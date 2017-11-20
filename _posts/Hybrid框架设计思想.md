title: Hybrid框架设计思想
author: Jaren
date: 2017-11-11 17:03:02
tags:
---
>**前言**：作为一种混合开发的模式，Hybrid APP底层依赖于Native提供的容器（UIWebview），上层使用Html&Css&JS做业务开发，底层透明化、上层多多样化，这种场景非常有利于前端介入，H5的低成本、高效率、跨平台等特性非常适合业务快速迭代。
<!-- more -->  
先上图一张：

![ico原来的样子](/assets/blogImg/hybrid1.png)
在设计Hybrid之初，首先是要考虑本地化包的版本管理，如下图所示，正常APP启动会如下逻辑：
![ico原来的样子](/assets/blogImg/hybrid2.png)  
1.网页资源提前加载到本地

在App启动时会请求一个配置文件，包含版本号和包地址，举例我们的文件如下：


    {"errorMsg": "", "code": 0, "data": [{"zipDownloadUrl": "https://192.168.10.10/cdn/updatepkg/matrix_v2.2.606.zip", "version": "2.2.606", "packageName": "dist"}]}

 
 这个配置文件每次启动会去下载，并比对version跟本地的version是否一致，不一致则下载，一致则略过。

部分对用户体验要求较高的页面使用原生开发

我和我的团队在对用户体验要求较高的页面（比如首页），我们最好还是使用原生开发。这也是Hybrid的真正意义所在吧。但随之也会产生一个新的问题：原生页面和H5的跳转实现。

**1.http和https的处理**

由于现在对网络安全的重视（当然苹果也由于要求），越来越多的App加入了https的支持。有一种非常普遍的问题需要我们注意：跨域。举个例子，我们的UIWebView加载的页面是     https://www.baidu.com ，但baidu.com这个页面中的某段js代码会执行页面跳转，而跳转的协议是hybrid，也就是诸如hybrid://AViewController类似的URL地址。App意识到这可能是一段不安全的代码，因此不给于执行，之前就是遇到了这个问题导致我们App不能响应任何【H5->原生】跳转。那如何“鱼和熊掌兼得”呢，解决方案有两种，各有利弊：

在调用UIWebView的loadRequest方法时，先去判断本地有没有资源，有的话加载本地资源，这样其实就变相的走了http协议或者file协议，就不存在https跨域的问题。但是对于UIWebView的js请求我们还是要进行拦截，走本地的https请求即可。这个方案的优点在于，由于html文件资源在本地所以加载资源速度比较快，用户体验不错；缺点就是我们必须通过URLProtocol重新定义所有的请求，这就需要我们在Request请求的时候避免在httpbody或者httpbodystream中设置参数，转而在header中设置。另外一种解决方案是，请求的时候将URL地址直接从https替换成http，这样的好处就是不需要定义所有的网络请求，但由于html文件资源在远端所以加载资源速度比较慢。

**2.参数的传递**

在原生和H5之间跳转的时候，参数传递都作为URL地址的一部分，例如AViewController中有属性username和tel，那我们在跳转到AViewController是路由的写法应该是类似：

    hybrid://AViewController?username=123&tel=1526181629x


这个很好理解，但有种情况，如果接受的是个url，并且url中也有参数怎么办？举例


    hybrid://AViewController?url=http://www.baidu.com?username=123&tel=1526181629x

那请问，tel是AViewController的参数还是url自带的参数？这是个问题。解决方案其实也很简单：一层层encode，比如有一个url那就encode一次，如果url中还带url那再encode一次。decode的时候也是一样，一层一层decode。

使用原生SDK编写一套可供js调用的API

路由除了可以用于原生页面和H5页面跳转，还可以用于调用本地的一些功能，下面是我们App预设的一些服务：
```
// 打开新页面

#define SERVICE_OPENNEWPAGE @"hybrid://openNewPage?param="

// 更新导航栏

#define SERVICE_UPDATEUIHEADER @"hybrid://updateNavigationBar?param="

// 回退

#define SERVICE_PAGEBACK @"hybrid://back?param="

// 获取定位

#define SERVICE_GETLOCATION @"hybrid://getLocation?param="

// 获取网络类型，状态

#define SERVICE_NETWORKTYPE @"hybrid://getNetWorkType?param="
```   
因此只要在html中指明如上的链接即可。有时我们需要在调用服务成功后给个反馈，那就需要在native调H5的方法，幸好JSContext帮我们实现了这个想法。

如图，当一个Request 发起后（这个Request 可以是UIWebView中的某个链接点击产生，也可以UIWebView的Ajax请求），通过Watermelon 进行拦截，如果是正常的HTTP请求则放过，如果是我们定义的scheme则单独做处理。OHHttpStub是基于NSURLProtocol实现的URL Request 拦截系统。方法

    +(id)stubRequestsPassingTest:(OHHTTPStubsTestBlock)testBlock withStubResponse:(OHHTTPStubsResponseBlock)responseBlock;

而参数testBlock就是需要拦截的Request请求，拦截后我们可以自己实现自己的Request请求，并返回数据给responseBlock。至于这里为什么使用OHHttpStub不使用原生的NSURLProtocol是为了避免与其他的使用NSURLProtocol 的三方库（比如Bulgly）冲突。当拦截成功后，我们再把结果分发给WebView。

**总结：事实上，Hybrid的核心有3个**

    1.离线资源包的管理。

    2.webview对所有请求的拦截（Native核心功能）

    3.交互协议的制定 


**一些问题遇到的问题：**

事实上，在iOS开发当中，你会发现如果你采用了WKwebview会后会出现各种各样的问题，比如说跨域的问题（可以多了解http协议本身，这个问题已经解决），比如说某些弹框失败的问题包括cookie的问题，当然这都是wk范畴的问题，有些在我后来的实践当中已经解决了，有些还没有，我后面有时间的话会把这些解决方案逐一分享给大家。当然目前针对上述问题我发现包括当前开源的无论是豆瓣还是腾讯团队的iOS版Hybrid框架都没有解决这个问题，统一采用的UIwebview控件，我个人以为使用WKwebview一定是大势所趋，所有问题一定会要解决毕竟技术是需要一步步来推动的。最后来让大家看一下我们之前团队做的这个开源的Hybrid框架源码，代码写的很一般，希望大家不要喷，它只是给了一个设计思路以供大家参考而已：


[Hybrid框架][Hybrid框架]。

[Hybrid框架]:https://github.com/jilei6/Watermelon