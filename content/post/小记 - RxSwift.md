---
title: "小记 - RxSwift"
date: 2016-04-23 09:30:49
categories: 
- 记录
tags: 
- RxSwift
metaAlignment: center
autoThumbnailImage: no
coverImage: //d1u9biwaxjngwg.cloudfront.net/welcome-to-tranquilpeak/city.jpg

---

又有一个月没有写过博客了，真是罪过，深深的内疚的感觉。之前写完了[ Alamofire 源码学习 ](http://lynchwong.com/2016/03/19/Alamofire-源码学习-Request/)然后让另一个做iOS的同学看，他说看不懂，写的乱。
<!--more-->

其实我看了下貌似是挺乱的，但是 Alamofire 的模式很简单，明白了就没什么东西了。大神的代码写的很简洁、简单，思路很清晰，结构很明朗。我觉得这样的代码就是最好的，特别是在公司里面，大家合作的时候，代码越简单、简洁，思路越清晰越好，这样才能降低维护的成本。

这段时间除了忙公司项目外，还在搞RxSwift。学习了下RxSwift，做了些小Demo，感觉用起来很爽。跟之前使用RAC相比，我感觉RxSwift更容易上手。RAC和RxSwift的源码都看了下，看懂了一小部分，后面会接着看。这两个框架的对比，网上有很多，之前我也用过RAC，项目里面也用过。但是RxSwift给我的感觉就是更顺，所以现在我更倾向于RxSwift。

这里贴一些RxSwift的学习资料，方便查阅：

ReactiveX社区，不仅仅是RxSwift，几乎包含了所有的编程语言：

* [ ReactiveX ](http://reactivex.io)
* [ ReactiveX文档中文翻译 ](https://mcxiaoke.gitbooks.io/rxdocs/content/)
* [ RxMarbles 图解运算符 ](http://rxmarbles.com)

讲解RxSwift的文章，以及一些Demo：

* [ RxSwift 函数响应式编程 ](https://realm.io/cn/news/slug-max-alexander-functional-reactive-rxswift/?hmsr=toutiao.io)
* [ 通过 Moya+RxSwift+Argo 完成网络请求 ](http://blog.callmewhy.com/2015/11/01/moya-rxswift-argo-lets-go/)
* [ RxSwift 入坑手册 Part0 - 基础概念 ](http://blog.callmewhy.com/2015/09/21/rxswift-getting-started-0/)
* [ RxSwift 入坑手册 Part1 - 示例实战 ](http://blog.callmewhy.com/2015/09/23/rxswift-getting-started-1/)
* [ RxSwift的第一印象 ](http://geek.csdn.net/news/detail/59759)
* [ RxSwift 学习指导索引 ](http://t.swift.gg/d/2-rxswift)(列出的参考资源很好，很丰富)

RAC和RxSwift的对比：

* [ iOS响应式编程：ReactiveCocoa vs RxSwift 选谁好 ](http://www.cnblogs.com/andy-zhou/p/5321798.html)

RAC也好、RxSwift也好，都跟FRP有关，理解FRP的思想：

* [ 函数式反应型编程(FRP) —— 实时互动应用开发的新思路 ](http://www.infoq.com/cn/articles/functional-reactive-programming)

隆重推荐最后这篇，超级棒。很多人学习了RxSwift，但是可能不知道在项目里怎么用，那么读了这篇你就明白了。讲解了如何以响应式编程的方式来思考：一切皆是流。

* [ 那些年我们错过的响应式编程 ](https://github.com/hehonghui/android-tech-frontier/tree/master/androidweekly/那些年我们错过的响应式编程)

当然看源码也是很好的，项目中的 Rx.playground 也是学习理解的好地方。

以上。
