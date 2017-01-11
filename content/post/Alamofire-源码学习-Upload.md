---
title: "Alamofire 源码学习 - Upload"
date: 2016-03-19 13:11:25
categories: 
- 源码学习
tags: 
- Alamofire
metaAlignment: center
autoThumbnailImage: no
coverImage: //d1u9biwaxjngwg.cloudfront.net/welcome-to-tranquilpeak/city.jpg

---

创建这篇文章的时候是3月19号，今天已经是4月7号了。感觉时间过的好快啊，但是博客一直没更新，顿时感觉堕落了，没有成长。废话不多说了，直接进入正题。
<!--more-->

其实Upload跟前面讲的Request模式基本是一样的，所以这篇文章应该就讲个大概，尽量简单些，很细分的东西自行了解学习了。

Alamofire有更新过代码，我当前的版本是3.3.0。上篇的版本是多少，忘记了，所以代码可能会有不一样的地方。

# Alamofire.swift

这个文件里面包含了很多的便捷方法，上篇有截图展示过，所以我们还是从这里开始。

这个文件里关于上传的方法有八个，但是仔细看下，作者分为了4类：

* File
* Data
* Stream
* MultipartFormData

每一类都有URLString和URLRequest两种，所以我们只需要看一种就行了。

这里我们主要把前面三种当做一类，最后面一种当做一类来，所以主要就简单说明下这两种情况。因为前三种NSURLSession的API都提供了支持(方法源码如下所示)，只需要讲解一种就可以了。最后一种是作者自己实现的，但是后面也是借用了NSURLSession的API。

    /* Creates an upload task with the given request.  The body of the request will be created from the file referenced by fileURL */
    public func uploadTaskWithRequest(request: NSURLRequest, fromFile fileURL: NSURL) -> NSURLSessionUploadTask
    
    /* Creates an upload task with the given request.  The body of the request is provided from the bodyData. */
    public func uploadTaskWithRequest(request: NSURLRequest, fromData bodyData: NSData) -> NSURLSessionUploadTask
    
    /* Creates an upload task with the given request.  The previously set body stream of the request (if any) is ignored and the URLSession:task:needNewBodyStream: delegate will be called when the body payload is required. */
    public func uploadTaskWithStreamedRequest(request: NSURLRequest) -> NSURLSessionUploadTask

