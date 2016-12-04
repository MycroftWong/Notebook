# HTTPS 学习笔记

以下摘自百度百科
```
HTTPS和HTTP的区别主要为以下四点：
一、https协议需要到ca申请证书，一般免费证书很少，需要交费。
二、http是超文本传输协议，信息是明文传输，https 则是具有安全性的ssl加密传输协议。
三、http和https使用的是完全不同的连接方式，用的端口也不一样，前者是80，后者是443。
四、http的连接很简单，是无状态的；HTTPS协议是由SSL+HTTP协议构建的可进行加密传输、身份认证的网络协议，比http协议安全。
```

## 认知
对于没有系统学过网络通信的我来说，这样的说法未免太难以理解了。还是以应用开发者的角度来理解这个。

关键字：证书、安全、端口

### 证书
根据[Tomcat启用HTTPS协议配置过程](http://blog.csdn.net/gane_cheng/article/details/53001846)，其实不一定要证书，但是需要发布对外使用，那么就一定需要使用证书。

1. 如果一个HTTPS的链接有合法的证书，那么可以直接进行访问。
2. 单向认证（保证服务器是真实的）：在客户端有一个证书，需要在http请求时使用该证书，进行网路请求
3. 双向认证（保证服务器和客户端都是真是的）：客户端和服务端都需要配置证书，一般不会这么麻烦。

### 安全
HTTP网络请求传输的数据是明文可读的，而HTTPS则将数据进行过加密。

### 端口
HTTP的默认端口是80，HTTPS的默认端口是443。（Tomcat开发环境中，HTTP端口是8080，HTTPS端口是8443）

## Android okhttp使用HTTPS
参考文章[Android Https相关完全解析 当OkHttp遇到Https](http://blog.csdn.net/lmj623565791/article/details/48129405)

实际上，这只是一个Java的操作过程，与Android并没有太大联系，需要知道其原理，根本上还是要从Java入手。

## 参考

[Tomcat启用HTTPS协议配置过程](http://blog.csdn.net/gane_cheng/article/details/53001846)

[HttpsURLConnection请求的单向认证](https://my.oschina.net/ososchina/blog/500925)

[Android Https相关完全解析 当OkHttp遇到Https](http://blog.csdn.net/lmj623565791/article/details/48129405)