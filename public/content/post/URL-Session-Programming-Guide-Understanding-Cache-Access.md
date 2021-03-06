---
title: "URL Session Programming Guide - Understanding Cache Access"
date: 2016-01-29T14:12:39
categories: 
- 编程指南
tags: 
- NSURLSession
- NSURLConnection
metaAlignment: center
autoThumbnailImage: no
coverImage: //d1u9biwaxjngwg.cloudfront.net/welcome-to-tranquilpeak/city.jpg

---

[官方文档](https://developer.apple.com/library/prerelease/tvos/documentation/Cocoa/Conceptual/URLLoadingSystem/Concepts/CachePolicies.html#//apple_ref/doc/uid/20001843-BAJEAIEE)
<!--more-->

# Understanding Cache Access
(理解缓存访问)

URL加载系统为请求提供了综合磁盘和内存的缓存。这些缓存能够减少应用程序对网络的依赖并提高其性能。

## Using the Cache for a Request
(为请求使用缓存)

通过设置缓存策略为NSURLRequestCachePolicy的值：NSURLRequestUseProtocolCachePolicy，NSURLRequestReloadIgnoringCacheData，NSURLRequestReturnCacheDataElseLoad，或者NSURLRequestReturnCacheDataDontLoad来指定NSURLRequest实例如何使用本地缓存。

NSURLRequest实例的默认缓存策略是NSURLRequestUseProtocolCachePolicy。NSURLRequestUseProtocolCachePolicy的行为是协议指定的，以及定义为该协议最适合的策略。

设置缓存策略为NSURLRequestReloadIgnoringCacheData会导致URL加载系统从原始资源加载数据，完全无视缓存。

NSURLRequestReturnCacheDataElseLoad缓存策略会导致URL加载系统加载缓存数据，无视缓存存在的时间以及是否过期，如果没有缓存本本时会从原始资源加载数据。

NSURLRequestReturnCacheDataDontLoad策略允许一个应用程序指定只有缓存中的数据应该返回。当尝试使用这种策略创建NSURLConnection或者NSURLDownload实例时如果本地缓存中没有响应会立即返回nil。这根离线模式的功能很像，并且绝对不会带来网络连接。

**注意**：目前，只有HTTP和HTTPS请求的响应会缓存。FTP和文件协议尝试访问原始资源是允许使用缓存策略。自定义NSURLProtocol类可以选择提供缓存。

## Cache Use Semantics for the HTTP Protocol
(对于HTTP协议缓存的使用语义)

最复杂的缓存使用场景就是当一个请求使用HTTP协议，然后设置缓存策略为NSURLRequestUseProtocolCachePolicy。

如果对于一个请求NSCachedURLResponse不存在，然后URL加载系统就会从原始资源获取数据。

如果请求有缓存的响应，URL加载系统会检查响应，确定指定的内容是否必需重新验证。

如果内容必需重新验证，URL加载系统会向原始资源发起一个HEAD请求查看资源是否改变了。如果没有改变，URL加载系统返回缓存的响应。如果改变了，URL加载系统从原始资源获取数据。

如果缓存的响应没有指定内容必需要重新验证，URL加载系统会检查缓存中的响应指定的最长存在时间和过期时间。如果缓存的响应是最近的，然后URL加载系统返回缓存的响应。如果响应是陈旧过时的，URL加载系统会向原始资源发起一个HEAD请求来确定资源是否改变了。如果改变，URL加载系统就从原始资源获取数据。否则，返回缓存的响应。

[ RFC 2616, Section 13 ](http://www.w3.org/Protocols/rfc2616/rfc2616-sec13.html#sec13)指定了语义涉及到的细节。

## Controlling Caching Programmatically
(缓存控制编程)

默认情况下，一个连接的缓存的数据根据请求的缓存策略来缓存， 由NSURLProtocol子类来处理请求的执行。

如果你的应用程序需要更精确的控制缓存(如果该协议支持缓存)，你可以实现代理方法允许你的应用程序来确定每个请求的响应是否应该被缓存。

* 对于NSURLSession的data 和 upload tasks，实现URLSession:dataTask:willCacheResponse:completionHandler:方法。这个代理方法只会为data 和 upload tasks调用。下载任务的缓存策略由专门指定的缓存策略决定。
* 对于NSURLConnection，实现connection:willCacheResponse:方法。

对于NSURLSession，你的代理方法会调用一个完成处理的Block告诉会话什么要缓存。对于NSURLConnection，你的代理方法会返回连接应该缓存的对象。

其这两种情况，代理通常提供如下之一的值：

* 提供允许缓存的响应对象。
* 一个新创建的响应对象来缓存改变的响应，比如一个storage policy的响应允许缓存到内存但不是磁盘。
* NULL防止缓存。

你的代理方法也能够向NSCachedURLResponse对象相关的userInfo字典插入对象，将这些对象作为缓存响应的一部分。

**重要**：如果你使用NSURLSession并且实现了代理方法，你的代理方法必需始终调用提供的完成处理句柄。否则，你的应用程序会内存泄漏。

Listing 7-1的例子防止缓存HTTPS的响应到磁盘。它同样也向用户的userInfo字典插入了当前日期，来标示缓存响应的时间。

Listing 7-1  Example connection:withCacheResponse: implementation

    -(NSCachedURLResponse *)connection:(NSURLConnection *)connection
    willCacheResponse:(NSCachedURLResponse *)cachedResponse
    {
        NSCachedURLResponse *newCachedResponse = cachedResponse;
        
        NSDictionary *newUserInfo;
        newUserInfo = [NSDictionary dictionaryWithObject:[NSDate date]
                                                  forKey:@"Cached Date"];
        if ([[[[cachedResponse response] URL] scheme] isEqual:@"https"]) {
    #if ALLOW_IN_MEMORY_CACHING
            newCachedResponse = [[NSCachedURLResponse alloc]
                                 initWithResponse:[cachedResponse response]
                                 data:[cachedResponse data]
                                 userInfo:newUserInfo
                                 storagePolicy:NSURLCacheStorageAllowedInMemoryOnly];
    #else // !ALLOW_IN_MEMORY_CACHING
            newCachedResponse = nil
    #endif // ALLOW_IN_MEMORY_CACHING
            } else {
                newCachedResponse = [[NSCachedURLResponse alloc]
                                     initWithResponse:[cachedResponse response]
                                     data:[cachedResponse data]
                                     userInfo:newUserInfo
                                     storagePolicy:[cachedResponse storagePolicy]];
            }
            return newCachedResponse;
        }
