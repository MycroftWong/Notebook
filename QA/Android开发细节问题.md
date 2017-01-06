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

### 参考

[Android-ProgressFragment](https://github.com/johnkil/Android-ProgressFragment)

[Fragment全解析系列（一）：那些年踩过的坑](http://www.jianshu.com/p/d9143a92ad94#)

[Fragment全解析系列（二）：正确的使用姿势](http://www.jianshu.com/p/fd71d65f0ec6#)

[http://www.jianshu.com/p/38f7994faa6b](Fragment之我的解决方案：Fragmentation)

[Android4.0-Fragment框架实现方式剖析（一）](http://blog.csdn.net/weihan1314/article/details/7997421)

## TODO

RecyclerRefreshLayout 和 RecyclerView.Adapter 的统一封装总结。