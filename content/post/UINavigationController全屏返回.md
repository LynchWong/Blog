---
categories:
- ios
coverImage: //d1u9biwaxjngwg.cloudfront.net/welcome-to-tranquilpeak/city.jpg
date: 2017-01-15T13:06:21+08:00
keywords:
- uinavigationcontroller
- full
- back
metaAlignment: center
autoThumbnailImage: no
showTags: true
tags:
- UINavigationController
- Swift
title: UINavigationController全屏返回
---

UINavigationController 全屏返回的意思就是导航栏跟着一起动画返回，而不是系统默认的那种效果：导航栏没有动画，只有按钮有些动画效果。

<!--more-->

<!-- toc -->

# 效果对比 #

这里我就不贴 gif 图片来进行对比了，找个使用系统默认 UINavigationController 的 App 来 Push 或者 Pop 一下；然后对比使用下 iPhone 版的 网易云音乐 就知道是什么效果了。使用边界返回手势，效果更明显。

这两种效果哪种更好，全凭个人喜欢，当我更偏向全屏返回。

# 实现思路 #

在网上搜索过不少资料，这里贴两个链接：

* [https://gold.xitu.io/entry/57fcd57a67f356005886e867](https://gold.xitu.io/entry/57fcd57a67f356005886e867)
* [http://jerrytian.com/2016/01/07/用Reveal分析网易云音乐的导航控制器切换效果/](http://jerrytian.com/2016/01/07/用Reveal分析网易云音乐的导航控制器切换效果/)

第一种思路是用专场动画实现的，效果挺不错。但是貌似不完整，只有 Pop 的时候才有全屏的效果，Push 的时候是没有的。
第二种思路就是第二篇博文博主逆向网易云音乐提供的思路。

当然还有其它的思路，第二篇博文里面也详细介绍了其它思路，以及各种思路的优缺点。最终我选择的是网易云音乐类似的思路，使用 Swift 实现，在自己的项目里面使用，还没出现什么问题。

## 详解 ##

首先我们有一个 UINavigationController，然后它的导航栏是隐藏的。然后我们 Push 的是一个 Controller 容器，这个容器的子 Controller 就是一个 UINavigationController，而这个 UINavigationController 的 rootController 就是我们真正要显示的内容 Controller。

就相当于我们每个内容 Controller 的 UINavigationController 都是独立的，所以返回的时候自然就是全屏了。UINavigationController 应该是不能包含 UINavigationController 的，具体行不行没试过，应该没人这么干吧。所以得用容器包一下，只是这样一来，层级就很臃肿了。在我的项目里看来，效果、性能都还行吧

# 代码 #

[GitHub](https://github.com/LynchWong/FullBack)

代码其实并不多，也很简单，思路跟第二篇博文一样，可以参考。这里不一一贴出来讲解，就说个大概。

## GlobalNavigationController ##

最外层的 UINavigationController，已经隐藏了导航栏。代码量很少，没有什么好讲的。主要遇到一个返回手势的坑 **interactivePopGestureRecognizer is freezing app** ，后来在 StackOverflow 上找到了答案。搜前面加粗的关键字就能搜到，还好遇到这问题的人很多。其它的就没什么好说了，就是一些基本设置，状态栏样式，自动旋转之类的。

## WarpperController ##

这个就是前面提到的容器 Controller，它的作用很简单，就是把我们内容 Controller 的 UINavigationController 包装成 Child。所以我们 GlobalNavigationController Push 和 Pop 的都是 WarpperController。注意这里只提供了代码初始化。

## BaseNavigationController ##

所有内容 Controller 的 UINavigationController，BaseNavigationController 继承自 UINavigationController，主要重写了 Push 和 Pop 的一些方法。

比如在 Push 的时候，我们先给内容 Controller 添加返回按钮；然后初始化一个新的 BaseNavigationController，将内容 Controller 设置成 BaseNavigationControll 的 rootController；之后再将 BaseNavigationControll 包装成 WarpperController 的 Child；最后再 Push WarpperController。

{{< alert warning >}}
注意有些 Controller 是不支持SB或者XIB的；其次只实现了基本的功能，比如你直接设置 GlobalNavigationController 的 viewControllers 属性，可能全屏返回的效果就没了；或者使用 UINavigationController 替代了 BaseNavigationController，那么也会没有效果。所以在做有些操作前请思考下，以免效果跟你所想的不一样。最好还是看懂代码，明白每一行代码的作用，让代码的行为在你的控制之下。其实理解代码最重要的一点就是理清 Controller 的层级结构。
{{< /alert >}}

