---
title: "URL Session Programming Guide - Cookies and Custom Protocols"
date: 2016-01-29 14:12:55
categories: 
- 编程指南
tags: 
- NSURLSession
- NSURLConnection
metaAlignment: center
autoThumbnailImage: no
coverImage: //d1u9biwaxjngwg.cloudfront.net/welcome-to-tranquilpeak/city.jpg

---

[官方文档](https://developer.apple.com/library/prerelease/tvos/documentation/Cocoa/Conceptual/URLLoadingSystem/CookiesandCustomProtocols/CookiesandCustomProtocols.html#//apple_ref/doc/uid/10000165i-CH10-SW3)
<!--more-->

# Cookies and Custom Protocols
(Cookies和自定义协议)

如果你的应用程序需要编程管理Cookies，比如添加、删除Cookies或者决定哪一个Cookies应该接收，阅读 Cookie Storage。

如果你的应用程序需要支持基于URL的协议，但是NSURL没有原生支持，你可以注册你自己的自定义协议类提供需要的支持。更多信息，阅读 Protocol Support。

## Cookie Storage
(Cookie的存储)

由于HTTP协议无状态，客户端经常使用Cookie在URL请求之间提供持久存储数据。URL加载系统提供了创建和管理Cookie的接口，为了把Cookie当作HTTP请求的一部分发送，以及在解释Web服务器的响应时接收Cookie。

NSHTTPCookie类封装了Cookie，提供了访问Cookie很多常用属性的访问器。这个类还提供了方法将HTTP cookie headers转换为NSHTTPCookie实例，以及将NSHTTPCookie实例转换为NSURLRequest对象适合使用的headers。URL加载系统会自动发送存储的合适的Cookie给NSURLRequest对象，除非请求指定不要发送Cookie。同样的，一个NSURLResponse对象返回的Cookie按照当前的Cookie策略接受。

NSHTTPCookieStorage类提供了管理所有应用程序共享的NSHTTPCookie对象的集合的接口。

**iOS注意**：iOS中应用程序并不共享Cookies。

NSHTTPCookieStorage允许应用程序指定Cookies的接受策略。Cookies的接受策略控制Cookies是否应该始终被接受，从不接受，或者只接受相同域的URL。

**注意**：改变一个应用程序的Cookies接受策略会影响所有运行的应用程序的接受策略。

当另一个应用程序改变了Cookies存储或者Cookies接受策略，NSHTTPCookieStorage会发送NSHTTPCookieManagerCookiesChangedNotification和NSHTTPCookieStorageAcceptPolicyChangedNotification通知来提醒应用程序。

更多信息，参见[ NSHTTPCookieStorage Class Reference ](https://developer.apple.com/library/prerelease/tvos/documentation/Cocoa/Reference/Foundation/Classes/NSHTTPCookieStorage_Class/index.html#//apple_ref/doc/uid/TP40003665)和[ NSHTTPCookie Class Reference ](https://developer.apple.com/library/prerelease/tvos/documentation/Cocoa/Reference/Foundation/Classes/NSHTTPCookie_Class/index.html#//apple_ref/doc/uid/TP40003664)。

## Protocol Support
(协议支持)

URL加载系统被设计成允许客户端应用程序拓展传输数据的协议。URL加载系统原生支持HTTP，HTTPS，FILE，FTP，和 DATA 协议。

你可以通过子类化NSURLProtocol来实现自定义协议然后使用URL加载系统的NSURLProtocol类的registerClass:方法注册新的协议子类。当NSURLSession，NSURLConnection，或者NSURLDownload 对象为一个NSURLRequest对象初始化了一个连接，URL加载系统按照注册的相反顺序参考每一个类。第一个为canInitWithRequest:消息返回YES的类会用来处理请求。

如果你自定义的协议需要请求或者响应额外的属性，你可以为NSURLRequest，NSMutableURLRequest，和 NSURLResponse创建类别来提供这些属性的访问器。NSURLProtocol类在这些访问器中提供了设置和获取这些属性值的方法。

URL加载系统在连接开始和完成时负责创建和释放NSURLProtocol实例。你的应用程序始终不应该直接创建NSURLProtocol实例。

当NSURLProtocol子类被URL加载系统初始化时，它会提供了一个适配了NSURLProtocolClient协议的客户端对象。NSURLProtocol子类会从NSURLProtocolClient协议发送消息到客户端对象来通知URL加载系统它的操作，比如创建响应，接收数据，重定向新的URL，请求认证，以及完成加载。如果自定义的协议支持认证，它必需适配NSURLAuthenticationChallengeSender协议。

更多信息，参见[ NSURLProtocol Class Reference ](https://developer.apple.com/library/prerelease/tvos/documentation/Cocoa/Reference/Foundation/Classes/NSURLProtocol_Class/index.html#//apple_ref/doc/uid/TP40003761)。
