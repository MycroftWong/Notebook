# gradle的使用

gradle是什么：它是一个打包工具。

什么是打包：将代码和资源文件组合成为可以在操作系统中运行的程序。

打包工具是什么：一个打包过程分为很多步骤，打包工具可以自动化完成这些工作，减少时间开销。

打包工具的常用作用：

1. 删除无用的资源和代码
2. 更改代码
3. 混淆代码
4. 动态配置程序，如渠道号、关闭调试模式、更改包名、更改文件名等等

## 概念
领域：一个gradle文件中，以块的方式存在的叫做领域，其中有多个领域，如 buildscript, android, dependencies

每个领域有不同的作用。

## gradle的常规使用

### settings.gradle
项目根目录下有一个```settings.gradle```, 其中有一句话：
```gradle
include ':app', ':androidtest'
```
表示这个项目中有两个module.

### 项目根目录 build.gradle
为整个项目进行一些配置。

默认生成的有一个buildscript领域，在其中可以配置 repositories 仓库，如 jcenter, mavenCentral; dependencies 依赖，如android的 com.android.tools.build:gradle:2.2.2, greendao 的 org.greenrobot:greendao-gradle-plugin:3.0.0

allprojects 配置整个项目可能会使用到的，如repositories。另外可以在其中定义一些其他的属性，则可以直接在所有的module中的build.gradle中使用，如可以配置Android的SDK版本等。

另外还可以自定义领域，用于在module中的build.gradle中使用。

### module build.gradle

在第一行通常有这样一句话（它也算是一个领域）
```gradle
apply plugin: 'com.android.application'
```
这句话表示，这个module是一个Android的项目module, 如果是
```gradle
apply plugin: 'com.android.library'
```
则表示一个library.

另外，如果是java项目的话则是
```gradle
apply plugin: 'java'
```

在module的build.gradle中还有另个领域：
android 在其中进行各种android的配置
dependencies 在其中添加各种依赖



















<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
