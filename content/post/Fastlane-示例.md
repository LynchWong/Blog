---
title: Fastlane - 示例
date: 2016-07-07 23:08:32
categories: 
- iOS
tags: 
- Fastlane
- Automate
metaAlignment: center
autoThumbnailImage: no
coverImage: //d1u9biwaxjngwg.cloudfront.net/welcome-to-tranquilpeak/city.jpg

---

**说明：**之前把关于 `Fastlane` 的工具集的文档简单翻译了下，翻译的比较糙。由于最近太忙了，没时间回去校对。之后有时间了也不会回去校对，so be it。
<!--more-->

既然文档什么的都整完了，是时候写自己的 `lane` 了。最近花了点时间研究了下各个工具，还是挺方便好用的。这里我把我自己的配置过程、以及一些自己的体会记录下来。

首先说明下我的配置满足的需求：

* 管理团队的证书，包括开发、发布、AdHoc
* 能够上传App到 ITC(iTunes Connect) 进行审核
* 能够使用 InHouse 证书打包，然后上传服务器

Fastlane 是个工具集，里面所有的工具都可以单独使用。

# **Matchfile**文件： #

	git_url "https://LynchWong@bitbucket.org/LynchWong/eplayermatch.git"

	type "development" # The default type, can be: appstore, adhoc or development

	app_identifier "com.lynch.ePlayer" 
	username "lynch.wong@me.com" # Your Apple Developer Portal username

    # For all available options run `match --help`
    # Remove the # in the beginning of the line to enable the other options

