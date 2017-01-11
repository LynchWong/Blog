---
title: "URL Session Programming Guide - Introduction"
date: 2016-01-29 14:10:37
categories: 
- 编程指南
tags: 
- NSURLSession
- NSURLConnection
metaAlignment: center
autoThumbnailImage: no
coverImage: //d1u9biwaxjngwg.cloudfront.net/welcome-to-tranquilpeak/city.jpg

---

前一份编程指南是关于并发的，算是完成了，凑合着看吧。其实整理这些编程指南蛮花时间的，我空闲的时间基本都是在弄这个。有的时候觉得没什么意义，其实蛮有用的。特别是开发这么久了，看了苹果的说明，这些编程指南，感觉以前蒙在自己眼前的雾都散开了。前两天正好要做一个图片轮循的功能，正好我就是用的定时器类型的**dispatch_source**实现定时轮循。也是因为正好看到**dispatch_source**这里，如果没整理并发编程指南我可能就用**NSTimer**了。所以你现在的努力、做的事情、学习的知识和技能你觉得没什么用，你也许现在不知道，但是总有一天会用上回报你的。感觉都有点像鸡汤了，就不扯了。
<!--more-->

之前说了在弄一个代码库，想自己写个HTTP请求的框架。觉得之前的写的很渣，正好最近在看**Alamofire**的源码，自愧不如。反正我也准备是用**NSURLSession**来实现，所以就想先把这份指南整理了再开始开发。

那就开始吧，下面是**NSURLSession**编程指南的正文了。

==============================

