---
title: Fastlane - Spaceship
date: 2016-06-28 09:03:31
categories: 
- iOS
tags: 
- Fastlane
- Automate
- Spaceship
metaAlignment: center
autoThumbnailImage: no
coverImage: /img/FastlaneSpaceship/spaceship.png

---

> 说明：翻译的 `Spaceship` 的指南，[ 原文地址 ](https://github.com/fastlane/fastlane/tree/master/spaceship)。
<!--more-->

![alt text](/img/FastlaneSpaceship/spaceship.png)

# spaceship #

`spaceship` 暴露了 Apple Developer Center 和 iTunes Connect API。这些快速和强大的API已经是 fastlane 的一部分，能够实现更多 fastlane 的高级功能。脚本化 Developer Center 的工作流从未如此简单。

# spaceship 是什么东西？ #

到现在为止， [fastlane tools](https://fastlane.tools) 都是使用的网页抓取的方式与 Apple's Web Services 进行交互。使用 spaceship 只需要一个简单的 HTTP Client 就能直接访问底层的 API。

使用 spaceship， [sigh](https://github.com/fastlane/fastlane/tree/master/sigh) 的执行时间从1分钟减少到5秒内。

spaceship 使用3中不同的API端点的组合，被 Apple Developer Portal 和 Xcode 使用。由于没有为我们提供所有需要的API，所以 spaceship 为你组合了所有的API。 [关于API的更多细节](More details about the APIs)。

更多关于为什么 spaceship 有用的说明，查看 [spaceship.airforce](https://spaceship.airforce)。

> 不管你有多少 App 或者 Certificates，spaceship 都能处理。

够了，这里是代码：

	Spaceship.login

    # Create a new app
	app = Spaceship.app.create!(bundle_id: "com.krausefx.app", name: "Spaceship App")

    # Use an existing certificate
	cert = Spaceship.certificate.production.all.first

    # Create a new provisioning profile
	profile = Spaceship.provisioning_profile.app_store.create!(bundle_id: app.bundle_id,
                                                         certificate: cert)

    # Print the name and download the new profile
	puts "Created Profile " + profile.name
	profile.download

## 速度 ##

跟网页抓取比起来，使用 `spaceship` 工具有多快？

![alt text](/img/FastlaneSpaceship/SpaceshipRecording.gif)

# 安装 #

	sudo gem install spaceship

# 使用 #

## Playground ##

尝试 `spaceship` ，只需要运行 `spaceship` 。然后就会自动启动 `spaceship playground`。

![alt text](/img/FastlaneSpaceship/Playground.png)

这要求你使用 `sudo gem install pry` 安装 `pry` 。`pry` 不会默认安装，因为大部分的 [fastlane](https://fastlane.tools) 用户不需要 `spaceship playground` 。你可以在 `Gemfile` 文件里添加 `pry` 依赖。

## Apple Developer Portal API ##

[DeveloperPortal.md](https://github.com/fastlane/fastlane/blob/master/spaceship/docs/DeveloperPortal.md) 中有示例代码

## iTunes Connect API ##

[iTunesConnect.md](https://github.com/fastlane/fastlane/blob/master/spaceship/docs/iTunesConnect.md) 中有示例代码

## 2 Step Verification ##

如果你的苹果账户开启了两步验证，将会自动询问你使用手机进行验证。结果会话将会存储在 `~/.spaceship/[email]/cookie` 。这个会话在一个月内是有效的，当然没有办法真的等到一个月后来测试。

由于你的 CI 系统可能不允许你输入值(比如验证码)，你可以使用 `spaceauth` ：

	spaceauth -u apple@krausefx.com

这个将会给你授权，并提供一个字符串，可以传给你的 CI 系统：

	export FASTLANE_SESSION='---\n- !ruby/object:HTTP::Cookie\n  name: DES5c148586dfd451e55afbaaa5f62418f91\n  value: HSARMTKNSRVTWFla1+yO4gVPowH17VaaaxPFnUdMUegQZxqy1Ie1c2v6bM1vSOzIbuOmrl/FNenlScsd/NbF7/Lw4cpnL15jsyg0TOJwP32tC/NguPiyOaaaU+jrj4tf4uKdIywVaaaFSRVT\n  domain: idmsa.apple.com\n  for_domain: true\n  path: "/"\n  secure: true\n  httponly: true\n  expires: 2016-04-27 23:55:56.000000000 Z\n  max_age: \n  created_at: 2016-03-28 16:55:57.032086000 -07:00\n  accessed_at: 2016-03-28 19:11:17.828141000 -07:00\n'

从 `---\n` 开始复制所有的东西到你的 CI 服务器，然后当做环境变量提供，变量名是 `FASTLANE_SESSION` 。

## Spaceship in use ##

很多 [fastlane tools](https://fastlane.tools) 已经使用了 `spaceship`，比如 `sigh`， `cert`， `produce`， `pilot` 和 `boarding` 。

# 技术细节 #

## HTTP Client ##

到现在为止，所有的 [fastlane tools](https://fastlane.tools) 都是使用的网页抓取的方式与 Apple's Web Services 进行交互。spaceship 使用一个简单的 HTTP Client ，更少的开销，更快的速度。

跟网页抓取比起来，`spaceship` 的优势：

* 快90%
* 不需要再加载图像、HTML、JS 以及 CSS文件
* 通过稳定的服务器响应，更广的测试覆盖率
* 能够应对 Apple Developer Portal 的设计变更
* 当请求超时，会自动重试

## API Endpoints ##

这里不会深入讲解各种API，只是给一些参考：

* `https://idmsa.apple.com`：用来授权，获取有效的会话
* `https://developerservices2.apple.com`：
  * 获取可用的 Provisioning Profiles 的详细列表
  * 这个API会返回每个 Profiles 的设备，Certificates 和 App
  * 注册新的设备
* `https://developer.apple.com`：
  * 列出所有设备，Certificates，Apps 和 App Groups
  * 创建新的Certificates，Provisioning Profiles 和 Apps
  * 禁用/开启 应用的服务，然后赋值给 App Groups
  * 删除Certificates 和 Apps
  * 修复 Provisioning Profiles
  * 下载 Provisioning Profiles
  * 团队选择
* `https://itunesconnect.apple.com`：
  * 管理应用
  * 管理测试用户
  * 提交更新进行审核
  * 管理应用的元数据
* `https://du-itc.itunesconnect.apple.com`：
  * 上传 图标、截图、trailers...

`spaceship` 使用所有这些API提供了所有功能的无缝体验。

## Magic involved ##

`spaceship` 做了许多魔法的事情来让所有的工作都很整洁：

* **合理的默认值：**你只需要提供那些强制性的信息(比如新的 Provisioning Profiles，默认包含所有设备)
* **本地验证：**将所有的改变推送到 Apple Dev Portal 的时候，`spaceship` 会确保只有有效的数据发送给 Apple (比如自动修复了的 Provisioning Profiles)
* **各种请求响应类型：**当处理不同的API的时候，`spaceship` 必需处理 `JSON`，`XML`，`txt`，`plist`，有时甚至包括 HTML 的请求和响应
* **自动分页：**即使你有上千个App、Profiles 或者 Certificates，`spaceship` 都会为你处理。是经过测试的。
* **Session, Cookie and CSRF token：**`spaceship` 处理了安全的各个方面。
* **Profile Magic：**创建上传 Code Signing Requests，所有的都由`spaceship` 处理
* **Multiple Spaceship：**你可以使用不同的苹果账户来登录多个 `spaceship`，用来同步注册的设备
