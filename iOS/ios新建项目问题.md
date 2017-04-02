# iOS新建项目问题及解决方法

在最新的xcode 8.1版本中，使用了swift 3.0，新建项目并导入原来的lib出现了各种问题。

## 不使用storyboard，在AppDelegate中指定ViewController

新建的iOS项目默认使用了stroyboard来指定默认的`ViewController`，而我自己不会使用storyboard，希望在`AppDelegate`中指定第一个`ViewController`。

> 解决方法，引用文章[不使用Main.storyboard的步骤](http://www.myexception.cn/ai/1868403.html), [不用storyboard开发ios8界面](http://www.tuicool.com/articles/vUnE7f)
> 1. 保留Main.storyboard
> 2. 删除Main.storyboard中的controller
> 3. Info.plist中删除Main Storyboard Base Name对应关系
> didFinishLoadD代理中要实例化UIWindow
> self.window = UIWindow(frame: UIScreen.main.bounds)
> self.window?.makeKeyAndVisible()
> self.window?.backgroundColor = UIColor.white
> self.window?.rootViewController = ViewController()

