---
title: "URL Session Programming Guide - Authentication Challenges and TLS Chain Validation"
date: 2016-01-29 14:12:16
categories: 
- 编程指南
tags: 
- NSURLSession
- NSURLConnection
metaAlignment: center
autoThumbnailImage: no
coverImage: //d1u9biwaxjngwg.cloudfront.net/welcome-to-tranquilpeak/city.jpg

---

[官方文档](https://developer.apple.com/library/prerelease/tvos/documentation/Cocoa/Conceptual/URLLoadingSystem/Articles/AuthenticationChallenges.html#//apple_ref/doc/uid/TP40009507-SW1)
<!--more-->

# Authentication Challenges and TLS Chain Validation
(认证挑战和TLS链验证)

一个NSURLRequest对象经常会遇到认证挑战，或者在连接服务器的时候要求凭证。NSURLSession，NSURLConnection，和NSURLDownload类在遇到认证挑战的时候会通知他们的代理，以便采取相应的行动。

**重要**：URL加载系统的类不会调用他们的代理去处理请求挑战除非服务器响应包含了**WWW-Authenticate**首部字段。其它的认证类型，比如proxy authentication和TLS信任验证不需要这个首部。

## Deciding How to Respond to an Authentication Challenge
(决定如何应对认证挑战)

如果一个NSURLRequest对象要求认证，那么这个认证挑战呈现给你应用程序的方式取决于你是使用NSURLSession对象还是NSURLConnection对象还是NSURLDownload对象执行请求的。

* 如果请求和NSURLSession对象相关，所有的认证都会传递给代理，不管是什么认证类型。
* 如果请求和NSURLConnection或者NSURLDownload对象相关，对象的代理会接收到connection:canAuthenticateAgainstProtectionSpace:(或者download:canAuthenticateAgainstProtectionSpace:)消息。这允许代理在尝试反对认证之前分析服务器的属性，包括协议和认证的方法。如果你的代理不准备对服务器的保护空间进行认证，你可以返回NO，系统将尝试从用户的钥匙串中的信息进行认证。
* NSURLConnection或者NSURLDownload对象的代理没有实现connection:canAuthenticateAgainstProtectionSpace:(或者download:canAuthenticateAgainstProtectionSpace:)方法以及用户客户端证书认证的保护空间或者服务器信任认证，系统的行为就像你返回了NO。对于其它的认证类型，系统的行为就像返回了YES。

接下来，如果你的代理同意处理认证，以及没有有效的可用凭证，请求的URL或者共享的NSURLCredentialStorage都没有，代理会接收如下之一的消息：

* URLSession:didReceiveChallenge:completionHandler:
* URLSession:task:didReceiveChallenge:completionHandler:
* connection:didReceiveAuthenticationChallenge:
* download:didReceiveAuthenticationChallenge:

为了让连接能够继续，代理有如下三个选择：

* 提供认证凭证。
* 不要凭证尝试继续。
* 取消认证挑战。

为了帮助确定正确的操作，传递给方法的NSURLAuthenticationChallenge实例包含了有关触发认证挑战的信息，对挑战做了多少次尝试，以及之前尝试的任何凭证，NSURLProtectionSpace要求凭证和挑战的发送者。

如果之前尝试认证并且失败(比如，如果用户改变了它在服务器上的密码)，你可以通过在认证挑战上调用proposedCredential来获得尝试的凭证。代理可以使用这些凭证来填充它给用户呈现的对话框。

在认证挑战上调用previousFailureCount会返回之前认证尝试的总次数，包括不同的认证协议。代理可以提供这些信息给用户，以确定它是否先前提供的凭证是失败的，或限制认证尝试的最大次数。

## Responding to an Authentication Challenge
(响应认证挑战)

你可以使用如下三种方式来响应connection:didReceiveAuthenticationChallenge:代理方法。

### Providing Credentials
(提供凭证)

为了尝试认证，应用程序应创建服务器期望形式的认证信息的NSURLCredential对象。你可以通过在认证挑战的保护空间上调用authenticationMethod来确定服务器认证的方法。被NSURLCredential支持的认证方法有：

* HTTP basic authentication (NSURLAuthenticationMethodHTTPBasic)要求一个用户名和密码。提示用户必需的信息，使用credentialWithUser:password:persistence:方法创建NSURLCredential对象。
* HTTP digest authentication (NSURLAuthenticationMethodHTTPDigest)，类似于basic authentication，要求一个用户名和密码。(digest自动生成。)提示用户必需的信息，使用credentialWithUser:password:persistence:方法创建NSURLCredential对象。
* Client certificate authentication (NSURLAuthenticationMethodClientCertificate)要求系统的身份以及服务器认证需要的所有证书。使用credentialWithIdentity:certificates:persistence:创建一个NSURLCredential对象。
* Server trust authentication (NSURLAuthenticationMethodServerTrust) 要求认证挑战的保护空间提供一个信任证书。使用credentialForTrust:方法创建一个NSURLCredential对象。

创建NSURLCredential对象之后：

* 对于NSURLSession，使用提供的完成处理句柄将对象发送给认证挑战的发送者。
* 对于NSURLConnection和NSURLDownload，使用useCredential:forAuthenticationChallenge:方法将对象传递给认证挑战的发送者。

### Continuing Without Credentials
(继续但不要凭证)

如果代理选择不为认证挑战提供凭证，它仍然可以继续。

* 对于NSURLSession，传递如下之一的值给完成处理的block：
  1. NSURLSessionAuthChallengePerformDefaultHandling处理请求就像代理没有提供处理挑战的方法。
  2. NSURLSessionAuthChallengeRejectProtectionSpace拒绝挑战。取决于服务器响应允许的认证类型，URL加载系统可能为额外的保护空间多次调用代理的方法。
* 对于NSURLConnection和NSURLDownload，在[challenge sender]上调用continueWithoutCredentialsForAuthenticationChallenge:。取决于协议的实现，不要凭证继续可能会引起连接失败，导致接收到connectionDidFailWithError:消息，或者返回替代的没有要求认证的URL内容。

### Canceling the Connection
(取消连接)

代理可能也会选择取消认证挑战。

* 对于NSURLSession，传递NSURLSessionAuthChallengeCancelAuthenticationChallenge给完成处理的Block。
* 对于NSURLConnection或者NSURLDownload，在[challenge sender]上调用cancelAuthenticationChallenge:。代理会接收到connection:didCancelAuthenticationChallenge:消息，提供给用户反馈的机会。

### An Authentication Example
(一个认证例子)

Listing 6-1展示的例子实现了通过使用应用程序偏好设置提供的用户名和密码创建NSURLCredential实例来响应挑战。如果之前的认证失败了，它会取消认证挑战然后通知用户。

Listing 6-1  An example of using the connection:didReceiveAuthenticationChallenge: delegate method

    -(void)connection:(NSURLConnection *)connection
    didReceiveAuthenticationChallenge:(NSURLAuthenticationChallenge *)challenge
    {
        if ([challenge previousFailureCount] == 0) {
            NSURLCredential *newCredential;
            newCredential = [NSURLCredential credentialWithUser:[self preferencesName]
                                                       password:[self preferencesPassword]
                                                    persistence:NSURLCredentialPersistenceNone];
            [[challenge sender] useCredential:newCredential
                   forAuthenticationChallenge:challenge];
        } else {
            [[challenge sender] cancelAuthenticationChallenge:challenge];
            // inform the user that the user name and password
            // in the preferences are incorrect
            [self showPreferencesCredentialsAreIncorrectPanel:self];
        }
    }

如果代理没有实现connection:didReceiveAuthenticationChallenge:方法并且请求要求认证，有效的凭证必需在URL credential storage中或者作为URL请求的一部分提供。如果凭证不可用或者它们认证失败，continueWithoutCredentialForAuthenticationChallenge:消息会被底层实现发送。

## Performing Custom TLS Chain Validation
(执行自定义的TLS链验证)

在NSURL家族API中，TLS链验证由你应用程序的认证代理的方法处理，但不是提供凭证给用户(你应用程序)给服务器，你的应用程序检查凭证在服务器提供TLS握手期间，然后告诉URL加载系统是否应该接受或者拒绝这些凭证。

如果你需要以非标准的方式来执行链验证(比如接受指定的证书来测试)，你的应用程序必需做如下事情：

* 对于NSURLSession，实现URLSession:didReceiveChallenge:completionHandler:或者URLSession:task:didReceiveChallenge:completionHandler:代理方法。如果你都实现了，那么在会话层级的方法会响应处理认证。
* 对于NSURLConnection和NSURLDownload，实现connection:canAuthenticateAgainstProtectionSpace:或者download:canAuthenticateAgainstProtectionSpace:方法，如果认证的保护空间是NSURLAuthenticationMethodServerTrust类型然后返回YES。然后实现connection:didReceiveAuthenticationChallenge:或者download:didReceiveAuthenticationChallenge:方法来处理认证。

在你处理认证的代理方法中，你应该检查认证的保护空间的类型是否是NSURLAuthenticationMethodServerTrust，如果是，从保护空间获取serverTrust信息。

对于额外的详细信息和代码片段(基于NSURLConnection)，阅读 Overriding TLS Chain Validation Correctly。
