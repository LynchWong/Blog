---
title: "CoreData: Multiple Managed Object Contexts"
date: 2015-07-25 13:27:53
categories: 
- iOS
tags: 
- CoreData
metaAlignment: center
autoThumbnailImage: no
coverImage: //d1u9biwaxjngwg.cloudfront.net/welcome-to-tranquilpeak/city.jpg

---

之前就有一篇讲[ CoreData多线程处理 ](http://lynchwong.com/2015/06/09/CoreData%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%A4%84%E7%90%86/)。和[《Core Data by Tutorials: iOS 8 and Swift Edition》](http://www.raywenderlich.com/store/core-data-by-tutorials?source=matthewmorey)的**Chapter 10: Multiple Managed Object Contexts** 感觉上类似的，内容差不了太多。
<!--more-->

本篇博文也只讲讲结论以及一些编程实践，同样也没有Demo实例，推荐大家看英文原版。

如果我们使用Xcode给我们生成的CoreData模板，使用的都是单个的**NSManagedObjectContext**。大部分操作都是在主线程中完成，我们也知道UI更新的操作也是在主线程中进行的，所以有时候必然会造成界面卡顿的现象。比如我们需要执行一个耗时的任务，如果只有单个的在主线程上执行的**NSManagedObjectContext**，那么就会造成UI卡顿，导致用户体验很不好。

# Doing work in the background

假如我们的App需要将数据导出到一个CSV文件中，如果我们使用的是单个的在主线程上执行的**NSManagedObjectContext**。当数据量很庞大的时候就会造成界面好几秒的卡顿，让用户无法操作，以为程序已经死掉了。

我们可以如下的代码模板来进行操作，这样便不会造成界面无法响应的情况。

	//新建一个NSManagedObjectContext，设置为PrivateQueueConcurrencyType类型
    let privateContext = NSManagedObjectContext(
      concurrencyType: .PrivateQueueConcurrencyType)
    privateContext.persistentStoreCoordinator =
      coreDataStack.context.persistentStoreCoordinator
  
    privateContext.performBlock { () -> Void in
      
      //获取数据
      var fetchRequestError:NSError? = nil
      let results = privateContext.executeFetchRequest(
        fetchRequest,
          error: &fetchRequestError)
      if results == nil {
        println("ERROR: \(fetchRequestError)")
      }
      
      //写入数据到CSV文件
    
      //返回主线程更新界面
      dispatch_async(dispatch_get_main_queue(), { () -> Void in
        //提醒用户操作结果
      })
    }
    
首先我们创建了一个新的**NSManagedObjectContext**，类型为**PrivateQueueConcurrencyType**，叫做privateContext。这个context和一个私有的队列关联了起来。一旦你创建了新的context，将main managed object context的persistent store coordinator赋值给它。然后调用performBlock，这个方法会在context的队列里面异步执行block里面的代码。最后更新UI，值得注意的是根UI相关的所有操作，总是应该在主线程上执行，否则会出现无法预测的结果。

**NSManagedObjectContext**总共有三种concurrencyType可以使用。ConfinementConcurrencyType已经废弃不再使用，剩下的两种基本覆盖了所有的使用情况。PrivateQueueConcurrencyType会将context跟私有的队列进行关联，而不是主线程的队列，所以不会阻塞UI。MainQueueConcurrencyType是默认的类型，我们之前**CoreDataStack**都是这种类型，我们之所有没有感觉的卡顿是因为数据量太少，无法察觉到卡顿，之前我们也有提过。为table view创建fetched results controller必须使用这种类型的context。

# Editing on a scratchpad

有时候我们需要编辑修改一个实体的时候或者需要修改大量数据的时候，我们可以使用child context。原理可以参考[ CoreData多线程处理 ](http://lynchwong.com/2015/06/09/CoreData%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%A4%84%E7%90%86/)。简单说一下，我们可以创建一个临时的**NSManagedObjectContext**，temporary Context，然后将它的**parentContext**属性设置为main managed object context。然后在temporary Context执行save操作，改变并没有影响到parent context。这些改变并没有发送到persistent store coordinator，直到parent context也执行了save操作。

下面是代码模板，和[ CoreData多线程处理 ](http://lynchwong.com/2015/06/09/CoreData%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%A4%84%E7%90%86/)是类似的，直接复制了，是OC版本的：

	NSMangedObjectContext *temporaryContext = [[NSManagedObjectContext alloc] initWithConcurrencyType:NSPrivateQueueConcurrencyType];
	temporaryContext.parentContext = mainMOC;
	 
	[temporaryContext performBlock:^{
	   // do something that takes some time asynchronously using the temp context
	 
	   // push to parent
	   NSError *error;
	   if (![temporaryContext save:&amp;error])
	   {
	      // handle error
	   }
	 
	   // save parent to disk asynchronously
	   [mainMOC performBlock:^{
	      NSError *error;
	      if (![mainMOC save:&amp;error])
	      {
	         // handle error
	      }
	   }];
	}];
	
代码中有注释，简单说下。

首先我们创建一个临时的temporaryContext，然后设置了他的parentContext属性为mainMOC。然后执行performBlock进行需要的操作，然后执行save操作。最后将改变push到mainMOC，最后mainMOC执行save操作将数据保存到数据库进行持久化。

我们将本篇提到的两种模板进行对比下。其实最开始在后台执行的操作也可以使用这种模板。因为我们第一种模板并没有修改数据，只是简单的获取数据，所以我们只要把第二种模板的所有保存操作取消即可，这样就能实现第一种需求了。

写的比较简单，不知道有没有叙述清楚，还请参看原文以及之前的那篇博文。以上
