---
title: "iOS提交App包"
date: 2015-10-27 15:45:05
categories: 
- Code issue
tags: 
- 提交App包
metaAlignment: center
autoThumbnailImage: no
coverImage: //d1u9biwaxjngwg.cloudfront.net/welcome-to-tranquilpeak/city.jpg

---

前两天打包项目提交App Store的时候一直提交不上去，根据提示的错误信息网上都说是网络的问题，多提交几次就行了。所以刚开始的时候并不在意，过一会再提交。
<!--more-->

过了一会再去提交的时候还是相同的问题，提交不上去。后来验证二进制包，有时能通过有时不能通过，就算通过了还是上传不上去。后来就改用**Application Loader**来提交，然后就报了错误：

	ERROR ITMS-90056: "This bundle is invalid. The Info.plist file is missing the required key: CFBundleVersion."

回头去看项目的Info.plist文件，发现确实是有CFBundleVersion的，所以就重新打包，结果还是报这个错。各种修改版本号等等，又打包了几次仍然报这个错。

关键时刻还是**StackOverflow**给力啊，搜索到了解决方案：[ This bundle is invalid. The Info.plist file is missing the required key: CFBundleVersion ](http://stackoverflow.com/questions/33312621/this-bundle-is-invalid-the-info-plist-file-is-missing-the-required-key-cfbundl)。原因就是因为项目里面的框架的CFBundleVersion是空值导致的，比如Facebook Pop。正好另外一个同事在项目里面加了Facebook Pop，除了这个还把其他框架的CFBundleVersion的值改成了和项目相同的CFBundleVersion值。

这个Info.plist里的CFBundleVersion值就是我们项目Targets->General->Build的值。

最后解决了问题，提交成功。
