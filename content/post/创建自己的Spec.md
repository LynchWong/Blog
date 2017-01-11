---
title: "创建自己的Spec"
date: 2015-06-15 16:33:08
categories: 
- iOS
tags: 
- CocoaPods
metaAlignment: center
autoThumbnailImage: no
coverImage: //d1u9biwaxjngwg.cloudfront.net/welcome-to-tranquilpeak/city.jpg

---

CocoaPods相信大家都听说过了，是用来帮助我们管理第三方依赖库的工具。
<!--more-->

CocoaPods使用起来相当方便，简单的安装和使用就不说了，可以参考一下资料：

[CocoaPods安装和使用教程](http://code4app.com/article/cocoapods-install-usage)
[CocoaPods详解之----使用篇](http://blog.csdn.net/wzzvictory/article/details/18737437)
[CocoaPods官网](https://cocoapods.org/)

相信大家有自己重用的代码，而且很多都可以当做第三方库使用，我们完全可以创建自己的依赖库。

先给出参考链接：

[CocoaPods详解之----制作篇](http://blog.csdn.net/wzzvictory/article/details/20067595) 貌似上传podspec文件，已经不用该博主的方式了。CocoaPods在0.33加入了Trunk服务，后面会讲到。
[Cocoa​Pods](http://nshipster.cn/cocoapods/)，该博文也提到了Trunk服务。

下面给出创建步骤：

# 创建GitHub Repository

![alt text](/img/CreateCocoaPods/1.png)

如图所示，我创建了一个叫做**HZPersistentStack**的Repository，描述那里根据自己的需求随便写，设为Public。选择License，大家自行选择，这里的License是必须的。点击创建后如下图所示：

![alt text](/img/CreateCocoaPods/2.png)

然后我们Clone到本地，如下图：

![alt text](/img/CreateCocoaPods/3.png)

然后我们进入Clone的文件夹，创建一个叫做HZPersistentStack的spec，如下所示：

![alt text](/img/CreateCocoaPods/4.png)

# 创建Spec

创建了一个**HZPersistentStack.podspec**文件，下面我们就要编辑这个文件，你可以使用Vim直接打开编辑或者进入文件夹打开文件直接编辑。

博主用Sublime Text 2打开，打开后发现有很多可编辑的选项，大家可以自行研究下。直接替换为如下内容：

	Pod::Spec.new do |s|  
	  s.name             = "HZPersistentStack"  
	  s.version          = "0.0.1"  
	  s.summary          = "CoreData PersistentStack"  
	  s.homepage         = "https://github.com/LynchWong/HZPersistentStack"  
	  s.license          = 'MIT'  
	  s.author           = { "Lynch Wong" => "lynch.wong@me.com" }  
	  s.source           = { :git => "https://github.com/LynchWong/HZPersistentStack.git", :tag => s.version.to_s }    
	  s.platform     = :ios, '7.0'  
	  s.requires_arc = true  
	  s.source_files = 'Classes/*'  
	end
	
请注意将s.name、s.source替换成自己。

注意这里的s.source_files是相对于**HZPersistentStack.podspec**的路径，所以我们进入到文件夹创建**Classes**文件夹，如下图所示：

![alt text](/img/CreateCocoaPods/5.png)

然后我们将文件类文件放入到**Classes**里，如下图所示：

![alt text](/img/CreateCocoaPods/6.png)

# 验证Spec

然后设置版本和tag，注意版本和tag都是0.0.1，和**HZPersistentStack.podspec**里面的是一样的。如下图所示：

![alt text](/img/CreateCocoaPods/7.png)

然后执行

	git add -A && git commit -m "Release 0.0.1."  
	git tag '0.0.1'  
	git push --tags  
	git push origin master
	
然后执行**pod lib lint**验证Spec，如下图所示：
	
![alt text](/img/CreateCocoaPods/8.png)

如图所示，通过了验证。

# 发布Spec

其实这部分内容可以参考[Cocoa​Pods](http://nshipster.cn/cocoapods/)，也解释了为什么要使用Trunk服务。所以这里不再赘述，直接引用：

> CocoaPods 0.33中加入了Trunk服务。

> 虽然一开始使用GitHub Pull Requests来整理所有公共pods效果很好。但是，随着Pod数量的增加，这个工作对于spec维护人员Keith Smiley来说变得十分繁杂。甚至一些没有通过$ pod lint的spec也被提交上来，造成repo无法build。

> CocoaPods Trunk服务的引入，解决了很多类似的问题。CocoaPods作为一个集中式的服务，使得分析和统计平台数据变得十分方便。

> 要想使用Trunk服务，首先你需要注册自己的电脑。这很简单，只要你指明你的邮箱地址（spec文件中的）和名称即可。
> 
	pod trunk register mattt@nshipster.com "Mattt Thompson"
	
> 然后就会收到邮件验证等。
> 至此，你就可以通过以下命令来方便地发布和升级你的Pod！
> 
	pod trunk push NAME.podspec
>

最后发布Spec：

![alt text](/img/CreateCocoaPods/9.png)

现在你可以搜索你自己的依赖库了，如下：

![alt text](/img/CreateCocoaPods/10.png)

在项目的Podfile文件中添加，如下：

	source 'https://github.com/CocoaPods/Specs.git'
	platform :ios, '8.0'
	
	target 'Demooo' do
		pod 'HZPersistentStack', '~> 0.0.1'
	end
	
然后**pod install**，最后项目结构如下：

![alt text](/img/CreateCocoaPods/11.png)

在需要使用的地方引入：

	#import <HZPersistentStack/PersistentStack.h>