[官方文档](https://developer.apple.com/library/prerelease/tvos/documentation/Cocoa/Conceptual/URLLoadingSystem/URLLoadingSystem.html#//apple_ref/doc/uid/10000165-BCICJDHA)

# About the URL Loading System
(关于URL加载系统)

重要提示：这是一份正在开发中的API或者技术的初步文档。虽然这个文档已经被技术准确性审核，但是这并不是最终的。

本指南介绍了可用于交互的URL以及使用标准互联网协议与服务器进行通信的Foundation框架类。这些类一起被称为URL加载系统。

URL加载系统是一组类和协议的集合，允许你的应用程序访问URL引用的内容。这项技术的核心就是**NSURL**类，它可以让你的应用程序操作URL以及URL引用的资源。

为了支持这个类，Foundation框架提供了一组丰富的类让你能够加载URL内容，上传数据到服务器，管理Cookie，控制响应缓存，处理Credential，以及应用程序指定的认证方式，自定义的协议拓展。

URL加载系统支持使用以下协议访问资源：

* 文件传输协议(**ftp://**)
* 超文本传输协议(**http://**)
* 加密的超文本传输协议(**https://**)
* 本地文件URL(**file:///**)
* 数据URL(**data://**)

使用用户系统的偏好设置也能支持代理服务器和SOCKS。

**重要**：除了URL加载系统，在OS X和iOS中也提供了在其它应用程序中开放的URL的API，比如Safari。这些API没有在本文档中描述。更多关于在OS X中登陆服务器的信息，参阅**Launch Services Programming Guide**。更多关于OS X的**NSWorkSpace**类的**openURL:**方法，参阅**NSWorkspace Class Reference**。更多关于iOS的**UIApplication**类的[ openURL: ](https://developer.apple.com/library/prerelease/tvos/documentation/UIKit/Reference/UIApplication_Class/index.html#//apple_ref/occ/instm/UIApplication/openURL:)方法，参阅[ UIApplication Class Reference ](https://developer.apple.com/library/prerelease/tvos/documentation/UIKit/Reference/UIApplication_Class/index.html#//apple_ref/doc/uid/TP40006728)。

## At a Glance
(概览)

URL加载系统包含了大量重要的辅助类来帮助URL加载类修改它们的行为。主要的辅助类分为5个类别：协议支持，认证，Credentials，Cookie，配置管理，缓存管理。

![alt text](/img/URLSessionProgrammingGuideIntroduction/1.png)

### URL Loading
(URL加载)

URL加载系统中最常用的类允许你的应用程序从URL指定的资源检索内容。你可以使用很多方式来检索内容，取决于你应用程序的要求。API的选择取决于你应用程序中OS X或者iOS的目标版本，以及你是希望将数据放在文件或者内存块中。

* 在iOS7或者OS X v10.9及之后，NSURLSession是执行URL请求的最完美的API之选。
* 对于要支持老版本OS X的软件，你可以使用NSURLDownload来下载URL资源的内容到磁盘文件上。
* 对于要支持老版本iOS或者OS X，你可以使用NSURLConnection下载URL资源的内容到内存中。如果需要之后可以再写到磁盘上。

你使用的方法大部分取决于你是希望将数据写入内存或者下载到磁盘。

#### Fetching Content as Data (In Memory)
(获取内容数据(内存中))

在较高层面上来说，有两种基本的方法来获取URL数据：

* 对于简单的请求，使用NSURLSession的API直接从NSURL对象那里检索资源内容，NSData对象或者磁盘上的文件。
* 对于更复杂的请求－请求上传数据，比如－提供一个NSURLRequest对象(或者它可变的子类，NSMutableURLRequest)给NSURLSession 或 NSURLConnection。

不管你使用哪种方法，你的应用程序可以使用两种方法获取响应数据：

* 提供一个处理完成的Block。当URL加载的类完成从服务器获取数据后会调用这个Block。
* 提供一个自定义的代理，URL加载的类从原始资源那里查询到数据会定期的调用你代理的方法。你的应用程序负责积累这些数据，如果需要的话。

除了数据本身之外，URL加载的类给你的代理或者处理完成的Block提供了一个响应对象，封装了与请求相关的源数据，比如MIME类型和内容长度。

**相关章节**：[ Using NSURLSession ](https://developer.apple.com/library/prerelease/tvos/documentation/Cocoa/Conceptual/URLLoadingSystem/Articles/UsingNSURLSession.html#//apple_ref/doc/uid/TP40013509-SW1)，[ Using NSURLConnection ](https://developer.apple.com/library/prerelease/tvos/documentation/Cocoa/Conceptual/URLLoadingSystem/Tasks/UsingNSURLConnection.html#//apple_ref/doc/uid/20001836-BAJEAIEE)。

#### Downloading Content as a File
(获取内容文件)

在较高层面上来说，有两种基本的方法来获取URL文件：

* 对于简单的请求，使用NSURLSession的API直接从NSURL对象那里检索资源内容，NSData对象或者磁盘上的文件。
* 对于更复杂的请－请求上传数据，比如－提供一个NSURLRequest对象(或者它可变的子类，NSMutableURLRequest)给NSURLSession 或 NSURLConnection。

NSURLSession类通过NSURLDownload类提供了两个显著的优势：在iOS中可用，即使你的应用程序挂起、终止、崩溃了，下载任务仍然可以在后台运行。

**注意**：由NSURLDownload或者NSURLSession实例启动下载，不会缓存。如果你需要缓存结果，你的应用程序必须使用NSURLConnection或者NSURLSession将数据写入磁盘本身。

**相关章节**：[ Using NSURLSession ](https://developer.apple.com/library/prerelease/tvos/documentation/Cocoa/Conceptual/URLLoadingSystem/Articles/UsingNSURLSession.html#//apple_ref/doc/uid/TP40013509-SW1)，[ Using NSURLDownload ](https://developer.apple.com/library/prerelease/tvos/documentation/Cocoa/Conceptual/URLLoadingSystem/Tasks/UsingNSURLDownload.html#//apple_ref/doc/uid/20001839-BAJEAIEE)。

### Helper Classes
(辅助类)

URL加载类使用两种辅助类来提供额外的源数据－一个是用来请求的NSURLRequest和用作服务器响应的NSURLResponse。

#### URL Requests
(URL请求)

一个NSURLRequest对象封装了URL和任何特定协议的特性，使用独立于协议的方式封装。同时它也指定了关于使用本地缓存数据的策略，当使用NSURLConnection或者NSURLDownload，提供了一个设置连接超时的接口。(对于NSURLSession，超时是在每个会话的基础上进行配置。)

**注意**：当一个客户端应用程序使用NSMutableURLRequest的实例初始化一个connection或者download时，会要求一个深拷贝。当下载初始化后改变初始的请求是没有效果的。

某些协议支持特定协议的属性。比如，对于HTTP协议，NSURLRequest增加了返回HTTP请求body，header，以及transfer method的方法。同时NSMutableURLRequest也增加了设置这些值的方法。

使用URL请求对象的细节在本书中描述。

#### Response Metadata
(响应源数据)

服务器对于一个请求的响应可以看作两部分：描述内容的源数据和内容数据本身。大多数协议的源数据都是相同的，使用NSURLResponse类进行封装，包括MIME类型，期望的内容长度，字符编码，以及URL。协议特定的NSURLResponse子类可以提供额外的源数据。比如，NSHTTPURLResponse存储了服务器返回的headers(首部字段)和状态码。

**重要**：只有响应的源数据存储在NSURLResponse对象中。其它的URL加载类通过你应用程序的处理完成的Block或者对象的代理来提供响应数据。一个NSCachedURLResponse实例封装了NSURLResponse对象，URL内容数据，以及你应用程序提供的额外的信息。参见[ Cache Management ](https://developer.apple.com/library/prerelease/tvos/documentation/Cocoa/Conceptual/URLLoadingSystem/URLLoadingSystem.html#//apple_ref/doc/uid/20001834-155585)了解详细信息。

使用URL响应对象的细节在本书中描述。

#### Redirection and Other Request Changes
(重定向和其它请求更改)

一些协议，比如HTTP，为服务器提供了一种告知你应用程序请求的URL内容已经移动到其它URL的方式。当这种情况发生的时候，URL加载类会通知它们的代理。如果你应用程序提供了合适的代理方法，你的应用程序就直到如何重定向，返回重定向后的响应数据或者错误。

**相关章节**：[ Handling Redirects and Other Request Changes ](https://developer.apple.com/library/prerelease/tvos/documentation/Cocoa/Conceptual/URLLoadingSystem/Articles/RequestChanges.html#//apple_ref/doc/uid/TP40009506-SW1)。

#### Authentication and Credentials
(认证和Credentials)

一些服务器限制访问某些内容，要求用户通过提供某种凭证－客户端证书，用户名和密码等等，以获得访问权限。在web服务器的情况下，限制内容被分组成为需要凭证的领域。证书也被用来确定信任另一个方向来评估您的应用程序是否应该信任服务器。

URL加载系统提供了安全凭证持久化模型和保护区。你的应用程序可以为一个单一的请求指定持久凭证，在应用程序启动时，或者永久的在用户钥匙串中。

**注意**：Credentials持久化的存储在用户的钥匙串中，所有的应用程序是共享的。

NSURLCredential类封装了一个凭证，包括认证的信息(用户名和密码)和持久化的行为。NSURLProtectionSpace类表示需要特定凭证的区域。一个保护空间可以限定在一个单一的URL，包含web服务器的领域，或者代理。

NSURLCredentialStorage类共享的一个实例(单例)管理凭证，以及提供了NSURLCredential对象与对应的NSURLProtectionSpace对象的映射，提供了认证信息。

NSURLAuthenticationChallenge类封装了由NSURLProtocol协议实现认证请求所需的信息：一个提议凭证，涉及的保护空间，用来检测认证所需的错误或者响应，以及尝试认证的次数。NSURLAuthenticationChallenge实例也指定了发起认证的对象。启动对象称为发送者，必须适配了NSURLAuthenticationChallengeSender协议。

NSURLAuthenticationChallenge实例被NSURLProtocol子类用来通知URL加载系统需要的认证。同时也提供了便于定制认证处理的NSURLConnection和NSURLDownload的委托方法。

**相关章节**：[ Authentication Challenges and TLS Chain Validation ](https://developer.apple.com/library/prerelease/tvos/documentation/Cocoa/Conceptual/URLLoadingSystem/Articles/AuthenticationChallenges.html#//apple_ref/doc/uid/TP40009507-SW1)。

#### Cache Management
(缓存管理)

URL加载系统提供了磁盘和内存缓存，允许应用程序减少对网络连接的依赖，提供一个快速的响应缓存。缓存基于每个应用程序存储。NSURLConnection的缓存，通过初始的NSURLRequest对象实例的缓存策略指定。

NSURLCache类提供了配置缓存大小和存储在磁盘位置上的方法。同时也提供了管理包含响应缓存的NSCachedURLResponse对象的集合的方法。

NSCachedURLResponse对象封装了NSURLResponse对象和URL内容数据。NSCachedURLResponse同时也提供了用户信息字典，你的应用程序可以用来存储任何自定义的数据。

不是所有的协议实现都支持响应缓存。目前只有http和https请求支持缓存。

NSURLConnection对象可以通过实现**connection:willCacheResponse:**代理方法来控制一个响应是否缓存以及是否只缓存在内存中。

**相关章节**：[ Understanding Cache Access ](https://developer.apple.com/library/prerelease/tvos/documentation/Cocoa/Conceptual/URLLoadingSystem/Concepts/CachePolicies.html#//apple_ref/doc/uid/20001843-BAJEAIEE)。

#### Cookie Storage
(Cookie存储)

由于HTTP协议的无状态特性，客户端经常使用Cookie来提供跨URL请求数据的持久化存储。URL加载系统提供了接口创建和管理Cookie，将Cookie当作HTTP请求的一部分发送，然后在解释web服务器响应时接收Cookie。

OS X和iOS提供了NSHTTPCookieStorage类，又提供了一种管理NSHTTPCookie对象的结合的接口。在OS X中，所有的应用程序共享Cookie，在iOS中，Cookie是基于每个应用程序存储的。

**相关章节**：[ Cookie Storage ](https://developer.apple.com/library/prerelease/tvos/documentation/Cocoa/Conceptual/URLLoadingSystem/CookiesandCustomProtocols/CookiesandCustomProtocols.html#//apple_ref/doc/uid/10000165i-CH10-SW1)。

#### Protocol Support
(协议支持)

URL加载系统本身支持http，https，file，ftp 和 data协议。但是，URL加载系统也允许你的应用程序注册你自己的类来支持额外的应用程序层级的网络协议。你也可以添加特定协议的特性到URL请求和响应对象中。

**相关章节**：[ Cookies and Custom Protocols ](https://developer.apple.com/library/prerelease/tvos/documentation/Cocoa/Conceptual/URLLoadingSystem/CookiesandCustomProtocols/CookiesandCustomProtocols.html#//apple_ref/doc/uid/10000165i-CH10-SW3)。

## How to Use This Document
(如何使用本文档)

本文档根据章节描述的不同的URL加载类来分的。为了决定使用哪种API，参阅[ URL Loading ](https://developer.apple.com/library/prerelease/tvos/documentation/Cocoa/Conceptual/URLLoadingSystem/URLLoadingSystem.html#//apple_ref/doc/uid/10000165-SW1)。在你决定使用哪种API之后，请阅读API指定的章节或者如下章节：

* 对于使用NSURLSession类来异步获取URL内容到内存或者下载文件到磁盘，阅读[ Using NSURLSession ](https://developer.apple.com/library/prerelease/tvos/documentation/Cocoa/Conceptual/URLLoadingSystem/Articles/UsingNSURLSession.html#//apple_ref/doc/uid/TP40013509-SW1)。然后阅读[ Life Cycle of a URL Session ](https://developer.apple.com/library/prerelease/tvos/documentation/Cocoa/Conceptual/URLLoadingSystem/NSURLSessionConcepts/NSURLSessionConcepts.html#//apple_ref/doc/uid/10000165i-CH2-SW1)学习NSURLSession与代理之间交互的细节。
* 对于使用NSURLConnection异步获取URL内容到内存，阅读[ Using NSURLConnection ](https://developer.apple.com/library/prerelease/tvos/documentation/Cocoa/Conceptual/URLLoadingSystem/Tasks/UsingNSURLConnection.html#//apple_ref/doc/uid/20001836-BAJEAIEE)。
* 对于使用NSURLDownload异步下载文件到磁盘，阅读[ Using NSURLDownload ](https://developer.apple.com/library/prerelease/tvos/documentation/Cocoa/Conceptual/URLLoadingSystem/Tasks/UsingNSURLDownload.html#//apple_ref/doc/uid/20001839-BAJEAIEE)。

在阅读了一个或者多个API指定的章节后，你应该阅读以下章节，也是相关的API：

* [ Encoding URL Data ](https://developer.apple.com/library/prerelease/tvos/documentation/Cocoa/Conceptual/URLLoadingSystem/WorkingwithURLEncoding/WorkingwithURLEncoding.html#//apple_ref/doc/uid/10000165i-CH12-SW1)解释了如何编码具体的字符串，让它们在URL中能够安全的使用。
* [ Handling Redirects and Other Request Changes ](https://developer.apple.com/library/prerelease/tvos/documentation/Cocoa/Conceptual/URLLoadingSystem/Articles/RequestChanges.html#//apple_ref/doc/uid/TP40009506-SW1)描述了当你请求的URL的内容移动到了其它的URL时你能做的操作。
* [ Authentication Challenges and TLS Chain Validation ](https://developer.apple.com/library/prerelease/tvos/documentation/Cocoa/Conceptual/URLLoadingSystem/Articles/AuthenticationChallenges.html#//apple_ref/doc/uid/TP40009507-SW1)描述了当你的连接被安全服务器拒绝时认证的处理过程。
* [ Understanding Cache Access ](https://developer.apple.com/library/prerelease/tvos/documentation/Cocoa/Conceptual/URLLoadingSystem/Concepts/CachePolicies.html#//apple_ref/doc/uid/20001843-BAJEAIEE)描述了连接在请求期间如何使用缓存。
* [ Cookies and Custom Protocols ](https://developer.apple.com/library/prerelease/tvos/documentation/Cocoa/Conceptual/URLLoadingSystem/CookiesandCustomProtocols/CookiesandCustomProtocols.html#//apple_ref/doc/uid/10000165i-CH10-SW3)描述了用来管理Cookie的可用的类以及支持自定义的应用层级的协议。

## See Also
(请参阅)

可用的案例代码：

* LinkedImageFetcher (OS X) 和 AdvancedURLConnections (iOS)，使用NSURLConnection和自定义认证。
* SpecialPictureProtocol (OS X) 和 CustomHTTPProtocol (iOS)展示了如何实现自定义的NSURLProtocol子类。
* QuickLookDownloader (in the Mac Developer Library)使用NSURLDownload管理从互联网下载的文件。