仔细看下这些API的方法名称，正好对应前面三种，方法的注释都说明了一些相关的信息，自行了解。[ NSURLSession编程指南 ](https://developer.apple.com/library/tvos/documentation/Cocoa/Conceptual/URLLoadingSystem/Articles/UsingNSURLSession.html#//apple_ref/doc/uid/TP40013509-SW15)，[ 中文翻译 ](http://lynchwong.com/2016/01/29/URL-Session-Programming-Guide-Using-NSURLSession/)(翻译的不好见谅) 对这三种类型的上传也有说明：

>Your app can provide the request body content for an HTTP POST request in three ways: as an NSData object, as a file, or as a stream. In general, your app should:

>* Use an NSData object if your app already has the data in memory and has no reason to dispose of it.
>* Use a file if the content you are uploading exists as a file on disk, if you are doing background transfer, or if it is to your app’s benefit to write it to disk so that it can release the memory associated with that data.
>* Use a stream if you are receiving the data over a network or are converting existing NSURLConnection code that provides the request body as a stream.

>Regardless of which style you choose, if your app provides a custom session delegate, that delegate should implement the URLSession:task:didSendBodyData:totalBytesSent:totalBytesExpectedToSend: delegate method to obtain upload progress information.

>Additionally, if your app provides the request body using a stream, it must provide a custom session delegate that implements the URLSession:task:needNewBodyStream: method, described in more detail in Uploading Body Content Using a Stream.

所以这里就不再赘述这三种方法有什么区别，何时该使用哪种方法。

**MultipartFormData**其实到最后也是借助了**File**和**Data**来实现的，接下来就看源码。

# Upload.swift

上传相关的信息都在这个文件里，包括创建上传的请求和上传需要的代理方法。这个文件并没有创建新的类，就是对**Manager**类和**Request**类的拓展。

我们以**File**为例，找到入口方方法，源码如下：

    public func upload(
        method: Method,
        _ URLString: URLStringConvertible,
        headers: [String: String]? = nil,
        file: NSURL)
        -> Request
    {
        return Manager.sharedInstance.upload(method, URLString, headers: headers, file: file)
    }

这是Alamofire.swift文件里的入口方法，点击跳转到Upload.swift文件里，方法如下：

    public func upload(
        method: Method,
        _ URLString: URLStringConvertible,
        headers: [String: String]? = nil,
        file: NSURL)
        -> Request
    {
        let mutableURLRequest = URLRequest(method, URLString, headers: headers)
        return upload(mutableURLRequest, file: file)
    }

继续点击跳转到的方法如下：

    public func upload(URLRequest: URLRequestConvertible, file: NSURL) -> Request {
        return upload(.File(URLRequest.URLRequest, file))
    }

最后调用的方法：

    private func upload(uploadable: Uploadable) -> Request {
        var uploadTask: NSURLSessionUploadTask!
        var HTTPBodyStream: NSInputStream?

        switch uploadable {
        case .Data(let request, let data):
            dispatch_sync(queue) {
                uploadTask = self.session.uploadTaskWithRequest(request, fromData: data)
            }
        case .File(let request, let fileURL):
            dispatch_sync(queue) {
                uploadTask = self.session.uploadTaskWithRequest(request, fromFile: fileURL)
            }
        case .Stream(let request, let stream):
            dispatch_sync(queue) {
                uploadTask = self.session.uploadTaskWithStreamedRequest(request)
            }

            HTTPBodyStream = stream
        }

        let request = Request(session: session, task: uploadTask)

        if HTTPBodyStream != nil {
            request.delegate.taskNeedNewBodyStream = { _, _ in
                return HTTPBodyStream
            }
        }

        delegate[request.delegate.task] = request.delegate

        if startRequestsImmediately {
            request.resume()
        }

        return request
    }

如上的方法是一个工厂方法，其它上传类型**Data**和**Stream**最终也是从这个方法创建的。

作者定义了具有关联值的枚举来标示这三种类型的上传：

    private enum Uploadable {
        case Data(NSURLRequest, NSData)
        case File(NSURLRequest, NSURL)
        case Stream(NSURLRequest, NSInputStream)
    }

然后在这个工厂方法里面通过枚举创建了不同的上传请求(都调用的是对应的NSURLSession的API)，最后返回了创建的Request实例，跟我们上篇讲解的Request模式是一样的。

然后拓展了Request类，新添加了UploadTaskDelegate类：

    class UploadTaskDelegate: DataTaskDelegate {
        var uploadTask: NSURLSessionUploadTask? { return task as? NSURLSessionUploadTask }
        var uploadProgress: ((Int64, Int64, Int64) -> Void)!

        // MARK: - NSURLSessionTaskDelegate

        // MARK: Override Closures

        var taskDidSendBodyData: ((NSURLSession, NSURLSessionTask, Int64, Int64, Int64) -> Void)?

        // MARK: Delegate Methods

        func URLSession(
            session: NSURLSession,
            task: NSURLSessionTask,
            didSendBodyData bytesSent: Int64,
            totalBytesSent: Int64,
            totalBytesExpectedToSend: Int64)
        {
            if initialResponseTime == nil { initialResponseTime = CFAbsoluteTimeGetCurrent() }

            if let taskDidSendBodyData = taskDidSendBodyData {
                taskDidSendBodyData(session, task, bytesSent, totalBytesSent, totalBytesExpectedToSend)
            } else {
                progress.totalUnitCount = totalBytesExpectedToSend
                progress.completedUnitCount = totalBytesSent

                uploadProgress?(bytesSent, totalBytesSent, totalBytesExpectedToSend)
            }
        }
    }

这个代理类以及实现的代理方法是为了获得上传的进度，这些知识点前面引用的部分有提及，不再赘述。以及这里代理调用的模式与上篇讲过的DataTaskDelegate和TaskDelegate是一样的，故这里也不再赘述了。基本上前面三种就这么些东西，只要明白了上篇Request的模式，上传和下载基本也就明白了。

## MultipartFormData

主要讲讲这个，使用方法如下所示：

    Alamofire.upload(
        .POST,
        "https://httpbin.org/post",
        multipartFormData: { multipartFormData in
            multipartFormData.appendBodyPart(fileURL: unicornImageURL, name: "unicorn")
            multipartFormData.appendBodyPart(fileURL: rainbowImageURL, name: "rainbow")
        },
        encodingCompletion: { encodingResult in
            switch encodingResult {
                case .Success(let upload, _, _):
                    upload.responseJSON { response in
                        debugPrint(response)
                    }
                case .Failure(let encodingError):
                    print(encodingError)
            }
        }
    )

根据这个方法我们找到入口方法，然后找到最后处理的方法，源码如下：

    /**
        Encodes the `MultipartFormData` and creates a request to upload the result to the specified URL request.

        It is important to understand the memory implications of uploading `MultipartFormData`. If the cummulative
        payload is small, encoding the data in-memory and directly uploading to a server is the by far the most
        efficient approach. However, if the payload is too large, encoding the data in-memory could cause your app to
        be terminated. Larger payloads must first be written to disk using input and output streams to keep the memory
        footprint low, then the data can be uploaded as a stream from the resulting file. Streaming from disk MUST be
        used for larger payloads such as video content.

        The `encodingMemoryThreshold` parameter allows Alamofire to automatically determine whether to encode in-memory
        or stream from disk. If the content length of the `MultipartFormData` is below the `encodingMemoryThreshold`,
        encoding takes place in-memory. If the content length exceeds the threshold, the data is streamed to disk
        during the encoding process. Then the result is uploaded as data or as a stream depending on which encoding
        technique was used.

        If `startRequestsImmediately` is `true`, the request will have `resume()` called before being returned.

        - parameter URLRequest:              The URL request.
        - parameter multipartFormData:       The closure used to append body parts to the `MultipartFormData`.
        - parameter encodingMemoryThreshold: The encoding memory threshold in bytes.
                                             `MultipartFormDataEncodingMemoryThreshold` by default.
        - parameter encodingCompletion:      The closure called when the `MultipartFormData` encoding is complete.
    */
    public func upload(
        URLRequest: URLRequestConvertible,
        multipartFormData: MultipartFormData -> Void,
        encodingMemoryThreshold: UInt64 = Manager.MultipartFormDataEncodingMemoryThreshold,
        encodingCompletion: (MultipartFormDataEncodingResult -> Void)?)
    {
        dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0)) {
            let formData = MultipartFormData()
            multipartFormData(formData)

            let URLRequestWithContentType = URLRequest.URLRequest
            URLRequestWithContentType.setValue(formData.contentType, forHTTPHeaderField: "Content-Type")

            let isBackgroundSession = self.session.configuration.identifier != nil

            if formData.contentLength < encodingMemoryThreshold && !isBackgroundSession {
                do {
                    let data = try formData.encode()
                    let encodingResult = MultipartFormDataEncodingResult.Success(
                        request: self.upload(URLRequestWithContentType, data: data),
                        streamingFromDisk: false,
                        streamFileURL: nil
                    )

                    dispatch_async(dispatch_get_main_queue()) {
                        encodingCompletion?(encodingResult)
                    }
                } catch {
                    dispatch_async(dispatch_get_main_queue()) {
                        encodingCompletion?(.Failure(error as NSError))
                    }
                }
            } else {
                let fileManager = NSFileManager.defaultManager()
                let tempDirectoryURL = NSURL(fileURLWithPath: NSTemporaryDirectory())
                let directoryURL = tempDirectoryURL.URLByAppendingPathComponent("com.alamofire.manager/multipart.form.data")
                let fileName = NSUUID().UUIDString
                let fileURL = directoryURL.URLByAppendingPathComponent(fileName)

                do {
                    try fileManager.createDirectoryAtURL(directoryURL, withIntermediateDirectories: true, attributes: nil)
                    try formData.writeEncodedDataToDisk(fileURL)

                    dispatch_async(dispatch_get_main_queue()) {
                        let encodingResult = MultipartFormDataEncodingResult.Success(
                            request: self.upload(URLRequestWithContentType, file: fileURL),
                            streamingFromDisk: true,
                            streamFileURL: fileURL
                        )
                        encodingCompletion?(encodingResult)
                    }
                } catch {
                    dispatch_async(dispatch_get_main_queue()) {
                        encodingCompletion?(.Failure(error as NSError))
                    }
                }
            }
        }
    }

