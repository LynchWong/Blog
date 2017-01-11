---
title: Fastlane - Produce
date: 2016-06-27 17:24:12
categories: 
- iOS
tags: 
- Fastlane
- Automate
- Produce
metaAlignment: center
autoThumbnailImage: no
coverImage: /img/FastlaneProduce/produce.png

---

> 说明：翻译的 `Produce` 的指南，[ 原文地址 ](https://github.com/fastlane/fastlane/tree/master/produce)。
<!--more-->

![alt text](/img/FastlaneProduce/produce.png)

# produce #

使用你的命令行工具在 iTunes Connect 和 Dev Portal 上创建新的iOS应用。

`produce` 会使用最少的信息在 Apple Developer Portal 和 iTunes Connect 上创建新的iOS应用。

# 功能 #

* 在 iTunes Connect 和 Dev Portal 上**创建**新的应用
* **修改** Apple Developer Portal 上的应用服务
* 在 Apple Developer Portal 上**创建** App Groups
* **关联** Apple Developer Portal 上的应用和 App Groups
* 支持**多个苹果帐号**，你的证书安全的存储在 Keychain 中

# 安装 #

	sudo gem install produce

# 使用 #

## 创建新的应用程序 ##

	produce

列出可用的参数列表：

	produce --help

	Commands:
		associate_group  Associate with a group, which is create if needed or simply located otherwise
		create           Creates a new app on iTunes Connect and the Apple Developer Portal
		disable_services Disable specific Application Services for a specific app on the Apple Developer Portal
		enable_services  Enable specific Application Services for a specific app on the Apple Developer Portal
		group            Ensure that a specific App Group exists
		help             Display global or [command] help documentation

	Global Options:
		-u, --username STRING Your Apple ID Username (PRODUCE_USERNAME)
		-a, --app_identifier STRING App Identifier (Bundle ID, e.g. com.krausefx.app) (PRODUCE_APP_IDENTIFIER)
		-e, --bundle_identifier_suffix STRING App Identifier Suffix (Ignored if App Identifier does not ends with .*) (PRODUCE_APP_IDENTIFIER_SUFFIX)
		-q, --app_name STRING App Name (PRODUCE_APP_NAME)
		-z, --app_version STRING Initial version number (e.g. '1.0') (PRODUCE_VERSION)
		-y, --sku STRING     SKU Number (e.g. '1234') (PRODUCE_SKU)
		-m, --language STRING Primary Language (e.g. 'English', 'German') (PRODUCE_LANGUAGE)
		-c, --company_name STRING The name of your company. Only required if it's the first app you create (PRODUCE_COMPANY_NAME)
		-i, --skip_itc       Skip the creation of the app on iTunes Connect (PRODUCE_SKIP_ITC)
		-d, --skip_devcenter  Skip the creation of the app on the Apple Developer Portal (PRODUCE_SKIP_DEVCENTER)
		-b, --team_id STRING The ID of your team if you're in multiple teams (PRODUCE_TEAM_ID)
		-l, --team_name STRING The name of your team if you're in multiple teams (PRODUCE_TEAM_NAME)
		-h, --help           Display help documentation
		-v, --version        Display version information

## 启用／禁用应用程序服务 ##

如果你想要为一个 App ID 启用应用程序服务(Application Services)，这个例子中是 HomeKit 和 HealthKit：

	produce enable_services --homekit --healthkit

如果你想为一个 App ID 禁用应用程序服务，这个例子中是 iCloud：

	produce disable_service --icloud

如果你想创建新的 App Group：

	produce group -g group.krausefx -n "Example App Group"

如果你想关联应用和 App Group：

	produce associate_group -a com.krausefx.app group.krausefx

## 参数 ##

获取可使用的选项列表：

	produce enable_services --help

	--app-group          Enable App Groups
    --associated-domains Enable Associated Domains
    --data-protection STRING Enable Data Protection, suitable values are "complete", "unlessopen" and "untilfirstauth"
    --healthkit          Enable HealthKit
    --homekit            Enable HomeKit
    --wireless-conf      Enable Wireless Accessory Configuration
    --icloud STRING      Enable iCloud, suitable values are "legacy" and "cloudkit"
    --inter-app-audio    Enable Inter-App-Audio
    --passbook           Enable Passbook
    --push-notification  Enable Push notification (only enables the service, does not configure certificates)
    --vpn-conf           Enable VPN Configuration

	produce disable_services --help

	--app-group          Disable App Groups
    --associated-domains Disable Associated Domains
    --data-protection    Disable Data Protection
    --healthkit          Disable HealthKit
    --homekit            Disable HomeKit
    --wireless-conf      Disable Wireless Accessory Configuration
    --icloud             Disable iCloud
    --inter-app-audio    Disable Inter-App-Audio
    --passbook           Disable Passbook
    --push-notification  Disable Push notifications
    --vpn-conf           Disable VPN Configuration

## 环境变量 ##

所有可用的值也可以使用环境变量传递，运行 `produce --help` 来获取所有可用参数的列表。

## `[fastlane](https://github.com/fastlane/fastlane/tree/master/fastlane)` 整合 ##

你的 `fastlane` 看起来应该是这样的：

	lane :appstore do
		produce(
			username: 'felix@krausefx.com',
			app_identifier: 'com.krausefx.app',
			app_name: 'MyApp',
			language: 'English',
			app_version: '1.0',
			sku: '123',
			team_name: 'SunApps GmbH' # only necessary when in multiple teams
		)

	    deliver
	end

为了在 `deliver` 中使用你新生成的应用，你需要将下面的代码添加到 `Deliverfile` 文件中：

	apple_id ENV['PRODUCE_APPLE_ID']

当应用程序还在 App Store 中不可用的时候，这会告诉 `deliver` 使用哪一个 App ID 。

你仍然必需填写剩余的信息(比如截图，应用描述及价格)。你可以使用 [deliver](https://github.com/fastlane/fastlane/tree/master/deliver) 和 CLI 上传应用的元数据。

## 如何存储我的密码？ ##

`produce` 使用来自 `fastlane` 的 [password manager](https://github.com/fastlane/fastlane/tree/master/credentials_manager)。查看 [CredentialsManager README](https://github.com/fastlane/fastlane/tree/master/credentials_manager) 获取更多信息。

# 提示 #

## `fastlane`工具链

* [ `fastlane` ](https://fastlane.tools)：自动化构建和发布你 iOS 和 Android 应用程序的最简单方法
* [ `deliver` ](https://github.com/fastlane/fastlane/tree/master/deliver)：将你的应用、截图、元数据上传到App Store
* [ `snapshot` ](https://github.com/fastlane/fastlane/tree/master/snapshot)：将你iOS应用在每一种设备上进行本地化自动截图
* [ `frameit` ](https://github.com/fastlane/fastlane/tree/master/frameit)：快速将你的截图放入到适合的设备框中
* [ `pem` ](https://github.com/fastlane/fastlane/tree/master/pem)：自动生成和更新你的推送通知证书
* [ `sigh` ](https://github.com/fastlane/fastlane/tree/master/sigh)：管理你的 provisioning profiles
* [ `cert` ](https://github.com/fastlane/fastlane/tree/master/cert)：自动创建和维护iOS的 code signing certificates
* [ `spaceship` ](https://github.com/fastlane/fastlane/tree/master/spaceship)：一个 Ruby library，自动连接 Apple Dev Center 和 iTunes Connect
* [ `pilot` ](https://github.com/fastlane/fastlane/tree/master/pilot)：管理你TestFlight测试人员的最好方式，使用终端构建
* [ `boarding` ](https://github.com/fastlane/boarding)：邀请你 TestFlight 的beta测试人员的最简单方式
* [ `gym` ](https://github.com/fastlane/fastlane/tree/master/gym)：构建你iOS应用程序
* [ `scan` ](https://github.com/fastlane/fastlane/tree/master/scan)：为你iOS和Mac应用运行测试的最简单方法
* [ `match` ](https://github.com/fastlane/fastlane/tree/master/match)：使用Git在你的团队中同步你的 certificates 和 profiles 
* [ `supply` ](https://github.com/fastlane/fastlane/tree/master/supply)：将你的Android应用和数据上传到 Google Play
* [ `screengrab` ](https://github.com/fastlane/fastlane/tree/master/screengrab)：Android版`snapshot`，一样的功能

# 帮助 #

请提交 issue 到 GitHub，并提供你关于设置的信息。
