---
title: "CoreData: Measuring and Boosting Performance"
date: 2015-07-25 13:27:29
categories: 
- iOS
tags: 
- CoreData
metaAlignment: center
autoThumbnailImage: no
coverImage: //d1u9biwaxjngwg.cloudfront.net/welcome-to-tranquilpeak/city.jpg

---

很显然的一件事情就是我们应该努力的优化我们开发的程序性能。一个性能低下的应用程序最好的情况可能就只是收到差的评价，最坏的情况就是无法响应以及崩溃。
<!--more-->

在CoreData的应用程序里，我们更应该进行优化。CoreData的大部分实现速度都很快，也很轻量，得益于CoreData内部的不断优化。

CoreData的灵活性让其成为了一个伟大的工具，但是你在一些方面的使用仍然会让CoreData在性能方面遭受到负面的影响。

本章就讲解如何优化CoreData的性能。本章没有实例代码，仅将一些结论及代码实践。

性能即是平衡，内存与速度之间的平衡。我们应用程序中的数据存在于两个地方：内存或者磁盘上。访问内存中数据比磁盘要快的多，但是设备的内存容量要比磁盘小的多。

你加载到内存中的数据越多，你程序的速度就越快，但是消耗的内存就越多。你可以减少内存的使用，但是你程序的速度就会变慢，因为数据都在磁盘上。

#Measure,change,verify

应用程序中哪里有性能瓶颈不是猜测出来的，通过测量应用程序的性能能够节省你大量的时间。Xcode为我们提供了大量的工具。

理想的情况下，你应该先测量性能，然后进行改进，最后再次测量验证你的改进提升了性能。你应该多次重复测量-改进-验证的过程直到你的应用程序达到你所要求的性能。

本章不提供Demo，简单叙述下。有两个实体A和B，它们之间是一对多的关系。A有个属性是Binary Data的类型，用来存放图片。应用程序启动时将A实体的所有数据加载到UITableView里面进行显示。

接下来优化这个程序，按照前面提到的测量-改进-验证这个过程进行。

##测量

在Xcode中我们可以看到我们应用程序的内存占用情况。如果我们运行前面提到的那个App，内存占用一定会很大。因为实体中有存储图片的二进制数据，图片数据会被加载到内存中。所以如果我们A的实体记录比较多的话，在程序启动的时候就要加载A的数据到UITableView中进行显示，那么内存就会消耗的很多。

![alt text](/img/CoreDataPerformance/1.png)

在前面的章节我们也提到过，即使我们只是要修改A的一个属性，与这个图片属性无关，二进制数据也会全部加载到内存中，然而我们根本就用不到。

所以这里我们应该改进我们的数据模型。

##改进

我们应该再新增一个实体C，添加一个Binary Data类型的属性，然后勾选**Allows External Storage**，这个选项我们之前也提到过。通常情况下二进制数据存放在数据库中，当你勾选了这个选项，CoreData会自动决定是当做分离的文件存储在磁盘上还是存放在数据库中。将A和C设为一对一的关系，A中使用缩略图，当点击查看大图的时候通过与C一对一的关系来加载图片。

##验证

没有Demo，没有截图，真是不好意思。

理论上来说，内存占用会减少很多。因为A实体使用了缩略图，大图和小图内存占用当然不一样啦。

#Fetching and performance

任何时候我们要访问数据，都需要使用fetch request来查询。我们不应该获取我们需要之外的数据。

##Fetch batch size

CoreData的fetch requests包含fetchBatchSize属性，让我们能够简单的获取足够的数据，但是又不会太多。

如果你不设置batch size，CoreData会使用默认的0值，即不使用batching。

设置一个非零的正值batch size让你限制返回的batch size。

在Xcode中⌘I，选择Core Data，如下所示：

![alt text](/img/CoreDataPerformance/2.png)

然后点击红点进行记录，一段时间后，如下所示：

![alt text](/img/CoreDataPerformance/3.png)

看最后一列。这列包含了Objective-C版本的调用者，获取数据的条数，以及操作的时间(微秒)。

回到我们之前假设的ABC。每次启动的时候都会加载所有的A实体的数据。这里我们应该设置`fetchRequest.fetchBatchSize = 10`。这样我们获取的数据一次就是10条，当需要的时候CoreData会再自动获取数据。这个值应该设置为多大，通常来说，我们在UITableView显示数据。假如我们一屏能显示5条数据，那么应该设置为显示条数的两倍，即10。如果是10条，就设置为20。

##Advanced fetching

这一小节主要涉及到**NSFetchRequest**，之前我们有讲过，这里就不在详细讲解了。

举两个简单的例子。比如获取结果的条数的时候，我们可以设置**NSFetchRequest**的resultType属性为**NSCountResultType**。

还记得之前获取所有商品价格总和吗，同理设置resultType为**NSDictionaryResultType**。

详细参见[ CoreData: Fetching ](http://lynchwong.com/2015/07/22/CoreData-Fetching/)

这一章写的比较粗糙，主要赶时间。推荐大家去看英文原版，有完整的Demo及更详细的讲解。

[《Core Data by Tutorials: iOS 8 and Swift Edition》](http://www.raywenderlich.com/store/core-data-by-tutorials?source=matthewmorey)的**Chapter 9: Measuring and Boosting Performance**。

以上。
