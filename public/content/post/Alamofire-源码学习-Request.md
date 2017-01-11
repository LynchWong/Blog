---
title: "Alamofire 源码学习 - Request"
date: 2016-03-19 13:11:15
categories: 
- 源码学习
tags: 
- Alamofire
metaAlignment: center
autoThumbnailImage: no
coverImage: //d1u9biwaxjngwg.cloudfront.net/welcome-to-tranquilpeak/city.jpg

---

本来想先把框架的类图画出来，便于大家好理解类、枚举、结构体之间的关系。无奈UML全忘光了，好多画UML图的软件都是付费的，所以就打消了这个念头。
<!--more-->

既然这样就从头开始，就像是第一次接触这个框架。

所以最好的方式就是按照我们使用框架的代码来学习，从下面这个用例开始：

	Alamofire.request(.GET, "https://httpbin.org/get", parameters: ["foo": "bar"])
	    .responseJSON { response in
	        print(response.request)  // original URL request
	        print(response.response) // URL response
	        print(response.data)     // server data
	        print(response.result)   // result of response serialization
	
	        if let JSON = response.result.value {
	            print("JSON: \(JSON)")
	        }
	    }

上篇这段代码就是发起一个请求然后进行JSON解析的代码，我们就从这里开始。

# Alamofire.swift

如果你有链接框架在Xcode中编写上面代码，使用Option点击request方法能看到方法的一些信息，使用Command点击request会跳转到源码文件或者接口文件。

我们Command点击request方法，跳转到了Alamofire.swift文件里面，方法源码如下：

	public func request(
	    method: Method,
	    _ URLString: URLStringConvertible,
	    parameters: [String: AnyObject]? = nil,
	    encoding: ParameterEncoding = .URL,
	    headers: [String: String]? = nil)
	    -> Request
	{
	    return Manager.sharedInstance.request(
	        method,
	        URLString,
	        parameters: parameters,
	        encoding: encoding,
	        headers: headers
	    )
	}

现在来看这个方法，看方法参数列表和返回值。参数列表主要有用于请求的HTTP方法，请求的URL，请求的参数，请求的编码方式，请求的首部信息，然后返回值是一个自定义的**Request**类。方法的第二个参数忽略了外部参数，所以你调用的时候不会有外部参数名；后面三个参数都有默认值，所以你调用方法的时候可以省略，这些都是Swift的知识，这里再不详解了。

