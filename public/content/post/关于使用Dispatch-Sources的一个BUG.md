---
title: "关于使用Dispatch Sources的一个BUG"
date: 2016-03-03 09:26:11
categories: 
- Code issue
tags: 
- Dispatch Sources
metaAlignment: center
autoThumbnailImage: no
coverImage: //d1u9biwaxjngwg.cloudfront.net/welcome-to-tranquilpeak/city.jpg

---

之前有在这篇博客[ URL Session Programming Guide - Introduction ](http://lynchwong.com/2016/01/29/URL-Session-Programming-Guide-Introduction/)提到在翻译完[ 并发变成指南 ](http://lynchwong.com/2016/01/14/Concurrency-Programming-Guide-Introduction/)后有要做一个图片轮播的需求，是使用的**DISPATCH_SOURCE_TYPE_TIMER**类型的Dispatch Sources来做时间控制进行轮播的而没有使用NSTimer。这个在后来引起了一个BUG，实在是无心之过。
<!--more-->

# 源码

    import UIKit

    public class RollingPlayPicture: UIView {
        
        private let playTimer = dispatch_source_create(DISPATCH_SOURCE_TYPE_TIMER, 0, 0, dispatch_get_main_queue())
        
        ....
        ....
        
        private func setupData() {
            if imageNumbers > 1 {
                dispatch_source_set_timer(playTimer, dispatch_walltime(nil, 0), UInt64(interval) * NSEC_PER_SEC, 0 * NSEC_PER_SEC)
                dispatch_source_set_event_handler(playTimer, task)
                dispatch_resume(playTimer)
            }
        }
        
        private func task() {
            if currentIndex == imageNumbers - 1 {
                currentIndex = 0
            } else {
                currentIndex += 1;
            }
            let offsetX = bounds.width * CGFloat(currentIndex)
            UIView.animateWithDuration(0.2) { () -> Void in
                self.pageControl.currentPage = self.currentIndex
                self.scrollView.contentOffset = CGPoint(x: offsetX, y: self.scrollView.contentOffset.y)
            }
        }
        
    }

代码大概是这样子的，由于使用的Swift，所以我直接声明初始化了常量属性playTimer。然后在imageNumbers即图片数量大于1的时候就对playTimer进行相关的设置，当时我觉得这个判断真TMD机智。其实后来才发现，不加这个判断就不会有BUG，不会崩溃。每隔interval秒都会执行task，task就是轮播的简单动画。

# 定位BUG

当时出现BUG的时候定位了很久，首先崩溃的时候没有任何信息，没有定位到具体的哪一行代码。一开始我是在测试环境下测试的，那时候没有任何问题。但是崩溃是出现在线上环境，所以我们一直以为是环境的问题。但是也找不到具体问题是在哪。

后来才发现跟图片数量有关，而测试环境返回的图片数量是2，线上环境返回的1。

按照上面的源码，当图片的数量等于0的时候我在ViewController里面是不会创建轮播视图的，这时候是没有什么问题的。当图片数量大于1的时候，图片是会正常轮播的，没有任何问题。问题恰好就是当图片数量等于1的时候才会出现问题，BUG现象就是离开轮播的ViewController的时候会崩溃。所以产生BUG的条件就是图片的数量是等于1的时候。

# 解决BUG

跟图片数量有关，而正好是1的时候才会崩溃。根据之前的图片数量大于1的判断，当图片数量等于1时if里面的代码是没有执行的，其它情况都是正常的(等于1和大于1)。所以崩溃肯定是跟Dispatch Sources有关了。当我取消了图片大于1的时候崩溃就没有了，我做这个判断就是规避图片等于1时不需要轮播，因为一张图片没什么好轮播的。

做了其它测试发现只要**dispatch_source_create(DISPATCH_SOURCE_TYPE_TIMER, 0, 0, dispatch_get_main_queue())**和设置playTimer的代码(即if里面的代码)没有一起执行就会崩溃。

当时BUG出现的时候我已经回家了，是同事解决的，他直接使用了NSTimer，删除了Dispatch Sources相关的代码。

当然你也可以把这些代码放到一起执行也不会有BUG，即把playTimer的初始化方法图片数量判断的if里面去就行了。

# 产生BUG的原因

那是只知道只要这些代码没有一起执行就会引发崩溃，不知道真正的原因。所以我就先回头看了之前翻译的[ Concurrency Programming Guide - Dispatch Sources ](http://lynchwong.com/2016/01/14/Concurrency-Programming-Guide-Dispatch-Sources/)，还是不明白什么原因引起的。后来经同事提醒可能是因为挂起和恢复状态引起的崩溃。

在博文里面有提到：

># Creating Dispatch Sources

>**因为调度源在使用之前必须进行额外的配置才能被使用，dispatch_source_create函数返回一个挂起状态的调度源。在挂起的时候，调度源会接收事件，但是不会处理它们。这时你可以安装事件处理器并执行额外的配置。**

所以这里我们可以得出我们源码里面的playTimer是处于挂起状态的。

然后在最后一个章节Suspending and Resuming Dispatch Sources里面有提到：

># Suspending and Resuming Dispatch Sources

>**你可以使用dispatch_suspend 和 dispatch_resume函数临时的挂起和继续调度源的事件传递。这个函数分别增加和减少调度对象的挂起计数。因此，你必须每次dispatch_suspend调用之后，都需要dispatch_resume才能继续事件的传递。**

从这里我们得知，重点就是挂起和恢复的函数要成对调用(要达到平衡)，因为这两个函数会增加和减少挂起计数(这个计数可能跟引用计数类似或者就是同一个计数，我也不知道。)。

官方文档的原话就是：

># Suspending and Resuming Dispatch Sources

>**You can suspend and resume the delivery of dispatch source events temporarily using the dispatch_suspend and dispatch_resume methods. These methods increment and decrement the suspend count for your dispatch object. As a result, you must balance each call to dispatch_suspend with a matching call to dispatch_resume before event delivery resumes.**

文档里面提到的是suspend count，这个跟引用计数的英文应该不一样吧，我书读的少，我也不管了。

所以我们来分析下，只要我们创建轮播视图，就会声明初始化playTimer这个常量属性，并且playTimer是处于挂起状态。所以我同事就说了，一开始挂起计数就增加了。然后当图片数量等于1的时候，里面的dispatch_resume函数没有执行，然后在退出时，挂起计数没有减少。没有达到平衡，所以就出现了崩溃。

尼玛，我读文档的时候，文档根本就没说这些啊。文档只说了你的调用要成对出现啊，要平衡啊。我就初始化了下，什么都没做啊你就让我崩溃了。

我读文档的时候把重点放在了你的调用要成对出现，既然我什么都没调用，所以我觉得就是没问题的啊。我感觉我是被坑了，唉，其实是我书读的太少了，想的太少了，不知道文字背后这一层意思。

一开始的时候我各种Google、Stackoverflow发现了很多OC版写的**DISPATCH_SOURCE_TYPE_TIMER**类型的Dispatch Sources。他们也都是使用属性声明了dispatch_source_t，但是大家都知道OC不能像Swift这样直接初始化。肯定都在代码里面跟另外的代码一起执行了(即初始化和设置的代码放在一起执行)，并且我发现他们的代码路径不管是怎样的，那些代码都是一起执行的(即初始化和设置的代码放在一起执行)。所以我想我当时要是使用OC写这个轮播，可能这个BUG就这么错过了。因为我肯定也是先声明dispatch_source_t的属性，然后在使用的时候进行初始化，然后紧跟着就设置、调用dispatch_resume方法。

以上。
