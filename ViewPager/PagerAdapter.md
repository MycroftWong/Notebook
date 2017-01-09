# PagerAdapter 分析

相信使用过```ViewPager```的人都知道它的常规使用方法，当然用得最多的仍然是```FragmentPagerAdapter```和```FragmentStatePagerFragment```，但是会用不一定用得好，从刚开始开发APP到现在，我也用过无数遍，但是最近为了优化界面，查看源码才发现之前自己“不会用”。

## 翻译

```java
/**
 * Base class providing the adapter to populate pages inside of
 * a {@link ViewPager}.  You will most likely want to use a more
 * specific implementation of this, such as
 * {@link android.support.v4.app.FragmentPagerAdapter} or
 * {@link android.support.v4.app.FragmentStatePagerAdapter}.
 *
 * <p>When you implement a PagerAdapter, you must override the following methods
 * at minimum:</p>
 * <ul>
 * <li>{@link #instantiateItem(ViewGroup, int)}</li>
 * <li>{@link #destroyItem(ViewGroup, int, Object)}</li>
 * <li>{@link #getCount()}</li>
 * <li>{@link #isViewFromObject(View, Object)}</li>
 * </ul>
 *
 * <p>PagerAdapter is more general than the adapters used for
 * {@link android.widget.AdapterView AdapterViews}. Instead of providing a
 * View recycling mechanism directly ViewPager uses callbacks to indicate the
 * steps taken during an update. A PagerAdapter may implement a form of View
 * recycling if desired or use a more sophisticated method of managing page
 * Views such as Fragment transactions where each page is represented by its
 * own Fragment.</p>
 *
 * <p>ViewPager associates each page with a key Object instead of working with
 * Views directly. This key is used to track and uniquely identify a given page
 * independent of its position in the adapter. A call to the PagerAdapter method
 * {@link #startUpdate(ViewGroup)} indicates that the contents of the ViewPager
 * are about to change. One or more calls to {@link #instantiateItem(ViewGroup, int)}
 * and/or {@link #destroyItem(ViewGroup, int, Object)} will follow, and the end
 * of an update will be signaled by a call to {@link #finishUpdate(ViewGroup)}.
 * By the time {@link #finishUpdate(ViewGroup) finishUpdate} returns the views
 * associated with the key objects returned by
 * {@link #instantiateItem(ViewGroup, int) instantiateItem} should be added to
 * the parent ViewGroup passed to these methods and the views associated with
 * the keys passed to {@link #destroyItem(ViewGroup, int, Object) destroyItem}
 * should be removed. The method {@link #isViewFromObject(View, Object)} identifies
 * whether a page View is associated with a given key object.</p>
 *
 * <p>A very simple PagerAdapter may choose to use the page Views themselves
 * as key objects, returning them from {@link #instantiateItem(ViewGroup, int)}
 * after creation and adding them to the parent ViewGroup. A matching
 * {@link #destroyItem(ViewGroup, int, Object)} implementation would remove the
 * View from the parent ViewGroup and {@link #isViewFromObject(View, Object)}
 * could be implemented as <code>return view == object;</code>.</p>
 *
 * <p>PagerAdapter supports data set changes. Data set changes must occur on the
 * main thread and must end with a call to {@link #notifyDataSetChanged()} similar
 * to AdapterView adapters derived from {@link android.widget.BaseAdapter}. A data
 * set change may involve pages being added, removed, or changing position. The
 * ViewPager will keep the current page active provided the adapter implements
 * the method {@link #getItemPosition(Object)}.</p>
 */
```

翻译：
在```ViewPager```填充页面需要提供的adapter的基类。大多数时候会使用这个类的特定实现，如```FragmentPagerAdapter```或```FragmentStatePagerAdapter```.

当你实现一个```PagerAdapter```时，你一定至少得重写以下方法：

- ```instantiateItem(ViewGroup, int)```
- ```destroyItem(ViewGroup, int, Object)```
- ```getCount()```
- ```isViewFromObject(View, Object)```

相对使用```AdapterView```的adapter, 使用```PagerAdapter```更为简单。```ViewPager```使用回调来指定更新的步骤，而不是直接使用循环机制。如果期望实现```View```的回收，那么通过```PagerAdapter```也是可以实现的，或者使用更加复杂的方法来组织每一页的```View```, 例如```Fragment```事务那样，使用一个```Fragment```来管理```View```.

