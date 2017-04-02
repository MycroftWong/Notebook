# Mac golang环境搭建

## 下载安装包

官方包有多个版本，如`go1.8.darwin-amd64.pkg`，当前为1.8版本，目前mac OS是64位，所以选择amd64, pkg表示是mac安装文件，当然还有`tar.gz`, 属于压缩包形式。

> 摘自百度知道 
>
> i386 简单理解就是是32位的amd64 是64位的版本，因为是amd把64位率先引进桌面系统的，英特尔也是要追随amd并且保持兼容，一般在软件包里包含这样的字符

## 安装

双击`pkg`文件即可启动正常的mac软件安装流程。安装完成后，安装目录自动为`/usr/local/go`

## 配置环境变量

在个人空间根目录下打开`.bash_profile`文件（没有则使用`touch`命令创建），添加配置信息，如下

```
export GOPATH=/Users/MycroftWong/Documents/Go
export PATH=$PATH:${GOPATH//://bin:}/bin
export GOBIN=
```

## IDE选择

## IDEA配置


## 问题探讨

- `GOPATH`是工作环境目录，它应该被认为是一个项目目录吗，如果有多个项目的话，怎么解决多个项目在同一个文件夹下的冲突呢？

## 参考

[Golang下载地址](http://www.golangtc.com/download)

[MAC 设置环境变量path的几种方法](http://www.cnblogs.com/shineqiujuan/p/4693404.html)

[Golang环境变量解释](http://blog.csdn.net/Alsmile/article/details/48290223)