这个方法在Upload.swift文件中，我们一点一点来看这个方法，首先看作者的注释，大概意思：

使用`MultipartFormData`方式进行上传时需要明白的最重要的一点就是内存的占用。如果上传的数据很小，那么直接将数据编码在内存中上传到服务器是最高效的方式。然而，如果上传的数据很大，将数据编码在内存中可能导致你的应用程序被终结。所以应该将这些大的要上传的数据使用输入或输出流写到硬盘上，这样会使内存占用比较低，然后将写入的文件转换成流进行上传。encodingMemoryThreshold允许Alamofire自动检测是将数据编码到内存还是写入硬盘。如果MultipartFormData的内容长度比encodingMemoryThreshold小，就会编码到内存中。如果大，就会在编码的过程中使用流写入硬盘。然后根据使用的编码方式来决定是使用Data还是Stream来进行上传。

通过注释基本上也明白了参数的作用，encodingMemoryThreshold参数，默认值定义如下，大小应该是10M：

    /// Default memory threshold used when encoding `MultipartFormData`.
    public static let MultipartFormDataEncodingMemoryThreshold: UInt64 = 10 * 1024 * 1024

encodingCompletion参数的类型类似于Result，只是没有泛型。MultipartFormData类型现在先不管，只需要知道它的作用就是对数据进行编码的就行了，我们先看这个方法。

