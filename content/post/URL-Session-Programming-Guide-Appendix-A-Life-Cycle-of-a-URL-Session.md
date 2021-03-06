---
title: "URL Session Programming Guide - Appendix A: Life Cycle of a URL Session"
date: 2016-01-29 14:14:22
categories: 
- 编程指南
tags: 
- NSURLSession
- NSURLConnection
metaAlignment: center
autoThumbnailImage: no
coverImage: //d1u9biwaxjngwg.cloudfront.net/welcome-to-tranquilpeak/city.jpg

---

[官方文档](https://developer.apple.com/library/prerelease/tvos/documentation/Cocoa/Conceptual/URLLoadingSystem/NSURLSessionConcepts/NSURLSessionConcepts.html#//apple_ref/doc/uid/10000165i-CH2-SW1)
<!--more-->

# Life Cycle of a URL Session
(URL Session的生命周期)

你可以有两种方式使用NSURLSession的API：使用系统提供的代理或者你自己的代理。通常，如果你的应用程序需要做如下任何事情时你必须使用自己的代理：

* 在应用程序没有运行的时候使用后台会话下载或者上传内容。
* 执行自定义的认证。
* 执行自定义的SSL证书验证。
* 决定传输的内容是应该下载到磁盘或者基于服务器返回的MIME类型或者类似标准来显示。
* 从body stream上传数据(而不是一个NSData对象)。
* 编程限制缓存。
* 编程限制HTTP重定向。

如果你的应用程序不需要做以上任何事情，你的应用程序可以使用系统提供的代理。取决于你使用哪种技术，你应该阅读以下之一的章节：

* [ Life Cycle of a URL Session with System-Provided Delegates ](https://developer.apple.com/library/prerelease/tvos/documentation/Cocoa/Conceptual/URLLoadingSystem/NSURLSessionConcepts/NSURLSessionConcepts.html#//apple_ref/doc/uid/10000165i-CH2-SW2)提供了轻量级的关于如何创建和使用URL会话的代码。即使你尝试编写你自己的代理你也应该阅读这个章节，因为它给了一个你的代码配置使用对象的完整的模版。
* [ Life Cycle of a URL Session with Custom Delegates ](https://developer.apple.com/library/prerelease/tvos/documentation/Cocoa/Conceptual/URLLoadingSystem/NSURLSessionConcepts/NSURLSessionConcepts.html#//apple_ref/doc/uid/10000165i-CH2-SW42)提供了URL会话操作的每一个步骤的完整视图。你应该阅读本章节来帮助你明白会话是如何与代理交互的。具体来说，这章节解释了每个代理方法在何时调用。

## Life Cycle of a URL Session with System-Provided Delegates
(使用系统提供代理的URL Session的生命周期)

如果你使用NSURLSession类而没有提供代理对象，系统提供的代理会帮你处理很多细节。这里是当你使用系统提供代理的NSURLSession调用方法的基本顺序：

1. 创建一个会话配置。对于后台会话，这个配置必须包含一个唯一标识。存储那个标识，如果你的应用程序崩溃或者终结或者挂起时使用这个标识与会话重新关联。
2. 创建会话，指定一个配置对象和nil的代理。
3. 使用会话创建一个任务对象来表示资源请求。每一个任务都以挂起的状态开始。当你应用程序调用了任务对象的resume方法后，它就会开始下载指定的资源。任务对象是NSURLSessionTask的子类，如NSURLSessionDataTask，NSURLSessionUploadTask，或者NSURLSessionDownloadTask，取决于你想实现的行为。这些对象类似于NSURLConnection对象，但是给你更多的控制和统一的代理模式。尽管你的应用程序可以(通常来说应该)添加不止一个任务给会话，为了简单起见，其余的步骤描述了单个任务的生命周期。

**重要**：如果你使用NSURLSession类而没有提供代理，那么你应用程序调用创建任务的方法必须传入completionHandler参数，否则你无法从类获取到数据。

4. 对于一个下载任务，在与服务器传输期间，如果你的用户告诉应用程序暂停下载，通过调用cancelByProducingResumeData:方法取消下载。之后可以传递返回的数据给downloadTaskWithResumeData:或者downloadTaskWithResumeData:completionHandler:方法来创建新的下载任务继续下载。
5. 当任务完成时，NSURLSession对象会调用任务的完成处理句柄。

**注意**：NSURLSession不会通过error参数来报告服务器的错误。你应用程序通过这个错误接收到的都是客户端这边的错误，比如无法解析主机或者无法连接主机。错误码在[ URL Loading System Error Codes ](https://developer.apple.com/library/prerelease/tvos/documentation/Cocoa/Reference/Foundation/Miscellaneous/Foundation_Constants/index.html#//apple_ref/doc/constant_group/URL_Loading_System_Error_Codes)中描述。服务端的错误通过NSHTTPURLResponse对象里面的HTTP状态码来报告。更多信息，参见[ NSHTTPURLResponse ](https://developer.apple.com/library/prerelease/tvos/documentation/Cocoa/Reference/Foundation/Classes/NSHTTPURLResponse_Class/index.html#//apple_ref/occ/cl/NSHTTPURLResponse)和[ NSURLResponse ](https://developer.apple.com/library/prerelease/tvos/documentation/Cocoa/Reference/Foundation/Classes/NSURLResponse_Class/index.html#//apple_ref/occ/cl/NSURLResponse)类的文档。

6. 当你的应用程序不再需要会话，通过调用[ invalidateAndCancel ](https://developer.apple.com/library/prerelease/tvos/documentation/Foundation/Reference/NSURLSession_class/index.html#//apple_ref/occ/instm/NSURLSession/invalidateAndCancel)(取消未完成的任务)或者[ finishTasksAndInvalidate ](https://developer.apple.com/library/prerelease/tvos/documentation/Foundation/Reference/NSURLSession_class/index.html#//apple_ref/occ/instm/NSURLSession/finishTasksAndInvalidate)(允许未完成的任务完成之后再使会话无效)。

## Life Cycle of a URL Session with Custom Delegates
(使用自定义代理的URL Session的生命周期)

你可以经常使用NSURLSession的API而无需提供代理。但是，如果你使用NSURLSession的API进行后台下载和上传，或者你需要以非缺省的方式处理认证和缓存，那么你必须提供一个适配了会话代理协议的代理，一个或者多个任务代理协议，或者这些协议的组合。这个代理服务于许多用途：

* 当使用下载任务时，NSURLSession对象会使用代理给你的应用程序提供一个文件URL来获取下载的数据。所有的后台下载和上传都要求代理。这些代理必须提供了[ NSURLSessionDownloadDelegate ](https://developer.apple.com/library/prerelease/tvos/documentation/Foundation/Reference/NSURLSessionDownloadDelegate_protocol/index.html#//apple_ref/occ/intf/NSURLSessionDownloadDelegate)协议的所有代理方法。
* 代理可以处理某些认证挑战。
* 代理为基于流上传数据到服务器的任务提供body streams。
* 代理决定是否遵循HTTP重定向。
* NSURLSession对象使用代理为你的应用程序提供每个数据传输的状态。数据任务代理接收初始调用，你可以将请求转换成下载及后续调用，提供了从远程服务器接收的数据块。
* 代理是告诉你应用程序传输任务完成的方法之一。

如果你的URL会话(要求后台任务)使用自定义的代理，那么URL会话的生命周期就很复杂。下面是使用自定义代理时基本的代理方法的调用顺序：

1. 创建一个会话配置。对于后台会话，这个配置必须包含一个唯一标识。存储那个标识，如果你的应用程序崩溃或者终结或者挂起时使用这个标识与会话重新关联。
2. 创建会话，指定配置对象，可选的，一个代理。
3. 使用会话创建一个任务对象来表示资源请求。每一个任务都以挂起的状态开始。当你应用程序调用了任务对象的resume方法后，它就会开始下载指定的资源。任务对象是NSURLSessionTask的子类，如NSURLSessionDataTask，NSURLSessionUploadTask，或者NSURLSessionDownloadTask，取决于你想实现的行为。这些对象类似于NSURLConnection对象，但是给你更多的控制和统一的代理模式。尽管你的应用程序可以(通常来说应该)添加不止一个任务给会话，为了简单起见，其余的步骤描述了单个任务的生命周期。
4. 如果远程服务器返回一个状态码指示要求认证以及如果认证要求连接级别的挑战(比如SSL客户端证书)，NSURLSession就会调用认证挑战的代理方法。
   * 对于会话级别的挑战，NSURLAuthenticationMethodNTLM，NSURLAuthenticationMethodNegotiate，NSURLAuthenticationMethodClientCertificate，或者NSURLAuthenticationMethodServerTrust，NSURLSession对象调用会话代理的URLSession:didReceiveChallenge:completionHandler:方法。如果你的应用程序没有提供会话代理方法，NSURLSession对象会调用任务代理的URLSession:task:didReceiveChallenge:completionHandler:方法来处理认证挑战。
   * 对于非会话级别的挑战(其它所有情况)，NSURLSession对象会调用会话代理的URLSession:task:didReceiveChallenge:completionHandler:方法来处理挑战。如果你的应用程序提供了会话代理以及你需要处理认证，那么你必须在任务级别处理认证或者提供一个任务级别的处理器，显示调用每一个会话的处理器。会话的代理方法URLSession:didReceiveChallenge:completionHandler:对于非会话级别的挑战并不会调用。
   **注意**：Kerberos认证是透明处理的。
   如果上传任务的认证失败，任务的数据由流来提供，NSURLSession对象调用代理的URLSession:task:needNewBodyStream:代理方法。代理必须提供一个新的NSInputStream对象来为新的请求提供新的数据。
   关于如何实现NSURLSession认证的代理方法，参阅[ Authentication Challenges and TLS Chain Validation ](https://developer.apple.com/library/prerelease/tvos/documentation/Cocoa/Conceptual/URLLoadingSystem/Articles/AuthenticationChallenges.html#//apple_ref/doc/uid/TP40009507-SW1)。
5. 根据接收到的HTTP重定向响应，NSURLSession对象会调用代理的URLSession:task:willPerformHTTPRedirection:newRequest:completionHandler:方法。代理方法会调用提供的完成处理器，一个新的NSURLRequest对象(重定向到不同的URL)，或者nil(将重定向的响应当作有效的效应然后作为结果返回)。
   * 如果遵循重定向，会返回到步骤四(认证挑战处理)。
   * 如果代理没有实现这些方法，重定向会遵循最大的重定向次数。
6. 对于调用downloadTaskWithResumeData:或者downloadTaskWithResumeData:completionHandler:方法重新创建的下载任务，NSURLSession会使用新的任务对象调用URLSession:downloadTask:didResumeAtOffset:expectedTotalBytes:方法。
7. 对于一个数据任务，NSURLSession对象会调用代理的URLSession:dataTask:didReceiveResponse:completionHandler:方法。决定是否将数据任务转换成为下载任务，然后调用完成回调来接收数据或者下载数据。如果你的应用程序选择将数据任务转换成为下载任务，NSURLSession会使用新的下载任务当作参数调用代理的URLSession:dataTask:didBecomeDownloadTask:方法。在调用之后，代理不再接收来自数据任务更多的回调，然后接收来自下载任务的回调。
8. 如果是uploadTaskWithStreamedRequest:方法创建的任务，NSURLSession会调用代理的URLSession:task:needNewBodyStream:方法来提供body data。
9. 在初始化上传到服务器的内容期间，代理会定期的接收URLSession:task:didSendBodyData:totalBytesSent:totalBytesExpectedToSend:回调来报告上传的进度。
10. 在与服务器传输期间，任务的代理会定期的接收回调来报告上传的进度。对于一个下载任务，会话会调用代理的URLSession:downloadTask:didWriteData:totalBytesWritten:totalBytesExpectedToWrite:方法，该方法会携带成功写入磁盘的比特数。对于数据任务，会话会调用代理的URLSession:dataTask:didReceiveData:方法，该方法懈怠了接收的数据块。对于一个下载任务，在与服务器传输期间，如果用户告诉应用程序暂停下载，通过调用cancelByProducingResumeData:方法来取消任务。之后，如果用户要求你应用程序恢复下载，你可以调用 ownloadTaskWithResumeData:或者downloadTaskWithResumeData:completionHandler:方法，传入返回的恢复数据来创建一个新的下载任务继续之前的下载，然后返回步骤3(创建恢复的任务对象)。
11. 对于一个数据任务，NSURLSession对象会调用代理的URLSession:dataTask:willCacheResponse:completionHandler:方法。你的应用程序应该决定是否允许缓存。如果你没有实现这个方法，那么默认行为就是使用指定了会话配置对象的缓存策略。
12. 如果一个下载任务成功完成，NSURLSession对象就会调用任务的URLSession:downloadTask:didFinishDownloadingToURL:方法，该方法会携带一个下载了数据的本地的临时文件。你的应用程序在该方法返回之前要么从文件中读取数据要么移动你应用程序沙盒中的永久的地址。
13. 当任何任务完成时，NSURLSession对象会调用代理的URLSession:task:didCompleteWithError:方法，该方法会携带一个error对象或者nil(任务成功完成)。如果任务失败，大多数应用程序应该重新请求直到用户取消下载或者服务器返回错误指示请求永远都不会成功。你的应用程序不应该立即重试。相反的，你应该使用reachability的API来检测服务器是否是可达的，并且只有在接收到可达性改变的通知后发起一个新的请求。如果下载任务是可以恢复的，NSError对象的userInfo字典包含一个NSURLSessionDownloadTaskResumeData的键。你的应用程序应该传递这个值给downloadTaskWithResumeData:或者downloadTaskWithResumeData:completionHandler:来创建一个新的下载任务来继续已经存在的下载。如果任务是不可恢复的，你的应用程序应该创建一个新的下载任务然后重新开始任务。在任一情况下，如果传输失败超过一个服务器错误以外的任何原因，转到步骤3(创建和恢复任务对象)。
**注意**：NSURLSession不会通过error参数来报告任何服务器的错误。通过error参数你代理接收到的错误只是客户端这一边的错误，比如无法解析主机或者无法连接到主机。错误代码在[ URL Loading System Error Codes ](https://developer.apple.com/library/prerelease/tvos/documentation/Cocoa/Reference/Foundation/Miscellaneous/Foundation_Constants/index.html#//apple_ref/doc/constant_group/URL_Loading_System_Error_Codes)中描述。服务端的错误通过NSHTTPURLResponse对象的HTTP状态码来报告。关于更多信息，参阅[ NSHTTPURLResponse ](https://developer.apple.com/library/prerelease/tvos/documentation/Cocoa/Reference/Foundation/Classes/NSHTTPURLResponse_Class/index.html#//apple_ref/occ/cl/NSHTTPURLResponse)和[ NSURLResponse ](https://developer.apple.com/library/prerelease/tvos/documentation/Cocoa/Reference/Foundation/Classes/NSURLResponse_Class/index.html#//apple_ref/occ/cl/NSURLResponse)类的文档。
14. 如果响应是多部分编码的，会话可能会再次调用代理的didReceiveResponse方法，伴随着零次或者多次didReceiveData调用。如果发生这些，返回步骤7(处理didReceiveResponse调用)。
15. 当你不在需要会话时，通过调用invalidateAndCancel(取消未完成的任务)或者finishTasksAndInvalidate(允许未完成的任务完成之后再使会话对象无效)方法来使会话对象无效。在会话对象无效后，当所有的未完成的任务完成或者取消时，会话会给代理发送URLSession:didBecomeInvalidWithError:消息。当代理方法返回时，会话会处置对代理的强引用。
**重要**：会话对象会保持对代理的强引用直到你的应用程序显式的使会话无效。如果你不将会话失效，你的应用程序会内存泄漏。

如果你的应用程序取消了一个正在进行的下载，NSURLSession对象会由因为发生错误而调用代理的URLSession:task:didCompleteWithError:方法。
