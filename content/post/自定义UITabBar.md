---
title: "自定义UITabBar"
date: 2015-11-09 15:48:59
categories: 
- iOS
tags: 
- iOS自定义控件
metaAlignment: center
autoThumbnailImage: no
coverImage: //d1u9biwaxjngwg.cloudfront.net/welcome-to-tranquilpeak/city.jpg

---

先给出两个参考资料：

[ iOS 自定义TabBarController ](http://blog.csdn.net/gf771115/article/details/37890663)
[ 学习笔记：UITabBarController使用详解 ](http://www.cnblogs.com/martin1009/archive/2012/05/30/2526401.html)
<!--more-->

先说说思路，和第一个参考资料的思路基本类似。我们自己做一个类似**UITabBar**的视图，然后我们新建一个继承自**UITabBarController**的控制器，然后将我们自己的视图添加到系统的**UITabBar**。这样做的好处前面的参考资料里面也提到了，省去了隐藏TabBar时的许多麻烦。

这里就不手把手的一步一步的讲解怎么做了，我描述一下大概的步骤，提示一些细节。最后我会上传一个完整的Demo工程，大家参考工程源码就能明白了。

# 自定义视图 #

自定义视图很简单，不管你是用代码还是用Xib或者StoryBoard都可以。我工程中使用的是Xib，而且我做的这个视图并不具有通用性，完全参照UI设计的来做的。工程里面的代码没有什么难点的地方，约束也都很简单。这里我们需要一个委托来选择点击的按钮是对应的控制器。

# 添加视图 #

我们新建一个继承自**UITabBarController**的控制器，然后在**viewWillAppear:**方法里面添加如下代码：

    //去掉那条线
    self.tabBar.shadowImage = [UIImage imageNamed:@"TransparentPixel"];
    [self.tabBar setBackgroundImage:[UIImage imageNamed:@"Pixel"]];
    
    //去掉Tabbar原来的子视图
    CGRect rect = self.tabBar.bounds;
    for (UIView *view in self.tabBar.subviews) {
        [view removeFromSuperview];
        NSLog(@"removeFromSuperview : %@", [[view class] description]);
    }
    
    //添加我们自己的Tabbar
    YKTabBar *tabbar = [[[NSBundle mainBundle] loadNibNamed:@"YKTabBar" owner:nil options:nil] firstObject];
    tabbar.frame = rect;
    tabbar.delegate = self;
    [self.tabBar addSubview:tabbar];
    
    _preSelectedButton = tabbar.firstButton;
    _preSelectedButton.enabled = NO;


代码中我们循环了系统的UITabBar，然后我们移除了所有的子视图。最开始的时候我是在**viewDidLoad**方法里面做这些操作的，在循环中打印Log，什么都没有输出。

最开始的两行代码很有用，用来去掉UITabBar上面那条线的。如下图所示：

![alt text](/img/SelfDefineUITabBar/1.png)

左边的和右边的数字对应。图中1的UIVIew是我自己添加到视图中的，而2就是我们要去掉的那条线。我循环去掉UITabBar的子视图也没能把这个UIImageView去掉。这个方法是参考苹果官方Demo来实现的，[ Customizing UINavigationBar ](https://developer.apple.com/library/ios/samplecode/NavBar/Introduction/Intro.html)。

然后我们将我们自己的视图添加到系统的UITabBar，最后设置第一个按钮的状态。

# 实现委托 #

最后一步就是实现委托的方法，在委托方法中设置**selectedIndex**的值为对应按钮的Tag值就好了。

效果图如下：

![alt text](/img/SelfDefineUITabBar/2.png)

上传的源码最后的效果图和我这个不一样，图片资源就不提供了。

[ 工程源码 ](https://github.com/LynchWong/UITabbarDemo)