首先我直接进入到项目文件夹直接运行 `fastlane init` 进行初始化，具体做了些什么工作参见 [这里](https://github.com/fastlane/fastlane/blob/master/fastlane/docs/Guide.md) 或者 [这里](http://lynchwong.com/2016/06/11/Fastlane-Guide/)。

完成之后运行 `match init` 来配置 `Match`。这一步你需要一个私有的 Repository 。之后我运行了 `match development`、`match adhoc`、`match appstore` 创建证书和配置文件。完成之后在 `fastlane` 文件夹里面生成了 `Matchfile` 文件，如上面所示，这里我做了一些小修改。

关于 `Match` 的说明，参考 [这里](https://github.com/fastlane/fastlane/tree/master/match) 或者 [这里](http://lynchwong.com/2016/06/28/Fastlane-Match/)。貌似 `Match` 这里不对企业帐号的证书进行管理，我这里没有尝试。我用企业证书打包时是使用的 `cert` 和 `sigh`。

# **Appfile**文件 #

	app_identifier "com.lynch.ePlayer" # The bundle identifier of your app
	apple_id "lynch.wong@me.com" # Your Apple email address
	team_id ""  # Developer Portal Team ID

	for_platform :ios do
		team_id ''
		for_lane :ent do
			app_identifier 'com.lynch.ePlayerEnt'
			apple_id '企业帐号'
		end
	end

    # you can even provide different app identifiers, Apple IDs and team names per lane:
    # More information: https://github.com/fastlane/fastlane/blob/master/fastlane/docs/Appfile.md

关于 `Appfile` 文件可以参考 [这里](https://github.com/fastlane/fastlane/blob/master/fastlane/docs/Appfile.md) 。

这里我对lane `ent` 使用不同的 app_identifier 和 apple_id 。我这里的lane `ent` 就是使用 InHouse 证书进行打包。

# **Fastfile**文件 #

		# coding: utf-8
		# Customise this file, documentation can be found here:
        # https://github.com/fastlane/fastlane/tree/master/fastlane/docs
        # All available actions: https://github.com/fastlane/fastlane/blob/master/fastlane/docs/Actions.md
        # can also be listed using the `fastlane actions` command

        # Change the syntax highlighting to Ruby
        # All lines starting with a # are ignored when running `fastlane`

        # If you want to automatically update fastlane if a new version is available:
        # update_fastlane

        # This is the minimum version number required.
        # Update this, if you use features of a newer version
        fastlane_version "1.97.2"

        default_platform :ios

        platform :ios do
			before_all do
				# ENV["SLACK_URL"] = "https://hooks.slack.com/services/..."
				# cocoapods
    
		    end

		    desc "安装 FixCode 插件，禁用 Xcode 中的 Fix Issue 按钮。"
			lane :xcode do
				install_xcode_plugin(
					url: "https://github.com/fastlane/FixCode/releases/download/0.2.0/FixCode.xcplugin.zip"
				)
			end

		    desc "使用 InHouse 证书打包。"
			lane :ent do
				produce(
					skip_itc: true
				)
				cert
				sigh(
					force: false
				)
				gym(
					workspace: "ePlayer.xcworkspace",
					configuration: "Release",
					scheme: "ePlayer",
					silent: true,
					clean: false,
					output_directory: "IPA",
					output_name: "ePlayerEnt.ipa",
					use_legacy_build_api: true,
				)
			end

		    desc "证书管理，不能管理企业帐号的证书。"
			lane :iosmatch do
				match(app_identifier: "com.lynch.ePlayer", type: "development", readonly: true)
				match(app_identifier: "com.lynch.ePlayer", type: "adhoc", readonly: true)
				match(app_identifier: "com.lynch.ePlayer", type: "appstore", readonly: true)
			end

		    desc "Runs all the tests"
			lane :test do
				scan
			end

		    desc "Submit a new Beta Build to Apple TestFlight"
			desc "This will also make sure the profile is up to date"
			lane :beta do
				# match(app_identifier: "com.lynch.ePlayer", type: "appstore", readonly: true) # more information: https://codesigning.guide
				gym(
					workspace: "ePlayer.xcworkspace",
					configuration: "Release",
					scheme: "ePlayer",
					silent: true,
					clean: false,
					output_directory: "IPA",
					output_name: "ePlayerTestFlight.ipa",
					use_legacy_build_api: true,
				) # Build your app - more options available
				pilot

		        # sh "your_script.sh"
				# You can also use other beta testing services here (run `fastlane actions`)
			end

		    desc "Deploy a new version to the App Store"
			lane :appstore do
				# match(type: "appstore")
				# snapshot
				gym(scheme: "ePlayer") # Build your app - more options available
				deliver(force: true)
				# frameit
			end

		    # You can define as many lanes as you want

		    after_all do |lane|
				# This block is called, only if the executed lane was successful

		        # slack(
				#   message: "Successfully deployed new App Update."
				# )
			end

		    error do |lane, exception|
				# slack(
				#   message: exception.message,
				#   success: false
				# )
			end
		end


        # More information about multiple platforms in fastlane: https://github.com/fastlane/fastlane/blob/master/fastlane/docs/Platforms.md
        # All available actions: https://github.com/fastlane/fastlane/blob/master/fastlane/docs/Actions.md

        # fastlane reports which actions are used
        # No personal data is recorded. Learn more at https://github.com/fastlane/enhancer

## lane :ent ##

使用 InHouse 证书来打包。首先使用 produce，设置 skip_itc: true 不会在 ITC 上进行创建App，因为是企业帐号。然后使用 `cert` 和 `sigh` 进行签名创建证书(不使用 `Match` 参考前面)。然后就是一些打包的设置。

## lane :xcode ##

安装 Xcode 的创建，主要是禁止使用 Xcode 来管理证书。至于为什么，参考 [这里](https://github.com/fastlane/fastlane/tree/master/fastlane/docs/Codesigning)，不再赘述。

## lane :iosmatch ##

主要用来获取安装相关证书，这里设置了 readonly: true，只会获取已经存在的签名和证书，而不会去生成一个新的。

另外几个 lane 基本就是生成**Fastfile**文件默认配置的。没什么好说的。我尝试运行了 `beta` lane，主要就是多了 pilot，将打包好的上传到 TestFlight 进行测试。然而我发现 pilot 的几个命令也都是针对外部测试的，但是外部测试也是需要提交审核的，这个时间要多久我不知道。而内部测试也只有25个名额，每个名额10台设备。所以我觉得这个 pilot 没什么用啊，除此之外，另外一个工具 Boarding 进行配置的时候 Heroku 一直报错。所以我觉得还不如使用企业帐号打包，然后运行脚本(脚本这里就不提供了)，将IPA放到服务器提供下载安装。小团队还不如打包 AdHoc，不过使用 pilot 进行内部测试也很好。

`appstore` lane 没有尝试，留到后面发版的时候使用。

# 总结 #

[这里还有很多文档可以参考](https://github.com/fastlane/fastlane/blob/master/fastlane/docs/README.md)，这些文档没有翻译。每个工具都可以使用 `--help` 命令来查看帮助，很方便，看看就懂了。另外一点要提的就是你项目的 `.gitignore`，可以参考 [这里](https://github.com/fastlane/fastlane/blob/master/fastlane/docs/Gitignore.md)。比如我打包的IPA是在项目文件夹里面的，所以我的 `.gitignore` 还会忽略 `IPA` 这个文件夹。

翻译太花时间了，以后应该不会翻译了，以上。