**ViewPager**使用一个键对象来关联每一页，而不是管理```View```。这个键用于追踪和唯一标识在adapter中独立位置中的一页。调用方法```startUpdate(ViewGroup)```表明```ViewPager```中的内容需要更改。

通过调用一次或多次调用```instantiateItem(ViewGroup, int)```来构造页面视图。
调用```destroyItem(ViewGroup, int, Object)```来取消```ViewPager```关联的页面视图。
最后，当一次更新（添加和/或移除）完成之后将会调用```finishUpdate(ViewGroup)```来通知adapter, 提交关联和/或取消关联的操作。这三个方法就是用于```ViewPager```使用回调的方式来通知```PagerAdapter```来管理其中的页面。

一个非常简单的方式就是使用每页视图作为key来关联它们自己，在方法```instantiateItem(ViewGroup, int)```中创建和添加它们到```ViewGroup```之后，返回该页视图。与之相匹配的方法```destroyItem(ViewGroup, int, Object)```实现从```ViewGroup```中移除视图。当然必须在```isViewFromObject(View, Object)```中这样实现：```return view == object;```.

**PagerAdapter**支持数据改变，数据改变必须在主线程中调用，并在数据改变完成后调用方法```notifyDataSetChanged()```, 和```AdapterView```中派生自```BaseAdapter```相似。一次数据的改变可能关联着页面的添加、移除、或改变位置。```ViewPager```将根据adapter中实现```getItemPosition(Object)```方法返回的结果，来判断是否保留当前已经构造的活动页面（即重用，而不完全自行构造）。

## 原理详解

**ViewPager**+**PagerAdapter**的合作关系：```ViewPager```来控制一页界面构造和销毁的时机，使用回调来通知```PagerAdapter```具体做什么，```PagerAdapter```只需要按照相应的步骤做。当然为了使用得更好、提供更多的功能，又建议了使用```View```的回收工作和管理工作，同时提供当数据改变时的界面刷新工作。

**instantiateItem(ViewGroup, int)**: 构造指定位置的页面。adapter负责在这个方法中添加view到容器中，即使是在```finishUpdate(ViewGroup)```才保证完成的。在```FragmentPagerAdapter```和```FragmentStatePagerAdapter```中，都是返回一个构造的```Fragment```.

**destroyItem(ViewGroup, populate, Object)**: 移除指定位置的页面。adapter负责从容器中移除view, 即是最后实在```finishUpdate(ViewGroup)```保证完成的。在```FragmentPagerAdapter```和```FragmentStatePagerAdapter```中，分别使用```FragmentTransition.detach(Fragment)```和```FragmentTransition.remove(Fragment)```来逻辑上销毁```Fragment```.

**finishUpdate(ViewGroup)**: 当页面的显示变化完成式调用。在这里，你一定保证所有的页面从容器中合理的添加或移除掉。

**setPrimaryItem(ViewGroup, int, Object)**: 被```ViewPager```调用来通知adapter此时那个item应该被认为是主要的页面，这个页面将在当前页面展示给用户。正是因为这个方法，才有在```ViewPager```中实现```Fragment```懒加载的机制。

**isViewFromObject(View, Object)**: 指定当前页面```View```是否和指定的key对象相关联（这个key对象是在```instantiateItem(ViewGroup, int)```方法返回的）。这个方法需要```PagerAdapter```恰当的实现。即只要匹配好键值对即可。```FragmentPagerAdapter```和```FragmentStatePagerAdapter```的实现: ```return ((Fragment)object).getView() == view;```.

虽然简单或很少使用到的一些方法不想细究，不过还是一次性分析完为好，如```getPageTitle(int)```, ```getPageWidth(int)```, ```getItemPosition(Object)```等。

**getPageTitle(int)**: 返回每页的标题，多用于关联indicator

