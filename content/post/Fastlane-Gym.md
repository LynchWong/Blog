---
title: Fastlane - Gym
date: 2016-06-28 09:05:06
categories: 
- iOS
tags: 
- Fastlane
- Automate
- Gym
metaAlignment: center
autoThumbnailImage: no
coverImage: /img/FastlaneGym/gym.png

---

> 说明：翻译的 `Gym` 的指南，[ 原文地址 ](https://github.com/fastlane/fastlane/tree/master/gym)。
<!--more-->

![alt text](/img/FastlaneGym/gym.png)

# gym #

构建你的应用程序从未如此简单。

# gym 是什么？ #

`gym` 构建然后打包你的 iOS 和 OS X 应用程序。为你处理了很多繁重的事情，能够非常简单的生成签名的 `ipa` 或者 `app` 文件。

`gym` 是 [shenzhen](https://github.com/nomad/shenzhen) 的替代。

## `gym` 之前 ##

	xcodebuild clean archive -archivePath build/MyApp \
		                     -scheme MyApp
	xcodebuild -exportArchive \
	           -exportFormat ipa \
		       -archivePath "build/MyApp.xcarchive" \
			   -exportPath "build/MyApp.ipa" \
			   -exportProvisioningProfile "ProvisioningProfileName"

## 使用 `gym` ##

	gym

## 为什么使用 `gym` ##

`gym` 使用最新的API来构建签名你的应用程序，会节省大量的构建时间。

|Gym的功能|
|:------:|
|`gym` 比其它构建工具，比如 [shenzhen](https://github.com/nomad/shenzhen) 要快30%|
|漂亮的输出|
|帮助你解决常见的构建错误，比如 Code Signing issues|
|合理的默认值：自动检测Project、Schemes等|
|与其它工具以及 [fastlane](https://fastlane.tools) 完美配合|
|自动生成 `ipa` 和 压缩的 `dSYM` 文件|
|不需要记住任何复杂的命令，只有 `gym`|
|使用参数和环境变量能够简单的动态配置|
|在 `Gymfile` 文件中存储常用的构建设置|
|所有的 Archives 存储了，可以在 Xcode Organizer 中访问|
|支持 iOS 和 Mac 应用程序|

![alt text](/img/FastlaneGym/gymScreenshot.png)

![alt text](/img/FastlaneGym/gym.gif)

# 安装 #

	sudo gem install gym

确保你已经安装了最新版本的 Xcode 命令行工具：

	xcode-select --install

# 使用 #

	gym

这就是构建程序所需的所有命令。如果你需要更多的控制，这里有些可用的参数：

	gym --workspace "Example.xcworkspace" --scheme "AppName" --clean

如果你需要安装使用不同的 Xcode，使用 xcode-select 或者定义 DEVELOPER_DIR：

	DEVELOPER_DIR="/Applications/Xcode6.2.app" gym

获取可用的参数列表：

	gym --help

如果你遇到任何问题，使用 `verbose` 模式获取更多的信息：

	gym --verbose

通常来说，当你在导出 Archive 时遇到问题，尝试运行：

	gym --use_legacy_build_api

如果你不需要上传到 App Store 或者 TestFlight 设置正确的导出方法：

	gym --export_method ad-hoc

传递布尔参数确保是否使用 `gym` ：

	gym --include_bitcode true --include_symbols false

访问原始的 `xcodebuild` 输出，打开 `~/Library/Logs/gym` 。

## Gymfile ##

由于你可能想要手动触发一个新的构建而不想每次都指定所有的参数，你可以将默认值存储在 `Gymfile` 里。

运行 `gym init` 创建一个新的配置文件，比如：

	scheme "Example"

	sdk "iphoneos9.0"

	clean true

	output_directory "./build"    # store the ipa in this folder
	output_name "MyApp"           # the name of the ipa file

## 导出选项 ##

由于 Xcode7，`gym` 使用新的 Xcode API 允许我们使用 `plist` 文件指定导出选项。`gym` 会为你创建这个文件，然后你可以通过使用 `export_method`，`export_team_id`，`include_symbols` 或者 `include_bitcode` 来修改一些参数。如果你想要更多的选项，比如创建  manifest 文件 或者 app thinning，你可以提供你自己的 `plist` 文件：

	export_options "./ExportOptions.plist"

或者你可以在 `Gymfile` 文件里直接提供 hash 值：

	export_options(
		method: "ad-hoc",
		manifest: {
			appURL: "https://example.com/My App.ipa",
		},
		thinning: "<thin-for-all-variants>"
	)

列出可用的选项运行 `xcodebuild -help`。

## 设置 Code Signing ##

* [More information on how to get started with codesigning](https://github.com/fastlane/fastlane/tree/master/fastlane/docs/Codesigning)
* [Docs on how to set up your Xcode project](https://github.com/fastlane/fastlane/blob/master/fastlane/docs/Codesigning/XcodeProject.md)

## 自动处理所有过程 ##

`gym` 与 [fastlane](https://fastlane.tools) 工作的很好，将所有的发布工具整合到工作流里面。

你可以像这样定义 `fastlane` 的配置：

	lane :beta do
		xctool
		gym(scheme: "MyApp")
		crashlytics
	end

然后你可以非常容易的在其它 `lane` 之间切换(比如 `testflight`，`hockey`，`s3` 等等)。

更多信息，访问 [fastlane GitHub page](https://github.com/fastlane/fastlane/tree/master/fastlane)。

# 如何工作的？ #

`gym` 使用最新的API来构建签名你的应用程序。两个主要的控件：

* `xcodebuild`
* [xcpretty](https://github.com/supermarin/xcpretty)

当你运行 `gym` ，没有使用 `--silent` 模式，将会打印每一个执行的命令。

为了构建 Archive ，`gym` 使用如下命令：

	set -o pipefail && \
	xcodebuild -scheme 'Example' \
	-project './Example.xcodeproj' \
	-configuration 'Release' \
	-destination 'generic/platform=iOS' \
	-archivePath '/Users/felixkrause/Library/Developer/Xcode/Archives/2015-08-11/ExampleProductName 2015-08-11 18.15.30.xcarchive' \
	archive | xcpretty

构建之后，`gym` 会检查。如果有效，就会签名打包到 `ipa` 文件里。

`gym` 会根据你当前使用的 Xcode 的版本选择不同的打包方法。

## Xcode7 及以上 ##

	/usr/bin/xcrun path/to/xcbuild-safe.sh -exportArchive \
	-exportOptionsPlist '/tmp/gym_config_1442852529.plist' \
	-archivePath '/Users/fkrause/Library/Developer/Xcode/Archives/2015-09-21/App 2015-09-21 09.21.56.xcarchive' \
	-exportPath '/tmp/1442852529'

`gym` 使用 Xcode7 最新的API，允许我们使用 `plist` 文件来执行导出选项。你可以运行 `xcodebuild --help` 获取更多可用选项的信息。

Using this method there are no workarounds for WatchKit or Swift required, as it uses the same technique Xcode uses when exporting your binary.

Note: the [xcbuild-safe.sh script](https://github.com/fastlane/fastlane/blob/master/gym/lib/assets/wrap_xcodebuild/xcbuild-safe.sh) wraps around xcodebuild to workaround some incompatibilities.

## Xcode6 及以下 ##

	/usr/bin/xcrun /path/to/PackageApplication4Gym -v \
	'/Users/felixkrause/Library/Developer/Xcode/Archives/2015-08-11/ExampleProductName 2015-08-11 18.15.30.xcarchive/Products/Applications/name.app' -o \
	'/Users/felixkrause/Library/Developer/Xcode/Archives/2015-08-11/ExampleProductName.ipa' \
	--sign "identity" --embed "provProfile"

Note: the official PackageApplication script is replaced by a custom PackageApplication4Gym script. This script is obtained by applying a [set of patches](https://github.com/fastlane/fastlane/tree/master/gym/lib/assets/package_application_patches) on the fly to fix some known issues in the official Xcode PackageApplication script.

Afterwards the ipa file is moved to the output folder. The dSYM file is compressed and moved to the output folder as well.

# 提示 #

## `fastlane`工具链

* [ `fastlane` ](https://fastlane.tools)：自动化构建和发布你 iOS 和 Android 应用程序的最简单方法
* [ `deliver` ](https://github.com/fastlane/fastlane/tree/master/deliver)：将你的应用、截图、元数据上传到App Store
* [ `snapshot` ](https://github.com/fastlane/fastlane/tree/master/snapshot)：将你iOS应用在每一种设备上进行本地化自动截图
* [ `frameit` ](https://github.com/fastlane/fastlane/tree/master/frameit)：快速将你的截图放入到适合的设备框中
* [ `pem` ](https://github.com/fastlane/fastlane/tree/master/pem)自动生成和更新你的推送通知证书
* [ `sigh` ](https://github.com/fastlane/fastlane/tree/master/sigh)：管理你的 provisioning profiles
* [ `produce` ](https://github.com/fastlane/fastlane/tree/master/produce)：使用命令行工具在 iTunes Connect 和 Dev Portal 上创建新的iOS应用
* [ `cert` ](https://github.com/fastlane/fastlane/tree/master/cert)：自动创建和维护iOS的 code signing certificates
* [ `spaceship` ](https://github.com/fastlane/fastlane/tree/master/spaceship)：一个 Ruby library，自动连接 Apple Dev Center 和 iTunes Connect
* [ `pilot` ](https://github.com/fastlane/fastlane/tree/master/pilot)：管理你TestFlight测试人员的最好方式，使用终端构建
* [ `boarding` ](https://github.com/fastlane/boarding)：邀请你 TestFlight 的beta测试人员的最简单方式
* [ `scan` ](https://github.com/fastlane/fastlane/tree/master/scan)：为你iOS和Mac应用运行测试的最简单方法
* [ `match` ](https://github.com/fastlane/fastlane/tree/master/match)：使用Git在你的团队中同步你的 certificates 和 profiles 
* [ `supply` ](https://github.com/fastlane/fastlane/tree/master/supply)：将你的Android应用和数据上传到 Google Play
* [ `screengrab` ](https://github.com/fastlane/fastlane/tree/master/screengrab)：Android版`snapshot`，一样的功能

## 使用 'Provisioning Quicklook plugin' ##

下载安装 [Provisioning Plugin](https://github.com/chockenberry/Provisioning)。

# 帮助 #

请提交 issue 到 GitHub，并提供你关于设置的信息。
