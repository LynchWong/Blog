---
title: "CoreData多线程处理"
date: 2015-06-09 10:26:19
categories: 
- iOS
tags: 
- CoreData
metaAlignment: center
autoThumbnailImage: no
coverImage: //d1u9biwaxjngwg.cloudfront.net/welcome-to-tranquilpeak/city.jpg

---

首先要说明的是本篇博文不是教程，不会教你怎么从零构建使用CoreData，本篇博文主要讲的是多线程处理CoreData。
<!--more-->

先说说之前开发项目的时候遇到的问题。由于项目的需求，需要多线程处理CoreData。我遇到的问题就是每次新开线程处理数据后，发现数据是有更新的；但是当我重启App之后再看发现数据变回原来的了。所以我大致猜测是CoreData多线程的问题，因为CoreData多线程是不安全的。其实做项目之前就考虑使用CoreData的时候就知道不是线程安全的，自己在开发的时候是有留意多线程这方面的。

在网上搜索了很多资料，感觉很多资料都没有说清楚；可能有的说清楚了博主难以理解。最后搜到了一篇博文解决了我的问题[ Multi-Context CoreData ](http://www.cocoanetics.com/2012/07/multi-context-coredata/?utm_source=tuicool)。该篇博文讲解了iOS5之前的Multi-Context方式和iOS5及之后。根据这篇博文，博主发现少做了一些步骤，所以导致我重启App数据后变回了原来的。得到的原因就是虽然是处理了数据，但是并没有更新存储到磁盘，所以再重启的时候取出的还是原来的数据。

该博文给出的iOS5及之后的方法就是：

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
	
博主的问题就在于少了：

	// save parent to disk asynchronously
	[mainMOC performBlock:^{
	   NSError *error;
	   if (![mainMOC save:&amp;error])
	   {
	      // handle error
	   }
	}];
	
这个步骤，加上了这个步骤博主的问题就解决了。

下面就翻译下这篇博文。

# Multi-Context CoreData

当你开始使用CoreData为你的应用程序做持久化操作时，是使用单个的managed object context (MOC，即**NSManagedObjectContext**对象)。如果你创建项目的时候勾选了“Use Core Data”，Xcode自带的模板就是为你这样设置的，即单个的MOC。

使用CoreData和NSFetchedResultsController能够极大的简化我们处理要显示在table view里面的数据，比如说排序、刷选等。

有两种场景你可能需要拓展下范围，使用多个managed object contexts：

* 简化 adding/editing 新数据项的操作。
* 防止阻塞了UI。

本篇博文将会回顾设置你contexts的方式来达到你想要的效果。

首先，我们回顾下单一context的设置。你需要一个persistent store coordinator (PSC，持久化存储协调器)管理与磁盘上数据库文件的对话。所以那个PSC知道你需要模型的数据库的结构。这个模型是由所有定义包含在项目里面的模型合并而成的，告诉CoreData数据库的结构是怎样的。这个PSC通过属性设置给了MOC。需要记住的第一条规则就是：如果你调用MOC的**saveContext**方法，PSC将会把数据写入磁盘。

![alt text](/img/CoreDataMultiContext/first.png)

思考一下这张图。每当你在单个MOC中插入、更新或者删除实体类，这些改变将会通知给fetched results controller然后更新table view的内容。这些独立context的saving操作。你可以随心所欲的执行save操作。苹果给出的模板在每次添加了实体对象后以及在**applicationWillTerminate**方法里都会执行save操作。

这种方法对于大多数基本情况来说都可以很好的完成工作，但是对于我们之前提到的那两个问题。第一个问题是关于添加新的实体。你可能想使用同一个view controller来添加和修改一个实体。所以你可能想在展示VC之前新建一个实体对象然后填充好这个VC。这就会触发一个fetched results controller的update notifications(更新通知)，等等。一个空的Cell将会在modal view controller完全展现之前短暂的出现。

第二个问题很明显，如果在**saveContext**之前有大量的更新，那么save操作将会超过1/60分。所以在这种情况下，用户界面将会被阻塞直到save操作完成。这种情形就跟我们滚动视图的时候的那种卡顿、跳动感很像。

这些问题都能使用多个MOCs解决。

## 传统的Multi-Context方法

将每一个MOC想象成改变数据的临时暂存器。在iOS5之前你可能会监听其他MOCs的改变然后通过通知将改变合并到你的main MOC(主要的MOC)。一种典型的设置如下图所示：

![alt text](/img/CoreDataMultiContext/second.png)

你可能会创建一个temporary MOC(临时的MOC)给background queue(后台线程)使用。所以允许改变被保存，你应该设置跟main MOC相同的PSC给temporary MOC。

>Marcus Zarra：尽管**NSPersistentStoreCoordinator**也不是线程安全的，但是**NSManagedObjectContext**知道在使用的时候如何加锁。然而，我们可以尽可能的多的将**NSManagedObjectContext**对象使用同一个**NSPersistentStoreCoordinator**而不必担心冲突。

在background MOC上调用**saveContext**方法就会把改变写入到存储文件中同时也会触发一个**NSManagedObjectContextDidSaveNotification**通知。

代码大致如下：

	dispatch_async(_backgroundQueue, ^{
	   // create context for background
	   NSManagedObjectContext *tmpContext = [[NSManagedObjectContext alloc] init];
	   tmpContext.persistentStoreCoordinator = _persistentStoreCoordinator;
	 
	   // something that takes long
	 
	   NSError *error;
	   if (![tmpContext save:&error])
	   {
	      // handle error
	   }
	});
	
创建一个temporary MOC是很快的，所以你不需要担心频繁的创建或者释放这些temporary MOCs。重点是设置与mainMOC相同的persistentStoreCoordinator，所以写入操作也可以发生在background(后台，上面代码中的_backgroundQueue)。

我可能会如下简单的设置CoreData stack(大部分开发者都是自己定义CoreData的堆栈，而不是使用Xcode模板生成的，原因可能就是单一的MOC，以及使AppDelegate文件很臃肿等等)：

	- (void)_setupCoreDataStack
	{
	   // setup managed object model
	   NSURL *modelURL = [[NSBundle mainBundle] URLForResource:@"Database" withExtension:@"momd"];
	   _managedObjectModel = [[NSManagedObjectModel alloc] initWithContentsOfURL:modelURL];
	 
	   // setup persistent store coordinator
	   NSURL *storeURL = [NSURL fileURLWithPath:[[NSString cachesPath] stringByAppendingPathComponent:@"Database.db"]];
	 
	   NSError *error = nil;
	   _persistentStoreCoordinator = [[NSPersistentStoreCoordinator alloc] initWithManagedObjectModel:_managedObjectModel];
	 
	   if (![_persistentStoreCoordinator addPersistentStoreWithType:NSSQLiteStoreType configuration:nil URL:storeURL options:nil error:&amp;error]) 
	   {
	   	// handle error
	   }
	 
	   // create MOC
	   _managedObjectContext = [[NSManagedObjectContext alloc] init];
	   [_managedObjectContext setPersistentStoreCoordinator:_persistentStoreCoordinator];
	 
	   // subscribe to change notifications
	   [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(_mocDidSaveNotification:) name:NSManagedObjectContextDidSaveNotification object:nil];
	}
	
现在我们来考虑下接收到didSave notification(上文提及的**NSManagedObjectContextDidSaveNotification**)通知时该如何处理。

	- (void)_mocDidSaveNotification:(NSNotification *)notification
	{
	   NSManagedObjectContext *savedContext = [notification object];
	 
	   // ignore change notifications for the main MOC
	   if (_managedObjectContext == savedContext)
	   {
	      return;
	   }
	 
	   if (_managedObjectContext.persistentStoreCoordinator != savedContext.persistentStoreCoordinator)
	   {
	      // that's another database
	      return;
	   }
	 
	   dispatch_sync(dispatch_get_main_queue(), ^{
	      [_managedObjectContext mergeChangesFromContextDidSaveNotification:notification];
	   });
	}
	
通过第一个if判断我们需要避免合并自己的改变。如果我们同一个App里面有多个CoreData数据库，我们需要避免合并其他数据库的改变。我在自己的App里面遇到过这个问题，所以我回检查PSC。最后我们通过提供的**mergeChangesFromContextDidSaveNotification: **方法来合并改变。所有的改变都在notification的字典里面，这个方法知道如何将这些改变整合到MOC里面。

## 在Contexts之间传递Managed Objects

你从一个MOC获取的managed object是被严格禁止传递给另外一个MOC的。有一个简单的方法通过managed object的ObjectID来排序它的镜像。这个ObjectID是线程安全的，你总是可以从NSManagedObject的实例中获取然后在你想要传递给的MOC上调用objectWithID:。第二个MOC将会检索这个managed objects的拷贝来工作。

	NSManagedObjectID *userID = user.objectID;
	 
	// make a temporary MOC
	dispatch_async(_backgroundQueue, ^{
	   // create context for background
	   NSManagedObjectContext *tmpContext = [[NSManagedObjectContext alloc] init];
	   tmpContext.persistentStoreCoordinator = _persistentStoreCoordinator;
	 
	   // user for background
	   TwitterUser *localUser = [tmpContext objectWithID:userID];
	 
	   // background work
	});
	
上面描述的方法完整的反相兼容到最初介绍CoreData的iOS版本，iOS3。如果你App的deployment target是iOS5，有一种更现代的方式，下面讲解。

## Parent/Child Contexts

iOS5介绍了MOCs可以有parentContext的能力。调用**saveContext**方法会将child context的改变push到parent而不需要求助于描述改变的字典来合并内容。苹果也为MOCs增加了自己专用的队列来执行同步或者异步的改变。

队列的类型在**NSManagedObjectContext**新的初始化器**initWithConcurrencyType**指定。注意在下面的插图中我增加了多个child MOCs，并且所有的parent都是同一个main queue MOC。

![alt text](/img/CoreDataMultiContext/third.png)

每当child MOC执行save操作时，parent都会得知这些改变，在这种情形下，fetched results controllers也会知悉这些改变。然后这些数据还没有被持久化，因为background MOCs并不知道PSC。为了将数据写入磁盘你需要在main queue MOC上执行额外的**saveContext:**操作。

首先我们需要改变main MOC的concurrency type(并发类型)为**NSMainQueueConcurrencyType**。在上面提到的_setupCoreDataStack方法中修改初始化的那一行，然后merge notification(合并的通知)不再需要。如下所示：

	_managedObjectContext = [[NSManagedObjectContext alloc] initWithConcurrencyType:NSMainQueueConcurrencyType];
	[_managedObjectContext setPersistentStoreCoordinator:_persistentStoreCoordinator];
	
后台操作如下所示：

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
	
现在每一个MOC都需要使用**performBlock:** (async) 或者 **performBlockAndWait:** (sync)来执行操作。这样就确保了block里面的操作在正确队列里面执行。上面的示例中的操作将在background queue里执行。一旦操作完成并且通过saveContext将改变push到parent之后，然后在mainMOC上执行异步的saving操作。同样的，这些操作被performBlock强制在正确的队列里面执行。

Child MOCs并不会自动的从它们的parents那里得到更新。你可以重新加载来得到这些更新，但是在大部分情况下它们都是临时的，所以并不需要困扰这个问题。只要main queue MOC得到了那些改变fetched results controllers就会更新，然后在main MOC执行saving操作做持久化。

这种方式简化了操作，你可以为任何有保存和取消按钮的view controller创建一个temporary MOC(as child)。如果你传递一个managed object(通过objectID)给temp context用来编辑。用户可以更新managed object的所有元素。如果用户点击了保存按钮，那么你就在temporary context上执行save操作。如果用户点击了取消，就不会有任何操作，而且所有的改变都与temporary MOC一同被废弃了。

## Asynchronous Saving(异步执行save操作)

Marcus Zarra向我展示了如下的方法，该方法构建在Parent/Child方法之上，但是增加了一个额外的context专门负责磁盘的写入。如之前提及的长时间的写入操作可能会阻塞主线程造成用户界面的卡顿。该方法很聪明的将写入操作放到自己的private queue里，然后保持用户界面操作很流畅。

![alt text](/img/CoreDataMultiContext/four.png)

设置CoreData也很简单了。只需要将persistentStoreCoordinator移到我们新的用来写入的MOC，然后将main MOC作为它的child。

	// create writer MOC
	_privateWriterContext = [[NSManagedObjectContext alloc] initWithConcurrencyType:NSPrivateQueueConcurrencyType];
	[_privateWriterContext setPersistentStoreCoordinator:_persistentStoreCoordinator];
	 
	// create main thread MOC
	_managedObjectContext = [[NSManagedObjectContext alloc] initWithConcurrencyType:NSMainQueueConcurrencyType];
	_managedObjectContext.parentContext = _privateWriterContext;
	
现在我们必须为每一次更新执行3次save操作：temporary MOC，main UI MOC以及写入磁盘。如之前所示的那样，很简单。现在用户界面将不会被阻塞当我们在执行大量数据库操作(比如导入大量数据记录)的时候以及写入磁盘的时候。

## Conclusion(结论)

iOS5极大的简化了在background queues处理CoreData的工作，以及parents分别从child MOCs得到改变。如果你仍然需要支持iOS 3/4那么这些对你来说远远不够。但是如果你的项目最低要求是iOS 5那么你可以立即根据上述Marcus Zarra的方法来设计。

Zach Waldowski指出使用private queue的并发类型来处理“editing view controllers”也许是杀鸡用牛刀。如果你使用**NSContainmentConcurrencyType**替代child context那么你就不需要performBlock的包装了。但是你仍然需要在mainMOC上调用performBlock执行saving操作。

>confinement的并发类型使用“the old way”的方式来执行contexts，但是那并不意味着是旧的，遗留的。它简单的将context的操作绑到自己管理的线程模式。为每一个view controller创建一个新的private queue是很浪费的，不需要的，而且很慢。**-performBlock:** 和 **-performBlockAndWait:**不能与confinement的并发类型一起工作是有原因的，因为blocks和locking是必须的，当你在处理multiple contexts时，正如你在“editing” view controller里设置的方式一样。

> **NSManagedObjectContext**知道如何智能的保存和合并，因为main thread context 绑到main thread，合并总是被安全的执行。editing view controller被绑到了main thread正如main view controller；这就是为什么在这里使用confinement的并发类型是合适的。这个editing context从概念上来将并不是是什么“new”的，只是将改变推迟了，但是仍然允许你完全的废弃改变。

所以它真的可以归结为你的个人喜好:private queue和performBlock或confinement。个人更倾向于private queues,因为使用它们时给我安全感。

有些地方翻译的不好，前面给出了原文地址，请自行参照原文阅读。
