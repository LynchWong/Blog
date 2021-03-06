---
title: "ReactiveCocoa"
date: 2015-10-29 11:27:32
categories: 
- iOS
tags: 
- ReactiveCocoa
metaAlignment: center
autoThumbnailImage: no
coverImage: //d1u9biwaxjngwg.cloudfront.net/welcome-to-tranquilpeak/city.jpg

---

原文地址：[ ReactiveCocoa ](https://github.com/ReactiveCocoa/ReactiveCocoa)
<!--more-->

说明：现在RAC的版本已经是4.0了，使用Swift2.0开发的。从3.0开始就已经全面转向Swift，并且跟之前OC版本的相比已经有很大不同了。2.5及以前的OC版本能找到很多的学习资料，包括国内以及国外的，就不多说了。我们这里主要面向4.0的版本，以及Swift2.0。

本篇是翻译RAC4.0的README.md，原文地址在开头已经给出了。你也可以在框架的Tag里面找到其他版本的README.md以及其他的文档，可以看看2.5OC版本和现在的版本有什么不一样。

# ReactiveCocoa

ReactiveCocoa(RAC) 是受[ Functional Reactive Programming ](https://en.wikipedia.org/wiki/Functional_reactive_programming)启发而开发的一个Cocoa框架。它为我们提供了组合和转换流的值的API。

如果你已经熟悉了函数响应式编程或者了解ReactiveCocoa是什么了，你可以查看这些[ 文档 ](https://github.com/ReactiveCocoa/ReactiveCocoa/tree/master/Documentation)来了解关于ReactiveCocoa是如何工作的更深入的信息。然后直接进入[ 文档注释 ](https://github.com/ReactiveCocoa/ReactiveCocoa/tree/master/ReactiveCocoa)学习更多关于OC和Swift的API。

兼容性：本篇文档是关于RAC 4的，Swift2.x。Swift1.2请参考[ RAC3 ](https://github.com/ReactiveCocoa/ReactiveCocoa/tree/v3.0.0)。

## 介绍

ReactiveCocoa 受[ Functional Reactive Programming ](https://en.wikipedia.org/wiki/Functional_reactive_programming)启发。比起使用可变变量来替换或者修改值，RAC提供了“事件流”，用**Signal**和**SignalProducer**类型来表示，随着时间的推移会一直传输值。

事件流统一了Cocoa里面为异步操作以及事件处理的常见的编程模式，包括：

* 代理方法
* 回调Block
* 通知
* 控制事件
* Futures and promises
* KVO

因为这些所有不同的机制可以用同一种方式来表示，能够很容易的通过声明的方式链接组合在一起，能给我们的App带来更少的意大利面似得代码，以及更少的状态。

关于ReactiveCocoa更多的概念，参见[ Framework Overview ](https://github.com/ReactiveCocoa/ReactiveCocoa/blob/master/Documentation/FrameworkOverview.md)。

## 举例：联网搜索

假设你有一个输入框，不管用户输入什么，你都想执行一个网络请求来搜索输入的内容。

### Observing text edits(观察用户的输入)

第一步就是观察用户的输入，我们使用RAC里面**UITextField**的一个拓展来完成这项工作：

	let searchStrings = textField.rac_textSignal()
		.toSignalProducer()
		.map { text in text as! String }

我们得到了一个发送**String**类型值的一个**SignalProducer**。

### Making network requests(进行网络请求)

对于每一次输入的字符串，我们都想执行网络请求。幸运的是，RAC提供的**NSURLSession**的一个拓展能够完成这项工作：

	let searchResults = searchStrings
		.flatMap(.Latest) { (query: String) -> SignalProducer<(NSData, NSURLResponse), NSError> in
			let URLRequest = self.searchRequestWithEscapedQuery(query)
			return NSURLSession.sharedSession().rac_dataWithRequest(URLRequest)
		}
		.map { (data, URLResponse) -> String in
			let string = String(data: data, encoding: NSUTF8StringEncoding)!
			return self.parseJSONResultsFromString(string)
		}
		.observeOn(UIScheduler())

首先降我们发送**String**类型值的**SignalProducer**转换为了一个发送包含搜索结果数组的**SignalProducer**，都会在主线程上发送。

除此之外，.flatMap(.Latest)会保证只有一次搜索－最新的－会被允许执行。如果正在网络请求的时候用户又进行了输入，在开始新的请求前这个网络请求会被取消。想想你自己实现这些功能需要手敲多少代码。

### Receiving the results(接收结果)

其实这些代码并不会执行，因为**SignalProducer**必须*started*才能接收到结果(防止做了工作而并没有使用结果)。很简单：

	searchResults.startWithNext { results in
		print("Search results: \(results)")
	}

这里我们监控了**Next**事件，包含了我们的请求结果，然后打印到控制台。或者你可以在这里做一些其它的事情，比如更新列表视图或者界面上的标签。

### Handling failures(处理失败)

在这个例子中，到目前为止，任何网络请求错误都将会生成**Failed**事件，将会结束这个事件流。不幸的是，这意味着之后的查询甚至都不会被尝试执行。

为了解决这种情况，我们需要决定当失败发生时要做些什么。最快速的解决方案就是打印到日志，然后忽视它们：

	.flatMap(.Latest) { (query: String) -> SignalProducer<(NSData, NSURLResponse), NSError> in
        let URLRequest = self.searchRequestWithEscapedQuery(query)

        return NSURLSession.sharedSession()
            .rac_dataWithRequest(URLRequest)
            .flatMapError { error in
                print("Network error occurred: \(error)")
                return SignalProducer.empty
            }
    }

通过使用**empty**事件流来替换失败，就能够忽视它们。

然后，在放弃之前至少尝试几次可能更合理。方便的是，正好有**retry**操作符。

改进后的**searchResults**如下：

	let searchResults = searchStrings
		.flatMap(.Latest) { (query: String) -> SignalProducer<(NSData, NSURLResponse), NSError> in
			let URLRequest = self.searchRequestWithEscapedQuery(query)

	        return NSURLSession.sharedSession()
			    .rac_dataWithRequest(URLRequest)
				.retry(2)
				.flatMapError { error in
					print("Network error occurred: \(error)")
					return SignalProducer.empty
            }
		}
		.map { (data, URLResponse) -> String in
			let string = String(data: data, encoding: NSUTF8StringEncoding)!
			return self.parseJSONResultsFromString(string)
		}
		.observeOn(UIScheduler())

### Throttling requests(请求节流)

现在，你只想在用户停止输入的时候才执行搜索，来减少请求拥堵。

ReactiveCocoa 定义了**throttle**操作符来实现上面的需求：

	let searchStrings = textField.rac_textSignal()
		.toSignalProducer()
		.map { text in text as! String }
		.throttle(0.5, onScheduler: QueueScheduler.mainQueueScheduler)

当用户停止编辑0.5秒后我们才会使用输入的字符串进行搜索。

如果我们手动的来实现这个需求，需要明显的状态变量，然后代码很难读。使用ReactiveCocoa，我们只需要一个操作符就能把将时间加入到事件流中。

## Objective-C and Swift

尽管ReactiveCocoa是以Objective-C框架开始的，3.0版本，所有主要的功能开发都集中到了Swift API上。

RAC的Objective-C API和Swift API是完全分开的，但是有桥接文件是它们可以互相转换。这样能够兼容老的ReactiveCocoa项目，同时也能使用还没添加到Swift API的Cocoa extensions。

Objective-C API将会继续存在一段时间，但是不再接收很多改进了。更多关于使用Objective-C API 的信息，参阅 [ legacy documentation ](https://github.com/ReactiveCocoa/ReactiveCocoa/tree/master/Documentation/Legacy) 。

强烈推荐所有的新项目使用Swift API。

剩下的内容就不翻译了，不是很重要。最后要说的一点就是3.0版本和之前版本最大的区别就是冷信号和热信号了。之前都是由**RACSignal**表示的，3.0之后就分开了，由**Signal**表示热信号，**SignalProducer**表示冷信号。