**getPageWidth(int)**: 返回指定的页面相对于```ViewPager```宽度的比例，范围(0.f-1.f]。默认值为1.f, 即占满整个屏幕。如果是0.5f, 那么在初始状态下，默认会出现前两个页面，而```primary```主页面是在```ViewPager```的起始位置（通常是屏幕左侧），直到最后一个页面在屏幕右侧，如果总共5个页面，返回值为0.2f, 那么将一次性出现所有的页面.

**getItemPosition(Object)**: 用于数据刷新时的页面处理方式。返回值包括三类：```POSITION_UNCHANGED```表示位置没有变化，即在添加或移除一页或多页之后该位置的页面保持不变，可以用于一个```ViewPager```中最后几页的添加或移除时，保持前几页仍然不变；```POSITION_NONE```，表示当前页不再作为```ViewPager```的一页数据，将被销毁，可以用于无视```View```缓存的刷新；根据传过来的参数```Object```来判断这个key所指定的新的位置，如总共有5个页面，我们想交换第0个和第4个的页面，那么在返回时，可以参考下面的代码。

```Java
    private List<Object> mItems = new ArrayList<>();

    {
        for (int i = 0; i < 5; i++) {
            mItems.add(null);
        }
    }

    @Override
    public int getItemPosition(Object object) {
        int position = mItems.indexOf(object);
        if (position == 0) {
            return 4;
        } else if (position == 4) {
            return 0;
        } else {
            // 下面两种都可以
//            return POSITION_UNCHANGED;
            return position;
        }
    }
```

**saveState()**: 保存页面状态。用得不多，见到的也是在```FragmentStatePagerAdapter```中使用，有不同于```FragmentPagerAdapter```的对于```Fragment```的管理机制。当然我们也可以使用这个来优化我们自己的```PagerAdapter```, 如主页Banner.

**restoreState(Parcelable, ClassLoader)**: 和```saveState()```搭配使用，用于恢复页面状态。

## FragmentPagerAdapter

可以从很多文章中看到类似这样的一句话：超出范围的```Fragment```会被销毁。~~所以之前，我一直认为的是，```FragmentPagerAdapter```中通常最多会保留3个```Fragment```, 超出左右两侧的```Fragment```将被销毁，滑动到时又会被重新构造。~~不过这是错误的。下面是翻译。

```java
/**
 * Implementation of {@link PagerAdapter} that
 * represents each page as a {@link Fragment} that is persistently
 * kept in the fragment manager as long as the user can return to the page.
 *
 * <p>This version of the pager is best for use when there are a handful of
 * typically more static fragments to be paged through, such as a set of tabs.
 * The fragment of each page the user visits will be kept in memory, though its
 * view hierarchy may be destroyed when not visible.  This can result in using
 * a significant amount of memory since fragment instances can hold on to an
 * arbitrary amount of state.  For larger sets of pages, consider
 * {@link FragmentStatePagerAdapter}.
 *
 * ...
 */
```

翻译：

**PagerAdapter**的实现类，使用将一直保留在```FragmentManager```中的```Fragment```来代表每一页，直到用户返回上一页。

当用于典型地使用多静态化的```Fragment```时，```FragmentPagerAdapter```无疑是最好使用的，例如一组tabs. 每个用户访问过的页面的```Fragment```都将会保留在内存中，即使它的视图层在不可见时已经被销毁。这可能导致使用比较大数量的内存，因为```Fragment```实例持有任意数量的状态。如果使用大数据的页面，考虑使用```FragmentStatePagerAdapter```.


从上面可以看出，即使是超出可视范围和缓存范围之外的```Fragment```，它的视图将会被销毁，但是它的实例将会保留在内存中，所以每一页的```Fragment```至始至终都只需要构造一次而已。通常是在主页中使用```FragmentPagerAdapter```, 但是超出范围的```Fragment```的视图会被销毁，我们也可以在```Fragment```中缓存```View```来避免状态的丢失，也可以使用另外的机制，如缓存```View```的状态。

