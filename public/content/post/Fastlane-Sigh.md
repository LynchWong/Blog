---
title: Fastlane - Sigh
date: 2016-06-27 12:39:46
categories: 
- iOS
tags: 
- Fastlane
- Automate
- Sigh
metaAlignment: center
autoThumbnailImage: no
coverImage: /img/FastlaneSigh/sigh.png

---

> 说明：翻译的 `Sigh` 的指南，[ 原文地址 ](https://github.com/fastlane/fastlane/tree/master/sigh)。
<!--more-->

![alt text](/img/FastlaneSigh/sigh.png)

# sigh #

因为你宁愿把时间花在其它方面，也不要花在和 Provisioning 文件作斗争上。

`sigh`可以创建、更新、下载和修复 Provisioning Profiles(只需一条命令)。它支持 App Store，Ad Hoc，Development 和 Enterprise Profiles 以及一些很棒的功能，比如自动添加所有的测试设备。

# 功能 #

* 为你的应用程序 **下载** 最新的 Provisioning Profile
* 当 Provisioning Profile 过期的时候会 **更新**
* 当 Provisioning Profile 损坏的时候会 **修复**
* 如果不存在 Provisioning Profile 就会 **创建** 一个新的
* 支持 **App Store**，**Ad Hoc** 和 **Development** Profiles
* 支持 **多个苹果账户**，你的证书(Credentials)安装的存储在Keychain中
* 支持 **多个团队**
* 支持 **Enterprise Profiles**

自动管理iOS的推送证书，你可以使用[ pem ](https://github.com/fastlane/fastlane/tree/master/pem)。

## 为什么不让 Xcode 做这些工作？ ##

* `sigh`可以很容易的集成到你的 CI-server(e.g. Jenkins)
* Xcode 有时会让 [所有存在的 Profiles](https://github.com/fastlane/fastlane/blob/master/sigh/assets/SignErrors.png) 失效
* 你可以对发生的事情进行控制
* 你仍然可以获取到 Signing Files，可用于你的脚本文件或者存储在Git中

参见`sigh`的执行：

![alt text](/img/FastlaneSigh/sighRecording.gif)

# 安装 #

**注意:**根据 [codesigning.guide](https://codesigning.guide)，生成和维护你的 Provisioning Profiles，我们推荐使用 [match](https://github.com/fastlane/fastlane/tree/master/match)。如果你想完整的控制要发生的事情以及知道Codesigning的更多知识，可以直接使用`sigh`。

	sudo gem install sigh

确保你已经安装了 Xcode 最新版本的命令行工具：

	xcode-select --install

# 使用 #

	sigh

是的，这就是所有的命令！

`sigh`默认会为 App Store 创建、修复和下载 Profiles。

你可以像如下这样传递 Bundle identifier 和 username：

	sigh -a com.krausefx.app -u username

如果你想生成 **Ad Hoc** 的Profile 替代 App Store 的Profile：

	sigh --adhoc

**Development** Profile：

	sigh --development

在指定的目录生成 Profile：

	sigh -o "~/Certificates/"

下载你所有的 Provisioning Profiles：

	sigh download_all

列出所有可用的参数和命令：

	sigh --help

## 高级 ##

默认 `sigh` 会在你的机器上安装下载的 Profile。如果你只想生成 Profile 跳过安装，使用如下标志：

	sigh --skip_install

使用指定的名字保存 Provisioning Profile，使用 -q 选项：

	sigh -a com.krausefx.app -u username -q "myProfile.mobileprovision"

出于某些原因，你不想 `sigh` 验证安装在你本地机器上的 Code Signing identity：

	sigh --skip_certificate_verification

如果你需要 Provisioning Profile 更新，而不管是什么状态，使用 `--force` 选项。这会给你的 Profile 最大的生命周期。`--force` 也会将所有可用的设备添加到这个Profile。

	sigh --force

默认情况下，`sigh`会在 Development Profiles 上包含所有的 Certificates，第一个 Certificate 是其它类型的。如果你需要指定使用哪一个 Certificate ，你可以使用环境变量 `SIGH_CERTIFICATE`，或者传递 Certificate 的 name 或者 expiry date 当做参数：

	sigh -c "SunApps GmbH"

列出可用的参数和命令：

	sigh --help

## 和[ `fastlane` ](https://github.com/fastlane/fastlane/tree/master/fastlane)一起使用 ##

当和[ `fastlane` ](https://github.com/fastlane/fastlane/tree/master/fastlane)中的[ `cert` ](https://github.com/fastlane/fastlane/tree/master/cert)一起使用时 `sigh` 就变的很有趣了。

更新你的 `Fastfile` 文件包含如下代码：

	lane :beta do
		cert
		sigh(force: true)
	end

`force: true` 会确保在每次运行的时候都会重新生成 Provisioning Profile 。这会保证 `sigh` 始终使用正确的 Signing Certificate ，会安装在本地机器上。

## 修复(Repair) ##

`sigh` 会自动修复你那些已经存在并且过期或者无效的所有 Provisioning Profiles 。

	sigh repair

# 重新签名(Resign) #

如果你生成了 `ipa` 文件，但是想给 ipa 文件应用不同的 Code Signing，你可以使用 `sigh resign`：

	sigh resign

如果 ipa 文件和 Provisioning Profile 位于当前文件夹，`sigh` 会为你找到它们。

你可以使用命令行传递更多的信息：

	sigh resign ./path/app.ipa --signing_identity "iPhone Distribution: Felix Krause" -n "my.mobileprovision"

## 管理 ##

使用 `sigh manage` ，会列出所有本地安装的 Provisioning Profiles ：

	sigh manage

删除所有过期的 Provisioning Profiles ：

	sigh manage -e

或者使用正则表达式删除所有的 `iOS Team Provisioning Profile` ：

	sigh manage -p "iOS\ ?Team Provisioning Profile:"

## 环境变量 ##

运行 `sigh --help` 列出所有可用的环境变量的列表。

如果你使用 [cert](https://github.com/fastlane/fastlane/tree/master/cert) 与 [fastlane](https://github.com/fastlane/fastlane/tree/master/fastlane)的组合，会自动为你选择 Signing Certificate。(确保在 `sigh` 之前运行 `cert`)

# 如何工作？ #

`sigh` 会访问 `iOS Dev Center` 下载、更新或者生成 `.mobileprovision`文件。它使用 [spaceship](https://spaceship.airforce) 和 Apple's web services 通信。

## 如何存储我的密码？ ##

`sigh` 使用来自 `fastlane` 的 [CredentialsManager](https://github.com/fastlane/fastlane/tree/master/credentials_manager)。

# 提示 #

## `fastlane`工具链

* [ `fastlane` ](https://fastlane.tools)：自动化构建和发布你 iOS 和 Android 应用程序的最简单方法
* [ `deliver` ](https://github.com/fastlane/fastlane/tree/master/deliver)：将你的应用、截图、元数据上传到App Store
* [ `snapshot` ](https://github.com/fastlane/fastlane/tree/master/snapshot)：将你iOS应用在每一种设备上进行本地化自动截图
* [ `frameit` ](https://github.com/fastlane/fastlane/tree/master/frameit)：快速将你的截图放入到适合的设备框中
* [ `pem` ](https://github.com/fastlane/fastlane/tree/master/pem)：自动生成和更新你的推送通知证书
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

它会像下图这样向你展示 `mobileprovision` 文件：

![alt text](/img/FastlaneSigh/QuickLookScreenshot.png)

## 找不到 App Identifier ##

如果你也想在 Apple Developer Portal 上创建一个新的 App Identifier，查看 [produce](https://github.com/fastlane/fastlane/tree/master/produce)，它就是做这个的。

## 对 Xcode 管理的 Profiles 有什么影响？ ##

`sigh` 永远不会触碰或者使用被 Xcode 创建、管理的 Profiles。相反， `sigh` 只会管理自己的 Provisioning Profiles 集合。

# 帮助？ #

请提交 issue 到 GitHub，并提供你关于设置的信息。
