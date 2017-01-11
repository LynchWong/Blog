---
title: "UITabBarController和UINavigationController混用的一个BUG"
date: 2015-12-08 09:45:11
categories: 
- iOS
tags: 
- BUG
metaAlignment: center
autoThumbnailImage: no
coverImage: //d1u9biwaxjngwg.cloudfront.net/welcome-to-tranquilpeak/city.jpg

---

上篇博文讲了[ 自定义UITabBar ](http://lynchwong.com/2015/11/09/自定义UITabBar/)，在项目上用着挺好的，没出现什么问题，直到......
<!--more-->

我们先来描述一下这个BUG：

导致BUG的原因：首先我们使用自定义的UITabBarController作为整个App的rootViewController；然后每个Tab对应的都是一个UINavigationController的子类。

产生BUG的操作：有时候我在Tab对应的UINavigationController的子类做**popToRootViewControllerAnimated:**操作时，我们的自定义的UITabBar就会出现问题。产生这个BUG的另外一个必要条件就是UINavigationController的viewControllers的count大于2时才会出现这个问题，小于或者等于2时是没有问题的。即我在Tab对应的UINavigationController里Push了3级及以后做**popToRootViewControllerAnimated:**操作就会出现问题。由于之前我们是移除了系统的UITabBar里面的UITabBarItem然后添加的自己做的，所以出现的问题就是系统把我们移除的又加回来了。

![alt text](/img/UITabBarMixUINavigation/1.png)

如上图所示，是我们刚启动App的时候系统视图层级的结构图，看起来跟我们预期的是一样的，系统UITabBar里面的UITabBarItem已经被我们移除，而且添加了我们自己做的，正常使用没有出现任何问题。

![alt text](/img/UITabBarMixUINavigation/2.png)

当我做了产生BUG的操作后就出现了问题，系统视图层级结构的截图如上所示，跟第一张图进行对比你会发现多了5个UITabBarButton，而我们正好是5个UITabBarItem。所以我们自己添加到UITabBar的视图就被覆盖了。单个Tab的效果图如下：

![alt text](/img/UITabBarMixUINavigation/3.png)

刚开始的时候各种定位问题出现的原因，试了很多种解决方法。觉得用For循环再移除UITabBar的UITabBarItem就好了，但是是没有效果的，不知道系统是在何时加进去的。定位了很久才确定上述的操作产生了这个BUG。而且不明白的是为什么UINavigationController在3级及以后做**popToRootViewControllerAnimated:**才会产生这个BUG，定位这个原因花了不少的时间。

既然找到了产生BUG的原因，那解决起来就好办了。由于我使用的都是UINavigationController的子类，所以我在子类里重写了**popToRootViewControllerAnimated:**方法，代码如下：

	- (NSArray<UIViewController *> *)popToRootViewControllerAnimated:(BOOL)animated {
      if (self.viewControllers.count > 2) {
		  self.viewControllers = @[self.viewControllers.firstObject, self.viewControllers.lastObject];
      }
      return [super popToRootViewControllerAnimated:animated];
	}

我们在viewControllers的count大于2的时候重新赋值viewControllers，使其count小于等于2，然后再调用父类的方法。经测试，BUG没有再出现。

说明：这里我的UITabBarController和UINavigationController都是它们的子类，即自定义的。我想如果使用系统自带的UITabBarUITabBarController应该是不会出现这个问题的，实际怎样没有测试。前前后后花了差不多一天的时间解决这个BUG，真是蛋疼。

以上。

========== 更新 =========

上面所说的BUG，后来证实在**popToViewController**这个方法里也会出现，产生BUG的原因是类似的。所以我也是重写了**popToViewController**方法，保证要跳转到的Controller与当前的Controller之间的级数不超过2，代码如下：

    - (NSArray<UIViewController *> *)popToViewController:(UIViewController *)viewController animated:(BOOL)animated {
        NSArray *viewControllers = self.viewControllers;
        NSMutableArray *newViewControllers = [[NSMutableArray alloc] init];
        for (UIViewController *controller in viewControllers) {
            [newViewControllers addObject:controller];
            if (controller == viewController) {
                [newViewControllers addObject:viewControllers.lastObject];
                break;
            }
        }
        self.viewControllers = newViewControllers;
        return [super popToViewController:viewController animated:animated];
    }