```Java
@Override
public Object instantiateItem(ViewGroup container, int position) {
    if (mCurTransaction == null) {
        mCurTransaction = mFragmentManager.beginTransaction();
    }

    final long itemId = getItemId(position);

    // Do we already have this fragment?
    String name = makeFragmentName(container.getId(), itemId);
    Fragment fragment = mFragmentManager.findFragmentByTag(name);
    if (fragment != null) {
        if (DEBUG) Log.v(TAG, "Attaching item #" + itemId + ": f=" + fragment);
        mCurTransaction.attach(fragment);
    } else {
        fragment = getItem(position);
        if (DEBUG) Log.v(TAG, "Adding item #" + itemId + ": f=" + fragment);
        mCurTransaction.add(container.getId(), fragment,
                makeFragmentName(container.getId(), itemId));
    }
    if (fragment != mCurrentPrimaryItem) {
        fragment.setMenuVisibility(false);
        fragment.setUserVisibleHint(false);
    }

    return fragment;
}

@Override
public void destroyItem(ViewGroup container, int position, Object object) {
    if (mCurTransaction == null) {
        mCurTransaction = mFragmentManager.beginTransaction();
    }
    if (DEBUG) Log.v(TAG, "Detaching item #" + getItemId(position) + ": f=" + object
            + " v=" + ((Fragment)object).getView());
    mCurTransaction.detach((Fragment)object);
}
```

从上面源码可以看出，当被销毁时，```Fragment```并没有从```FragmentTransition```中移除，而是调用了```FragmentTransition.detach(Fragment)```方法，这样销毁了```Fragment```的视图，但是没有移除```Fragment```本身。


## FragmentStatePagerAdapter

```Java
/**
 * Implementation of {@link PagerAdapter} that
 * uses a {@link Fragment} to manage each page. This class also handles
 * saving and restoring of fragment's state.
 *
 * <p>This version of the pager is more useful when there are a large number
 * of pages, working more like a list view.  When pages are not visible to
 * the user, their entire fragment may be destroyed, only keeping the saved
 * state of that fragment.  This allows the pager to hold on to much less
 * memory associated with each visited page as compared to
 * {@link FragmentPagerAdapter} at the cost of potentially more overhead when
 * switching between pages.
 *
 * ...
 */
```

**PagerAdapter**的实现类，使用```Fragment```来管理每一页。这个类也会管理保存和恢复```Fragment```的状态。

当使用一个大数量页面时，```FragmentStatePagerAdapter```将更加有用，工作机制类似于```ListView```. 当每页不再可见时，整个```Fragment```将会被销毁，只保留```Fragment```的状态。相对于```FragmentPagerAdapter```, 这个将允许页面持有更少的内存。

```Java
@Override
public Object instantiateItem(ViewGroup container, int position) {
    // If we already have this item instantiated, there is nothing
    // to do.  This can happen when we are restoring the entire pager
    // from its saved state, where the fragment manager has already
    // taken care of restoring the fragments we previously had instantiated.
    if (mFragments.size() > position) {
        Fragment f = mFragments.get(position);
        if (f != null) {
            return f;
        }
    }

    if (mCurTransaction == null) {
        mCurTransaction = mFragmentManager.beginTransaction();
    }

    Fragment fragment = getItem(position);
    if (DEBUG) Log.v(TAG, "Adding item #" + position + ": f=" + fragment);
    if (mSavedState.size() > position) {
        Fragment.SavedState fss = mSavedState.get(position);
        if (fss != null) {
            fragment.setInitialSavedState(fss);
        }
    }
    while (mFragments.size() <= position) {
        mFragments.add(null);
    }
    fragment.setMenuVisibility(false);
    fragment.setUserVisibleHint(false);
    mFragments.set(position, fragment);
    mCurTransaction.add(container.getId(), fragment);

    return fragment;
}

@Override
public void destroyItem(ViewGroup container, int position, Object object) {
    Fragment fragment = (Fragment) object;

    if (mCurTransaction == null) {
        mCurTransaction = mFragmentManager.beginTransaction();
    }
    if (DEBUG) Log.v(TAG, "Removing item #" + position + ": f=" + object
            + " v=" + ((Fragment)object).getView());
    while (mSavedState.size() <= position) {
        mSavedState.add(null);
    }
    mSavedState.set(position, fragment.isAdded()
            ? mFragmentManager.saveFragmentInstanceState(fragment) : null);
    mFragments.set(position, null);

    mCurTransaction.remove(fragment);
}
```

