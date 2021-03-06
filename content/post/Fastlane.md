---
title: Fastlane
date: 2016-06-10 10:08:51
categories: 
- iOS
tags: 
- Fastlane
- Automate
metaAlignment: center
autoThumbnailImage: no
coverImage: //d1u9biwaxjngwg.cloudfront.net/welcome-to-tranquilpeak/city.jpg

---

最近一直在研究持续集成、单元测试、自动化的东西。这些东西是明显能够提高我们工作效率、保证我们编码质量的工具。对我们的编码工作有很大的帮助，而且一次配置，一直受用的。
<!--more-->

# 持续集成

## Travis CI

这个我想大家都了解，对于开源项目的持续集成是免费的，但是对于私有的是收费的，而且价格还蛮高的。去了解了下，发现使用还是蛮简单的，跟GitHub搭配的很好，适合开源项目。

但是公司的代码都是托管在自己的服务器上的，如果要使用Travis CI。首先必须在GitHub上建私有的仓库，然后到Travis CI上付费才能使用。所以对于这一笔开销其实是不必要的，而且还有其它的持续集成的工具可以选择。

## Jenkins

估计很多公司都在用Jenkins，我司也是使用的Jenkins。我司Jenkins的安装、配置都是测试组的同事完成的，每天会定时打包，所以不会每一次提交代码都进行持续集成。而且我们项目里面没有测试的代码，所以我们把Jenkins基本就是当作一个打包工具在使用而已。

Jenkins的安装、配置、使用我就不啰嗦了，我也没去了解。

# 单元测试

单元测试这一块了解的不多，反正我自己的项目是很少写测试代码的。TDD、BDD类型的测试框架很多，没有研究过，之后空了单开一篇做下记录。

# 自动化

自动化工具也有很多，这里主要介绍`fastlane`。

![alt text](/img/Fastlane/fastlane_text.png)

`fastlane`是一个为iOS，Mac，Android开发人员做自动化任务的工具，比如生成截图、处理Provisioning Profiles，发布你的应用程序。准确的说，`fastlane`一个工具集合，整合了其它很多自动化工具。

你可以使用`lane`来定义你的处理步骤：

	lane :beta do
		increment_build_number
		cocoapods
		match
		testflight
		sh "./customScript.sh"
		slack
	end

然后你只需要运行`fastlane beta`就能为你的应用程序部署一个新的'beta'版本。

## 安装

	sudo gem install fastlane --verbose

确保你已经安装了最新版本的Xcode command line tools：

	xcode-select --install

如果你觉得fastlane启动缓慢，尝试运行：

	gem cleanup

**系统要求**：fastlane要求Mac OS X 或者 Linux使用Ruby2.0.0及以 上版本。

如果你想要查看已经使用了`fastlane`的项目，点击[ fastlane-examples ](https://github.com/fastlane/examples)，包括已经设置好`fastlane`的Wikipedia、Product Hunt、MindNode等项目。

## 快速开始

`fastlane`的设置助理会在你开始使用前创建所有必需的文件，使用从iTunes Connect或者Google Play上已经存在的App元数据。

* cd [your_project_folder]
* `fastlane init`
* 跟着设置助理来配置你的应用程序
* 使用额外的[ actions ](https://github.com/fastlane/fastlane/blob/master/fastlane/docs/Actions.md)来自定义`Fastfile`(基于Ruby)

更多细节，查看[ fastlane guide ](https://github.com/fastlane/fastlane/blob/master/fastlane/docs/Guide.md)。

## 可用命令

通常你会使用`fastlane`命令来触发独立的lane：

	fastlane [lane_name]

### 其它命令

* `fastlane actions`：列出所有可用的`fastlane`的actions
* `fastlane action [action_name]`：显示action的更多细节的描述
* `fastlane lanes`：列出所有可用的lane，附带描述
* `fastlane list`：列出所有可用的lane，没有描述
* `fastlane new_action`：为fastlane创建新的action

## `fastlane`工具链

除了`fastlane`的命令，你还可以连接`fastlane`的工具：

* [ `deliver` ](https://github.com/fastlane/fastlane/tree/master/deliver)：上传截图、数据、App到 App Store
* [ `supply` ](https://github.com/fastlane/fastlane/tree/master/supply)：将你的Android应用和数据上传到 Google Play
* [ `snapshot` ](https://github.com/fastlane/fastlane/tree/master/snapshot)：将你iOS应用在每一种设备上进行本地化自动截图
* [ `screengrab` ](https://github.com/fastlane/fastlane/tree/master/screengrab)：Android版`snapshot`，一样的功能
* [ `frameit` ](https://github.com/fastlane/fastlane/tree/master/frameit)：快速将你的截图放入到适合的设备框中
* [ `pem` ](https://github.com/fastlane/fastlane/tree/master/pem)：自动生成和更新你的推送通知证书
* [ `sigh` ](https://github.com/fastlane/fastlane/tree/master/sigh)：管理你的 provisioning profiles
* [ `produce` ](https://github.com/fastlane/fastlane/tree/master/produce)：使用命令行工具在 iTunes Connect 和 Dev Portal 上创建新的iOS应用
* [ `cert` ](https://github.com/fastlane/fastlane/tree/master/cert)：自动创建和维护iOS的 code signing certificates
* [ `spaceship` ](https://github.com/fastlane/fastlane/tree/master/spaceship)：一个 Ruby library，自动连接 Apple Dev Center 和 iTunes Connect
* [ `pilot` ](https://github.com/fastlane/fastlane/tree/master/pilot)：管理你TestFlight测试人员的最好方式，使用终端构建
* [ `boarding` ](https://github.com/fastlane/boarding)：邀请你 TestFlight 的beta测试人员的最简单方式
* [ `gym` ](https://github.com/fastlane/fastlane/tree/master/gym)：构建你iOS应用程序
* [ `match` ](https://github.com/fastlane/fastlane/tree/master/match)：使用Git在你的团队中同步你的 certificates 和 profiles 
* [ `scan` ](https://github.com/fastlane/fastlane/tree/master/scan)：为你iOS和Mac应用运行测试的最简单方法