首先是GCD的异步操作，队列是全局并发队列中的DISPATCH_QUEUE_PRIORITY_DEFAULT，这里多线程没什么好讲的。首先创建了一个MultipartFormData实例，然后作为了multipartFormData闭包的参数，结合最开始我们提到的使用方式，这里的作用就是将数据添加到这个实例中，以便于我们之后再编码。然后设置了HTTP请求的首部字段**Content-Type**，值由前面创建的MultipartFormData实例提供，我们后面再讲解。然后根据NSURLSession实例的identifier属性来判断是否是后台回话，属性源码如下：

    /* identifier for the background session configuration */
    public var identifier: String? { get }

然后根据要上传的数据的大小和encodingMemoryThreshold的比较，以及是否是后台回话进行条件判断，if else 二选一。进入if的条件就是要上传的数据比encodingMemoryThreshold小而且不是后台会话，然后我们直接调用MultipartFormData实例的encode()方法将数据编码到了内存中(因为数据比较小，跟encodingMemoryThreshold参数比较，所以直接编码到内存中，前面方法注释有提过)，然后创建一个MultipartFormDataEncodingResult实例encodingResult，在构造的方法中可以看出是使用Data的上传方式。然后返回主线程调用回调，传入前面创建的encodingResult当做参数。如果编码数据的时候直接产生错误就返回主线程调用回调，然后将错误传回去。