从源码可以看出，当销毁```Fragment```时，缓存了```Fragment```的状态，并移除了```Fragment```的引用。而在构造时，显示判断是否已经在构造，如果是则直接返回该```Fragment```, 如果不是，则重新构造一个新的```Fragment```, 并且如果已经缓存了状态，则将改状态传入```Fragment```用于恢复状态。


## Fragment懒加载的原理

概念：当需要时才加载，加载之后一直保持该对象。

而关于```Fragment```实现的```PagerAdapter```都没有完全保存其引用和状态。```FragmentPageAdapter```需要重建视图，```FragmentStatePageAdapter```使用状态恢复，```View```都被销毁，但是恢复的方式不同，而通常我们想得到的结果是，```Fragment```一旦被加载，其视图也不会被销毁，即不会再重新走一遍生命周期。而且```ViewPager```为了实现滑动效果，都是预加载左右两侧的页面。

我们通常想要实现的两种效果：不提供滑动，需要时才构造，并且只走一遍生命周期，避免在```Fragment```中做过多的状态保存和恢复，参考斗鱼、QQ、全名，这也是大多数APP的实现方式；提供滑动，但是如果没有真正显示在界面上，那么使用站位页面代替真正的数据页面，参考微信，使用这种实现的较少。

### 是否提供滑动

- 提供滑动，当然使用```ViewPager```是最为方便的，但是会预加载新的页面，所以不可避免得，会提前加载数据，或是使用站位页面的方式来实现懒加载。

- 不提供滑动，一种方式是使用```ViewPager```, 但是限制其不可滑动，同时在选择指定的页面时，直接跳转，而不是翻页的方式；另外的实现方式则是完全抛弃```ViewPager```，自己使用```FragmentManager```来管理```Fragment```, 这种方式的缺点是，离开了```ViewPager```，那么与之对应的indicator库也可能不能使用，所以会要求在这方面多做一些工作。

### ViewPager中实现懒加载的原理

#### 懒加载需要处理的几个问题

- 预加载

虽然没有显示在界面上，但是当前页面的上一页和下一页的```Fragment```已经执行了一个```Fragment```能够显示在界面上的所有生命周期方法，但是我们想在跳转到该页时才真正构造数据视图和请求数据。那么我们可以使用一个占位视图，那么可以想到使用```ViewStub```，当真正跳转到该页时，执行```ViewStub.inflate()```方法，加载真正的数据视图和请求数据。

- 视图保存

当某一页超出可视范围和预加载范围，那么它将会被销毁，```FragmentStatePagerAdapter```销毁整个```Fragment```, 我们可以自己保存该```Fragment```, 或使用```FragmentPagerAdapter```让```FragmentTransition```来保留```Fragment```的引用。虽然这样，但是它的周期方法已经走完，那么我们只能手动的保存```Fragment```根```View```的引用，当再次重新进入新的声明周期方法时，返回原来的```View```

- 是否已经被用户所看到

其实本身而言，```FragmentManager```并没有提供为```Fragment```被用户所看到的回调方法，而是在```FragmentPagerAdapter```和```FragmentStatePagerAdapter```中，调用了```Fragment.setUserVisibleHint(boolean)```来表明```Fragment```是否已经被作为**primary**```Fragment```. 所以这个方法可以被认为是一个回调方法。


#### 懒加载实现

通常想到的，我们是在```Fragment```中来根据生命周期来控制视图的缓存和数据的加载来实现懒加载的效果，但是我们也可以自定义```PagerAdapter```来实现懒加载。

- 重写```Fragment```实现懒加载

