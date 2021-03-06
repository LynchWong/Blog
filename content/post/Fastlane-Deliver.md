---
title: Fastlane - Deliver
date: 2016-06-18 12:49:42
categories: 
- iOS
tags: 
- Fastlane
- Automate
- Deliver
metaAlignment: center
autoThumbnailImage: no
coverImage: /img/FastlaneDeliver/deliver.png

---

> 说明：翻译的 `Deliver` 的指南，[ 原文地址 ](https://github.com/fastlane/fastlane/tree/master/deliver)。
<!--more-->

![alt text](/img/FastlaneDeliver/deliver.png)

# deliver

使用简单的指令就能将你的应用、截图、元数据上传到App Store。

`deliver`可以使用命令行工具上传 ipa 或者 pkg 文件，截图以及更多的数据到 iTunes Connect。

# 功能

* 自动化上传上百张本地化的截图
* 不需要使用任何一台 Mac 的 Xcode 就可以上传新的 ipa/pkg 文件到 iTunes Connect
* 在本地维你应用的元数据，然后将改变推送回 iTunes Connect
* 使用 fastlane 能够简单的实现真正的持续化的部署流程
* 使用 Git 管理配置文件，可以在任务一台 Mac 上部署，包括你持续集成的服务器
* 在将 App 元数据和截图上传到 iTC 之前会生成一个 HTML 的元数据预览

将构建的 App 上传到 TestFlight 查看 [ pilot ](https://github.com/fastlane/fastlane/tree/master/pilot)。

# 安装

安装 gem

	sudo gem install deliver

确保你已经安装了最新版本的 Xcode 命令行工具：

	xcode-select --install

# 快速开始

指引会使用 iTunes Connect 上已经存在的 App 元数据为你创建所有必需的文件。

* `cd [your_project_folder]`
* `deliver init`
* 输入 iTunes Connect Credentials
* 输入 App Identifier
* 在电脑为你做好所有工作之前，可以去先去来一发

从现在起，你可以使用 `deliver` 来部署一个新的版本更新，或者上传新的 App 元数据和截图。

# 使用

查看你本地 `./fastlane/metadata` 和 `./fastlane/screenshots`文件夹(如果你没有使用fastlane，`./metadata`替换)。

![alt text](/img/FastlaneDeliver/metadata.png)

你将会看见从 iTunes Connect 获取的元数据。你可以在本地修改这些数据，然后推送回 iTunes Connect。

运行 `deliver` 将你本地机器的元数据上传。

	deliver

提供 ipa 文件的路径，然后上传你的 App 进行审核：

	deliver --ipa "App.ipa" --submit_for_review

你页可以为 Mac OS X 的 apps 指定 pkg 文件的路径：

	deliver --pkg "MacApp.pkg"

如果你使用 fastlane，你不需要手动指定 ipa/pkg 文件的路径。

这些只是 `deliver` 能做的一小部分，查看完整文档[ Deliverfile.md ](https://github.com/fastlane/fastlane/blob/master/deliver/Deliverfile.md)。

从 iTunes Connect 下载已经存在的截图：

	deliver download_screenshots

获取可用的运行选项：

	deliver --help

选择之前上传的构建，然后提交审核：

	deliver submit_build --build_number 830

查看[ Deliverfile.md ](https://github.com/fastlane/fastlane/blob/master/deliver/Deliverfile.md)获取更多选项。

## 证书

关于你证书是如何处理的，在[ credentials_manager ](https://github.com/fastlane/fastlane/tree/master/credentials_manager)中可以得到详细的描述。

这些东西是如何工作的？是使用的魔法吗？

你的密码会存储在Mac OS X 的 keychain 中，但是也可以传递使用环境变量。([ CredentialsManager ](https://github.com/fastlane/fastlane/tree/master/credentials_manager)中有更多信息)

其实在上传任何东西到 iTunes 之前，`deliver` 会使用收集来的数据的摘要生成一个HTML。

`deliver`在底层使用如下技术：

* 使用 iTMSTransporter 工具来上传二进制文件到 iTunes Connect。iTMSTransporter 是 Apple 提供的一个命令行工具。
* 与所有actions相关的元数据，`deliver`使用的是[ spaceship ](https://github.com/fastlane/fastlane/tree/master/spaceship)。

# 提示

## `fastlane`工具链

* [ `fastlane` ](https://fastlane.tools)：自动化构建和发布你 iOS 和 Android 应用程序的最简单方法
* [ `snapshot` ](https://github.com/fastlane/fastlane/tree/master/snapshot)：将你iOS应用在每一种设备上进行本地化自动截图
* [ `frameit` ](https://github.com/fastlane/fastlane/tree/master/frameit)：快速将你的截图放入到适合的设备框中
* [ `pem` ](https://github.com/fastlane/fastlane/tree/master/pem)：自动生成和更新你的推送通知证书
* [ `sigh` ](https://github.com/fastlane/fastlane/tree/master/sigh)：管理你的 provisioning profiles
* [ `produce` ](https://github.com/fastlane/fastlane/tree/master/produce)：使用命令行工具在 iTunes Connect 和 Dev Portal 上创建新的iOS应用
* [ `cert` ](https://github.com/fastlane/fastlane/tree/master/cert)：自动创建和维护iOS的 code signing certificates
* [ `spaceship` ](https://github.com/fastlane/fastlane/tree/master/spaceship)：一个 Ruby library，自动连接 Apple Dev Center 和 iTunes Connect
* [ `pilot` ](https://github.com/fastlane/fastlane/tree/master/pilot)：管理你TestFlight测试人员的最好方式，使用终端构建
* [ `boarding` ](https://github.com/fastlane/boarding)：邀请你 TestFlight 的beta测试人员的最简单方式
* [ `gym` ](https://github.com/fastlane/fastlane/tree/master/gym)：构建你iOS应用程序
* [ `scan` ](https://github.com/fastlane/fastlane/tree/master/scan)：为你iOS和Mac应用运行测试的最简单方法
* [ `match` ](https://github.com/fastlane/fastlane/tree/master/match)：使用Git在你的团队中同步你的 certificates 和 profiles 
* [ `supply` ](https://github.com/fastlane/fastlane/tree/master/supply)：将你的Android应用和数据上传到 Google Play
* [ `screengrab` ](https://github.com/fastlane/fastlane/tree/master/screengrab)：Android版`snapshot`，一样的功能

## 可用的语言码

	no, en-US, en-CA, fi, ru, zh-Hans, nl-NL, zh-Hant, en-AU, id, de-DE, sv, ko, ms, pt-BR, el, es-ES, it, fr-CA, es-MX, pt-PT, vi, th, ja, fr-FR, da, tr, en-GB

## 默认值

Deliver 有指定的 `default` 语言码，允许你提供不是本地化的值，如果你没有提供本地化的值，将会使用默认值。

你可以在deliverfile中使用json设置，也可以在metadata文件夹中设置。

假如你有`en-US, de-DE, el, it`语言码的本地化数据。

你可以在deliverfile文件中如下设置：

	release_notes({
		'default' => "Shiny and new”,
		'de-DE' => "glänzend und neu"
	})

Deliver会为 en-US, el 和 it 使用 "Shiny and new"。

为 de-DE 使用 "glänzend und neu"。

你也可以使用文件夹达到同样目的：

	default
      keywords.txt
      marketing_url.txt
      name.txt
      privacy_url.txt
      support_url.txt
      release_notes.txt
	en-US
      description.txt
	de-DE
      description.txt
	el
      description.txt
	it
      description.txt

在这种情况下，keywords, urls, name 和 release notes 所有的本地化都会使用默认值，但是每一种语言都有完整的本地化描述。

## 自动创建截图

如果你想 `deliver` 集成[ snapshot ](https://github.com/fastlane/fastlane/tree/master/snapshot)，查看[ fastlane！ ](https://fastlane.tools)。

## Jenkins集成

关于如何在 `Jenkins` 中设置 `deliver` 和 `fastlane` 的详细说明，参见[ fastlane README ](https://github.com/fastlane/fastlane/blob/master/fastlane/docs/Jenkins.md)。

## 防火墙问题

`deliver` 使用 iTunes Transporter 来上传元数据和二进制。为了防止被防火墙阻挡，你可以指定不同的传输协议，使用方法如下：

	DELIVER_ITMSTRANSPORTER_ADDITIONAL_UPLOAD_PARAMETERS="-t DAV" deliver

## HTTP代理

iTunes Transporter 是一个 Java 应用程序，与 Xcode 绑定在一起。为了利用`DELIVER_ITMSTRANSPORTER_ADDITIONAL_UPLOAD_PARAMETERS="-t DAV"`，你需要配置 iTunes Transporter 使用独立于系统代理和任何环境代理的代理设置。你可以使用 Xcode 找到配置文件：

	TOOLS_PATH=$( xcode-select -p )
	REL_PATH='../Applications/Application Loader.app/Contents/itms/java/lib/net.properties'
	echo "$TOOLS_PATH/$REL_PATH"

根据[ Java Proxy Configuration ](http://docs.oracle.com/javase/8/docs/technotes/guides/net/proxies.html)为`net.properties`增加必需的代理配置值。

## 限制

Apple 限制每天只能上传150个二进制文件。

## 编辑`Deliverfile`文件

将语法高亮改为Ruby。

# 需要帮助？

请在 GitHub 上提交 issue 并提供关于你设置的信息。
