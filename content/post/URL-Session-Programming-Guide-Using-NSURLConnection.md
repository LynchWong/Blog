---
title: "URL Session Programming Guide - Using NSURLConnection"
date: 2016-01-29 14:11:09
categories: 
- 编程指南
tags: 
- NSURLSession
- NSURLConnection
metaAlignment: center
autoThumbnailImage: no
coverImage: //d1u9biwaxjngwg.cloudfront.net/welcome-to-tranquilpeak/city.jpg

---

[官方文档](https://developer.apple.com/library/prerelease/tvos/documentation/Cocoa/Conceptual/URLLoadingSystem/Tasks/UsingNSURLConnection.html#//apple_ref/doc/uid/20001836-BAJEAIEE)
<!--more-->

# Using NSURLConnection
(使用NSURLConnection)

[ NSURLConnection ](https://developer.apple.com/library/prerelease/tvos/documentation/Cocoa/Reference/Foundation/Classes/NSURLConnection_Class/index.html#//apple_ref/occ/cl/NSURLConnection)提供了最灵活的方法来获取URL的内容。这个类提供了简单的接口来创建和取消一个连接，也提供了一组代理方法来反馈和控制连接的各个方面。这些类分为五个类别：URL加载，缓存管理，认证和凭证，Cookie存储以及协议支持。

## Creating a Connection
(创建连接)

NSURLConnection类支持三种方法获取URL内容：同步获取，使用完成处理器Block异步获取，使用自定义的代理对象异步获取。

同步获取URL内容：对于运行在后台线程的代码，你可以调用[ sendSynchronousRequest:returningResponse:error: ](https://developer.apple.com/library/prerelease/tvos/documentation/Cocoa/Reference/Foundation/Classes/NSURLConnection_Class/index.html#//apple_ref/occ/clm/NSURLConnection/sendSynchronousRequest:returningResponse:error:)方法来执行HTTP请求。这个方法会在请求完成或者发生错误的时候返回。更多信息，参见[ Retrieving Data Synchronously ](https://developer.apple.com/library/prerelease/tvos/documentation/Cocoa/Conceptual/URLLoadingSystem/Tasks/UsingNSURLConnection.html#//apple_ref/doc/uid/20001836-SW3)。
使用完成处理器获取内容：如果你不需要监视请求的状态，仅仅需要在数据完全接收后执行一些操作，你可以调用[ sendAsynchronousRequest:queue:completionHandler: ](https://developer.apple.com/library/prerelease/tvos/documentation/Cocoa/Reference/Foundation/Classes/NSURLConnection_Class/index.html#//apple_ref/occ/clm/NSURLConnection/sendAsynchronousRequest:queue:completionHandler:)方法，传递Block处理结果。更多细节，参阅[ Retrieving Data Using a Completion Handler Block ](https://developer.apple.com/library/prerelease/tvos/documentation/Cocoa/Conceptual/URLLoadingSystem/Tasks/UsingNSURLConnection.html#//apple_ref/doc/uid/20001836-SW1)。
使用代理对象获取内容：创建一个代理类，至少实现这些方法：connection:didReceiveResponse:，connection:didReceiveData:，connection:didFailWithError:，and connectionDidFinishLoading:。支持的代理方法定义在 [ NSURLConnectionDelegate ](https://developer.apple.com/library/prerelease/tvos/documentation/Foundation/Reference/NSURLConnectionDelegate_Protocol/index.html#//apple_ref/occ/intf/NSURLConnectionDelegate), [ NSURLConnectionDownloadDelegate ](https://developer.apple.com/library/prerelease/tvos/documentation/Foundation/Reference/NSURLConnectionDownloadDelegate_Protocol/index.html#//apple_ref/occ/intf/NSURLConnectionDownloadDelegate), and [ NSURLConnectionDataDelegate ](https://developer.apple.com/library/prerelease/tvos/documentation/Foundation/Reference/NSURLConnectionDataDelegate_protocol/index.html#//apple_ref/occ/intf/NSURLConnectionDataDelegate)协议中。

Listing 2-1的例子使用URL初始化了一个连接。这段代码开始使用URL创建了一个NSURLRequest实例，指定了访问的缓存策略和超时。然后创建了一个NSURLConnection实例，指定请求和代理。如果NSURLConnection不能为请求创建连接，initWithRequest:delegate:方法会返回nil。然后创建了NSMutableData实例来存储数据，这些数据由代理提供。

Listing 2-1  Creating a connection using NSURLConnection

    // Create the request.
    NSURLRequest *theRequest=[NSURLRequest requestWithURL:[NSURL URLWithString:@"http://www.apple.com/"]
                                              cachePolicy:NSURLRequestUseProtocolCachePolicy
                                          timeoutInterval:60.0];

    // Create the NSMutableData to hold the received data.
    // receivedData is an instance variable declared elsewhere.
    receivedData = [NSMutableData dataWithCapacity: 0];

    // create the connection with the request
    // and start loading the data
    NSURLConnection *theConnection=[[NSURLConnection alloc] initWithRequest:theRequest delegate:self];
    if (!theConnection) {
        // Release the receivedData object.
        receivedData = nil;
        
        // Inform the user that the connection failed.
    }

在收到initWithRequest:delegate:消息后传输会马上开始。通过给连接发送cancel消息可以在代理接收connectionDidFinishLoading: 或者 connection:didFailWithError:消息之前任何时间取消。

当服务器已经提供了足够了数据来创建一个NSURLResponse对象，代理会接收一个connection:didReceiveResponse:消息。代理方法可以检测提供的NSURLResponse对象，侦测期望的数据的内容长度，MIME类型，建议的文件名，以及服务器提供的其他源数据。

你应该让你的代理准备好单个连接会接收多次connection:didReceiveResponse:消息；当响应是多部分的MIME编码的时候就会发生这种情况。每一次代理接收connection:didReceiveResponse:消息，它应该重设进度指示以及放弃之前接收的数据(除了多响应这种情况)。Listing 2-2展示了每一次调用都将接收的数据的长度设为0.

Listing 2-2  Example connection:didReceiveResponse: implementation

    - (void)connection:(NSURLConnection *)connection didReceiveResponse:(NSURLResponse *)response
    {
        // This method is called when the server has determined that it
        // has enough information to create the NSURLResponse object.
        
        // It can be called multiple times, for example in the case of a
        // redirect, so each time we reset the data.
        
        // receivedData is an instance variable declared elsewhere.
        [receivedData setLength:0];
    }

当接收到数据时，代理会定期的发送connection:didReceiveData:消息。代理实现负责存储新接收到的数据。Listing 2-3的实现例子中，新的数据拼接到了Listing 2-1中创建的NSMutableData对象上了。

Listing 2-3  Example connection:didReceiveData: implementation

    - (void)connection:(NSURLConnection *)connection didReceiveData:(NSData *)data
    {
        // Append the new data to receivedData.
        // receivedData is an instance variable declared elsewhere.
        [receivedData appendData:data];
    }

你也可以使用connection:didReceiveData:方法来向用户提供指示连接的进度。为了实现这些，你必须首先获取期望的内容长度，通过在connection:didReceiveResponse:代理方法中调用响应对象的expectedContentLength方法。如果服务器没有提供长度信息，expectedContentLength返回[ NSURLResponseUnknownLength. ](https://developer.apple.com/library/prerelease/tvos/documentation/Cocoa/Reference/Foundation/Classes/NSURLResponse_Class/index.html#//apple_ref/c/macro/NSURLResponseUnknownLength)。

如果在传输过程中发生了错误，代理会接收到connection:didFailWithError:消息。NSError对象会当作参数传递过去，对象中指定了错误的详细信息。在user info字典中也提供了请求失败的URL，使用[ NSURLErrorFailingURLStringErrorKey ](https://developer.apple.com/library/prerelease/tvos/documentation/Cocoa/Reference/Foundation/Classes/NSError_Class/index.html#//apple_ref/c/data/NSURLErrorFailingURLStringErrorKey)键获取。

在代理接收了connection:didFailWithError:消息，就不会再接收指定连接的任何消息了。

Listing 2-4的例子释放了连接，以及接收的数据，然后打印出错误。

Listing 2-4  Example connection:didFailWithError: implementation

    - (void)connection:(NSURLConnection *)connection
      didFailWithError:(NSError *)error
    {
        // Release the connection and the data object
        // by setting the properties (declared elsewhere)
        // to nil.  Note that a real-world app usually
        // requires the delegate to manage more than one
        // connection at a time, so these lines would
        // typically be replaced by code to iterate through
        // whatever data structures you are using.
        theConnection = nil;
        receivedData = nil;
        
        // inform the user
        NSLog(@"Connection failed! Error - %@ %@",
              [error localizedDescription],
              [[error userInfo] objectForKey:NSURLErrorFailingURLStringErrorKey]);
    }

最后，如果连接成功的检索了请求，代理会接收connectionDidFinishLoading:消息。之后便不会再接收连接的任何消息，应用程序可以释放NSURLConnection对象。

Listing 2-5的列子，实现了打印接收的数据长度，然后释放了连接对象和接收到的数据。

Listing 2-5  Example connectionDidFinishLoading: implementation

    - (void)connectionDidFinishLoading:(NSURLConnection *)connection
    {
        // do something with the data
        // receivedData is declared as a property elsewhere
        NSLog(@"Succeeded! Received %d bytes of data",[receivedData length]);
        
        // Release the connection and the data object
        // by setting the properties (declared elsewhere)
        // to nil.  Note that a real-world app usually
        // requires the delegate to manage more than one
        // connection at a time, so these lines would
        // typically be replaced by code to iterate through
        // whatever data structures you are using.
        theConnection = nil;
        receivedData = nil;
    }

这个例子展示了使用NSURLConnection的一个简单实现。另外的代理方法提供了自定义处理重定向，请求认证，以及响应缓存的能力。

## Making a POST Request
(创建一个POST请求)

你可以像发起其他请求(在[ An Authentication Example ](https://developer.apple.com/library/prerelease/tvos/documentation/Cocoa/Conceptual/URLLoadingSystem/Articles/AuthenticationChallenges.html#//apple_ref/doc/uid/TP40009507-SW2)中描述)一样的方式发起一个HTTP或者HTTPS的POST请求。主要的区别就是你必须首先配置提供给initWithRequest:delegate:方法的[ NSMutableURLRequest ](https://developer.apple.com/library/prerelease/tvos/documentation/Cocoa/Reference/Foundation/Classes/NSMutableURLRequest_Class/index.html#//apple_ref/occ/cl/NSMutableURLRequest)对象。

你也需要构建正文数据(body data)。你可以使用如下三种方式之一构建：

* 对于上传小的，内存中的数据，你应该URL编码存在的数据，正如[ Continuing Without Credentials ](https://developer.apple.com/library/prerelease/tvos/documentation/Cocoa/Conceptual/URLLoadingSystem/Articles/AuthenticationChallenges.html#//apple_ref/doc/uid/TP40009507-SW7)中描述。
* 对于上传磁盘文件数据，调用[ setHTTPBodyStream: ](https://developer.apple.com/library/prerelease/tvos/documentation/Cocoa/Reference/Foundation/Classes/NSMutableURLRequest_Class/index.html#//apple_ref/occ/instm/NSMutableURLRequest/setHTTPBodyStream:)方法告诉NSMutableURLRequest对象从NSInputStream中读取，然后将读取的结果数据当作正文内容(body content)。
* 对于构建大数据块，调用CFStreamCreateBoundPair创建一对流，然后调用setHTTPBodyStream:方法告诉NSMutableURLRequest使用这些流中的一个当作正文内容的资源。通过写入其他数据流，你可以一次发送一小段数据。取决于你服务端是怎么处理这些数据的，你可以希望想要编码你发送的数据。(详情，参见[ Continuing Without Credentials ](https://developer.apple.com/library/prerelease/tvos/documentation/Cocoa/Conceptual/URLLoadingSystem/Articles/AuthenticationChallenges.html#//apple_ref/doc/uid/TP40009507-SW7)。)

如果你上传数据到兼容的服务端，URL加载系统也支持100(Continue)HTTP状态码，允许在认证失败或者其它错误的情况下继续上传。如果要支持上传continuation，在请求对象中设置Expect:首部字段为100-continue。

Listing 6-1展示怎么配置一个POST的NSMutableURLRequest。

Listing 2-6  Configuring an NSMutableRequest object for a POST request

    // In body data for the 'application/x-www-form-urlencoded' content type,
    // form fields are separated by an ampersand. Note the absence of a
    // leading ampersand.
    NSString *bodyData = @"name=Jane+Doe&address=123+Main+St";

    NSMutableURLRequest *postRequest = [NSMutableURLRequest requestWithURL:[NSURL URLWithString:@"https://www.apple.com"]];

    // Set the request's content type to application/x-www-form-urlencoded
    [postRequest setValue:@"application/x-www-form-urlencoded" forHTTPHeaderField:@"Content-Type"];

    // Designate the request a POST request and specify its body data
    [postRequest setHTTPMethod:@"POST"];
    [postRequest setHTTPBody:[NSData dataWithBytes:[bodyData UTF8String] length:strlen([bodyData UTF8String])]];

    // Initialize the NSURLConnection and proceed as described in
    // Retrieving the Contents of a URL

使用setValue:forHTTPHeaderField:方法可以为请求指定不同的内容类型。如果这样做，请确保你的正文内容类型和这个匹配。

为了获取POST请求的进度，在连接的代理中实现connection:didSendBodyData:totalBytesWritten:totalBytesExpectedToWrite:方法。注意这不是上传进度的准确的测量，因为连接可能失败或者连接可能遇到认证挑战。

## Retrieving Data Using a Completion Handler Block
(使用完成处理器Block获取数据)

NSURLConnection类提供支持异步管理，并且在结果返回或者发生错误或者超时时回调Block。要做到这一点，调用类方法sendAsynchronousRequest:queue:completionHandler:方法，提供请求对象，一个回调Block，一个操作队列(用来运行Block)，当请求完成或者错误发生时，URL加载系统会调用这个Block返回结果数据或者错误信息。

如果请求成功，请求的内容传递给回调Block，会返回NSData对象和NSURLResponse对象。如果NSURLConnection不能检索URL，一个NSError对象会当作第三个参数传递过来。

**注意**：这个方法有两个显著的局限性：

* 最小程度的支持需要认证的请求。如果请求需要认证然后连接，有效的凭证必须已经在NSURLCredentialStorage对象中可用了，或者当作URL请求对象的一部分提供了。如果凭证不可用或者认证失败，URL加载系统负责发送连接的continueWithoutCredentialForAuthenticationChallenge:消息到NSURLProtocol的子类处理。
* 没有办法修改默认的响应缓存的行为或者接收服务器重定向。当连接遇到服务器重定向时，重定向始终是被允许的。同样的，根据提供的协议的默认实现，响应的数据存储在缓存中。

NSURLSession提供了相似的功能当没有这些限制。更多信息，阅读[ Using NSURLSession ](https://developer.apple.com/library/prerelease/tvos/documentation/Cocoa/Conceptual/URLLoadingSystem/Articles/UsingNSURLSession.html#//apple_ref/doc/uid/TP40013509-SW1)。

## Retrieving Data Synchronously
(同步获取数据)

NSURLConnection类提供支持同步管理，调用类方法[ sendSynchronousRequest:returningResponse:error: ](https://developer.apple.com/library/prerelease/tvos/documentation/Cocoa/Reference/Foundation/Classes/NSURLConnection_Class/index.html#//apple_ref/occ/clm/NSURLConnection/sendSynchronousRequest:returningResponse:error:)来实现同步获取数据。并不推荐使用这种方法，因为它有几点限制：

* 除非你在开发command-line工具，否则你必须添加额外的代码保证请求没有运行在你应用程序的主线程上。
* 最小程度的支持请求的认证。
* 没有办法修改默认的响应缓存的行为或者接收服务器重定向。

**重要**：如果你同步获取数据，你必须确保请求的代码不会运行在你应用程序的主线程上。网络操作需要人任意长的时间才能完成。如果你尝试在主线程上同步执行网络操作，这些操作会阻塞应用程序的执行直到数据完全获取到，或者发生错误，或者请求超时。这回造成糟糕的用户体验，也会引起iOS终止你的应用程序。

如果请求成功，会通过NSData对象和NSURLResponse对象的引用返回请求的内容。如果NSURLConnection无法检索URL，这方法会返回nil，以及一个NSError实例的引用。

如果请求需要认证之后连接，有效的凭证必须已经在NSURLCredentialStorage对象且可用，或者必须作为URL请求的一部分提供。如果凭证不可用或者认证失败，URL加载系统负责发送连接的continueWithoutCredentialForAuthenticationChallenge:消息到NSURLProtocol的子类处理。

如果同步连接遇到服务器重定向，重定向始终是被允许的。同样的，根据提供的协议的默认实现，响应的数据存储在缓存中。