```Java
/**
 * Created by Mycroft on 2017/1/9.
 */
public abstract class LazyFragment extends Fragment {

    // Fragment的根View
    private View mRootView;

    // 检测声明周期中，是否已经构建视图
    private boolean mViewCreated = false;

    // 占位图
    private ViewStubCompat mViewStub;

    @Nullable
    @Override
    public final View onCreateView(LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {
        if (mRootView != null) {
            mViewCreated = true;
            return mRootView;
        }

        final Context context = inflater.getContext();
        FrameLayout root = new FrameLayout(context);
        mViewStub = new ViewStubCompat(context, null);
        mViewStub.setLayoutResource(getResId());
        root.addView(mViewStub, new FrameLayout.LayoutParams(FrameLayout.LayoutParams.MATCH_PARENT, FrameLayout.LayoutParams.MATCH_PARENT));
        root.setLayoutParams(new ViewGroup.MarginLayoutParams(ViewGroup.MarginLayoutParams.MATCH_PARENT, ViewGroup.MarginLayoutParams.MATCH_PARENT));

        mRootView = root;

        mViewCreated = true;
        if (mUserVisible) {
            realLoad();
        }
        return mRootView;
    }

    private boolean mUserVisible = false;

    @Override
    public final void setUserVisibleHint(boolean isVisibleToUser) {
        super.setUserVisibleHint(isVisibleToUser);
        mUserVisible = isVisibleToUser;
        if (mUserVisible && mViewCreated) {
            realLoad();
        }
    }

    // 判断是否已经加载
    private boolean mLoaded = false;

    /**
     * 控制只允许加载一次
     */
    private void realLoad() {
        if (mLoaded) {
            return;
        }

        mLoaded = true;
        onRealViewLoaded(mViewStub.inflate());
    }

    @Override
    public void onDestroyView() {
        mViewCreated = false;
        super.onDestroyView();
    }

    /**
     * 获取真正的数据视图
     *
     * @return
     */
    protected abstract int getResId();

    /**
     * 当视图真正加载时调用
     */
    protected abstract void onRealViewLoaded(View view);
}
```

