---
title: "Alamofire 源码学习 - Download"
date: 2016-03-19 13:11:31
categories: 
- 源码学习
tags: 
- Alamofire
metaAlignment: center
autoThumbnailImage: no
coverImage: //d1u9biwaxjngwg.cloudfront.net/welcome-to-tranquilpeak/city.jpg

---

其实下载很简单，模式上来说和上传基本是一样的，至少在创建请求上模式没有区别。感觉上已经多说无益了，也是浪费时间。
<!--more-->

首先这里下载也分两种情况，全新的下载和从指定的数据继续下载。所以在Alamofire.swift文件里面的那三个入口方法就很好理解；包括在Download.swift文件中的代码和Upload.swift文件中的代码形式上都是和上传类似的，这些也很好理解。

所以这里就不再赘述了，其它下载相关的一些细节：比如NSURLSession的API下载完成后返回的是临时文件的地址，所以你最好将文件保存到别处；以及后台下载等等的一些处理，请参考编程指南。

到这里就结束了，当然Alamofire还有很多内容我这里没有涉及到，比如SSL、验证、时间线、网络检测等等很多内容。除了框架核心的代码，还有测试的代码、示例项目的代码都没有提及，相信这些代码里面还有很多内容、知识。

从开始看Alamofire源码以来，对于我来说感觉还是学了不少知识的，也许在这几篇博文里面难以体现。至少又去熟悉了一遍HTTP协议，又熟悉了一遍Swift语法，仔细看了苹果的编程指南。当然最重要的就是学习了大神的代码，学习了编码风格，还有思想。

诚惶诚恐，以上。
