---
title: "URL Session Programming Guide - Using NSURLDownload"
date: 2016-01-29 14:11:29
categories: 
- 编程指南
tags: 
- NSURLSession
- NSURLConnection
metaAlignment: center
autoThumbnailImage: no
coverImage: //d1u9biwaxjngwg.cloudfront.net/welcome-to-tranquilpeak/city.jpg

---

[官方文档](https://developer.apple.com/library/prerelease/tvos/documentation/Cocoa/Conceptual/URLLoadingSystem/Tasks/UsingNSURLDownload.html#//apple_ref/doc/uid/20001839-BAJEAIEE)
<!--more-->

# Using NSURLDownload
(使用NSURLDownload)

在OS X中，NSURLDownload让应用程序具有直接将URL的内容下载到磁盘的能力。它提供了类似于NSURLConnection的接口，增加了一个额外的方法用于指定文件的目的地。NSURLDownload还可以解码常用的编码方案，比如MacBinary，BinHex和gzip。不同于NSURLConnection，使用NSURLDownload下载的数据不会存储到缓存系统。

如果你的应用程序并不局限于使用Foundation框架的类，WebKit框架包含了WebDownload，是NSURLDownload的子类，提供了用于认证的用户界面。

**iOS Note**：NSURLDownload类在iOS中不可用，因为并不鼓励直接下载内容到文件系统。使用NSURLSession或者NSURLConnection类代替。参见[ Using NSURLSession ](https://developer.apple.com/library/prerelease/tvos/documentation/Cocoa/Conceptual/URLLoadingSystem/Articles/UsingNSURLSession.html#//apple_ref/doc/uid/TP40013509-SW1)和[ Using NSURLConnection ](https://developer.apple.com/library/prerelease/tvos/documentation/Cocoa/Conceptual/URLLoadingSystem/Tasks/UsingNSURLConnection.html#//apple_ref/doc/uid/20001836-BAJEAIEE)获取更多信息。

## Downloading to a Predetermined Destination
(下载到预定的目的地)

NSURLDownload的一种使用模式就是使用预定的文件名下载到磁盘。如果应用程序直到下载的目标地址，它可以显式的使用setDestination:allowOverwrite:方法。多次给NSURLDownload的实例发送setDestination:allowOverwrite:消息会被忽略。

当接收到initWithRequest:delegate:消息时下载会立即开始。在代理接收到downloadDidFinish:或download:didFailWithError:方法之前的任何时间发送cancel消息就能取消下载。

Listing 3-1的例子设置了目标地址，因此代理只需要实现download:didFailWithError:和downloadDidFinish:方法。

Listing 3-1  Using NSURLDownload with a predetermined destination file location

    - (void)startDownloadingURL:sender
    {
        // Create the request.
        NSURLRequest *theRequest = [NSURLRequest requestWithURL:[NSURL URLWithString:@"http://www.apple.com"]
                                                    cachePolicy:NSURLRequestUseProtocolCachePolicy
                                                timeoutInterval:60.0];
        
        // Create the connection with the request and start loading the data.
        NSURLDownload  *theDownload = [[NSURLDownload alloc] initWithRequest:theRequest
                                                                    delegate:self];
        if (theDownload) {
            // Set the destination file.
            [theDownload setDestination:@"/tmp" allowOverwrite:YES];
        } else {
            // inform the user that the download failed.
        }
    }


    - (void)download:(NSURLDownload *)download didFailWithError:(NSError *)error
    {
        // Dispose of any references to the download object
        // that your app might keep.
        ...
        
        // Inform the user.
        NSLog(@"Download failed! Error - %@ %@",
              [error localizedDescription],
              [[error userInfo] objectForKey:NSURLErrorFailingURLStringErrorKey]);
    }

    - (void)downloadDidFinish:(NSURLDownload *)download
    {
        // Dispose of any references to the download object
        // that your app might keep.
        ...
        
        // Do something with the data.
        NSLog(@"%@",@"downloadDidFinish");
    }

代理可以实现其它额外的方法来自定义处理认证，服务器重定向，以及文件解码。

## Downloading a File Using the Suggested Filename
(使用建议的文件名下载文件)

有时候应用程序需要从下载的数据本身推断出目标文件名。这要求你实现download:decideDestinationWithSuggestedFilename:代理方法然后使用建议的文件名调用setDestination:allowOverwrite:方法。Listing 3-2的例子使用建议的文件名将下载的文件保存在了桌面。

Listing 3-2  Using NSURLDownload with a filename derived from the download

    - (void)startDownloadingURL:sender
    {
        // Create the request.
        NSURLRequest *theRequest = [NSURLRequest requestWithURL:[NSURL URLWithString:@"http://www.apple.com/index.html"]
                                                    cachePolicy:NSURLRequestUseProtocolCachePolicy
                                                timeoutInterval:60.0];
        
        // Create the download with the request and start loading the data.
        NSURLDownload  *theDownload = [[NSURLDownload alloc] initWithRequest:theRequest delegate:self];
        if (!theDownload) {
            // Inform the user that the download failed.
        }
    }

    - (void)download:(NSURLDownload *)download decideDestinationWithSuggestedFilename:(NSString *)filename
    {
        NSString *destinationFilename;
        NSString *homeDirectory = NSHomeDirectory();
        
        destinationFilename = [[homeDirectory stringByAppendingPathComponent:@"Desktop"]
                               stringByAppendingPathComponent:filename];
        [download setDestination:destinationFilename allowOverwrite:NO];
    }


    - (void)download:(NSURLDownload *)download didFailWithError:(NSError *)error
    {
        // Dispose of any references to the download object
        // that your app might keep.
        ...
        
        // Inform the user.
        NSLog(@"Download failed! Error - %@ %@",
              [error localizedDescription],
              [[error userInfo] objectForKey:NSURLErrorFailingURLStringErrorKey]);
    }

    - (void)downloadDidFinish:(NSURLDownload *)download
    {
        // Dispose of any references to the download object
        // that your app might keep.
        ...
        
        // Do something with the data.
        NSLog(@"%@",@"downloadDidFinish");
    }

使用index.html文件名将下载的文件保存在桌面，文件名根据下载内容推断出。给setDestination:allowOverwrite:方法传入NO参数阻止一个已经存在文件被下载的文件重写。相反，通过在文件名后面插入有序的数据来创建一个唯一的文件名，比如index-1.html。

如果代理实现了download:didCreateDestination:方法，当一个文件在磁盘上创建时会通知代理。这个方法同样也给了应用程序机会来决定保存下载内容使用的最终的文件名。

Listing 3-3的例子打印了最终使用的文件名。

Listing 3-3  Logging the finalized filename using download:didCreateDestination:

    -(void)download:(NSURLDownload *)download didCreateDestination:(NSString *)path
    {
        // path now contains the destination path
        // of the download, taking into account any
        // unique naming caused by -setDestination:allowOverwrite:
        NSLog(@"Final file destination: %@",path);
    }

在代理有机会能够响应download:shouldDecodeSourceDataOfMIMEType:和download:decideDestinationWithSuggestedFilename:消息后这个消息就会发送给代理。

## Displaying Download Progress
(显示下载进度)

通过实现代理的download:didReceiveResponse:和download:didReceiveDataOfLength:方法你可以确定下载的进度。

download:didReceiveResponse:方法给代理提供了期望从NSURLResponse获取确定内容长度的机会。代理在每次接收到这个消息的时候应该重新设置进度。

Listing 3-4的例子实现了使用该方法向用户反馈进度。

Listing 3-4  Displaying the download progress

    - (void)setDownloadResponse:(NSURLResponse *)aDownloadResponse
    {
        // downloadResponse is an instance variable defined elsewhere.
        downloadResponse = aDownloadResponse;
    }

    - (void)download:(NSURLDownload *)download didReceiveResponse:(NSURLResponse *)response
    {
        // Reset the progress, this might be called multiple times.
        // bytesReceived is an instance variable defined elsewhere.
        bytesReceived = 0;
        
        // Store the response to use later.
        [self setDownloadResponse:response];
    }

    - (void)download:(NSURLDownload *)download didReceiveDataOfLength:(unsigned)length
    {
        long long expectedLength = [[self downloadResponse] expectedContentLength];
        
        bytesReceived = bytesReceived + length;
        
        if (expectedLength != NSURLResponseUnknownLength) {
            // If the expected content length is
            // available, display percent complete.
            float percentComplete = (bytesReceived/(float)expectedLength)*100.0;
            NSLog(@"Percent complete - %f",percentComplete);
        } else {
            // If the expected content length is
            // unknown, just log the progress.
            NSLog(@"Bytes received - %d",bytesReceived);
        }
    }

在代理开始接收download:didReceiveDataOfLength:消息之前会接收download:didReceiveResponse:消息。

## Resuming Downloads
(恢复下载)

在某些情形下，你可以恢复取消或者失败的下载。首先你必须确保你下载的原始数据没有被删除，通过给下载的setDeletesFileUponFailure:方法传递NO来实现。如果原始的下载失败，你可以使用resumeData方法来获取下载的数据。然后你可以使用initWithResumeData:delegate:path:方法来初始化一个新的下载。当下载恢复时，下载的代理会接收到download:willResumeWithResponse:fromByte:消息。

你可以只恢复连接协议和文件MIME类型支持恢复的下载。你可以使用canResumeDownloadDecodedWithEncodingMIMEType:方法来确定文件的MIME类型是否支持。

## Decoding Encoded Files
(解码编码的文件)

NSURLDownload提供了支持MacBinary，BinHex和gzip文件格式的解码。如果NSURLDownload确定了文件解码支持的格式，它会尝试给代理发送download:shouldDecodeSourceDataOfMIMEType:消息。如果代理实现了这个方法，它应该检查传递的MIME类型，如果文件应该解码就返回YES。

Listing 3-5的例子检查了文件的MIME类型，然后允许使用MacBinary和BinHex解码文件内容。

Listing 3-5  Example implementation of download:shouldDecodeSourceDataOfMIMEType: method

    - (BOOL)download:(NSURLDownload *)download
    shouldDecodeSourceDataOfMIMEType:(NSString *)encodingType
    {
        BOOL shouldDecode = NO;
        
        if ([encodingType isEqual:@"application/macbinary"]) {
            shouldDecode = YES;
        } else if ([encodingType isEqual:@"application/binhex"]) {
            shouldDecode = YES;
        } else if ([encodingType isEqual:@"application/x-gzip"]) {
            shouldDecode = NO;
        }
        return shouldDecode;
    }