示例可以参考[FragmentApp](https://github.com/MycroftWong/FragmentApp)

- 重写```PagerAdapter```

重写```PagerAdapter```解决的是```Fragment```生命周期所带来的视图保存的问题。

```Java
/**
 * 和{@link android.support.v4.app.FragmentPagerAdapter}唯一的不同是
 * 使用{@link FragmentTransaction#add(int, Fragment, String)}和{@link FragmentTransaction#remove(Fragment)}
 * 来代替{@link FragmentTransaction#attach(Fragment)}和{@link FragmentTransaction#detach(Fragment)}
 * <p>
 * 这样{@link Fragment}只会走一遍生命周期
 * <p>
 * Created by Mycroft on 2017/1/9.
 */
public abstract class LazyFragmentPagerAdapter extends PagerAdapter {
    private static final String TAG = "LazyFragmentPagerAdapter";
    private static final boolean DEBUG = true;

    private final FragmentManager mFragmentManager;
    private FragmentTransaction mCurTransaction = null;
    private Fragment mCurrentPrimaryItem = null;

    public LazyFragmentPagerAdapter(FragmentManager fm) {
        mFragmentManager = fm;
    }

    /**
     * Return the Fragment associated with a specified position.
     */
    public abstract Fragment getItem(int position);

    @Override
    public void startUpdate(ViewGroup container) {
        if (container.getId() == View.NO_ID) {
            throw new IllegalStateException("ViewPager with adapter " + this
                    + " requires a view id");
        }
    }

    @Override
    public Object instantiateItem(ViewGroup container, int position) {
        if (mCurTransaction == null) {
            mCurTransaction = mFragmentManager.beginTransaction();
        }

        final long itemId = getItemId(position);

        // Do we already have this fragment?
        String name = makeFragmentName(container.getId(), itemId);
        Fragment fragment = mFragmentManager.findFragmentByTag(name);
        if (fragment != null) {
            if (DEBUG) Log.v(TAG, "Attaching item #" + itemId + ": f=" + fragment);
            mCurTransaction.show(fragment);
        } else {
            fragment = getItem(position);
            if (DEBUG) Log.v(TAG, "Adding item #" + itemId + ": f=" + fragment);
            mCurTransaction.add(container.getId(), fragment,
                    makeFragmentName(container.getId(), itemId));
        }
        if (fragment != mCurrentPrimaryItem) {
            fragment.setMenuVisibility(false);
            fragment.setUserVisibleHint(false);
        }

        return fragment;
    }

    @Override
    public void destroyItem(ViewGroup container, int position, Object object) {
        if (mCurTransaction == null) {
            mCurTransaction = mFragmentManager.beginTransaction();
        }
        if (DEBUG) Log.v(TAG, "Detaching item #" + getItemId(position) + ": f=" + object
                + " v=" + ((Fragment) object).getView());
        mCurTransaction.hide((Fragment) object);
    }

    @Override
    public void setPrimaryItem(ViewGroup container, int position, Object object) {
        Fragment fragment = (Fragment) object;
        if (fragment != mCurrentPrimaryItem) {
            if (mCurrentPrimaryItem != null) {
                mCurrentPrimaryItem.setMenuVisibility(false);
                mCurrentPrimaryItem.setUserVisibleHint(false);
            }
            if (fragment != null) {
                fragment.setMenuVisibility(true);
                fragment.setUserVisibleHint(true);
            }
            mCurrentPrimaryItem = fragment;
        }
    }

    @Override
    public void finishUpdate(ViewGroup container) {
        if (mCurTransaction != null) {
            mCurTransaction.commitNowAllowingStateLoss();
            mCurTransaction = null;
        }
    }

    @Override
    public boolean isViewFromObject(View view, Object object) {
        return ((Fragment) object).getView() == view;
    }

    @Override
    public Parcelable saveState() {
        return null;
    }

    @Override
    public void restoreState(Parcelable state, ClassLoader loader) {
    }

    /**
     * Return a unique identifier for the item at the given position.
     * <p>
     * <p>The default implementation returns the given position.
     * Subclasses should override this method if the positions of items can change.</p>
     *
     * @param position Position within this adapter
     * @return Unique identifier for the item at position
     */
    public long getItemId(int position) {
        return position;
    }

    private static String makeFragmentName(int viewId, long id) {
        return "android:switcher:" + viewId + ":" + id;
    }
}
```

而同时，仍然需要重写```Fragment```来进行预加载

```Java
/**
 * Created by Mycroft on 2017/1/9.
 */
public abstract class BaseLazyFragment extends Fragment {

    // 检测声明周期中，是否已经构建视图
    private boolean mViewCreated = false;

    // 占位图
    private ViewStubCompat mViewStub;

    @Nullable
    @Override
    public final View onCreateView(LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {

        final Context context = inflater.getContext();
        FrameLayout root = new FrameLayout(context);
        mViewStub = new ViewStubCompat(context, null);
        mViewStub.setLayoutResource(getResId());
        root.addView(mViewStub, new FrameLayout.LayoutParams(FrameLayout.LayoutParams.MATCH_PARENT, FrameLayout.LayoutParams.MATCH_PARENT));
        root.setLayoutParams(new ViewGroup.MarginLayoutParams(ViewGroup.MarginLayoutParams.MATCH_PARENT, ViewGroup.MarginLayoutParams.MATCH_PARENT));

        mViewCreated = true;
        if (mUserVisible) {
            realLoad();
        }
        return root;
    }

    private boolean mUserVisible = false;

    @Override
    public final void setUserVisibleHint(boolean isVisibleToUser) {
        super.setUserVisibleHint(isVisibleToUser);
        mUserVisible = isVisibleToUser;
        if (mUserVisible && mViewCreated) {
            realLoad();
        }
    }

    // 判断是否已经加载
    private boolean mLoaded = false;

    /**
     * 控制只允许加载一次
     */
    private void realLoad() {
        if (mLoaded) {
            return;
        }

        mLoaded = true;
        onRealViewLoaded(mViewStub.inflate());
    }

    @Override
    public void onDestroyView() {
        mViewCreated = false;
        super.onDestroyView();
    }

    /**
     * 获取真正的数据视图
     *
     * @return
     */
    protected abstract int getResId();

    /**
     * 当视图真正加载时调用
     */
    protected abstract void onRealViewLoaded(View view);
}
```

和```LazyFragment```唯一的不同是不用自己来保留根```View```.

示例可以参考[FragmentApp](https://github.com/MycroftWong/FragmentApp)


### 不使用ViewPager，实现懒加载的原理

不使用```ViewPager```就避免了预加载的问题，同时也不会有```Fragment```是否用户可见的问题，因为只有加载时，用户才可见。

关于这一点，已经有了开源项目，可以参考[FragmentNavigator](https://github.com/Aspsine/FragmentNavigator)

使用示例可以参考[FragmentApp](https://github.com/MycroftWong/FragmentApp)