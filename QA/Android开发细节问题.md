# Android开发细节问题

## 场景：主页通常有一个ViewPager，断网状态下，第一次进入，至少有两个Fragment同时加载，如果每个加载都使用Toast提示，那么可能出现多个Toast提示的问题

引出第一个问题，也是体验好坏非常重要的问题。

从下面**加载错误的提示时机**的分析，我们将加载错误的场景组合，分为两类（忽略数据逻辑错误）

- 第一次进入页面时，加载失败（通常是断网）<br/> 通用解决办法：使用一个“刷新界面”“覆盖”正常显示的界面，不使用```Toast```等进行提示。这个刷新界面至少需要一个按钮，点击后重新模拟第一次进入时加载数据

- 请求更多数据、或刷新当前页面时，加载失败（即界面上本来就有数据）<br/>解决方案：1. 可以使用Toast提示 2. 如果界面是一个列表，可以使用footer来显示加载失败的信息，同时附加点击加载更多的标识

### 加载错误的提示时机

#### 加载失败的分类

1. 断网、弱网络，总之网络数据请求失败，通常是抛出异常

2. 加载的数据逻辑错误，如签名错误。这种错误实现上本不应该出现。

#### 错误出现时机的分类

1. 第一次进入页面时

2. 在请求更多数据，或刷新当前页面时


### 统一封装

常用的地方莫过于```Activity```和```Fragment```, 刷新界面在加载成功过一次之后应该隐藏或移除掉。


## 斗鱼首页Fragment的处理

目的：懒加载与构造新Fragment的平衡。

原理：懒加载、Fragment数据保存机制。

| 层 | 使用```ViewPager``` | 使用```Fragment```懒加载 | 分析 |
|--|:--|:--|:--|
| 第一层 | 否 | 是 | 需要时才构造```Fragment``` |
| 第二层 | 是 | 是 | 便于滑动 |
| 第三层 | 是 | 否 | 内存优化 |

### 是否使用```ViewPager```

- 使用```ViewPager```是最方便处理```Fragment```的方式，只要合理掌握其对```FragmentPagerAdapter```和```FragmentStatePagerAdapter```对```Fragment```的控制，那么开发将行云流水。而```ViewPager```的问题在于与```Fragment```懒加载的某些冲突，如果使用懒加载，那么实际上，```ViewPager```是一次性保留所有可能被加载的```Fragment```, 这样实际上非常耗用内存。

- 如果不使用```ViewPager```, 那么自行控制```Fragment```的构造与生命周期保留，那么就可以避免这个问题。可以实现像微信、QQ这样，需要时才加载```Fragment```, 并且比较好的实现```Fragment```懒加载。

### TODO

实现使用和不使用```ViewPager```时的```Fragment```懒加载。即通常在主页使用```ViewPager```与否对```Fragment```的控制。

### 参考

[Android-ProgressFragment](https://github.com/johnkil/Android-ProgressFragment)

[Fragment全解析系列（一）：那些年踩过的坑](http://www.jianshu.com/p/d9143a92ad94#)

[Fragment全解析系列（二）：正确的使用姿势](http://www.jianshu.com/p/fd71d65f0ec6#)

[http://www.jianshu.com/p/38f7994faa6b](Fragment之我的解决方案：Fragmentation)

[Android4.0-Fragment框架实现方式剖析（一）](http://blog.csdn.net/weihan1314/article/details/7997421)

## TODO

## 一些重复性的工作，如何封装

使用最频繁的封装当然莫过于继承，在父类中提供一些通用方法，子类就不必在使用时再重新实现一遍。如显示/隐藏一个```ProgressDialog```, 每次再使用时都需要设置很多参数，而这些参数大部分是固定的，并且，除了构造对象时需要的一些操作，如隐藏，我们还有一些额外的方法，我们就可以把构造这个类的对象的代码在父类中封装成方法，使用时调用即可。

但是如果是构造一个对象，对这个对象进行一些定制化的操作呢？如最近常用的[RecyclerRefreshLayout](https://github.com/dinuscxj/RecyclerRefreshLayout), 我想为我的项目统一设置一个刷新头部，那么再不改动源码的方式下怎么来封装。同样的，仍然是定义一个类，继承```RecyclerRefreshLayout```. 不过这里需要注意的是，我们不是继承使用```RecyclerRefreshLayout```的类，而是直接继承被使用的类。在编程时可能会陷入一种定向思维。同样的，可以封装```WebView```, 简单的浏览器实现都是一些很统一的设置, [BaseRecyclerViewAdapterHelper](https://github.com/CymChad/BaseRecyclerViewAdapterHelper), 设置风格相同的empty view, loading footer, loading failed footer等。

- 第一种方式重点在于是**方法**， 封装的是方法，这些方法都是比较常用的，使用简单，但是其中的内容过多，那么就把多余的工作统一处理，只提供使用的接口
- 第二种方式重点在于**构造**，封装的是对象，这个对象的构造是非常简单的，但是我们需要一些定制化的设定，那么就把这些定制化的设定在构造对象时就一并完成，实现一个子类，在子构造器中定制对象。

问题：如果一些方法使用过于复杂，即使封装成简单的调用方法，在调用时必须按照一定的规范呢，如何封装比较合适？

例如：统一为所有的```Activity```添加一个断网提示页面，当进入这个```Activity```时，如果断网，没有请求到这个数据，则显示断网提示页面，如果刷新得到数据，则永久隐藏断网提示页面，如果是需要多次刷新数据，那么势必会影响到断网提示页面的显示，所以即使封装成简单的调用方法，调用时，仍然需要非常多的逻辑，那么怎样处理（或是说服自己，使用起来就是这么麻烦）？

## 参考

[ Android中clipChildren属性的用法](http://blog.csdn.net/flymoon1201/article/details/44646473)

[ 一个卡片式的ViewPager，带你玩转ViewPager的PageTransformer属性！](http://blog.csdn.net/u012702547/article/details/52334161)

[clipToPadding和clipChildren](http://blog.csdn.net/litefish/article/details/52471273)

[viewpager实现画廊(一屏多个Fragment)效果](http://www.trinea.cn/android/viewpager-multi-fragment-effect/)

[android:clipToPadding和android:clipChildren](http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2015/0317/2613.html)

[Android布局属性android:clipToPadding的UI设计妙用](http://blog.csdn.net/zhangphil/article/details/48680055)

[ViewPagerCards](https://github.com/rubensousa/ViewPagerCards)