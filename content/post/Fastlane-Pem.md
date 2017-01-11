---
title: Fastlane - Pem
date: 2016-06-27 10:31:25
categories: 
- iOS
tags: 
- Fastlane
- Automate
- Pem
metaAlignment: center
autoThumbnailImage: no
coverImage: /img/FastlanePem/pem.png

---

> 说明：翻译的 `Pem` 的指南，[ 原文地址 ](https://github.com/fastlane/fastlane/tree/master/pem)。
<!--more-->

![alt text](/img/FastlanePem/pem.png)

# pem

自动生成和更新您的推送通知的配置文件。

厌倦了为你的iOS应用手动创建和维护推送通知的配置文件？厌倦了为你的服务器生成PEM文件？

只需要简单的运行`pem`命令，`pem`就会为你做所有的这些工作。

如果需要一个有效的推送通知的配置文件，`pem`会创建新的 .pem， .cer 和 .p12 文件并上传到你的推送服务器。

`pem`不会覆盖上传到你服务器的文件。

自动管理 iOS Provisioning profiles，你可以使用[ sigh ](https://github.com/fastlane/fastlane/tree/master/sigh)。

# 功能

其实只有一个功能：为你的服务器生成`pem`文件。

查看这张Gif：

![alt text](/img/FastlanePem/PEMRecording.gif)

# 安装

	sudo gem install pem

确保你已经安装了 Xcode 最新版本的命令行工具：

	xcode-select --install

# 使用

	pem

是的，这就是所有的命令！

会做如下事情：

* 创建新的签名请求(signing request)
* 创建新的推送证书(certification)
* 下载证书(certificate)
* 在当前工作目录创建新的 `.pem` 文件，你可以上传到你的服务器

`pem` 永远不会废除(revoke)你已经存在的证书(certificates)。

如果你已经有一个可用的推送证书(certificate)，至少还有30以上的激活时间，`pem`就不会创建新的证书(certificate)。如果你想要创建新的，使用`force`：

	pem --force

你可以像下面这样传递参数：

	pem -a com.krausefx.app -u username

如果你想要生成开发的证书(development certificate)：

	pem --development

为 `p12` 文件设置密码：

	pem -p "MyPass"

可以为输出文件指定一个名字：

	pem -o my.pem

获取可用的选项，运行：

	pem --help

## 注意空的`p12`密码和 Keychain Access.app

`pem`会产生一个有效的没有指定密码的 `p12` ，或者使用空的字符串当做密码。一旦这个文件有效，如果不指定密码，Mac OS X的Keychain Access不会允许你打开这个文件。

相反，你可以使用 OpenSSL 来验证这个文件是有效的：

	openssl pkcs12 -info -in my.p12

## 环境变量

运行 `pem --help` 获取可用的环境变量的列表。

# 如何工作？

`pem`使用[ spaceship ](https://spaceship.airforce)来和 Apple Developer Portal 交流，为你请求新的推送证书。

## 如何存储我的密码？

`pem` 使用来自 `fastlane` 的 [password manager](https://github.com/fastlane/fastlane/tree/master/credentials_manager)。查看 [CredentialsManager README](https://github.com/fastlane/fastlane/tree/master/credentials_manager)获取更多信息。

# 提示

## `fastlane`工具链

* [ `fastlane` ](https://fastlane.tools)：自动化构建和发布你 iOS 和 Android 应用程序的最简单方法
* [ `deliver` ](https://github.com/fastlane/fastlane/tree/master/deliver)：将你的应用、截图、元数据上传到App Store
* [ `snapshot` ](https://github.com/fastlane/fastlane/tree/master/snapshot)：将你iOS应用在每一种设备上进行本地化自动截图
* [ `frameit` ](https://github.com/fastlane/fastlane/tree/master/frameit)：快速将你的截图放入到适合的设备框中
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

## 使用'Provisioning Quicklook plugin'

下载然后安装 [Provisioning Plugin](https://github.com/chockenberry/Provisioning)。

它会像下图这样向你展示 `pem` 文件：

![alt text](/img/FastlanePem/QuickLookScreenshot.png)

# 帮助

请提交 issue 到 GitHub，并提供你关于设置的信息。
