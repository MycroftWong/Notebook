# iOS新建项目问题及解决方法

在最新的xcode 8.1版本中，使用了swift 3.0，新建项目并导入原来的lib出现了各种问题。

## 不使用storyboard，在AppDelegate中指定ViewController

新建的iOS项目默认使用了stroyboard来指定默认的`ViewController`，而我自己不会使用storyboard，希望在`AppDelegate`中指定第一个`ViewController`。

> 解决方法，引用文章[不使用Main.storyboard的步骤](http://www.myexception.cn/ai/1868403.html), [不用storyboard开发ios8界面](http://www.tuicool.com/articles/vUnE7f), [Xcode升级到8.0，Swift3.0之后，出现的alamofire问题](http://blog.csdn.net/philar2016/article/details/52710060)
> 1. 保留Main.storyboard
> 2. 删除Main.storyboard中的controller
> 3. Info.plist中删除Main Storyboard Base Name对应关系
> didFinishLoadD代理中要实例化UIWindow
> self.window = UIWindow(frame: UIScreen.main.bounds)
> self.window?.makeKeyAndVisible()
> self.window?.backgroundColor = UIColor.white
> self.window?.rootViewController = ViewController()

## 使用CocoaPods导入lib出现swift 3.0 convert失败的问题

虽然github上很多的lib支持swift3.0，但是在导入时仍然出现编译的问题。

> 解决方法：参考文章[Xcode8中Swift3.0适配问题](http://www.jianshu.com/p/cbd650c9daad)
> 在使用`CocoaPods`导入了所有的lib时不进行convert, 分别对所有的lib更改语言版本设定。
> Pods -> 选择lib -> Build Settings -> 搜索关键字swift -> 找到 "Swift Language Version" -> 选择 Swift 3

## 添加网络权限

解决方法参考文章：[Xcode7、iOS9网络访问权限问题](http://www.ithao123.cn/content-10828113.html)，[iOS开发 适配Xcode8以及iOS10-权限问题](http://blog.csdn.net/baidu_25743639/article/details/52586525)

## SnapKit使用的问题

在对`UIView`进行尺寸设置之前需要提前添加到`UIView`中。

参考文章：[Swift - 自动布局库SnapKit的使用详解1（配置、使用方法、样例）](http://www.hangge.com/blog/cache/detail_1097.html)

## 获取statusbar的高度

`UIApplication.shared.statusBarFrame.size`

参考文章：[状态栏高度变化处理](http://www.jianshu.com/p/8c8303f7d439)

## `UIScrollView`的使用及实现引导页

```swift
scrollView = UIScrollView(frame: CGRect(x: 0, y: 0, width: self.view.frame.width, height: self.view.frame.height))
self.view.addSubview(scrollView)

scrollView.alwaysBounceVertical = false
scrollView.alwaysBounceHorizontal = false
scrollView.isScrollEnabled = true

scrollView.showsHorizontalScrollIndicator = false
scrollView.showsVerticalScrollIndicator = false
scrollView.isPagingEnabled = true

scrollView.bounces = false

let frame = scrollView.frame
let lableArray = ["Welcome", "Hello", "New World"]
for i in 0..<lableArray.count {
    let v = UILabel(frame: CGRect(x: CGFloat(i) * frame.width, y: 0, width: frame.width, height: frame.height))
    v.text = lableArray[i]
    v.textAlignment = .center
    
    scrollView.addSubview(v)
}

// height: 0 限制 UIScrollView 的上下滚动
scrollView.contentSize = CGSize(width: scrollView.frame.width * 3, height: scrollView.frame.height)

scrollView.delegate = self
```

上面是一个简单的例子，`UIScrollView`的各种属性不一一赘述，主要说几个比较关键的问题。

### 概念
`UIScrollView`是`UIView`的子类，实现了滑动，所有可滑动的控件都是它的子类。frame表示其位置和大小，这个是继承自`UIView`的属性，而其中最重要的属性是`contentSize`, 这个表示其内容的大小。**想象一下，我们是井底之蛙，通过`UIScrollView`这个井口，看到了天空的画面。`contentSize`则是这个天空的大小**，所以可以理解，`contentSize`决定了`UIScrollView`的可滑动的范围。

在上面的例子中，`UIScrollView`占用了`ViewController`的整个空间，而其中的内容则是三倍宽于`UIScrollView`的，所以可以左右滑动。同时注意，为了实现翻页的效果，指定了属性`isPagingEnable`属性为true，同时为了避免弹性边缘的效果（上下边缘），指定了属性`bounces`为false

### 配合`UIPageControl`的使用

两者可以认为是没有任何关系的，只是在使用上，通常将两者作为一体来使用。

`UIPageControl`继承自`UIView`，所以也是一个控件，对于其的使用就相对比较简单了。

```swift
pageControl.numberOfPages = lableArray.count
pageControl.currentPage = 0
```

关于`UIPageControl`的构造和位置大小的设置不再代码支出，主要是上面这两个属性，`numberOfPages`表示页数，即显示可以看到的点的个数，`currentPage`表示当前页数，即高亮的点的位置。仍然可以使用`addTarget`方法添加点击事件处理。另外如果想改变点的颜色，通常是自定义控件，所以在网上找即可。

```swift
func scrollViewDidScroll(_ scrollView: UIScrollView) {
    pageControl.currentPage = Int(scrollView.contentOffset.x / scrollView.frame.width)
}
```
如上所示，关于两者的交互，只需要在`UIScrollViewDelegate`中进行处理`UIPageControl`即可。

## APPIcon的设置

![APPIcon](./APPIcon.png)
打开Assets.xcassets -> AppIcon可以看到上图所示的布局，在下面都明确标注了图片需要的大小，我们需要做的是制作相应大小的图片，并放入相应的位置。例如，在标注40pt的位置上，2x则放入80x80像素的图片，3x则放入120x120像素的图片。需要注意的是，并没有方式20pt大小的图片，好像没有必要。