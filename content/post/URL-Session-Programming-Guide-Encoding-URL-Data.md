---
title: "URL Session Programming Guide - Encoding URL Data"
date: 2016-01-29 14:11:44
categories: 
- 编程指南
tags: 
- NSURLSession
- NSURLConnection
metaAlignment: center
autoThumbnailImage: no
coverImage: //d1u9biwaxjngwg.cloudfront.net/welcome-to-tranquilpeak/city.jpg

---

[官方文档](https://developer.apple.com/library/prerelease/tvos/documentation/Cocoa/Conceptual/URLLoadingSystem/WorkingwithURLEncoding/WorkingwithURLEncoding.html#//apple_ref/doc/uid/10000165i-CH12-SW1)
<!--more-->

# 编码URL数据

为了编码URL字符串，使用Core Foundation的[ CFURLCreateStringByAddingPercentEscapes ](https://developer.apple.com/library/prerelease/tvos/documentation/CoreFoundation/Reference/CFURLRef/index.html#//apple_ref/c/func/CFURLCreateStringByAddingPercentEscapes)和[ CFURLCreateStringByReplacingPercentEscapesUsingEncoding ](https://developer.apple.com/library/prerelease/tvos/documentation/CoreFoundation/Reference/CFURLRef/index.html#//apple_ref/c/func/CFURLCreateStringByReplacingPercentEscapesUsingEncoding)函数。这些函数允许你指定一组编码的字符，除了high-ASCII (0x80–0xff)和非打印字符。

根据[ RFC 3986 ](http://tools.ietf.org/html/rfc3986)，URL中保留如下字符：

	reserved    = gen-delims / sub-delims
	gen-delims  = ":" / "/" / "?" / "#" / "[" / "]" / "@"
	sub-delims  = "!" / "$" / "&" / "'" / "(" / ")"
                  / "*" / "+" / "," / ";" / "="

因此，正确编码UTF-8字符串包含在URL中，按照如下执行：

    CFStringRef originalString = ...

    CFStringRef encodedString = CFURLCreateStringByAddingPercentEscapes(
                                                                        kCFAllocatorDefault,
                                                                        originalString,
                                                                        NULL,
                                                                        CFSTR(":/?#[]@!$&'()*+,;="),
                                                                        kCFStringEncodingUTF8);

如果要解码一个URL片段，你必须首先把URL字符串分割成其组成部分(域和路径部分)。如果你不对其进行解码，那你无法分辨&符号是原始内容部分还是指示域结束的符号。

在你将URL分成几部分后，你可以按照如下解码每一部分：

    CFStringRef decodedString = CFURLCreateStringByReplacingPercentEscapesUsingEncoding(
                                                                                        kCFAllocatorDefault,
                                                                                        encodedString,
                                                                                        CFSTR(""),
                                                                                        kCFStringEncodingUTF8);

**重要**： 尽管NSString类提供了内建增加百分号转义的方法，通常你不应该使用它们。这些方法会假定你传递给它们的字符串包含一系列的&分割值，其结果是，你不能正确的URL编码任何包含&符号的字符串。 
