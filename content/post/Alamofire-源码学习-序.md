---
title: "Alamofire 源码学习 - 序"
date: 2016-03-16 13:33:16
categories: 
- 源码学习
tags: 
- Alamofire

---

本来这篇博文的标题是“Alamofire 源码分析”，后来想想不太恰当，所以就改成了现在的标题。

前段时间正好翻译完了并发编程指南和NSURLSession相关的一些东西，所以这篇博文也是必须的。
<!--more-->

一直想做个自己的网络请求框架，所以这段时间准备了下：

1. 先是翻译了并发编程指南和NSURLSession的东西。
2. 详细了解了下HTTP协议。
3. 用Go语言写了个Web Service用于测试。

当这些都弄好了要开始写框架的时候却是一头雾水，所以就找了些网络请求的开源框架学习。主要是用Swift写，所以直接看了Alamofire的源码。Alamofire的作者就是AFNetworking的作者，据说99.8%的iOS开发人员的网络请求框架都是AFNetworking。所以这个框架肯定是很适合来学习的，毕竟这作者是大神。

当看完源码着手开始写的时候，又一个问题出现了。我发现我已经跳不出Alamofire的实现方式了，不知不觉就设计的跟Alamofire一样了。首先编写网络请求框架方面，我没有多少经验，所以在实现功能的时候，除了Alamofire的实现方式之外，我想不到更好的方法了。所以我到最后其实就是从新写了一个Alamofire，或者说是山寨了一个。目前我这个山寨框架已经写完了，测试了下数据请求、上传、下载都实现了，但没有经过严格测试，并不完善，不适合在项目中使用。但我会在自己的项目中使用，然后不断完善，改进。

之前翻译的并发编程指南和NSURLSession的文档帮助我理解Alamofire的源码起了很大的作用，所以这个翻译是值得的。Alamofire是完全基于NSURLSession的，看源码的时候你会发现它是按照苹果的编程指南来实现的，简直就是最佳实践。这也说明了苹果官方的编程指南对于开发人员是很重要的，在开发之前先了解官方的编程指南会让我们少走很多弯路，避免很多不必要的BUG。看完了源码之后觉得其实很简单，但是就会有自己怎么想不到的疑问，但是再看第二遍、第三遍的时候发现好像又并不是那么简单。现在我也不敢说全部代码都看懂了，甚至都没看完。感觉看了60%吧，还有很多代码路径没有覆盖到，包括测试代码和Demo的代码。

所以在这里还是声明下：这篇博文只是记录一些自己看源码时候的一些理解，由于博主颜值太高导致智商不是很高，难免会有错误，如果博主有什么不对的地方，那NTMD来打我啊。最后我会将自己写的山寨框架和Go的Web Service上传到GitHub，但是请不要在正式项目里面使用这些源码，要是出了什么纰漏，NTMD就不要来打我了，这个锅你自己背。

接下来的博文就从Request、Upload、Download三个方面进行源码学习。

[ Go WebService ](https://github.com/LynchWong/SpeedyServer)
[ Speedy ](https://github.com/LynchWong/Speedy)
