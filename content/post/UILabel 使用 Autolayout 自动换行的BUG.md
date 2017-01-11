---
title: "UILabel 使用 Autolayout 自动换行的BUG"
date: 2016-08-26 10:29:18
archives:
- A
categories: 
- iOS
tags: 
- BUG
metaAlignment: center
autoThumbnailImage: no
coverImage: /img/2521-8.jpg

---

之前有一篇 Blog 写的就是关于 UILabel 自动换行的内容，只不过不是通过代码计算，而是使用 xib、storyboard、AutoLayout 来自动换行。
<!--more-->

最近貌似产生了一个BUG，最近做界面的时候按照这种方式自动换行的时候有些瑕疵，而之前的界面没有产生这个BUG。

![alt text](/img/UILabelBUG/1.png)

BUG已经在图中标注出来了，可以明显看到 UILabel 的那个高度跟我们期望的不一样。后来在排查BUG的时候，我用代码计算的 **"VOA常速新闻:特朗普不担心缺少党内支持"** 的高度差不多是25.0左右。但是我用 Xcode 的 View UI Hierarchy 调试的时候高度是差不多50.0左右。所以可以断定的是 UILabel 进行了换行，但是我们是不需要换行的。另外，**"永不退缩 Won't Back Down (2012)"**也没有显示完全。

排查 BUG 的过程还是比较蛋疼的，以为是图片影响了、或者是 Cell 重用影响了，代码逻辑看了好几遍都觉得没错。所以后来就 Google 了 "UILabel Xib Storyboard 自动换行"，找到了如下资料：

* [ Autolayout下UILabel的自动换行实现 ](http://blog.csdn.net/lihogjun/article/details/30365269)
* [ UILABEL AUTOLAYOUT自动换行 版本区别 ](http://www.cnblogs.com/allen123/p/4521746.html)

所及基本上我就觉得是 **preferredMaxLayoutWidth** 这个属性的问题，应该设置成跟 UILabel 的宽度一样。

从图中可以看出我的 UILabel 的宽度是动态变化的，所以其实这个值应该也是变化的。但是我看我的 storyboard 里面设置的是 290.0，且是不变的。

![alt text](/img/UILabelBUG/2.png)

后来验证确实是这个的问题，其实界面是按照 6S 为基准做的，所以宽度是 375.0 减去左右边距和 30.0，应该是 345.0。所以这个 290.0 其实是不对的，以 6S 为标准应该是 345.0。

![alt text](/img/UILabelBUG/3.png)

所以按照这样的设置，在 6S 上没有图片的Cell显示应该是没问题的。截图如下：

![alt text](/img/UILabelBUG/4.png)

但是箭头指的那里还是没有显示完全，后来设置成零就没有问题了，就没有深究了。截图如下：

![alt text](/img/UILabelBUG/5.png)

这样看来就正常了。

以上。
