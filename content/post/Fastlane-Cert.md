---
title: Fastlane - Cert
date: 2016-06-28 08:46:19
categories: 
- iOS
tags: 
- Fastlane
- Automate
- Cert
metaAlignment: center
autoThumbnailImage: no
coverImage: /img/FastlaneCert/cert.png

---

> 说明：翻译的 `Cert` 的指南，[ 原文地址 ](https://github.com/fastlane/fastlane/tree/master/cert)。
<!--more-->

![alt text](/img/FastlaneCert/cert.png)

# cert #

自动创建维护 iOS 的 Code Signing Certificates 。

`cert` 只专注于 Code Signing 。你可以为不同的环境(development 和 distribution)创建新的 Code Signing identities 然后使用任何已经存在的有效的 Certificates 替换本地的。

# 安装 #

**注意：**根据 [codesigning.guide](https://codesigning.guide) 推荐使用 [match](https://github.com/fastlane/fastlane/tree/master/match) 来生成维护你的 Certificates 。

如果你想完整的控制要发生的事情以及知道 Codesigning 的更多知识，可以直接使用 `cert` 。

	sudo gem install cert

确保你已经安装了 Xcode 最新版本的命令行工具：

	xcode-select --install

# 为什么使用？ #

请查看 [this guide](https://github.com/fastlane/fastlane/blob/master/cert/ManualSteps.md) ，向你演示了如何使用 Apple Developer Portal 手动创建 iOS Code Signing Profile 和 Provisioning Profile 。

查看完成**之后**，再看 `cert` 和 [`sigh`](https://github.com/fastlane/fastlane/tree/master/sigh) 如何为你做这些工作。

![alt text](/img/FastlaneCert/cert.gif)

如上所示，我使用了 `cert && sigh` ，首先会创建 iOS Code Signing Certificate ，如果 `cert` 成功，然后会为你的 App 创建 Provisioning Profile 。

# 使用 #

	cert

这个会检测你本地机器是否已经安装了可用的 Signing Certificates 。

如果需要创建新的 Certificate ，`cert` 会做如下事情：

* 创建新的 Private Key
* 创建新的 Signing Request
* 生成、下载、安装 Certificate
* 导入所有生成的文件到你的 Keychain

`cert` 永远不会废除(revoke)你已经存在的 Certificates 。如果不能再创建任何新的 Certificates，`cert` 会产生一个异常，意味着你必需废除(revoke)一个已经存在的 Certificates 给新的腾出空间。

你可以传递你的 Apple ID：

	cert -u cert@krausefx.com

可用的命令：

	cert --help

记住，`cert` 没有办法从 Apple Develop Portal 下载已经存在的 Certificates 和 Private Keys ，因为 Private Key 永远不会离开?(leaves)你的电脑。

## 环境变量 ##

运行 `cert --help` 列出所有可用的环境变量的列表。

## 和 [`sigh`](https://github.com/fastlane/fastlane/tree/master/sigh) 一起使用 ##

当 `cert` 和 [`fastlane`](https://github.com/fastlane/fastlane/tree/master/fastlane) 中的 [`sigh`](https://github.com/fastlane/fastlane/tree/master/sigh) 组合使用时就变的很有趣了。

更新你的 `Fastfile` 包含如下代码：

	lane :beta do
		cert
		sigh(force: true)
	end

`force: true` 会确保在每次运行的时候都会重新生成 Provisioning Profile 。这会导致 `sigh` 始终使用的是对的 Signing Certificate，会安装在本地机器上。

## 如何存储我的密码？ ##

`cert` 使用来自 `fastlane` 的 [password manager](https://github.com/fastlane/fastlane/tree/master/credentials_manager)。查看 [CredentialsManager README](https://github.com/fastlane/fastlane/blob/master/credentials_manager/README.md)获取更多信息。

# 提示 #

## `fastlane`工具链

* [ `fastlane` ](https://fastlane.tools)：自动化构建和发布你 iOS 和 Android 应用程序的最简单方法
* [ `deliver` ](https://github.com/fastlane/fastlane/tree/master/deliver)：将你的应用、截图、元数据上传到App Store
* [ `snapshot` ](https://github.com/fastlane/fastlane/tree/master/snapshot)：将你iOS应用在每一种设备上进行本地化自动截图
* [ `frameit` ](https://github.com/fastlane/fastlane/tree/master/frameit)：快速将你的截图放入到适合的设备框中
* [ `pem` ](https://github.com/fastlane/fastlane/tree/master/pem)自动生成和更新你的推送通知证书
* [ `sigh` ](https://github.com/fastlane/fastlane/tree/master/sigh)：管理你的 provisioning profiles
* [ `produce` ](https://github.com/fastlane/fastlane/tree/master/produce)：使用命令行工具在 iTunes Connect 和 Dev Portal 上创建新的iOS应用
* [ `spaceship` ](https://github.com/fastlane/fastlane/tree/master/spaceship)：一个 Ruby library，自动连接 Apple Dev Center 和 iTunes Connect
* [ `pilot` ](https://github.com/fastlane/fastlane/tree/master/pilot)：管理你TestFlight测试人员的最好方式，使用终端构建
* [ `boarding` ](https://github.com/fastlane/boarding)：邀请你 TestFlight 的beta测试人员的最简单方式
* [ `gym` ](https://github.com/fastlane/fastlane/tree/master/gym)：构建你iOS应用程序
* [ `scan` ](https://github.com/fastlane/fastlane/tree/master/scan)：为你iOS和Mac应用运行测试的最简单方法
* [ `match` ](https://github.com/fastlane/fastlane/tree/master/match)：使用Git在你的团队中同步你的 certificates 和 profiles 
* [ `supply` ](https://github.com/fastlane/fastlane/tree/master/supply)：将你的Android应用和数据上传到 Google Play
* [ `screengrab` ](https://github.com/fastlane/fastlane/tree/master/screengrab)：Android版`snapshot`，一样的功能

## 使用'Provisioning Quicklook plugin' ##

下载安装 [Provisioning Plugin](https://github.com/chockenberry/Provisioning) ，方便查看 Provisioning Profile files 和 Certificates 。

# 帮助 #

请提交 issue 到 GitHub，并提供你关于设置的信息。
