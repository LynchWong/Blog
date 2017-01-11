---
title: Fastlane - Match
date: 2016-06-28 09:05:34
categories: 
- iOS
tags: 
- Fastlane
- Automate
- Match
metaAlignment: center
autoThumbnailImage: no
coverImage: /img/FastlaneMatch/match.png

---

> 说明：翻译的 `Match` 的指南，[ 原文地址 ](https://github.com/fastlane/fastlane/tree/master/match)。
<!--more-->

![alt text](/img/FastlaneMatch/match.png)

# match #

在你的团队之间使用 Git 轻松的同步你的证书和配置文件。

iOS Code Signing 的新方法：在你的开发团队中共享一个 Code Signing identity，简化 CodeSigning 的设置工作，防止其它问题出现。

`match` 是 [https://codesigning.guide](https://codesigning.guide) 概念的实现。`match` 创建请求所有的证书和配置文件，然后存储在另外一个 Git Repository 中。团队中的每一个成员都可以访问这个 Repository，使用这些证书来 CodeSigning。`match` 也会自动修复损坏的，过期的证书。`match` 是在团队中共享签名证书的最简单方法。

# 为什么使用match？ #

在开始使用 `match` 前，确保已经阅读了 [codesigning.guide](https://codesigning.guide) 了。

> When deploying an app to the App Store, beta testing service or even installing it on a device, most development teams have separate code signing identities for every member. This results in dozens of profiles including a lot of duplicates.

> You have to manually renew and download the latest set of provisioning profiles every time you add a new device or a certificate expires. Additionally this requires spending a lot of time when setting up a new machine that will build your app.

新方法：

> Share one code signing identity across your development team to simplify your setup and prevent code signing issues. What if there was a central place where your code signing identity and profiles are kept, so anyone in the team can access them during the build process?

(这些我就不翻译了，就是说了一大堆团队成员之间证书管理混乱、各种revoke)

## 为什么不让 Xcode 处理这些？ ##

* 让要发生的事情都完全在你控制中
* 你可以访问所有的证书和配置文件，这些都安全的存储在 Git 中
* 在团队中共享一个 Code Signing identity，少量的证书和配置文件
* 有时 Xcode 会 revoke(废除) 证书，打乱你的设置，导致项目构建失败
* 比起 `自动` 设置，通过显式设置配置文件，构建项目时我们更能预测会发生什么
* It just works™(这只是工作？ Xcode 只是有点作用，少了其它方面的考量？)

## `match` 为你做了什么工作？ ##

|match|
|:---:|
|在团队成员中使用 Git 自动同步你的 iOS Keys 和 Profiles|
|处理所有繁重的事情，比如创建存储你的证书和配置文件|
|一分钟就能在新机器上设置好 Code Signing|
|能够与你的应用的多个的 Targets 和 Bundle identity 很好的工作|
|你能够完全控制 Git Repo，没有第三方的服务参与|
|Provisioning Profile 始终与正确的 Certificate 匹配|
|如果你当前账户过期了或者 Profiles 无效，很方便的重置你已经存在的 Profiles 和 Certificates|
|使用 `--force` 选项能够自动更新你的 Provisioning Profiles，会包含你所有的设备|
|支持多个苹果账户和多个团队|
|与 [fastlane](https://fastlane.tools) 紧密集成，与 [gym](https://github.com/fastlane/fastlane/tree/master/gym) 等其它工具能够无缝衔接|

更多关于概念上的信息，访问 [codesigning.guide](https://codesigning.guide)。

# 安装 #

	sudo gem install match

确保你已经安装了 Xcode 最新版本的命令行工具：

	xcode-select --install

# 使用 #

## 设置 ##

1. 创建一个**新的私有 Git Repo**(比如在 [GitHub](https://github.com/new) 或者 [BitBucket](https://bitbucket.org/account/signin/?next=/repo/create) 上)命名为诸如 `certificates` 这样的。**重要：**确保 Repository 设置为了私有的。
2. 可选：创建一个**新的，共享的 Apple Developer Portal 账户**，诸如 `office@company。com` ，现在开始团队共享这个帐号(更多信息访问 [codesigning.guide](https://codesigning.guide))。
3. 在项目文件夹中运行如下命令开始使用 `match` ：

		match init

![alt text](/img/FastlaneMatch/match_init.gif)

会要求你输入 Git Repo 的 URL。可以是 `https://` 或者 `git` URL。`match init` 不会读取或者修改你的证书或者配置文件。

会在你当前目录(或者在你的 `./fastlane/` 文件夹)创建 `Matchfile` 。

示例内容(更多高级设置，查看 [fastlane section](https://github.com/fastlane/fastlane/tree/master/match#fastlane))：

	git_url "https://github.com/fastlane/fastlane/tree/master/certificates"

	app_identifier "tools.fastlane.app"
	username "user@fastlane.tools"

### **重要：每个团队使用一个 Git 分支** ###

`match` 也支持在一个 Repo 中为多个团队存储证书，通过使用不同的分支。如果你在多个团队中工作，确保你为每个团队设置了唯一的 `git_branch` 参数值。`match` 会自动为创建使用指定的分支。

	match(git_branch: "team1", username: "user@team1.com")
	match(git_branch: "team2", username: "user@team2.com")

## 运行 ##

> Before running match for the first time, you should consider clearing your existing profiles and certificates using the [match nuke command](https://github.com/fastlane/fastlane/tree/master/match#nuke).

在运行 `match init` 之后，你可以运行如下的命令来生成新的证书和配置文件：

	match appstore
	match development

![alt text](/img/FastlaneMatch/match_appstore_small.gif)

这回创建新的证书和配置文件(如果需要)然后存储在你的 Git Repo 中。如果你之前运行 `match` ，它会自动安装 Git Repo 里已经存在的 Profiles 。

当 Certificates 和 Private keys 安装在 Keychain，Provisioning Profiles 安装在 `~/Library/MobileDevice/Provisioning Profiles` 。

使用如下命令获得更详细的输出：

	match --verbose

列出可用的选项：

	match --help

### 处理多个 Targets ###

如果你有不同的 Bundle identifiers，多个 Targets，为它们的每一个调用 `match` 。

	match appstore -a tools.fastlane.app
	match appstore -a tools.fastlane.app.watchkitapp

你可以使用 [fastlane](https://github.com/fastlane/fastlane/tree/master/fastlane) 让这过程更简单点：

	lane :match do
		match(app_identifier: "com.krausefx.app1", readonly: true)
		match(app_identifier: "com.krausefx.app2", readonly: true)
		match(app_identifier: "com.krausefx.app3", readonly: true)
	end

然后所有团队需要做的就是 `fastlane match`，然后为所有 targets 的 keys、certs 和 profiles 都会同步。

### 密码 ###

当在一台新机器上第一次运行 `match` ，会询问你 Git Repository 的密码。这是一个额外的安全措施：所有的文件都会被 OpenSSL 加密。请务必记住密码，当你在不同机器上运行 `match` 的时候你会需要。

使用环境变量来设置密码，使用 `MATCH_PASSWORD` 。

### 新机器 ###

在新机器上设置证书和配置文件，使用如下命令：

	match development

你也可以运行 `match` 的是使用 `readonly` 确保不会创建新的证书或者配置文件。

	match development --readonly

### 访问控制 ###

使用 `match` 的一个好处就是你允许团队成员可以访问 Code Signing Certificates，而不需要给每个成员访问 Developer Portal 的权限：

1. 运行 `match` 在 Git Repo 中存储证书
2. 给开发人员授予访问 Git Repo 的权限，并给他们密码
3. 开发人员现在可以运行 `match` 安装最新版本的 Code Signing Profile，所以他们可以直接构建签名应用程序，而不需要去访问 Developer Portal
4. 每次你运行 `match` 更新 Profiles(比如 添加新设备)，所有的开发人员在运行 `match` 时都会自动获取最新版本的 Profiles

如果你决定运行 `match` 的时候不访问 Developer Portal，使用 `--readonly` 选项，就不会询问你 Developer Portal 的密码。

这个方法的优势就是团队中的人不会因为手误废除(revoke)了证书。另外推荐安装 [FixCode Xcode Plugin](https://github.com/neonichu/FixCode) 禁用 `Fix Issue` 按钮。

### Git Repo ###

在第一次运行 `match` 之后，你的 Git Repo 会包含两个目录：

* `certs` 文件夹包含所有的 Certificates 和它们的 Private Keys
* `profiles` 文件夹包含所有的 Provisioning Profiles

除此之外，`match` 会为你创建一个漂亮的 Repo `README.md`，方便新的团队成员登船(入坑，哈哈，开玩笑)：

![alt text](/img/FastlaneMatch/github_repo.png)

### fastlane ###

添加 `match` 到你的 `Fastfile` ，使用 [fastlane](https://fastlane.tools) 自动获取最新版本的 Code Signing Certificates 。

	match(type: "appstore")
	
	match(git_url: "https://github.com/fastlane/fastlane/tree/master/certificates",
      type: "development")

	match(git_url: "https://github.com/fastlane/fastlane/tree/master/certificates",
      type: "adhoc",
      app_identifier: "tools.fastlane.app")

	# `match` should be called before building the app with `gym`
	gym
	...

### 多个 Targets ###

如果你的应用程序有多个 Targets(比如 Today Widget or WatchOS Extension)

	match(app_identifier: "tools.fastlane.app", type: "appstore")
	match(app_identifier: "tools.fastlane.app.today_widget", type: "appstore")

`match` 甚至可以为所有的 Bundle identifiers 使用同一个 Git Repository 。

## 设置 Xcode 项目 ##

为了确保 Xcode 为每一个 target 使用了正确的 Provisioning Profile，请不要为选择 Profile 的功能使用 `Automatic` 。

另外，推荐使用 [FixCode Xcode Plugin](https://github.com/neonichu/FixCode) 禁用 `Fix Issue`按钮。`Fix Issue` 按钮可以废除(revoke)你已经存在 Certificates，会使你的 Provisioning Profiles 无效。

### 使用 [fastlane](https://fastlane.tools) 从命令行构建 ###

`match` 会使用正确的 Provisioning Profiles 的 UUID 来自动填充环境变量，准备在你的 Xcode 项目中使用。

![alt text](/img/FastlaneMatch/UDIDPrint.png)

打开你的 target 设置，打开 `Provisioning Profile` 选择 `Other` ：

![alt text](/img/FastlaneMatch/XcodeProjectSettings.png)

Profile 环境变量以 `$(sigh_<bundle_identifier>_<profile_type>)` 命名

比如 `$(sigh_tools.fastlane.app_development)`

### 从 Xcode 手动构建 ###

当你使用 Development Profile 安装应用程序到设备上的时候非常有用。

你可以在 Xcode 项目中静态的选择正确的 Provisioning Profile(名字与 `Development tools.fastlane.app` 匹配)。

## 持续集成 ##

### 访问 Repo ###

为了让 CI 系统和 `match` 工作，只需要让 CI 系统能够访问 Repo。通常你只需要将 CI 系统的 public ssh key 当做 deploy key 添加到你的 `match` Repo 中，但由于你的 CI 系统可能已经使用 public ssh key 来访问代码的 Repo，[那你就不能那么做了。](https://help.github.com/articles/error-key-already-in-use/)

有些 Repo Hosts 可能允许你为不同的 Repo 使用相同的 deploy 可以，但是 GitHub 不允许。如果你的 Host 也是，不需要担心，只需要将 CI 的 public ssh key 当做 deploy key 添加到你的 `match` Repo 中，然后滚动到"Encryption password"。

大约有这几个方面：

1. 在你的 Repo Host 上创建新的帐号，对于你的 `match` Repo 只有 read-only 的访问权限。[这里有描述。](http://devcenter.bitrise.io/docs/adding-projects-with-submodules)
2. 有些 CI 允许你手动上传 Signing Credientials，但是这就意味着 profiles/keys/certs 每次改变你都要重新上传。

这两种解决方法都不完美。两种方法需要你的权衡。看你是更在意设置额外的账户还是更在意证书的同步。

### 加密密码 ###

一旦你决定使用哪种方法，所有剩下要做的就是设置你加密的密码作为秘密的环境变量，命名为 `MATCH_PASSWORD` 。`match` 在运行的时候会带上这个参数。

## Nuke ##

如果你从来不再以 Code Signing，并且 Apple Developer 账户有大量无效的，过期的，Xcode管理的 profiles／certificates，你可以使用 `match nuke` 命令来废除你的 Certificates 和 Provisioning Profiles。不要担心，已经在 App Store 里可用的应用程序仍然会继续工作。在 `nuke` 你的账户之后，通过 TestFlight 构建的 distributed 就不能再用了，所以你必需重新上传新的构建。在清理了你的账户之后，你将会从干净的状态开始，然后你就可以运行 `match` 再次生成 Certificates 和 Profiles。

对指定环境的 Certificates 和 Provisioning Profiles 废除：

	match nuke development
	match nuke distribution

![alt text](/img/FastlaneMatch/match_nuke.gif)

在删除 profiles / certificates 之前你必需确认。

## 修改密码 ##

修改你 Repo 的密码，加密和解密所有文件运行：

	match change_password

在你所有机器上下一次运行的时候会被询问新的密码。

## 手动解密 ##

如果你想手动解密文件：

	openssl aes-256-cbc -k "<password>" -in "<fileYouWantToDecryptPath>" -out "<decryptedFilePath>" -a -d

# 这个安全吗？ #

你的 Keys 和 Provisioning Profiles 都使用了 OpenSSL 和 密码进行加密。

将 Private Keys 存储在 Git Repo 中第一次听起来倒胃口。我们进行了一次深层次的潜在安全问题的分析，然后得出如下结论：

## 如果某人偷取了 Private Key 会发生什么？ ##

如果攻击者有了你的 Certificate 和 Provisioning Profile，他们就可以使用跟你相同的 Bundle identifier 来签名应用。

对于每种类型的 Profile，最糟糕的情况是什么？

### App Store Profiles ###

只要不是苹果重新签名的 App Store Profile 就没有任何的用处。而获取重新签名的唯一方法就是提交应用进行审核(将会花7天的时候左右)。攻击者只能提交应用进行审核，如果他们也获得了访问你 iTunes Connect 的 凭证(credentials，帐号？密码？)(这个凭证没有存储在 Git 中，而是在你本地的 Keychain 中)。另外，每一次的构建上传、取消，甚至是你的应用进入审核阶段之前你都会收到通知邮件。

### Development and Ad Hoc Profiles ###

通常来说这些 Profiles 是无害的，因为它们签名的应用程序只能安装在小部分的设备上。为了新增设备，攻击者也会需要你的 Apple Developer Portal 的凭证(这个凭证没有存储在 Git 中，而是在你本地的 Keychain 中)。

### Enterprise Profiles ###

攻击者可以使用 In-House Profile 来签名应用程序然后分发给潜在的没有数量限制的设备。所有的这些都运行在你的公司名下，最终会导致苹果撤销你的 In-House 账户。然而可以很容易的废除(revoke)证书，然后远程的破坏所有设备上的这个 App 。

因为 In-House Profile 潜在的危险性，所以我们决定不允许 `match` 使用企业账户。

总结一下：

* 你可以完全控制 Git Repo，没有第三方服务的参与
* 即使你的 Certificates 泄漏了，他们没有你的 iTunes Connect 登录凭证也无法引起任何损害
* `match` 现在不支持 In-House Enterprise Profiles ，因为它们太难控制
* 如果你使用 GitHub 或者 Bitbucket 我们强烈鼓励你开启所有访问 Certificates Repo 账户的两部验证
* `match` 的所有的源码都开源在了 [Git Hub](https://github.com/fastlane/fastlane/tree/master/match) 上

# `fastlane` 工具链 #

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
* [ `supply` ](https://github.com/fastlane/fastlane/tree/master/supply)：将你的Android应用和数据上传到 Google Play
* [ `screengrab` ](https://github.com/fastlane/fastlane/tree/master/screengrab)：Android版`snapshot`，一样的功能

# 帮助 #

请提交 issue 到 GitHub，并提供你关于设置的信息。