`method`参数的类型是**Method**，我们还是点击过去看源码。源码在ParameterEncoding.swift文件中，**Method**类型是枚举类型的，rawValue类型为String。这里访问权限有必要提下，特别是在写框架的时候要把访问权限弄清楚，public，internal，private针对到每一个类、枚举、结构体、变量、属性、方法、函数。这里就提下，不详解了，参考[ Swift之访问控制 ](http://lynchwong.com/2015/03/07/Swift%E4%B9%8B%E8%AE%BF%E9%97%AE%E6%8E%A7%E5%88%B6/)。**Method**的定义很简单，定义了基本所有的HTTP方法，不再赘述。

`URLString`参数的类型是**URLStringConvertible**。**URLStringConvertible**类型是协议，该协议只有一个只读的URLString属性，类型为String。然后String、NSURL、NSURLComponents、NSURLRequest都适配了这个协议，所以这些类型都有了一个只读的URLString属性。这样的好处就是你可以把所有适配了这个协议的类型当作URLString的参数，然后我们编码的时候只需要读取URLString属性就可以得到URL字符串。**URLRequestConvertible**协议是类似的，后面我们遇到了就不再赘述了。

`parameters`参数是可选字典类型，键为String类型，值为AnyObject类型，默认值为nil。

`encoding`参数类型是**ParameterEncoding**，看参数默认值的形式应该是枚举。参数编码的方式，这里先不讲解，后面来。

`headers`参数类型是可选的字典类型，键值都是String类型，默认值为nil。这个就是指定HTTP首部字段的值。

返回值我们现在只需要知道是返回的一个自定义的**Request**类，我们再来看看整个Alamofire.swift文件。

整个文件结构很清晰，没有定义类。首先定义了两个协议，然后就是request、upload、download的方法，其实这些方法都是一些便捷方法，方便开发者直接使用。如下图所示，结构就很清晰了：

![alt text](/img/AlamofireRequest/1.png)

# ParameterEncoding.swift

这个文件除了之前提到的**Method**类型外就是**ParameterEncoding**类型了，两个都是枚举类型。

**ParameterEncoding**类型定义了五种值，源码上面有作者的注释：

1. `URL`：对于`GET`、`HEAD`、`DELETE`方法，编码后的参数拼接在URL后面，其他方法编码后的参数作为表单提交，设为HTTP的Body。如果编码后的参数作为HTTP的Body，那么HTTP首部字段`Content-Type`设为`application/x-www-form-urlencoded; charset=utf-8`。因为没有一个公开的标准来编码集合类型，所以我们约定键后面跟`[]`来编码数组值(`foo[]=1&foo[]=2`)。`foo[bar]=baz`编码字典值。
2. `URLEncodedInURL`：编码方式与上面一样，但是编码的结果总是拼接在URL后面。
3. `JSON`：使用`NSJSONSerialization`来序列化参数对象，设置为HTTP请求的Body。`Content-Type`的值设为`application/json`。
4. `PropertyList`：关联值为NSPropertyListFormat和NSPropertyListWriteOptions，根据这些信息来创建plist对象。编码后的值设为HTTP的请求Body，`Content-Type`的值设为`application/json`。
5. `Custom`：关联值是一个闭包，闭包参数是URLRequestConvertible类型和[String: AnyObject]?可选类型，返回值为(NSMutableURLRequest, NSError?)的元组类型。该闭包的参数类型和返回值类型跟我们编码的方法的参数和返回值类型是一样的。

编码的方法的具体实现的细节就不详细讲解了，主要是HTTP协议相关的一些事情，不懂就搜索下，是能看懂的。这个文件中的代码，是可以高度重用的，完全可以学习保留。

# Manager.swift

方法的方法体中的代码如下所示：

	return Manager.sharedInstance.request(
	    method,
	    URLString,
	    parameters: parameters,
	    encoding: encoding,
	    headers: headers
	)

所以Manager类的实例的request方法的返回值类型就是**Request**。

sharedInstance的实现代码如下：

	public static let sharedInstance: Manager = {
	    let configuration = NSURLSessionConfiguration.defaultSessionConfiguration()
	    configuration.HTTPAdditionalHeaders = Manager.defaultHTTPHeaders
	
	    return Manager(configuration: configuration)
	}()

虽然返回的都是Manager对象的实例，但是并不是单列，每次都是新建的一个实例。看上面的代码，首先创建了用于创建NSURLSession的配置对象，然后设置了默认的HTTP首部字段。最后将配置对象传递给Manager类的初始化方法，最后返回Manager类的实例。如果你之前有看过关于NSURLSession的编程指南，你肯定还记得NSURLSessionConfiguration对象，比如有default、ephemeral、background三种类型的配置对象。另外还有NSURLSessionTask、NSURLSessionDataTask、NSURLSessionUploadTask、NSURLSessionDownloadTask，以及这些类之间的关系。这些你应该了解才行，看起源码来才更容易理解作者的意图。这一方面的知识，这里不再赘述，只会提及一下。

defaultHTTPHeaders属性这里也不再说明了，与HTTP协议相关的内容，自行了解下。接下来我们就来看Manager的初始化方法。

初始化方法都跟属性是相关的，我们先看属性：

	let queue = dispatch_queue_create(nil, DISPATCH_QUEUE_SERIAL)

定义了一个常量属性，并初始化赋值，创建了一个串行队列。

	public let session: NSURLSession
	public let delegate: SessionDelegate
	public var startRequestsImmediately: Bool = true

定义了NSURLSession和它的代理，SessionDelegate是自定义的代理对象，这里我们先不具体展开。startRequestsImmediately属性标识是否立即开始请求。

	public var backgroundCompletionHandler: (() -> Void)?

这个属性作者有注释说明。是后台处理完成的回调闭包，由UIApplicationDelegate的`application:handleEventsForBackgroundURLSession:completionHandler:`方法提供。通过设置后台处理完成的回调，SessionDelegate的`sessionDidFinishEventsForBackgroundURLSession`闭包实现会自动调用回调。如果你需要在回调之前处理自己的事件，你需要重写SessionDelegate的`sessionDidFinishEventsForBackgroundURLSession`闭包，然后在结束的时候手动调用回调。

接下来看初始化方法：

	public init(
	    configuration: NSURLSessionConfiguration = NSURLSessionConfiguration.defaultSessionConfiguration(),
	    delegate: SessionDelegate = SessionDelegate(),
	    serverTrustPolicyManager: ServerTrustPolicyManager? = nil)
	{
	    self.delegate = delegate
	    self.session = NSURLSession(configuration: configuration, delegate: delegate, delegateQueue: nil)
	
	    commonInit(serverTrustPolicyManager: serverTrustPolicyManager)
	}
	
	public init?(
	    session: NSURLSession,
	    delegate: SessionDelegate,
	    serverTrustPolicyManager: ServerTrustPolicyManager? = nil)
	{
	    self.delegate = delegate
	    self.session = session
	
	    guard delegate === session.delegate else { return nil }
	
	    commonInit(serverTrustPolicyManager: serverTrustPolicyManager)
	}

Manager是基类，不需要调用父类的指定初始化器，属性赋值完成之后就可以使用self了，以及方法调用。第一个初始化方法很简单，参数都有默认值。第二个初始化方法是直接使用NSURLSession来初始化的，是一个可失败的初始化器，当NSURLSession的代理不是传入的delegate代理时就初始化失败。这里使用了===符来判断对象是否是同一个，两个初始化方法都调用了如下的私有方法：

	private func commonInit(serverTrustPolicyManager serverTrustPolicyManager: ServerTrustPolicyManager?) {
	    session.serverTrustPolicyManager = serverTrustPolicyManager
	
	    delegate.sessionDidFinishEventsForBackgroundURLSession = { [weak self] session in
	        guard let strongSelf = self else { return }
	        dispatch_async(dispatch_get_main_queue()) { strongSelf.backgroundCompletionHandler?() }
	    }
	}

这行代码`session.serverTrustPolicyManager = serverTrustPolicyManager`我们先放着，后面再说。方法的最后是给delegate的`sessionDidFinishEventsForBackgroundURLSession`赋值，如作者注释的那样，闭包会自动调用`backgroundCompletionHandler`闭包，所以如果你重写delegate的`sessionDidFinishEventsForBackgroundURLSession`闭包，那么在结束的时候要手动调用`backgroundCompletionHandler`闭包。为了避免强引用环，作者这里定义了捕获列表。在闭包中做了guard判断，确保弱引用不是nil。最后在主线程上可选调用了闭包，当闭包为nil时什么都不做。

	deinit {
	    session.invalidateAndCancel()
	}

析构函数这里就不多说了。

在sharedInstance之后就调用请求方法，所以接下来我们就看请求的方法：

	public func request(
	    method: Method,
	    _ URLString: URLStringConvertible,
	    parameters: [String: AnyObject]? = nil,
	    encoding: ParameterEncoding = .URL,
	    headers: [String: String]? = nil)
	    -> Request
	{
	    let mutableURLRequest = URLRequest(method, URLString, headers: headers)
	    let encodedURLRequest = encoding.encode(mutableURLRequest, parameters: parameters).0
	    return request(encodedURLRequest)
	}
	
	public func request(URLRequest: URLRequestConvertible) -> Request {
	    var dataTask: NSURLSessionDataTask!
	    dispatch_sync(queue) { dataTask = self.session.dataTaskWithRequest(URLRequest.URLRequest) }
	
	    let request = Request(session: session, task: dataTask)
	    delegate[request.delegate.task] = request.delegate
	
	    if startRequestsImmediately {
	        request.resume()
	    }
	
	    return request
	}

请求的方法如上所示，第一个方法最后调用了第二个方法，所以我们主要看第二个方法。第一个方法很简单，根据参数构造了请求，然后对参数进行编码，最后将构造的请求传递给第二个方法并返回。

现在我们主要看第二个方法，一开始就声明了一个变量，类型为隐式解析可选的NSURLSessionDataTask。然后同步执行我们提交到queue队列上的Block对象，这行代码会在调用request方法的线程上执行，并且会阻塞这个线程，直到操作完成。这行代码保证了dataTask变量在后面使用的时候是赋值完成了的，赋值的过程是调用NSURLSession的API完成的，这里不赘述了。然后我们使用session属性和dataTask变量来初始化**Request**对象，创建的**Request**的实例就是我们函数的返回值。接下来的这行代码也很简单，使用了Swift的下标脚本，参考[ Swift之下标脚本 ](http://lynchwong.com/2015/03/05/Swift%E4%B9%8B%E4%B8%8B%E6%A0%87%E8%84%9A%E6%9C%AC/)。这里下标脚本接收NSURLSessionTask类型的值，可以设置和返回request.delegate类型的值。request.delegate我们这里先理解为：每个**Request**对象也都有自己的代理，这个代理我们后面再说。然后我们根据startRequestsImmediately属性的值，判断是否马上开始请求，最后返回我们创建的**Request**的实例。

其实到这里，默认情况下我们的请求已经开始了。这里我们发现Alamofire.swift文件完全可以不需要的，我们可以自己创建Manager的实例，然后发起请求。Alamofire.swift文件里的方法只是方便了开发者而已。

我们最开始的请求的那个方法只剩最后一步了，因为默认情况下我们的请求已经开始了，所以接下来的事情就是接收服务器的响应，然后解析数据等等一些操作了。我们看最后一步是`responseJSON`，这个操作，不管是方法还是闭包属性，都是属于**Request**实例的。但是我们现在先不急着去看**Request**的源码，因为还有SessionDelegate的源码，接下来我们先看SessionDelegate的源码。

## SessionDelegate

SessionDelegate定义在Manager内部，是内部类，所以在Manager以外的地方使用需要加上前缀：｀Manager.SessionDelegate｀。使用final修饰，说明不能被继承。

	    private var subdelegates: [Int: Request.TaskDelegate] = [:]
	    private let subdelegateQueue = dispatch_queue_create(nil, DISPATCH_QUEUE_CONCURRENT)
	
	    subscript(task: NSURLSessionTask) -> Request.TaskDelegate? {
	        get {
	            var subdelegate: Request.TaskDelegate?
	            dispatch_sync(subdelegateQueue) { subdelegate = self.subdelegates[task.taskIdentifier] }
	
	            return subdelegate
	        }
	
	        set {
	            dispatch_barrier_async(subdelegateQueue) { self.subdelegates[task.taskIdentifier] = newValue }
	        }
	    }

首先定义了两个私有属性，然后就是前面提到的下标脚本的源码。第一个属性subdelegates是字典类型，键为Int类型，值为Request.TaskDelegate类型，然后初始化赋值为空的字典。第二个属性subdelegateQueue是一个并发队列，接下来就来看下标脚本的实现。

下标脚本接收NSURLSessionTask的参数，返回的是Request.TaskDelegate?类型。获取的时候会阻塞当前线程，但是队列是并发的，根据参数的taskIdentifier作为键来获取subdelegates字典中对应的值。当使用下标脚本赋值的时候dispatch\_barrier\_async会等待之前的操作完成，然后再执行，执行完成之后再执行后面的操作。

	public final class SessionDelegate: NSObject, NSURLSessionDelegate, NSURLSessionTaskDelegate, NSURLSessionDataDelegate, NSURLSessionDownloadDelegate {
	    ...
	}
 
SessionDelegate继承了NSObject，主要是为了适配NSObjectProtocol协议。然后就是适配了一连串的代理协议：NSURLSessionDelegate、NSURLSessionTaskDelegate、NSURLSessionDataDelegate、NSURLSessionDownloadDelegate。

	public func dataTaskWithRequest(request: NSURLRequest, completionHandler: (NSData?, NSURLResponse?, NSError?) -> Void) -> NSURLSessionDataTask

我们可以使用上面的方法来创建NSURLSessionDataTask，提供一个完成后的回调闭包。但是NSURLSession编程指南有提到：

> Note: Completion callbacks are primarily intended as an alternative to using a custom delegate. If you create a task using a method that takes a completion callback, the delegate methods for response and data delivery are not called.

> 注意：完成回调主要用来当作一种可替代的自定义代理。如果你使用接收完成回调的方法创建了一个任务，那么用来传输响应和数据的代理方法就不会被调用。

所以要注意这些细节。

然后SessionDelegate提供了大量的如下属性，这些属性都是可选的闭包类型，让开发者可以通过给这些闭包赋值来重写作者提供的默认行为。这种属性太多，就不一一列举了：

	    /// Overrides default behavior for NSURLSessionDelegate method `URLSession:didBecomeInvalidWithError:`.
	    public var sessionDidBecomeInvalidWithError: ((NSURLSession, NSError?) -> Void)?
	
	    /// Overrides default behavior for NSURLSessionDelegate method `URLSession:didReceiveChallenge:completionHandler:`.
	    public var sessionDidReceiveChallenge: ((NSURLSession, NSURLAuthenticationChallenge) -> (NSURLSessionAuthChallengeDisposition, NSURLCredential?))?
	
	    /// Overrides default behavior for NSURLSessionDelegate method `URLSessionDidFinishEventsForBackgroundURLSession:`.
	    public var sessionDidFinishEventsForBackgroundURLSession: ((NSURLSession) -> Void)?

大部分的代理方法都是调用默认的闭包实现，如下：

	    public func URLSessionDidFinishEventsForBackgroundURLSession(session: NSURLSession) {
	        sessionDidFinishEventsForBackgroundURLSession?(session)
	    }

当然还有一些方法除了是调用默认闭包实现外，还有调用Request.TaskDelegate的实现的，如下所示：

	    public func URLSession(session: NSURLSession, task: NSURLSessionTask, didCompleteWithError error: NSError?) {
	        if let taskDidComplete = taskDidComplete {
	            taskDidComplete(session, task, error)
	        } else if let delegate = self[task] {
	            delegate.URLSession(session, task: task, didCompleteWithError: error)
	        }
	
	        NSNotificationCenter.defaultCenter().postNotificationName(Notifications.Task.DidComplete, object: task)
	
	        self[task] = nil
	    }

其实到这里基本上Alamofire框架的架构都出来了，每个NSURLSession都有一个代理，默认情况下代理使用默认的闭包实现，要么调用Request.TaskDelegate的实现。所以SessionDelegate就像是一个枢纽或者代理一样，处理一些基本的操作或者将操作分发给**Request**对象的代理（Request.TaskDelegate）。

当然还有一些处理认证、缓存、重定向等的代理方法，基本都是这样的模式，这里就不再一一讲解，等到遇到的时候再说。

相关知识[ NSURLSession的生命周期 ](http://lynchwong.com/2016/01/29/URL-Session-Programming-Guide-Appendix-A-Life-Cycle-of-a-URL-Session/)和[ 使用NSURLSession ](http://lynchwong.com/2016/01/29/URL-Session-Programming-Guide-Using-NSURLSession/)，这些知识比较重要，特别是NSURLSession的生命周期，代理方法的调用顺序等等知识很重要，不然就算你看懂了框架的源码，也不能理解框架这样写的原因、框架设计的精髓。以及其他的一些HTTP协议相关的知识、认证授权等等。这些知识官方编程指南都有提及，参见官方编程指南或者那方面的专业知识。

# Request.swift

这个文件里面的内容看起来还蛮多的，但是`Request`类的内容并不多。我们一步步的来，先看类的属性和初始化方法。

	// MARK: - Properties
	
	/// The delegate for the underlying task.
	public let delegate: TaskDelegate
	
	/// The underlying task.
	public var task: NSURLSessionTask { return delegate.task }
	
	/// The session belonging to the underlying task.
	public let session: NSURLSession
	
	/// The request sent or to be sent to the server.
	public var request: NSURLRequest? { return task.originalRequest }
	
	/// The response received from the server, if any.
	public var response: NSHTTPURLResponse? { return task.response as? NSHTTPURLResponse }
	
	/// The progress of the request lifecycle.
	public var progress: NSProgress { return delegate.progress }
	
	var startTime: CFAbsoluteTime?
	var endTime: CFAbsoluteTime?

大部分属性作者都有注释，主要有当前任务的代理、当前任务、当前任务所属的NSURLSession、原始请求、响应、请求所在的生命周期、开始时间、结束时间等等。大部分属性是计算型属性，还有两个可选属性，剩下的属性只进行了声明没有赋值，所以在初始化里这些属性需要赋值。

	init(session: NSURLSession, task: NSURLSessionTask) {
	    self.session = session
	
	    switch task {
	    case is NSURLSessionUploadTask:
	        delegate = UploadTaskDelegate(task: task)
	    case is NSURLSessionDataTask:
	        delegate = DataTaskDelegate(task: task)
	    case is NSURLSessionDownloadTask:
	        delegate = DownloadTaskDelegate(task: task)
	    default:
	        delegate = TaskDelegate(task: task)
	    }
	
	    delegate.queue.addOperationWithBlock { self.endTime = CFAbsoluteTimeGetCurrent() }
	}

初始化方法也很简单，先赋值`session`属性。然后根据不同的任务类型，使用任务来初始化不同的代理并赋值给`delegate`属性，最后我们向代理里面的一个队列添加了任务，任务很简单，就是赋值结束时间。这些不同类型的代理，我们留到后面。注意这里应该结合Manager类的request方法一起看，因为Request的实例是在那个方法里面创建的，这样前后联系起来看更便于理解。

两个认证的方法以及一些其他的方法我们现在先不看，主要把一开始用例涉及到的代码走以一遍。`resume`、`suspend`很简单，方法最后都post相应的通知，所以你可以在应用里面注册这些通知，当这些方法调用时你能接收到通知。`cancel`方法这里看起来复杂点，其实也很简单，首先判断当前任务的代理和任务，如果属于下载任务，将已经下载的数据保存到代理里面；如果不是下载任务，直接取消，然后post通知。

这里只是`Request`类的一部分，其他内容在另外的一些文件里面，使用`extension`实现。在我们一开始的用例里面就差`responseJSON`这个操作没有说明了，但是说明这一步涉及到的内容挺多的，如果你去看会发现这个方法的定义和实现在`ResponseSerialization.swift`文件里面，然后里面还有各种协议、泛型、还有个新的类型。使用`extension`将`responseJSON`这个行为添加到`Request`类。

所以我们先看一些相关的内容。

## TaskDelegate

这个类我们先不说属性、初始化，先说说这个类的作用。

> The task delegate is responsible for handling all delegate callbacks for the underlying task as well as executing all operations attached to the serial operation queue upon task completion.

`TaskDelegate`类继承自`NSObject`，作者注释如上所示。大概意思就是这个代理类为任务处理所有代理回调，以及执行所有添加到串行队列(这个队列就是代理属性`queue`)里面的任务。

其实这个框架的核心就在上面这句话里，理解了上面的这句话，基本这个框架就懂了大半，框架的结构、代码的执行流程也就很清晰了。

下面我就从两个方面来解释上面的那句话：

* 处理所有代理的回调。
* 执行串行队列里面的所有任务。

### 处理所有代理的回调

首先我们得搞清楚这里所指的代理到底是什么，其实这里的代理指的是NSURLSession的代理，即我们在Manager里面初始化时为NSURLSession指定的代理`SessionDelegate`。

这个代理类我们前面就讲解过了，只是没有详细分析而已。这个代理类适配了所有与NSURLSession和NSURLSessionTask相关的代理：NSURLSessionDelegate、NSURLSessionTaskDelegate、NSURLSessionDataDelegate、NSURLSessionDownloadDelegate。需要注意的事情就是这些代理之间的继承关系以及有些代理方法是可选有些是必须的。

现在我们看一个具体的代理实现，代码如下；

	    public func URLSession(session: NSURLSession, task: NSURLSessionTask, didCompleteWithError error: NSError?) {
	        if let taskDidComplete = taskDidComplete {
	            taskDidComplete(session, task, error)
	        } else if let delegate = self[task] {
	            delegate.URLSession(session, task: task, didCompleteWithError: error)
	        }
	
	        NSNotificationCenter.defaultCenter().postNotificationName(Notifications.Task.DidComplete, object: task)
	
	        self[task] = nil
	    }

这个代理方法属于NSURLSessionTaskDelegate，不管什么任务，当任务完成时，都会调用这个方法，所以这是代理生命周期中的最后一个方法。首先方法用可选绑定判断开发者是否重写了代理的行为(前面我们说过，代理提供了很多可选的闭包类型给开发者来重写代理的默认行为)，如果重写了就调用开发者的重写闭包；然后用可选绑定来获取代理对象(这里涉及到我们前面讲的下标脚本)，注意这里的代理对象的类型是`Request.TaskDelegate`(其他的代理方法会有类型转换，可能为Request.UploadTaskDelegate、Request.DownloadTaskDelegate、Request.DataTaskDelegate)，如果代理存在就调用与本方法类似的代理方法。然后发送通知，任务完成，最后从`subdelegates`字典中移除任务和任务的代理(任务和任务的代理是在Request实例创建后添加到字典中的，参见Manager的request方法)。

所以`SessionDelegate`要么调用开发者重写代理行为的闭包，要么调用子代理(Request.TaskDelegate、Request.TaskDelegateDataTaskDelegate等)实现的相应的代理方法。大部分的代理方法都是这种类似的模式，或者有些不必要处理的代理方法就直接带过。

所以Request.TaskDelegate、Request.TaskDelegateDataTaskDelegate，以及后面会涉及到的Request.UploadTaskDelegate、Request.DownloadTaskDelegate为任务处理所有代理回调都是上面这种模式。

### 执行串行队列里面的所有任务

这里的串行队列指的是TaskDelegate里面的queue属性，类型为NSOperationQueue，NSOperationQueue默认就是异步并发的。

	    init(task: NSURLSessionTask) {
	        self.task = task
	        self.progress = NSProgress(totalUnitCount: 0)
	        self.queue = {
	            let operationQueue = NSOperationQueue()
	            operationQueue.maxConcurrentOperationCount = 1
	            operationQueue.suspended = true
	
	            if #available(OSX 10.10, *) {
	                operationQueue.qualityOfService = NSQualityOfService.Utility
	            }
	
	            return operationQueue
	        }()
	    }
	
	    deinit {
	        queue.cancelAllOperations()
	        queue.suspended = false
	    }

初始化函数主要就是初始化赋值了queue属性，设置了队列的maxConcurrentOperationCount属性1，那么队列就是串行的了，然后将队列的suspended属性设置为true，那么添加的队列的任务就不会执行，直到suspended属性为false。析构函数取消了队列的所有操作，然后设置suspended属性为false。

前面Request初始化的时候给队列添加了任务，代码如下：

	delegate.queue.addOperationWithBlock { self.endTime = CFAbsoluteTimeGetCurrent() }

因为队列是挂起的，所以并不会开始执行，除非队列的suspended属性为false。我们再看看何时将队列的suspended属性设置为了false，除了在析构器设置之外。

仔细找找就会发现在Request.TaskDelegate的一个方法里面设置了队列的suspended属性为false。方法源码如下：

	    func URLSession(session: NSURLSession, task: NSURLSessionTask, didCompleteWithError error: NSError?) {
	        if let taskDidCompleteWithError = taskDidCompleteWithError {
	            taskDidCompleteWithError(session, task, error)
	        } else {
	            if let error = error {
	                self.error = error
	
	                if let
	                    downloadDelegate = self as? DownloadTaskDelegate,
	                    userInfo = error.userInfo as? [String: AnyObject],
	                    resumeData = userInfo[NSURLSessionDownloadTaskResumeData] as? NSData
	                {
	                    downloadDelegate.resumeData = resumeData
	                }
	            }
	
	            queue.suspended = false
	        }
	    }

这个方法的源码很简单，首先判断开发者是否重写了默认的行为，如果重写了就调用开发者重写的闭包，然后结束；如果没有就进行一些相应的处理，然后设置队列的suspended属性为false，队列就开始执行之前添加到队列里面的任务。

之所以在这个方法里面设置队列suspended属性的原因就是这个方法是代理的生命周期中的最后一个方法(前面就说过，SessionDelegate的相应代理方法默认情况下就会调用这个方法)，这时候请求、响应都已经结束了。那为什么一定要等请求和响应结束之后才开始执行队列里面的任务呢？试想一下，如果我们队列里面的任务是要获取响应数据，而请求和响应都还没结束，我们从何获取数据。事实上，`responseJSON`操作就是把我们传入的闭包参数添加到了`queue`队列里面，而闭包的输入参数就是响应数据，所以我们传入的闭包是等到了请求和响应结束之后才执行的。`queue`队列是异步串行的，所以不会阻塞调用`responseJSON`操作的线程。包括其它的诸如`response`、`responseData`、`responseString`、`responsePropertyList`操作也都是添加到队列里面，然后等到请求和响应结束之后才执行的。

所以一开始的用例基本上都已经讲解完了，而且你可以在`responseJSON`操作之后再接其它操作(`response`、`responseData`、`responseString`等)。实际上这些操作只是先添加到了队列上，并没有立即开始执行，而是等到请求响应结束之后再执行。

# ResponseSerialization.swift

剩下的最后内容就是对响应数据的处理了，先看两个简单的类型。

## Result.swift

定义了泛型枚举`Result`类型，方便我们封装值，定义很简单，这里就不讲解了。

## Response.swift

这里定义的是泛型结构体`Response`类型，用于封装响应的数据，都很简单，看源码就明白了。

然后这两个类型都适配了`CustomStringConvertible`、`CustomDebugStringConvertible`协议，便于开发调试，接下来我们主要看ResponseSerialization.swift文件里面的内容。

## ResponseSerializerType协议

该协议源码如下：

	// MARK: ResponseSerializer
	
	/**
	    The type in which all response serializers must conform to in order to serialize a response.
	*/
	public protocol ResponseSerializerType {
	    /// The type of serialized object to be created by this `ResponseSerializerType`.
	    typealias SerializedObject
	
	    /// The type of error to be created by this `ResponseSerializer` if serialization fails.
	    typealias ErrorObject: ErrorType
	
	    /**
	        A closure used by response handlers that takes a request, response, data and error and returns a result.
	    */
	    var serializeResponse: (NSURLRequest?, NSHTTPURLResponse?, NSData?, NSError?) -> Result<SerializedObject, ErrorObject> { get }
	}

在写这篇博客的时候，Xcode已经升至7.3、iOS9.3、Swift2.2版本。所以在协议里已经推荐使用`associatedtype`关键字了，而不是`typealias`，解释说这样更符合语义，确实当初也觉得`typealias`理解起来总是感觉有点别扭。

这个协议很简单，定义了`SerializedObject`和`ErrorObject`关联类型，`ErrorObject`类型必须适配了`ErrorType`协议，然后定义了一个只读的闭包属性`serializeResponse`。

所有处理响应数据的序列化器必须适配该协议。

然后定义了泛型结构体`ResponseSerializer<Value, Error: ErrorType>`，并且适配了上述的`ResponseSerializerType`协议，所以之后我们会使用这个结构体来处理响应的数据。

	/**
	    A generic `ResponseSerializerType` used to serialize a request, response, and data into a serialized object.
	*/
	public struct ResponseSerializer<Value, Error: ErrorType>: ResponseSerializerType {
	    /// The type of serialized object to be created by this `ResponseSerializer`.
	    public typealias SerializedObject = Value
	
	    /// The type of error to be created by this `ResponseSerializer` if serialization fails.
	    public typealias ErrorObject = Error
	
	    /**
	        A closure used by response handlers that takes a request, response, data and error and returns a result.
	    */
	    public var serializeResponse: (NSURLRequest?, NSHTTPURLResponse?, NSData?, NSError?) -> Result<Value, Error>
	
	    /**
	        Initializes the `ResponseSerializer` instance with the given serialize response closure.
	
	        - parameter serializeResponse: The closure used to serialize the response.
	
	        - returns: The new generic response serializer instance.
	    */
	    public init(serializeResponse: (NSURLRequest?, NSHTTPURLResponse?, NSData?, NSError?) -> Result<Value, Error>) {
	        self.serializeResponse = serializeResponse
	    }
	}

首先使用泛型制定了`SerializedObject`和`ErrorObject`的类型，然后实现了`serializeResponse`属性，到这里就适配完了`ResponseSerializerType`协议，最后创建了一个初始化方法用来赋值`serializeResponse`属性。注意`serializeResponse`属性是闭包类型，所以初始化器接收的参数是一个闭包类型。

接下来我们就来具体看看`responseJSON`操作，方法源码如下：

	/**
	    Adds a handler to be called once the request has finished.
	
	    - parameter options:           The JSON serialization reading options. `.AllowFragments` by default.
	    - parameter completionHandler: A closure to be executed once the request has finished.
	
	    - returns: The request.
	*/
	public func responseJSON(
	    options options: NSJSONReadingOptions = .AllowFragments,
	    completionHandler: Response<AnyObject, NSError> -> Void)
	    -> Self
	{
	    return response(
	        responseSerializer: Request.JSONResponseSerializer(options: options),
	        completionHandler: completionHandler
	    )
	}

该方法在`ResponseSerialization.swift`文件中，通过拓展`extension`添加给`Request`类，是实例方法。方法接收的参数`options`类型为`NSJSONReadingOptions`且有默认值，调用的时候可以省略；`completionHandler`参数为`Response<AnyObject, NSError> -> Void`类型，即对应了我们最开始用例里面的：

	{ response in
	    print(response.request)  // original URL request
	    print(response.response) // URL response
	    print(response.data)     // server data
	    print(response.result)   // result of response serialization
	
	    if let JSON = response.result.value {
	        print("JSON: \(JSON)")
	    }
	}

这一部分。注意这里Response的实际类型，如果你是`responseString`操作，那么就是`Response<String, NSError>`；如果是`responseData`，那么就是`Response<NSData, NSError>`。这就是泛型的好处，可以写更通用、更精简的代码。方法的返回值为`Self`，这是Swift里面的知识 ，当方法的返回值为该类型本身的时候，你可以使用`Self`代替，不用明确的写出类型。这些方法返回了类型的实例，所以我们才能在`responseData`等这些操作之后再接其它的类似的操作。

实现`responseJSON`操作的源码如下所示：

    return response(
        responseSerializer: Request.JSONResponseSerializer(options: options),
        completionHandler: completionHandler
    )

直接返回了response方法的返回值，第二个参数即是我们传入的闭包，这个不多做解释。主要说说第一个参数：

	Request.JSONResponseSerializer(options: options)

第一个参数就是`JSONResponseSerializer`方法的返回值，方法的源码如下；

    /**
        Creates a response serializer that returns a JSON object constructed from the response data using 
        `NSJSONSerialization` with the specified reading options.

        - parameter options: The JSON serialization reading options. `.AllowFragments` by default.

        - returns: A JSON object response serializer.
    */
    public static func JSONResponseSerializer(
        options options: NSJSONReadingOptions = .AllowFragments)
        -> ResponseSerializer<AnyObject, NSError>
    {
        return ResponseSerializer { _, response, data, error in
            guard error == nil else { return .Failure(error!) }

            if let response = response where response.statusCode == 204 { return .Success(NSNull()) }

            guard let validData = data where validData.length > 0 else {
                let failureReason = "JSON could not be serialized. Input data was nil or zero length."
                let error = Error.errorWithCode(.JSONSerializationFailed, failureReason: failureReason)
                return .Failure(error)
            }

            do {
                let JSON = try NSJSONSerialization.JSONObjectWithData(validData, options: options)
                return .Success(JSON)
            } catch {
                return .Failure(error as NSError)
            }
        }
    }

这个方法也是通过拓展`extension`添加给`Request`类的，而且是类方法，不需要初始化实例，直接调用。这个方法的返回值就是我们前面讲过的泛型结构体`ResponseSerializer<Value, Error: ErrorType>`，这个方法的实现也很简单，就是初始化了一个实例而已。之前就说了初始化方法接收的参数是一个闭包，是用来序列化响应数据的。具体的序列化的过程其实也很简单，就是把响应数据序列化为JSON对象，然后将序列化后的数据封装在`Result`里面，作为闭包的返回值。详细的序列化过程不再讲解，主要涉及到错误处理、`guard`关键字等，都是Swift的知识。

然后我们再来看response方法，源码如下：

    /**
        Adds a handler to be called once the request has finished.

        - parameter queue:              The queue on which the completion handler is dispatched.
        - parameter responseSerializer: The response serializer responsible for serializing the request, response, 
                                        and data.
        - parameter completionHandler:  The code to be executed once the request has finished.

        - returns: The request.
    */
    public func response<T: ResponseSerializerType>(
        queue queue: dispatch_queue_t? = nil,
        responseSerializer: T,
        completionHandler: Response<T.SerializedObject, T.ErrorObject> -> Void)
        -> Self
    {
        delegate.queue.addOperationWithBlock {
            let result = responseSerializer.serializeResponse(
                self.request,
                self.response,
                self.delegate.data,
                self.delegate.error
            )

            let requestCompletedTime = self.endTime ?? CFAbsoluteTimeGetCurrent()
            let initialResponseTime = self.delegate.initialResponseTime ?? requestCompletedTime

            let timeline = Timeline(
                requestStartTime: self.startTime ?? CFAbsoluteTimeGetCurrent(),
                initialResponseTime: initialResponseTime,
                requestCompletedTime: requestCompletedTime,
                serializationCompletedTime: CFAbsoluteTimeGetCurrent()
            )

            let response = Response<T.SerializedObject, T.ErrorObject>(
                request: self.request,
                response: self.response,
                data: self.delegate.data,
                result: result,
                timeline: timeline
            )

            dispatch_async(queue ?? dispatch_get_main_queue()) { completionHandler(response) }
        }

        return self
    }

这个方法是个泛型方法，首先指定了一个占位类型，该类型适配了`ResponseSerializerType`协议。然后方法的第二个参数就是该占位类型，即适配了`ResponseSerializerType`协议的所有类型都可以作为第二个参数，比如我们刚刚说过的泛型结构体`ResponseSerializer<Value, Error: ErrorType>`，因为它适配了`ResponseSerializerType`协议，完全符合这个占位类型，而这里也的确是传入了这个结构体的实例作为参数。第三个参数就是我们传入的闭包，而Response的类型由占位类型来确定。

然后我们看方法的实现，首先就是给队列添加任务，即我们传入的闭包也包含在这个任务中，也就是在这里把`responseJSON`操作添加到队列里面的。简单讲解下这个任务具体做了什么事情，首先使用第二个参数来获取序列化后的结果，其它的就先不讲解了，接着使用这些数据初始化了一个`Response`的实例，最后将这个实例当作我们传入闭包的输入参数，然后执行闭包。

`??`是空合运算符，自行了解。所以任务里面的最后一行代码在默认情况下，我们的闭包是在主线程上执行的。

`responseData`、`responseString`、`responsePropertyList`都是类似的，这里不再赘述。到这里基本就差不多了，其实上传和下载也是这种类似的模式，只是有些细节不一样，大概的模式以及流程是相似的。

以上。
