---
title: Fastlane - Guide
date: 2016-06-11 19:42:14
categories: 
- iOS
tags: 
- Fastlane
- Automate
metaAlignment: center
autoThumbnailImage: no
coverImage: //d1u9biwaxjngwg.cloudfront.net/welcome-to-tranquilpeak/city.jpg

---

> 说明：翻译的 `Fastlane` 的指南，[ 原文地址 ](https://github.com/fastlane/fastlane/blob/master/fastlane/docs/Guide.md)。
<!--more-->

## 安装

要求：

* Mac OS 10.9 或者更新
* Ruby 2.0 或者更新(ruby -v)
* Xcode

除了安装Xcode之外，还需要安装Xcode command line tools：

	xcode-select --install

如果你之前没有使用过command line tools，你需要接受服务条款：

	sudo xcodebuild -license accept

### fastlane

安装gem和所有依赖(可能要花几分钟)：

	sudo gem install fastlane --verbose

## 设置`fastlane`

在修改你项目之前，建议你在当前的目录中已经使用Git提交过了。

当你运行设置命令时，请阅读终端中的说明，它们出现在那里通常是有原因的。

使用如下命令时`fastlane`将会为你创建所有必需的文件和文件夹。将会使用从项目工作目录检测到的变量。

	fastlane init

将会提示你需要你的Apple ID，为了验证已经存在在iTunes Connect和Apple Developer Portal上的应用程序。如果没有，`fastlane`会问你是否需要自动创建。

然后应该会接收安装成功的信息。

这些设置会做什么工作？

* 创建一个`fastlane`文件夹。
* 将已经存在的 deliver 和 snapshot 的配置文件移动到`fastlane`文件夹(如果存在)。
* 创建`fastlane/Appfile`，保存你的Apple ID 和 Bundle Identifier。
* 创建`fastlane/Fastfile`，保存你的lanes。

这个设置会自动检测你使用的什么工具(比如deliver，CocoaPods等等)。

### 独立工具

在运行`fastlane`命令前，确保所有的工具都被正确设置了。

比如，尝试运行如下工具(取决于你的计划)：

* deliver
* snapshot
* sigh

所有这些工具都有如何设置它们的详细说明，很容易设置。你可以现在或者之后再设置。

### 配置`Fastfile`

首先，思考你到底需要构建什么。一些想法：

* 新的App Store发布版本
* 为 TestFlight 或者[ HockeyApp ](https://www.hockeyapp.net/features/)
* 运行测试
* InHouse发布

使用你喜欢的文本编辑器打开`fastlane/Fastfile`，然后将语法高亮改为Ruby。

取决于你已经存在的一些设置，看起来类似如下：

    before_all do
      # increment_build_number
      cocoapods
    end

    lane :test do
      snapshot
    end

    lane :beta do
      sigh
      gym
      pilot
      # sh "your_script.sh"
    end

    lane :deploy do
      snapshot
      sigh
      gym
      deliver(force: true)
      # frameit
    end

    after_all do |lane|
      # This block is called, only if the executed lane was successful
    end

    error do |lane, exception|
      # Something bad happened
    end

你可以将如下文本复制粘贴到你的`fastlane/Fastfile`文件，然后使用`fastlane`命令运行：

    lane :example do
      say "It works"
    end

然后在你的终端中运行：

	fastlane example

如果成功，你应该听见你的电脑对你说话！

可用的actions列表，参见[ Actions documentation ](https://github.com/fastlane/fastlane/blob/master/fastlane/docs/Actions.md)。

接下来，自动化部署流程是很重大的一步。你应该使用`increment_build_number`action，当你想上传构建的包到iTunes Connect时([ Activate incrementing build numbers ](https://developer.apple.com/library/ios/qa/qa1827/_index.html))。

### 使用你已经存在的构建脚本

	sh "./script.sh"

这会执行你已经存在的构建脚本。所有在`"`里的都会在shell里执行。

### 创建你自己的actions(构建脚本)

如果你想使用花哨的命令(就像snapshot拥有的那样)，你可以使用[ fastlane new_action ](https://github.com/fastlane/fastlane/blob/master/fastlane/docs/README.md#extensions)很简单的构建你自己的拓展。

## 示例项目

参见[ Wikipedia ](https://github.com/fastlane/examples#wikipedia-by-wikimedia-foundation)，[ Product Hunt ](https://github.com/fastlane/examples#product-hunt)，[ MindNode ](https://github.com/fastlane/examples#mindnode)如何使用`fastlane`来自动化他们iOS应用的处理流程。

参见[ Actions documentation ](https://github.com/fastlane/fastlane/blob/master/fastlane/docs/Actions.md)查看可用的集成和选项。

## Help

如果还有什么不清楚或者你需要帮助，开个[ issue ](https://github.com/fastlane/fastlane/issues/new)。
