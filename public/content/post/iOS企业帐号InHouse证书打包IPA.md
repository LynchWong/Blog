---
title: iOS企业帐号InHouse证书打包IPA
date: 2016-06-09 14:12:48
categories: 
- iOS
tags: 
- iOS开发者
- InHouse
metaAlignment: center
autoThumbnailImage: no
coverImage: //d1u9biwaxjngwg.cloudfront.net/welcome-to-tranquilpeak/city.jpg

---

参考上篇博文，使用iOS企业帐号的InHouse证书打包了IPA。然后将IPA包放在公司内网服务器，给出下载链接。只要连接到公司内网的同事，都可以直接下载安装。节省了我们手动给同事手机安装App的时间，也避免了在开发时被打扰。

在实现这些的时候踩了不少坑，所以抽时间记录下来。
<!--more-->

# iOS开发者

首先iOS开发者有好几种，这里就不讲解区别、以及如何申请了，这些资源网上都能查找到。这里我们主要涉及的是企业开发者，如何制作证书这里也不讲解了。

# 打包IPA

我们假设已经打包完成，出现了如下界面。

![alt text](/img/InHouse/1.png)

选择**Save for Enterprise Deployment**，点击**Next**。

![alt text](/img/InHouse/2.png)

选择帐号，点击**Choose**。

![alt text](/img/InHouse/3.png)

这里我选择适合所有的设备，没有指定设备。通用的IPA包应该会比特定设备的包要大些，点击**Next**。

![alt text](/img/InHouse/4.png)

之前网上查资料的时候说Xcode6.0不会自动生成.plist文件了，Xcode7又加回来了，但是需要勾选上我红线圈起来的地方。

![alt text](/img/InHouse/5.png)

这里其实是在配置.plist文件，下载App的URL，App的图标等。这里我们先随便填下，反正之后还是可以再编辑.plist文件的。然后我们点击**Export**，导出IPA包和.plist文件。

![alt text](/img/InHouse/6.png)

# .plist文件请求URL

我们下载的时候并不是直接下载的IPA包，而是下载的.plist文件。然后解析.plist文件后再去下载的IPA包，请求.plist文件必须是HTTPS的。

请求的形式如下所示：

	itms-services:///?action=download-manifest&url=你的.plist文件的URL

'url='后面的部分就是.plist文件的HTTPS的URL。

一开始想自己用Go搭建个HTTPS的服务器，按照网上的资料做了后发现下载不成功。不知道是证书的问题还是什么，后来不想浪费时间就放弃了。

所以我将.plist文件托管在了GitHub上面，用的Public的Repository，Private的应该不行。到GitHub上点击具体的.plist文件，然后点击Raw。那个页面的URL地址就是.plist文件的HTTPS的URL，需要跟上面给的那个形式做拼接。

![alt text](/img/InHouse/7.png)

假设最后拼接的结果如下所示：

	itms-services:///?action=download-manifest&url=https://raw.githubusercontent.com/LynchWong/InHouse/master/OnLine/manifest.plist

如果我们在手机上的Safari上请求这个地址就能下载安装我们的App了，当然前提是.plist文件已经配置好了。所以接下来我们就来配置.plist文件。

# 配置.plist文件

![alt text](/img/InHouse/8.png)

这里我们需要修改的地方是1、2、3处：

* 1处就是我们IPA的存放地址，这个地址不需要是HTTPS的。比如我是放在公司内网服务器上的，所以只有连接我们公司内网的同事才能下载到IPA。这样就避免了外网直接获取到IPA，然后对IPA包做逆向工程获取App资源等其它操作。虽然越狱的用户也可以做这些操作，但是总比直接送上IPA包要好。
* 2和3是应用图标的图片地址，大小是57和512的。

其它信息根据需要进行修改。比如**title**并不是你安装后App要显示的名称，这个是在打包前项目里设置的。这个**title**是你要安装时系统提示时使用的，你安装的App叫什么名字。

至此就配置完成，在手机Safari浏览器里输入我们上面拼接后的itms-services协议的URL就能下载IPA包进行安装了。
