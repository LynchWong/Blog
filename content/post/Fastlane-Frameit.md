---
title: Fastlane - Frameit
date: 2016-06-26 11:21:05
categories: 
- iOS
tags: 
- Fastlane
- Automate
- Frameit
metaAlignment: center
autoThumbnailImage: no
coverImage: /img/FastlaneFrameit/frameit.png

---

> 说明：翻译的 `Frameit` 的指南，[ 原文地址 ](https://github.com/fastlane/fastlane/tree/master/frameit)。
<!--more-->

![alt text](/img/FastlaneFrameit/frameit.png)

# frameit

快速将你的截图放入合适的设备边框中。

运行一条简单的命令`frameit`就能允许你将iOS截图使用华丽的设备框起来。

使用`frameit`为你的 App Store、网站、QA 或者 邮件准备完美的截图。

# 功能

运行一条简单的命令`frameit`就能将你的iOS截图放入到华丽的设备框中。支持：

* iPhone 6 Plus, iPhone 6, iPhone 5s 和 iPad mini
* 横屏和竖屏
* 黑色和白色设备

这里有一个漂亮的Gif，展示了`frameit`的执行：

![alt text](/img/FastlaneFrameit/FrameitGit.gif)

## 结果

![alt text](/img/FastlaneFrameit/ScreenshotsBig.png)
![alt text](/img/FastlaneFrameit/ScreenshotsOverview.png)
![alt text](/img/FastlaneFrameit/MacExample.png)

`frameit`2.0的更新算是[ MindNode ](https://mindnode.com)的赞助，参见上面的截图。

# 安装

确保你已经安装了命令行工具

	xcode-select --install

安装 gem

	sudo gem install frameit

由于法律原因，我不能预先将设备边框打包到`frameit`。

添加边框的过程非常简单，只需要运行`frameit`，之后将会引导帮助你设置。在每台电脑上你只需要做一次。

* 运行`frameit`
* 点击`Enter.`[ Apple page ](https://developer.apple.com/app-store/marketing/guidelines/#images)应该在你的浏览器中打开了，然后下载设备边框的图片
* 下载你想要使用的设备边框
* 点击`Enter`
* 解压并将解压后的文件移动到`~/.frameit/devices_frames`
* 点击`Enter`

# 使用

只是给截图添加一个边框，干嘛还要使用Photoshop呢？

只需导航到你存放截图的文件夹，然后运行如下命令：

	frameit

要使用银色版本的边框：

	frameit silver

添加新的边框再次运行设置的过程：

	frameit setup

当你使用`frameit`的时候没有标题，这个截图将会使用`full`的解决方法，这意味着这个截图不能直接上传到 App Store 。应该使用在网站、多媒体打印、邮件里面。查看下面的章节制作 App Store 的截图。

## 标题和背景(可选)

`frameit`2.0 现在可以添加自定义的背景、标题、还有文字颜色到截图上。

可以在[ fastlane examples ](https://github.com/fastlane/examples/tree/master/MindNode/screenshots)项目中找到可用的例子。

### Framefile.json

使用这个文件来定义通用的信息：

	{
		"default": {
			"keyword": {
				"font": "./fonts/MyFont-Rg.otf"
			},
			"title": {
				"font": "./fonts/MyFont-Th.otf",
				"color": "#545454"
			},
			"background": "./background.jpg",
			"padding": 50,
			"show_complete_frame": false
		},

	    "data": [
			{
				"filter": "Brainstorming",
				"keyword": {
					"color": "#d21559"
				}
			},
			{
				"filter": "Organizing",
				"keyword": {
					"color": "#feb909"
				}
			},
			{
				"filter": "Sharing",
				"keyword": {
					"color": "#aa4dbc"
				}
			},
			{
				"filter": "Styling",
				"keyword": {
					"color": "#31bb48"
				}
			}
		]
	}

`show_complete_frame`的值指定`frameit`是否应该收缩设备和边框，让它们能够完整的显示在边框的截图中。如果值是 false，那么就会超过截图的底部。

`filter`的值是截图名字的一部分，用来决定给定的选项用在哪些截图上。如果一个截图的名字叫做`iPhone5_Brainstorming.png`那么第一个输入的`data`数组会被使用。

你可以找到更复杂的[ 配置 ](https://github.com/fastlane/examples/blob/master/MindNode/screenshots/Framefile.json)，也支持中文、日文、韩文。

`Framefile.json`应该放在`screenshots`文件夹中，如[ example ](https://github.com/fastlane/examples/tree/master/MindNode/screenshots)看到的那样。

### `.strings`文件

为了定义标题和可选的关键字，放入两个`.strings`文件到language文件夹(e.g.[ en-US in the example project ](https://github.com/fastlane/examples/tree/master/MindNode/screenshots/en-US))。

`keyword.strings` 和 `title.strings`是已经为你的iOS应用使用的`.strings`文件，使其易于使用您现有的翻译服务来获得本地化的标题。

**注意：**这些`.strings`文件必需utf-16编码(UTF-16 LE with BOM)。也必须以空行开始。如果有问题，参见[ issue #1740 ](https://github.com/fastlane/fastlane/issues/1740)。

**注意：**如果你想要标题，你必需提供背景。如果没有指定背景，`frameit`不会添加标题。

### 上传截图到iTC

使用[ deliver ](https://github.com/fastlane/fastlane/tree/master/deliver)自动上传所有截图到 iTunes Connect 。

## Mac

`frameit`2.0 也可以 Mac OS X 应用框起来，你必需提供如下信息：

* `offset`信息，`frameit`才知道把你的截图放到哪
* `background`的路径，包含背景和 Mac
* `titleHeight`：标题的高度，以像素为单位

### 例子

	{
		"default": {
			"title": {
				"color": "#545454"
			},
			"background": "Mac.jpg",
			"offset": {
				"offset": "+676+479",
				"titleHeight": 320
			}
		},
		"data": [
			{
				"filter": "Brainstorming",
				"keyword": {
					"color": "#d21559"
				}
			}
		]
	}

查看[ MindNode example project ](https://github.com/fastlane/examples/tree/master/MindNode/screenshots)。

# 提示

## `fastlane`工具链

* [ `fastlane` ](https://fastlane.tools)：自动化构建和发布你 iOS 和 Android 应用程序的最简单方法
* [ `deliver` ](https://github.com/fastlane/fastlane/tree/master/deliver)：将你的应用、截图、元数据上传到App Store
* [ `snapshot` ](https://github.com/fastlane/fastlane/tree/master/snapshot)：将你iOS应用在每一种设备上进行本地化自动截图
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

## 生成本地化截图

查看[ snapshot ](https://github.com/fastlane/fastlane/tree/master/snapshot)。

## 存储device_frames的可选地址

如果你不想将 Device frames 存储在`~/.frameit/device_frames`目录，你可以存储在`./fastlane/screenshots/devices_frames`。但是你如果这样做的话要注意苹果的图片是有版权的，不应该当做你的 Repository 重写发布出去。所以你应该想要在 .gitignore 文件中包含它们。

## 边框白色背景

苹果提供的一些老旧的图片还有白色背景，而不是透明的。你必需编辑这些图片的 Photoshop 文件去掉白色的背景，删除生成的`.png`文件，再次运行`frameit`。

## 使用干净的状态栏

你可以使用[ SimulatorStatusMagic ](https://github.com/shinydevelopment/SimulatorStatusMagic)来清理状态栏。

## 文本周围的灰色东西

如果您遇到任何质量问题，就像字体周围有边框，通常重装 `imagemagick` 会有帮助。你可以运行如下命令来重新安装：

	brew uninstall imagemagick
	brew install imagemagick

## 卸载

* `sudo gem uninstall frameit`
* `rm -rf ~/.frameit`

# 帮助

请提交 issue 到 GitHub，并提供你关于设置的信息。