然后是else分支，首先是构建目录、创建路径。首先在临时目录下创建一个"com.alamofire.manager/multipart.form.data"目录，然后使用这个目录和NSUUID().UUIDString生成的文件名生成了文件路径。然后在创建目录，将数据写入文件路径，这些都有错误处理。使用临时目录的好处就是iOS系统会自动清理临时目录。然后就是跟if里面类似的，创建MultipartFormDataEncodingResult实例encodingResult，构造方中可以看出是使用了File的上传方式，其它的都是一样的不再赘述。把是否是后台会话加入判断是因为基于后台会话的上传方式只支持基于文件的上传，基于Data和Stream的上传任务在程序退出时就会失败，这点在编程指南里面也有提及。关于后台传输(包括上传和下载)的一些信息，参考[ Background Transfer Considerations ](https://developer.apple.com/library/tvos/documentation/Cocoa/Conceptual/URLLoadingSystem/Articles/UsingNSURLSession.html#//apple_ref/doc/uid/TP40013509-SW44)。

## MultipartFormData.swift

MultipartFormData上传方式的主要核心就是这个文件里面的内容了，前面那个方法只是一些逻辑的处理。其实这个文件里面的内容也没什么难度，当然是代码上没什么难度，难的地方可能就是你不懂MultipartFormData的相关概念和定义，所以可能看起代码来不知道到底是在干什么。

所以首先应该搞清楚MultipartFormData相关的概念、定义、知识等。这里就直接贴两个我之前研究学习的链接，就不详细讲解了。

* [ 模拟HTML表单上传文件（RFC 1867） ](http://blog.zhaojie.me/2011/03/html-form-file-uploading-programming.html)
* [ Android文件图片上传的详细讲解（一）HTTP multipart/form-data 上传报文格式实现手机端上传GOOD ](http://blog.csdn.net/baple/article/details/45890405)

第二篇虽然是Android的，但是没关系，只需了解MultipartFormData的东西就可以了。

理解了MultipartFormData相关的知识和概念后再看这个文件里面的代码就很简单了。

之前有说过我山寨了Alamofire，但有些地方也做了些小修改。比如我写的MultipartFormData方式上传的方法支持参数，这里并不是Alamofire不支持，其实Alamofire是支持的，我这里做了些小修改只是在调用上更清晰而已。(经测试是可以使用的)

源码如下：

    public class func upload(
        request: NSURLRequest,
        parameters: [String: AnyObject]? = nil,
        encodingMemoryThreshold: UInt64 = Manager.MultipartFormDataEncodingMemoryThreshold,
        multipartFormData: MultipartFormData -> Void)
    {
        Manager.sharedInstance.upload(
            request,
            parameters: parameters,
            encodingMemoryThreshold: encodingMemoryThreshold,
            multipartFormData: multipartFormData
        )
    }

然后在MultipartFormData.swift的基础上也做了些小修改，[ 源码链接 ](https://github.com/LynchWong/Speedy/blob/master/Speedy/Speedy/MultipartFormData.swift)。

其实我们只需要将参数当做数据编码到HTTP请求Body中就可以了，[ 模拟HTML表单上传文件（RFC 1867） ](http://blog.zhaojie.me/2011/03/html-form-file-uploading-programming.html)中的例子就是这种情况。

假设我们上传的时候需要携带`userName`这个参数，那么使用Alamofire的代码如下：

        Alamofire.upload(
            .POST,
            "",
            multipartFormData: { multipartFormData in
                multipartFormData.appendBodyPart(data: "Nobodyknows+".dataUsingEncoding(NSUTF8StringEncoding)!, name: "userName")
                multipartFormData.appendBodyPart(fileURL: unicornImageURL, name: "unicorn")
                multipartFormData.appendBodyPart(fileURL: rainbowImageURL, name: "rainbow")
            }) { encodingResult in
                switch encodingResult {
                    case .Success(let upload, _, _):
                        upload.responseJSON { response in
                            debugPrint(response)
                        }
                    case .Failure(let encodingError):
                        print(encodingError)
                }
        }

理论上这样是可以的，实际没有测试过。

以上。
