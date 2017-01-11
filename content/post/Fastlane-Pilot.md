---
title: Fastlane - Pilot
date: 2016-06-28 09:03:59
categories: 
- iOS
tags: 
- Fastlane
- Automate
- Pilot
metaAlignment: center
autoThumbnailImage: no
coverImage: /img/FastlanePilot/PilotTextTransparentSmall.png

---

> 说明：翻译的 `Pilot` 的指南，[ 原文地址 ](https://github.com/fastlane-old/pilot)。
<!--more-->

![alt text](/img/FastlanePilot/PilotTextTransparentSmall.png)

# Pilot #

管理你 TestFlight 用户的最简单的方法。

Pilot 让你能够非常容易的管理你应用的 Apple's TestFlight。你可以：

* 上传 & 分发 你构建的应用
* 添加 & 移除 测试用户
* 获取 测试用户 & 设备 的信息
* 导出/导入 所有可用的测试用户

`pilot` 使用 [spaceship.airforce](https://spaceship.airforce) 和 iTunes Connect 交互。

`pilot` 现在是 [fastlane](https://fastlane.tools) 的一部分：自动构建、发布你 iOS 和 Android 应用的最简单方法。

# 安装 #

	sudo gem install pilot

# 使用 #

你可以使用 `-u felix@krausefx.com` 命令来指定 Apple ID。如果你的项目已经使用了 [fastlane](https://fastlane.tools) ，用户名和 App identifier 会自动检测。

## 上传构建的应用 ##

上传一个新的构建，只需运行：

	pilot upload

将会在你当前目录查找 `ipa` 文件，然后会尝试从 [fastlane setup](https://fastlane.tools) 中获取登录凭证。

然后会向你询问漏掉的信息。另外，你可以传递所有种类的参数：

	pilot --help

你可以使用下面的命令传递修改日志：

	pilot upload --changelog "Something that is new here"

你也可以跳过二进制的提交，这意味着，`ipa` 文件只会被上传，不会分发给测试用户：

	pilot upload --skip_submission

`pilot` 为你做了所有神奇的工作：

* 从你的 `ipa` 文件中自动检测 Bundle identifier
* 基于 Bundle identifier，自动获取你 App 的 AppID

`pilot` 使用 [spaceship](https://spaceship.airforce) 来提交构建的元数据，然后使用 iTunes Transporter 来上传二进制。

## 列出构建的应用 ##

列出指定应用程序的所有构建：

	pilot builds

会列出所有激活的构建和处理中的构建：

	+-----------+---------+----------+----------+----------+
	|                   Great App Builds                   |
	+-----------+---------+----------+----------+----------+
	| Version # | Build # | Testing  | Installs | Sessions |
	+-----------+---------+----------+----------+----------+
	| 0.9.13    | 1       | Expired  | 1        | 0        |
	| 0.9.13    | 2       | Expired  | 0        | 0        |
	| 0.9.20    | 3       | Expired  | 0        | 0        |
	| 0.9.20    | 4       | Internal | 5        | 3        |
	+-----------+---------+----------+----------+----------+

# 管理测试用户 #

## 列出所有的测试人员 ##

这个命令会列出所有的测试人员，包括内部人员和外部人员。

	pilot list

输出如下：

	+--------+--------+--------------------------+-----------+
	|                    Internal Testers                    |
	+--------+--------+--------------------------+-----------+
	| First  | Last   | Email                    | # Devices |
	+--------+--------+--------------------------+-----------+
	| Felix  | Krause | felix@krausefx.com       | 2         |
	+--------+--------+--------------------------+-----------+

	+-----------+---------+----------------------------+-----------+
	|                       External Testers                       |
	+-----------+---------+----------------------------+-----------+
	| First     | Last    | Email                      | # Devices |
	+-----------+---------+----------------------------+-----------+
	| Max       | Manfred | email@email.com            | 0         |
	| Detlef    | Müller  | detlef@krausefx.com        | 1         |
	+-----------+---------+----------------------------+-----------+

## 添加新的测试人员 ##

使用 `pilot add` 命令来为你的 iTunes Connect 账户和应用添加新的测试人员。这个会创建新的测试人员(如果必要的话)或者添加一个已经存在的测试人员到App进行测试。

	pilot add email@invite.com

除此之外，你可以指定 App 的 identifier(如果必要)：

	pilot add email@email.com -a com.krausefx.app

## 查找一个测试人员 ##

查找一个指定的测试人员：

	pilot find felix@krausefx.com

输出结果如下：

	+---------------------+---------------------+
	|            felix@krausefx.com             |
	+---------------------+---------------------+
	| First name          | Felix               |
	| Last name           | Krause              |
	| Email               | felix@krausefx.com  |
	| Latest Version      | 0.9.14 (23          |
	| Latest Install Date | 03/28/15 19:00      |
	| 2 Devices           | • iPhone 6, iOS 8.3 |
	|                     | • iPhone 5, iOS 7.0 |
	+---------------------+---------------------+

## 移除一个测试人员 ##

这个命令只会移除外部的测试人员：

	pilot remove felix@krausefx.com

## 导出测试人员 ##

将所有的外部测试人员导出到 CSV 文件。这个非常有用，如果你需要将测试人员的信息导入到另一个系统或者新的账户。

	pilot export

## 导入测试人员 ##

从一个 CSV 文件中添加外部测试人员。[这里有一个可用的 CSV 文件](https://itunesconnect.apple.com/itc/docs/tester_import.csv)。

	pilot import

你也可以使用下面的命令来指定目录：

	pilot export -c ~/Desktop/testers.csv
	pilot import -c ~/Desktop/testers.csv

# 提示 #

## [`fastlane`](https://fastlane.tools) 工具链##

* [ `fastlane` ](https://fastlane.tools)：自动化构建和发布你 iOS 和 Android 应用程序的最简单方法
* [ `deliver` ](https://github.com/fastlane/fastlane/tree/master/deliver)：将你的应用、截图、元数据上传到App Store
* [ `snapshot` ](https://github.com/fastlane/fastlane/tree/master/snapshot)：将你iOS应用在每一种设备上进行本地化自动截图
* [ `frameit` ](https://github.com/fastlane/fastlane/tree/master/frameit)：快速将你的截图放入到适合的设备框中
* [ `pem` ](https://github.com/fastlane/fastlane/tree/master/pem)自动生成和更新你的推送通知证书
* [ `sigh` ](https://github.com/fastlane/fastlane/tree/master/sigh)：管理你的 provisioning profiles
* [ `produce` ](https://github.com/fastlane/fastlane/tree/master/produce)：使用命令行工具在 iTunes Connect 和 Dev Portal 上创建新的iOS应用
* [ `cert` ](https://github.com/fastlane/fastlane/tree/master/cert)：自动创建和维护iOS的 code signing certificates
* [ `spaceship` ](https://github.com/fastlane/fastlane/tree/master/spaceship)：一个 Ruby library，自动连接 Apple Dev Center 和 iTunes Connect
* [ `boarding` ](https://github.com/fastlane/boarding)：邀请你 TestFlight 的beta测试人员的最简单方式
* [ `gym` ](https://github.com/fastlane/fastlane/tree/master/gym)：构建你iOS应用程序
* [ `scan` ](https://github.com/fastlane/fastlane/tree/master/scan)：为你iOS和Mac应用运行测试的最简单方法
* [ `match` ](https://github.com/fastlane/fastlane/tree/master/match)：使用Git在你的团队中同步你的 certificates 和 profiles 
* [ `supply` ](https://github.com/fastlane/fastlane/tree/master/supply)：将你的Android应用和数据上传到 Google Play
* [ `screengrab` ](https://github.com/fastlane/fastlane/tree/master/screengrab)：Android版`snapshot`，一样的功能

## 调试信息 ##

如果你遇到了什么问题，你可以使用 `verbose` 模式来获取更多细节的输出：

	pilot --verbose

## 防火墙问题 ##

`pilot` 使用 iTunes Transporter 上传元数据和二进制。以防万一被防火墙挡住，你可以指定不同的传输协议，使用如下命令：

	DELIVER_ITMSTRANSPORTER_ADDITIONAL_UPLOAD_PARAMETERS="-t DAV" pilot ...

如果你使用 `pilot` 通过 [fastlane action](https://github.com/fastlane/fastlane/blob/master/fastlane/docs/Actions.md#pilot) ，添加如下代码到你的 `Fastfile` ：

	ENV["DELIVER_ITMSTRANSPORTER_ADDITIONAL_UPLOAD_PARAMETERS"] = "-t DAV"
	pilot...

## 凭证问题 ##

如果你的密码包含特殊字符，`pilot` 可能会抛出一个模糊的错误，“你的 Apple ID 或者 密码输入不正确”。最简单的修复方法就是修改你的密码，不包含特殊字符。

## 如何保存我的密码？ ##

`pilot` 使用来自 `fastlane` 的 [CredentialsManager](https://github.com/fastlane/fastlane/tree/master/credentials_manager) 管理密码。

# 帮助 #

请提交 issue 到 GitHub，并提供你关于设置的信息。
