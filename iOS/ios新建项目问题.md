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