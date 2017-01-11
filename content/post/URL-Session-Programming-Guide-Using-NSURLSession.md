---
title: "URL Session Programming Guide - Using NSURLSession"
date: 2016-01-29 14:10:55
categories: 
- 编程指南
tags: 
- NSURLSession
- NSURLConnection
metaAlignment: center
autoThumbnailImage: no
coverImage: //d1u9biwaxjngwg.cloudfront.net/welcome-to-tranquilpeak/city.jpg

---

[官方文档](https://developer.apple.com/library/prerelease/tvos/documentation/Cocoa/Conceptual/URLLoadingSystem/Articles/UsingNSURLSession.html#//apple_ref/doc/uid/TP40013509-SW26)
<!--more-->

# Using NSURLSession
(使用NSURLSession)

NSURLSession及其相关的类提供了通过HTTP协议下载内容的API。这些API提供了丰富的代理方法支持认证，并且让你的应用程序在没有运行(在iOS中即是挂起时)时具有后台执行下载任务的能力。

为了使用NSURLSession的API，你的应用程序创建了一系列的会话，每一个会话都协调了一组数据传输相关的任务。比如，如果你在开发一个web浏览器，你的应用程序可能会每个标签或者窗口创建一个会话。对于每个会话，你的应用程序可能会添加一系列的任务到会话中，每个任务表示了一个特定URL的请求(以及任何返回的HTTP重定向)。

类似于大多数网络请求API，NSURLSession的API是高度异步的。如果你使用默认的NSURLSession，系统会提供代理，你必须提供一个处理完成的Block，用来在传输成功完成或者发生了错误时给你的应用程序返回数据。另外，如果你提供了你自己自定义的代理对象，任务对象在接收来自服务器的数据后调用这些代理的方法。

**注意**：完成回调主要用来当作一种可替代的自定义代理。如果你使用接收完成回调的方法创建了一个任务，那么用来传输响应和数据的代理方法就不会被调用。

NSURLSession的API提供了状态和进度属性，除了将这些信息传递给代理。它也支持取消，重新开始(恢复)，挂起任务，也提供了恢复挂起，取消，或者下载失败的任务的能力。

## Understanding URL Session Concepts
(理解URL Session的概念)

在一个会话中，任务的行为依赖于三件事情：会话的类型(取决于创建会话时使用的配置对象)，任务的类型，以及任务创建时应用程序是否在前台。

### Types of Sessions
(会话类型)

NSURLSession的API支持三种类型的会话，由创建会话时配置对象的类型决定：

* Default类型的会话的行为类似于其它用来下载URL内容的Foundation方法。使用持久化的基于磁盘的缓存和存储在用户钥匙串理的凭证。
* Ephemeral类型的会话不会存储任何数据到磁盘；所有的缓存，凭证等等都保存在内存中，与会话绑定。因此，当你的应用程序无效的时候，他们也被自动抹去了。
* Background类型的会话类似于Default类型，除了一个分开的进程处理所有的数据传输。Background会话有些额外的限制，在[ Background Transfer Considerations ](https://developer.apple.com/library/prerelease/tvos/documentation/Cocoa/Conceptual/URLLoadingSystem/Articles/UsingNSURLSession.html#//apple_ref/doc/uid/TP40013509-SW44)中描述了。

### Types of Tasks
(任务类型)

对于会话，NSURLSession类支持三种类型的任务：data tasks，download tasks，and upload tasks。

* Data tasks使用NSData对象发送和接收数据。Data tasks主要针对你应用程序中那些与服务器短的、经常交互的请求。Data tasks能够一次返回一小段数据，或者通过完成处理句炳一次返回。
* Download tasks在文件形式中检索数据，支持在应用程序没有运行时进行后台下载任务。
* Upload tasks以文件形式发送数据，支持在应用程序没有运行时进行后台上传任务。

### Background Transfer Considerations
(后台传输的注意事项)

NSURLSession类支持应用程序被挂起时后台传输。只有使用后台会话的配置对象创建的NSURLSession对象才支持后台传输(使用[ backgroundSessionConfiguration: ](https://developer.apple.com/library/prerelease/tvos/documentation/Foundation/Reference/NSURLSessionConfiguration_class/index.html#//apple_ref/occ/clm/NSURLSessionConfiguration/backgroundSessionConfiguration:)创建后台会话的配置对象)。

因为实际传输在分开的进程中处理，以及重启你应用程序的进程是相对昂贵的，所以一些功能无法实现，导致了如下所示的限制：

* 会话必须给每一个传输提供一个代理。(对于上传和下载，代理的行为类似于处理传输。)
* 只有HTTP和HTTPS协议支持(自定义协议不支持)。
* 始终遵循重定向。
* 只支持基于文件的上传任务(基于数据对象和流的上传任务在程序退出时就会失败)。
* 如果后台传输是应用程序在后台时初始化的，那么配置对象的[ discretionary ](https://developer.apple.com/library/prerelease/tvos/documentation/Foundation/Reference/NSURLSessionConfiguration_class/index.html#//apple_ref/occ/instp/NSURLSessionConfiguration/discretionary)属性应该设置为**true**。

**注意**：在iOS8和OS X 10.10之前，Data tasks不支持后台会话。

iOS和OS X重新启动应用程序的行为有一些不一样。

在iOS中，当一个后台传输完成或者需要凭证，如果你的应用程序没有在运行，iOS在后台自动重启你的应用程序然后在应用程序的UIApplicationDelegate对象上调用**application:handleEventsForBackgroundURLSession:completionHandler:**方法。这个方法提供了导致你应用程序重启的会话的identifier。你的应用程序应该存储完成处理句柄，使用这个相同的identifier创建一个后台配置对象，然后使用这个后台配置对象创建一个会话。新的会话会自动与后台活动的关联。之后，当会话完成了最后一个后台下载任务，它会给会话的代理发送一个**URLSessionDidFinishEventsForBackgroundURLSession:**消息。你的会话代理应该调用存储的完成句柄。

在iOS和OS X中，当用户重启了你的应用程序，你的应用程序应该立即使用相同的identifier为应用程序最后运行的任务创建后台配置的对象，然后为每一个配置对象创建一个会话。这些新的会话会自动与后台活动的关联。

**注意**：每个identifier只能创建一个会话(创建配置对象的时候指定)。多个会话共享同一个identifier的行为是不确定。

当应用程序挂起的时候，有任务完成，代理的**URLSession:downloadTask:didFinishDownloadingToURL:**方法会被调用，以及与新的下载文件相关的任务和URL。

类似的，如果任务需要凭证，NSURLSession对象调用代理的**URLSession:task:didReceiveChallenge:completionHandler:**方法或者**URLSession:didReceiveChallenge:completionHandler:**方法。

后台会话中的上传和下载的任务在网络错误后，URL加载系统会自动重试。所以没有必要使用reachability的API来侦测网络确定何时重试失败的任务。

NSURLSession后台传输的例子，参见**Simple Background Transfer**。

### Life Cycle and Delegate Interaction
(声明周期和代理交互)

取决于你使用NSURLSession类做什么，了解会话完整的生命周期可能很有帮助，包括和代理的交互，以及代理调用的顺序，当服务器返回重定向的时候会发生什么，以及当你的应用程序恢复一个下载失败的任务时发生了什么，等等。

关于会话生命周期的完整描述，参见[ Life Cycle of a URL Session ](https://developer.apple.com/library/prerelease/tvos/documentation/Cocoa/Conceptual/URLLoadingSystem/NSURLSessionConcepts/NSURLSessionConcepts.html#//apple_ref/doc/uid/10000165i-CH2-SW1)。

### NSCopying Behavior
(NSCopying行为)

会话和任务对象按照如下适配**NSCopying**协议：

* 当你的应用程序复制会话或者任务对象时，你会得到同一个对象。
* 当你的应用程序复制一个配置对象，你会得到一个新的拷贝的对象，便于你可以独立的修改。

## Sample Delegate Class Interface
(简单的代理类接口)

接下来的代码片段基于Listing 1-1所示的类接口。

Listing 1-1  Sample delegate class interface

    #import <Foundation/Foundation.h>


    typedef void (^CompletionHandlerType)();

    @interface MySessionDelegate : NSObject <NSURLSessionDelegate, NSURLSessionTaskDelegate, NSURLSessionDataDelegate, NSURLSessionDownloadDelegate>

    @property NSURLSession *backgroundSession;
    @property NSURLSession *defaultSession;
    @property NSURLSession *ephemeralSession;

    #if TARGET_OS_IPHONE
    @property NSMutableDictionary *completionHandlerDictionary;
    #endif

    - (void) addCompletionHandler: (CompletionHandlerType) handler forSession: (NSString *)identifier;
    - (void) callCompletionHandlerForSession: (NSString *)identifier;


    @end

## Creating and Configuring a Session
(创建配置会话)

NSURLSession的API提供了广泛的配置选项：

* 支持给单个的会话指定私有的缓存，Cookie，凭证，和协议。
* 依赖于特定请求(task)或者一组请求(session)的认证。
* 通过URL上传和下载文件，鼓励数据(文件内容)和源数据分离。
* 配置每个主机的最大连接数。
* 如果一整个资源在一定的时间下不能下载就会触发每个资源超时。
* 最低和最高的TLS版本支持。
* 自定义的代理字典。
* 控制Cookie策略。
* 控制HTTP管道行为。

因为大部分设置都包含在一个分开的配置对象里，你可以重用相同的设置。当你初始化一个会话对象时，按照如下：

* 一个配置对象，管理会话和任务的行为。
* 一个可选的代理对象，用来处理接收的数据以及处理指定给会话和任务的其它事件，比如服务器认证，确定资源请求是否应该被转换成下载等。

如果你没有提供代理对象，NSURLSession对象会使用系统提供的代理。通过这种方式，你可以很容易的使用NSURLSession来替换已经存在的使用[ sendAsynchronousRequest:queue:completionHandler: ](https://developer.apple.com/library/prerelease/tvos/documentation/Cocoa/Reference/Foundation/Classes/NSURLConnection_Class/index.html#//apple_ref/occ/clm/NSURLConnection/sendAsynchronousRequest:queue:completionHandler:)方法的代码。

**注意**：如果的应用程序需要执行后台传输，你必须提供一个自定义的代理。

在你初始化一个会话对象后，你不能改变配置或者代理，除了重新创建一个新的会话。

Listing 1-2展示了如何创建一个normal， ephemeral， 和 background的会话的例子。

Listing 1-2  Creating and configuring sessions

    #if TARGET_OS_IPHONE
    self.completionHandlerDictionary = [NSMutableDictionary dictionaryWithCapacity:0];
    #endif

    /* Create some configuration objects. */

    NSURLSessionConfiguration *backgroundConfigObject = [NSURLSessionConfiguration backgroundSessionConfiguration: @"myBackgroundSessionIdentifier"];
    NSURLSessionConfiguration *defaultConfigObject = [NSURLSessionConfiguration defaultSessionConfiguration];
    NSURLSessionConfiguration *ephemeralConfigObject = [NSURLSessionConfiguration ephemeralSessionConfiguration];


    /* Configure caching behavior for the default session.
     Note that iOS requires the cache path to be a path relative
     to the ~/Library/Caches directory, but OS X expects an
     absolute path.
     */
    #if TARGET_OS_IPHONE
    NSString *cachePath = @"/MyCacheDirectory";

    NSArray *myPathList = NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES);
    NSString *myPath    = [myPathList  objectAtIndex:0];

    NSString *bundleIdentifier = [[NSBundle mainBundle] bundleIdentifier];

    NSString *fullCachePath = [[myPath stringByAppendingPathComponent:bundleIdentifier] stringByAppendingPathComponent:cachePath];
    NSLog(@"Cache path: %@\n", fullCachePath);
    #else
    NSString *cachePath = [NSTemporaryDirectory() stringByAppendingPathComponent:@"/nsurlsessiondemo.cache"];

    NSLog(@"Cache path: %@\n", cachePath);
    #endif





    NSURLCache *myCache = [[NSURLCache alloc] initWithMemoryCapacity: 16384 diskCapacity: 268435456 diskPath: cachePath];
    defaultConfigObject.URLCache = myCache;
    defaultConfigObject.requestCachePolicy = NSURLRequestUseProtocolCachePolicy;

    /* Create a session for each configurations. */
    self.defaultSession = [NSURLSession sessionWithConfiguration: defaultConfigObject delegate: self delegateQueue: [NSOperationQueue mainQueue]];
    self.backgroundSession = [NSURLSession sessionWithConfiguration: backgroundConfigObject delegate: self delegateQueue: [NSOperationQueue mainQueue]];
    self.ephemeralSession = [NSURLSession sessionWithConfiguration: ephemeralConfigObject delegate: self delegateQueue: [NSOperationQueue mainQueue]];

除了后台配置，你可以重用其他的配置对象创建额外的会话。(你不能重用后台会话配置，因为两个后台会话对象共享相同的identifier的行为不确定的。)

任何时候修改配置对象都是安全的。当你创建会话的时候，会话会对配置对象执行深拷贝，所以修改只会影响新创建的会话，不会影响已经存在的。比如，你可能创建了第二个会话，只在Wi-Fi连接下才获取内容，如Listing 1-3所示。

Listing 1-3  Creating a second session with the same configuration object

    ephemeralConfigObject.allowsCellularAccess = NO;

    // ...

    NSURLSession *ephemeralSessionWiFiOnly = [NSURLSession sessionWithConfiguration: ephemeralConfigObject delegate: self delegateQueue: [NSOperationQueue mainQueue]];

## Fetching Resources Using System-Provided Delegates
(使用系统提供的代理获取资源)

使用NSURLSession的最直接的方法就是使用NSURLSession上的方法替换[ sendAsynchronousRequest:queue:completionHandler: ](https://developer.apple.com/library/prerelease/tvos/documentation/Cocoa/Reference/Foundation/Classes/NSURLConnection_Class/index.html#//apple_ref/occ/clm/NSURLConnection/sendAsynchronousRequest:queue:completionHandler:)方法。使用这种方法，你只需要完成两部分代码：

* 创建一个配置对象，然后使用配置对象创建一个会话。
* 当数据完全接受后的处理句柄。

使用系统提供的代理，你可以只使用一行代码就能发起获取特定URL的请求。Listing 1-4演示了这个简单的例子。

**注意**：系统提供的代理只在自定义网络行为的时候有限制。如果你的应用程序需要除了基本请求之外的特别行为，比如自定义的认证或者后台下载，这种技术就不太适合。对于你必须实现完整代理的情形，参见**Life Cycle of a URL Session**。

Listing 1-4  Requesting a resource using system-provided delegates

    NSURLSession *delegateFreeSession = [NSURLSession sessionWithConfiguration: defaultConfigObject delegate: nil delegateQueue: [NSOperationQueue mainQueue]];

    [[delegateFreeSession dataTaskWithURL: [NSURL URLWithString: @"http://www.example.com/"]
                        completionHandler:^(NSData *data, NSURLResponse *response,
                                            NSError *error) {
                            NSLog(@"Got response %@ with error %@.\n", response, error);
                            NSLog(@"DATA:\n%@\nEND DATA\n",
                                  [[NSString alloc] initWithData: data
                                                        encoding: NSUTF8StringEncoding]);
                        }] resume];

## Fetching Data Using a Custom Delegate
(使用自定义代理获取数据)

如果你使用自定义代理获取数据，你的代理至少要实现如下方法：

* [ URLSession:dataTask:didReceiveData: ](https://developer.apple.com/library/prerelease/tvos/documentation/Foundation/Reference/NSURLSessionDataDelegate_protocol/index.html#//apple_ref/occ/intfm/NSURLSessionDataDelegate/URLSession:dataTask:didReceiveData:)方法提供了从请求中获取的数据，每次一小段。
* [ URLSession:task:didCompleteWithError: ](https://developer.apple.com/library/prerelease/tvos/documentation/Foundation/Reference/NSURLSessionTaskDelegate_protocol/index.html#//apple_ref/occ/intfm/NSURLSessionTaskDelegate/URLSession:task:didCompleteWithError:)方法向你的任务指示数据是否完全接收了。

如果你的应用程序需要使用URLSession:dataTask:didReceiveData:方法返回的数据，那么你的代码负责存储这些数据。

比如，一个Web浏览器可能在接收到数据的时候就要呈现数据。要做到这一点，你可能会使用一个字典映射一个任务对象和存储结果的NSMutableData，然后使用NSMutableData对象的[ appendData: ](https://developer.apple.com/library/prerelease/tvos/documentation/Cocoa/Reference/Foundation/Classes/NSMutableData_Class/index.html#//apple_ref/occ/instm/NSMutableData/appendData:)方法拼接新的接收的数据。

Listing 1-5展示了创建和开始一个data task。

Listing 1-5  Data task example

    NSURL *url = [NSURL URLWithString: @"http://www.example.com/"];

    NSURLSessionDataTask *dataTask = [self.defaultSession dataTaskWithURL: url];
    [dataTask resume];

## Downloading Files
(下载文件)

在更高层级上来说，下载文件类似于接收数据。你的应用程序应该实现如下代理方法：

* [ URLSession:downloadTask:didFinishDownloadingToURL: ](https://developer.apple.com/library/prerelease/tvos/documentation/Foundation/Reference/NSURLSessionDownloadDelegate_protocol/index.html#//apple_ref/occ/intfm/NSURLSessionDownloadDelegate/URLSession:downloadTask:didFinishDownloadingToURL:)方法给你的应用程序提供了一个URL，这个URL是临时存储的下载文件的URL。
**重要**：在这方法返回之前，你必须打开文件然后读取文件或者将文件移动到一个永久的地址。当这个方法返回之后，如果这个临时文件还在原来的位置就会被删除。
* [ URLSession:downloadTask:didWriteData:totalBytesWritten:totalBytesExpectedToWrite: ](https://developer.apple.com/library/prerelease/tvos/documentation/Foundation/Reference/NSURLSessionDownloadDelegate_protocol/index.html#//apple_ref/occ/intfm/NSURLSessionDownloadDelegate/URLSession:downloadTask:didWriteData:totalBytesWritten:totalBytesExpectedToWrite:)方法给你的应用程序提供下载进度的状态信息。
* [ URLSession:downloadTask:didResumeAtOffset:expectedTotalBytes: ](https://developer.apple.com/library/prerelease/tvos/documentation/Foundation/Reference/NSURLSessionDownloadDelegate_protocol/index.html#//apple_ref/occ/intfm/NSURLSessionDownloadDelegate/URLSession:downloadTask:didResumeAtOffset:expectedTotalBytes:)方法告诉你的应用程序尝试恢复之前下载失败的任务是否成功。
* [ URLSession:task:didCompleteWithError: ](https://developer.apple.com/library/prerelease/tvos/documentation/Foundation/Reference/NSURLSessionTaskDelegate_protocol/index.html#//apple_ref/occ/intfm/NSURLSessionTaskDelegate/URLSession:task:didCompleteWithError:)告诉你的应用程序是否下载失败。

如果你在后台会话上执行下载，当你应用程序没有下载的时候仍然会继续执行。如果你在默认或者短暂(ephemeral)的会话上运行，下载必须在你应用程序启动后重新开始。

在与服务器传输数据期间，如果用户告诉应用程序暂停下载，你的应用程序可以调用[ cancelByProducingResumeData: ](https://developer.apple.com/library/prerelease/tvos/documentation/Foundation/Reference/NSURLSessionDownloadTask_class/index.html#//apple_ref/occ/instm/NSURLSessionDownloadTask/cancelByProducingResumeData:)方法来取消任务。之后，你的应用程序可以传递已经下载好的数据给[ downloadTaskWithResumeData: ](https://developer.apple.com/library/prerelease/tvos/documentation/Foundation/Reference/NSURLSession_class/index.html#//apple_ref/occ/instm/NSURLSession/downloadTaskWithResumeData:)或者[ downloadTaskWithResumeData:completionHandler: ](https://developer.apple.com/library/prerelease/tvos/documentation/Foundation/Reference/NSURLSession_class/index.html#//apple_ref/occ/instm/NSURLSession/downloadTaskWithResumeData:completionHandler:)方法来创建一个继续之前下载的新的下载任务。

如果传输失败，你的代理的调用[ URLSession:task:didCompleteWithError: ](https://developer.apple.com/library/prerelease/tvos/documentation/Foundation/Reference/NSURLSessionTaskDelegate_protocol/index.html#//apple_ref/occ/intfm/NSURLSessionTaskDelegate/URLSession:task:didCompleteWithError:)方法，并传递一个NSError对象。如果任务是可恢复的，对象的userInfo字典的[ NSURLSessionDownloadTaskResumeData ](https://developer.apple.com/library/prerelease/tvos/documentation/Foundation/Reference/NSURLSession_class/index.html#//apple_ref/c/data/NSURLSessionDownloadTaskResumeData)键会包含一个已经下载了数据的值。你的应用程序可以传递这个已经下载好的数据给[ downloadTaskWithResumeData: ](https://developer.apple.com/library/prerelease/tvos/documentation/Foundation/Reference/NSURLSession_class/index.html#//apple_ref/occ/instm/NSURLSession/downloadTaskWithResumeData:)或者[ downloadTaskWithResumeData:completionHandler: ](https://developer.apple.com/library/prerelease/tvos/documentation/Foundation/Reference/NSURLSession_class/index.html#//apple_ref/occ/instm/NSURLSession/downloadTaskWithResumeData:completionHandler:)方法创建一个新的下载任务，重新开始下载。

Listing 1-6提供了一个下载中等文件大小的例子。Listing 1-7提供了下载任务代理方法的例子。

Listing 1-6  Download task example

    NSURL *url = [NSURL URLWithString: @"https://developer.apple.com/library/ios/documentation/Cocoa/Reference/"
                  "Foundation/ObjC_classic/FoundationObjC.pdf"];

    NSURLSessionDownloadTask *downloadTask = [self.backgroundSession downloadTaskWithURL: url];
    [downloadTask resume];

Listing 1-7  Delegate methods for download tasks

    -(void)URLSession:(NSURLSession *)session downloadTask:(NSURLSessionDownloadTask *)downloadTask didFinishDownloadingToURL:(NSURL *)location
    {
        NSLog(@"Session %@ download task %@ finished downloading to URL %@\n",
              session, downloadTask, location);
        
    #if 0
        /* Workaround */
        [self callCompletionHandlerForSession:session.configuration.identifier];
    #endif
        
    #define READ_THE_FILE 0
    #if READ_THE_FILE
        /* Open the newly downloaded file for reading. */
        NSError *err = nil;
        NSFileHandle *fh = [NSFileHandle fileHandleForReadingFromURL:location
                                                               error: &err];
        
        /* Store this file handle somewhere, and read data from it. */
        // ...
        
    #else
        NSError *err = nil;
        NSFileManager *fileManager = [NSFileManager defaultManager];
        NSString *cacheDir = [[NSHomeDirectory()
                               stringByAppendingPathComponent:@"Library"]
                              stringByAppendingPathComponent:@"Caches"];
        NSURL *cacheDirURL = [NSURL fileURLWithPath:cacheDir];
        if ([fileManager moveItemAtURL:location
                                 toURL:cacheDirURL
                                 error: &err]) {
            
            /* Store some reference to the new URL */
        } else {
            /* Handle the error. */
        }
    #endif
        
    }

    -(void)URLSession:(NSURLSession *)session downloadTask:(NSURLSessionDownloadTask *)downloadTask didWriteData:(int64_t)bytesWritten totalBytesWritten:(int64_t)totalBytesWritten totalBytesExpectedToWrite:(int64_t)totalBytesExpectedToWrite
    {
        NSLog(@"Session %@ download task %@ wrote an additional %lld bytes (total %lld bytes) out of an expected %lld bytes.\n",
              session, downloadTask, bytesWritten, totalBytesWritten, totalBytesExpectedToWrite);
    }

    -(void)URLSession:(NSURLSession *)session downloadTask:(NSURLSessionDownloadTask *)downloadTask didResumeAtOffset:(int64_t)fileOffset expectedTotalBytes:(int64_t)expectedTotalBytes
    {
        NSLog(@"Session %@ download task %@ resumed at offset %lld bytes out of an expected %lld bytes.\n",
              session, downloadTask, fileOffset, expectedTotalBytes);
    }

## Uploading Body Content
(上传正文内容)

你的应用程序可以有三种方式给HTTP POST请求提供请求的正文内容(Body Content)：作为一个NSData对象，文件或者流。通常来说，你的应用程序应该：

* 使用NSData对象，如果你的内存中已经有了数据并且没有理由去处理它。
* 如果你上传的内容已经在磁盘文件中，如果你使用后台传输，或者将数据写入磁盘减少内存对你的应用程序有好处，那么你就使用文件。
* 如果你通过网络接收数据，使用流。或者转换提供给NSURLConnection请求正文的流的代码。

不管你选择哪种方式，如果你的应用程序提供了自定义的会话代理，代理需要实现[ URLSession:task:didSendBodyData:totalBytesSent:totalBytesExpectedToSend: ](https://developer.apple.com/library/prerelease/tvos/documentation/Foundation/Reference/NSURLSessionTaskDelegate_protocol/index.html#//apple_ref/occ/intfm/NSURLSessionTaskDelegate/URLSession:task:didSendBodyData:totalBytesSent:totalBytesExpectedToSend:)代理方法来获得上传进度的信息。

另外，如果你的应用程序使用流提供请求正文，必须提供一个自定义会话代理，实现[ URLSession:task:needNewBodyStream: ](https://developer.apple.com/library/prerelease/tvos/documentation/Foundation/Reference/NSURLSessionTaskDelegate_protocol/index.html#//apple_ref/occ/intfm/NSURLSessionTaskDelegate/URLSession:task:needNewBodyStream:)方法，更多详细内容，参见[ Uploading Body Content Using a Stream ](https://developer.apple.com/library/prerelease/tvos/documentation/Cocoa/Conceptual/URLLoadingSystem/Articles/UsingNSURLSession.html#//apple_ref/doc/uid/TP40013509-SW23)。

### Uploading Body Content Using an NSData Object
(使用NSData对象上传正文内容)

使用NSData对象上传正文内容，你的应用程序调用[ uploadTaskWithRequest:fromData: ](https://developer.apple.com/library/prerelease/tvos/documentation/Foundation/Reference/NSURLSession_class/index.html#//apple_ref/occ/instm/NSURLSession/uploadTaskWithRequest:fromData:)或者[ uploadTaskWithRequest:fromData:completionHandler: ](https://developer.apple.com/library/prerelease/tvos/documentation/Foundation/Reference/NSURLSession_class/index.html#//apple_ref/occ/instm/NSURLSession/uploadTaskWithRequest:fromData:completionHandler:)方法创建一个上传任务，然后通过fromData参数提供请求的正文数据。

会话对象会基于数据对象的大小来计算Content-Length首部字段。

你的应用程序必须提供服务器要求的额外的首部字段信息，比如内容类型(content type)，作为URL请求对象的一部分。

### Uploading Body Content Using a File
(使用文件上传正文内容)

使用文件上传正文内容，你的应用程序可以调用[ uploadTaskWithRequest:fromFile: ](https://developer.apple.com/library/prerelease/tvos/documentation/Foundation/Reference/NSURLSession_class/index.html#//apple_ref/occ/instm/NSURLSession/uploadTaskWithRequest:fromFile:)和[ uploadTaskWithRequest:fromFile:completionHandler: ](https://developer.apple.com/library/prerelease/tvos/documentation/Foundation/Reference/NSURLSession_class/index.html#//apple_ref/occ/instm/NSURLSession/uploadTaskWithRequest:fromFile:completionHandler:)方法创建上传任务，然后提供一个文件的URL，上传任务会从这个URL读取正文内容。

会话对象根据数据对象计算首部字段Content-Length的值。如果你的应用程序没有为首部字段提供Content-Type的值，会话对象也会提供一个。

你的应用程序应该提供服务器要求的首部字段信息，作为URL请求对象的一部分。

### Uploading Body Content Using a Stream
(使用流上传正文内容)

使用流上传正文内容，调用[ uploadTaskWithStreamedRequest: ](https://developer.apple.com/library/prerelease/tvos/documentation/Foundation/Reference/NSURLSession_class/index.html#//apple_ref/occ/instm/NSURLSession/uploadTaskWithStreamedRequest:)方法创建一个上传任务。你应用程序提供一个请求对象，这个对象与流相关联，上传任务会从流里面读取正文内容。

你的应用程序必须提供服务器要求的首部字段的信息，内容类型和内容长度，作为URL请求对象的一部分。

### Uploading a File Using a Download Task
(使用下载任务(Download Task)上传文件)

为下载任务(download task)上传正文内容，你的应用程序必须提供NSData对象或者流作为URL请求对象的一部分，然后用这个请求创建下载请求。

如果你使用流提供数据，你的应用程序必须提供一个[ URLSession:task:needNewBodyStream: ](https://developer.apple.com/library/prerelease/tvos/documentation/Foundation/Reference/NSURLSessionTaskDelegate_protocol/index.html#//apple_ref/occ/intfm/NSURLSessionTaskDelegate/URLSession:task:needNewBodyStream:)代理方法在认证失败的事件中来提供新的内容流。这个方法的详细信息，参见[ Uploading Body Content Using a Stream ](https://developer.apple.com/library/prerelease/tvos/documentation/Cocoa/Conceptual/URLLoadingSystem/Articles/UsingNSURLSession.html#//apple_ref/doc/uid/TP40013509-SW23)。

下载任务(download task)的行为类似于数据任务(data task)，除了返回数据的方式不一样。

## Handling Authentication and Custom TLS Chain Validation
(处理认证以及自定义TLS链验证)

如果远程服务器返回了一个状态码指示需要认证，如果认证要求一个连接级别的挑战(比如SSL客户端证书)，NSURLSession调用认证的代理方法。

* 对于会话级别的挑战－[ NSURLAuthenticationMethodNTLM ](https://developer.apple.com/library/prerelease/tvos/documentation/Cocoa/Reference/Foundation/Classes/NSURLProtectionSpace_Class/index.html#//apple_ref/c/data/NSURLAuthenticationMethodNTLM)，[ NSURLAuthenticationMethodNegotiate ](https://developer.apple.com/library/prerelease/tvos/documentation/Cocoa/Reference/Foundation/Classes/NSURLProtectionSpace_Class/index.html#//apple_ref/c/data/NSURLAuthenticationMethodNegotiate)，[ NSURLAuthenticationMethodClientCertificate ](https://developer.apple.com/library/prerelease/tvos/documentation/Cocoa/Reference/Foundation/Classes/NSURLProtectionSpace_Class/index.html#//apple_ref/c/data/NSURLAuthenticationMethodClientCertificate)， 或者 [ NSURLAuthenticationMethodServerTrust ](https://developer.apple.com/library/prerelease/tvos/documentation/Cocoa/Reference/Foundation/Classes/NSURLProtectionSpace_Class/index.html#//apple_ref/c/data/NSURLAuthenticationMethodServerTrust)－NSURLSession对象调用会话的[ URLSession:didReceiveChallenge:completionHandler: ](https://developer.apple.com/library/prerelease/tvos/documentation/Foundation/Reference/NSURLSessionDelegate_protocol/index.html#//apple_ref/occ/intfm/NSURLSessionDelegate/URLSession:didReceiveChallenge:completionHandler:)代理方法。如果你的应用程序不提供会话代理方法，NSURLSession对象会调用任务的[ URLSession:task:didReceiveChallenge:completionHandler: ](https://developer.apple.com/library/prerelease/tvos/documentation/Foundation/Reference/NSURLSessionTaskDelegate_protocol/index.html#//apple_ref/occ/intfm/NSURLSessionTaskDelegate/URLSession:task:didReceiveChallenge:completionHandler:)代理方法来处理挑战。
* 对于非会话级别的挑战(其他所有情况)，NSURLSession对象会调用会话代理的[ URLSession:task:didReceiveChallenge:completionHandler: ](https://developer.apple.com/library/prerelease/tvos/documentation/Foundation/Reference/NSURLSessionTaskDelegate_protocol/index.html#//apple_ref/occ/intfm/NSURLSessionTaskDelegate/URLSession:task:didReceiveChallenge:completionHandler:)方法处理挑战。如果你的应用提供会话代理以及你需要处理认证，那么你必须在任务级别处理认证或者提供一个任务级别的处理器处理每个会话处理的显示调用。会话的代理方法[ URLSession:didReceiveChallenge:completionHandler: ](https://developer.apple.com/library/prerelease/tvos/documentation/Foundation/Reference/NSURLSessionDelegate_protocol/index.html#//apple_ref/occ/intfm/NSURLSessionDelegate/URLSession:didReceiveChallenge:completionHandler:)在非会话级别是不会调用的。

**注意**：Kerberos认证的处理是透明的。

当一个基于流的上传任务的认证失败时，任务不能回退且安全的重用流。相反，NSURLSession对象会调用代理的[ URLSession:task:needNewBodyStream: ](https://developer.apple.com/library/prerelease/tvos/documentation/Foundation/Reference/NSURLSessionTaskDelegate_protocol/index.html#//apple_ref/occ/intfm/NSURLSessionTaskDelegate/URLSession:task:needNewBodyStream:)方法去获得一个新的NSInputStream对象，给新的请求提供数据。(如果上传任务是基于文件或者NSData对象，会话对象就不会调用这个方法。)

关于NSURLSession代理中的认证的方法的更多信息，参见[ Authentication Challenges and TLS Chain Validation ](https://developer.apple.com/library/prerelease/tvos/documentation/Cocoa/Conceptual/URLLoadingSystem/Articles/AuthenticationChallenges.html#//apple_ref/doc/uid/TP40009507-SW1)。

## Handling iOS Background Activity
(处理iOS后台活动)

如果你在iOS中使用NSURLSession，当下载完成时应用程序会自动重启。你应用程序的app delegate的[ application:handleEventsForBackgroundURLSession:completionHandler: ](https://developer.apple.com/library/prerelease/tvos/documentation/UIKit/Reference/UIApplicationDelegate_Protocol/index.html#//apple_ref/occ/intfm/UIApplicationDelegate/application:handleEventsForBackgroundURLSession:completionHandler:)方法会负责重新创建合适的会话，存储完成处理器，当会话调用会话代理的[ URLSessionDidFinishEventsForBackgroundURLSession: ](https://developer.apple.com/library/prerelease/tvos/documentation/Foundation/Reference/NSURLSessionDelegate_protocol/index.html#//apple_ref/occ/intfm/NSURLSessionDelegate/URLSessionDidFinishEventsForBackgroundURLSession:)方法时会调用那个处理器。

Listing 1-8 和 Listing 1-9分开展示会话及app delegate的方法。

Listing 1-8  Session delegate methods for iOS background downloads

    #if TARGET_OS_IPHONE
    -(void)URLSessionDidFinishEventsForBackgroundURLSession:(NSURLSession *)session
    {
        NSLog(@"Background URL session %@ finished events.\n", session);
        
        if (session.configuration.identifier)
            [self callCompletionHandlerForSession: session.configuration.identifier];
    }

    - (void) addCompletionHandler: (CompletionHandlerType) handler forSession: (NSString *)identifier
    {
        if ([ self.completionHandlerDictionary objectForKey: identifier]) {
            NSLog(@"Error: Got multiple handlers for a single session identifier.  This should not happen.\n");
        }
        
        [ self.completionHandlerDictionary setObject:handler forKey: identifier];
    }

    - (void) callCompletionHandlerForSession: (NSString *)identifier
    {
        CompletionHandlerType handler = [self.completionHandlerDictionary objectForKey: identifier];
        
        if (handler) {
            [self.completionHandlerDictionary removeObjectForKey: identifier];
            NSLog(@"Calling completion handler.\n");
            
            handler();
        }
    }
    #endif

Listing 1-9  App delegate methods for iOS background downloads

    - (void)application:(UIApplication *)application handleEventsForBackgroundURLSession:(NSString *)identifier completionHandler:(void (^)())completionHandler
    {
        NSURLSessionConfiguration *backgroundConfigObject = [NSURLSessionConfiguration backgroundSessionConfiguration: identifier];
        
        NSURLSession *backgroundSession = [NSURLSession sessionWithConfiguration: backgroundConfigObject delegate: self.mySessionDelegate delegateQueue: [NSOperationQueue mainQueue]];
        
        NSLog(@"Rejoining session %@\n", identifier);
        
        [ self.mySessionDelegate addCompletionHandler: completionHandler forSession: identifier];
    }
