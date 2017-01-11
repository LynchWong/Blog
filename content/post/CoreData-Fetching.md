---
title: "CoreData: Fetching"
date: 2015-07-22 22:00:06
categories: 
- iOS
tags: 
- CoreData
metaAlignment: center
autoThumbnailImage: no
coverImage: //d1u9biwaxjngwg.cloudfront.net/welcome-to-tranquilpeak/city.jpg

---

前面我们从CoreData提取数据的时候都只是简单的使用了**NSFetchRequest**，没有对数据进行精确的控制和刷选。

本章就讲讲**NSFetchRequest**，但是没有Demo演示，大多都只是给出代码，相信大家都能看得懂，然后自己去尝试使用。

<!--more-->

# NSFetchRequest

从前面的章节我们得知，从CoreData获取数据的方法就是新建**NSFetchRequest**的实例，然后进行你需要的配置，再交给**NSManagedObjectContext**来完成操作。

看起来相当的简单，实际上，有四种方法来获取一个**NSFetchRequest**。如下所示：

    //1
    let fetchRequest1 = NSFetchRequest()
    let entity = NSEntityDescription.entityForName("Warehouse", inManagedObjectContext: managedContext)!
    fetchRequest1.entity = entity
    
    //2
    let fetchRequest2 = NSFetchRequest(entityName: "Warehouse")
    
    //3
    let fetchRequest3 = managedObjectModel.fetchRequestTemplateForName("FetchRequest")
    
    //4
    let fetchRequest4 = managedObjectModel.fetchRequestFromTemplateWithName("FetchRequest", substitutionVariables: ["name" : "apple"])

如上代码所示，第一种和第二种都很简单。第三种方法，我们从**NSManagedObjectModel**里面检索了**NSFetchRequest**。你可以在Xcode的数据模型编辑器里面存储你经常使用的一些**NSFetchRequest**，后面我们会讲解。第四种方法和第三种是类似的，但是传递了参数，`substitutionVariables`会用在谓词中来细化我们要获取的数据。
# Stored fetch requests
正如之前提到的，我们可以将一些频繁使用的**NSFetchRequest**存储到数据模型中。这样不仅便于我们访问，而且可视化的界面操作也便于我们设置参数。

打开**.xcdatamodeld**文件，然后长按**Add Entity**，然后选择**Add Fetch Request**。如下图所示：

![alt text](/img/CoreDataFetching/1.png)

你可以从新设置名字，这里我使用了默认的名字。也增加了谓词，如图中所示。查找name是apple的Good。使用如下代码来获取这个FetchRequest，以及使用：

    let fetchRequest = managedObjectModel.fetchRequestTemplateForName("FetchRequest")!
    var error: NSError?
    let results = managedContext.executeFetchRequest(fetchRequest, error: &error) as? [Good]
    if let fetchedResults = results {
        let good = fetchedResults[0]
        println("result : \(good.name)")
    } else {
        println("Could not fetch \(error), \(error!.userInfo)")
    }

使用方法和我们之前是类似的，只是获取的方式不一样。注意这里是使用的**managedObjectModel**来获取的，即是我们之前**CoreDataStack**里面的**model**属性，**NSManagedObjectModel**的实例。**fetchRequestTemplateForName()**方法接收一个字符串参数，这个字符串必须与我们设置的FetchRequest的名字匹配，否侧程序会抛出异常然后崩溃。

如果你使用的是上一章节的Demo，那么你就需要在**AppDelegate.swift**里设置**ViewController.swift**时添加如下代码，同理还要在**ViewController.swift**设置一个**managedObjectModel**属性：

	viewController.managedObjectModel = coreDataStack.model
	
	var managedObjectModel: NSManagedObjectModel!
	
最后输出结果：

	result : apple

# Fetching different result types

感觉上**NSFetchRequest**没有多大用处，实际上，**NSFetchRequest**就像一把多功能瑞士军刀。你可以用它获取独立的数据，计算过后的数据，比如平均值、最小值、最大值等等。

**NSFetchRequest**有一个**resultType**属性，到目前为止，我们使用的都是默认的**NSManagedObjectResultType**。如下列出了所有可能的值：

* **NSManagedObjectResultType**：返回managed objects(默认值)。
* **NSCountResultType**：返回结果的条数。
* **NSDictionaryResultType**：返回不同的计算结果。
* **NSManagedObjectIDResultType**：返回一个唯一的identifiers，而不是整个对象。

## NSCountResultType

    let fetchRequest = NSFetchRequest(entityName: "Good")
    fetchRequest.resultType = .CountResultType
    var error: NSError?
    let results = managedContext.executeFetchRequest(fetchRequest, error: &error) as? [NSNumber]
    if let fetchedResults = results {
        let number = fetchedResults[0]
        println("result : \(number.integerValue)")
    } else {
        println("Could not fetch \(error), \(error!.userInfo)")
    }
一旦你设置了resultType为**NSCountResultType**，返回值就变成了包含单个**NSNumber**对象的可选数组。因为没有添加谓词，所以应该会查询所有的Good，CoreData里面只有两条数据，最后输出如下：
	result : 2

**注意：**你可能会觉得我们不需设置为**NSCountResultType**，只需使用默认值，然后使用数组的count属性就可以达到相同的目的。但是这样的话，返回值数组包含的是对象。假设我们有百万条数据，你觉得CoreData是给我们一个数值高效还是给我们百万条记录更高效，同时内存占用也完全不一样。

### An alternate way to fetch a count

还有另外一种可以替代设置resultType为**NSCountResultType**的方法，如下：

    let fetchRequest = NSFetchRequest(entityName: "Good")
    var error: NSError?
    let count = managedContext.countForFetchRequest(fetchRequest, error: &error)
    if count == NSNotFound {
        println("Could not fetch \(error), \(error!.userInfo)")
    }
    println("count : \(count)")
    
## NSDictionaryResultType

假设Good有个**price**的属性，类型是**Double**。现在我们有个新的需求，就是要得到所有商品的价格总和。实现这个需求并不难，我们可以把所有的商品查询出来，然后用一个循环把商品的价格加起来就实现了这个需求。幸运的是CoreData内建支持了大量不同的函数，比如求平均值的、总和的、最小值或最大值等等。

    //1
    let fetchRequest = NSFetchRequest(entityName: "Good")
    fetchRequest.resultType = .DictionaryResultType
    //2
    let sumExpressionDesc = NSExpressionDescription()
    sumExpressionDesc.name = "sumPrice"
    //3
    sumExpressionDesc.expression = NSExpression(forFunction: "sum:", arguments:[NSExpression(forKeyPath: "price")])
    sumExpressionDesc.expressionResultType = .Integer32AttributeType
    //4
    fetchRequest.propertiesToFetch = [sumExpressionDesc]
    //5
    var error: NSError?
    let result = managedContext.executeFetchRequest(fetchRequest, error: &error) as? [NSDictionary]
    if let resultArray = result {
        let resultDict = resultArray[0]
        let sumPrice: AnyObject? = resultDict["sumPrice"]
        println("SumPrice : \(sumPrice!)")
    } else {
        println("Could not fetch \(error), \(error!.userInfo)")
    }

最后输出结果：

	SumPrice : 100
	
我修改了下Good实体，添加了price属性，然后重新生成了NSManagedObject subclasses。然后在添加Good的方法里面设置所有添加的商品的price都为100。这里就不详细讲解怎么实现的了。

我们创建了**NSExpressionDescription**来请求总和，然后设置**name**属性为“sumPrice”。便于我们在后面字典里面获取这个值。然后我们指定了“sum:”这个函数以及需要求和的属性“price”；设置返回结果的类型，最后设置了propertiesToFetch属性wierd我们刚才创建的**NSExpressionDescription**。

**注意：**CoreData支持哪些函数，count, min, max, average, median, mode, absolute value等等，详见苹果官方文档的**NSExpression**。

## NSManagedObjectIDResultType

当设置为这种类型的时候，返回值数组包含的是**NSManagedObjectID**对象，而不是实体对象。**NSManagedObjectID**是一个全局唯一的identifier，就像是数据库里面的主键一样。

在iOS5之前，是用IDs来fetching数据很流行，因为它适配了并发模型。而现在线程限制已在更现代的并发模型中被废弃了，所以没有再使用这种类型的理由了。
    
# Sorting fetched results

**NSFetchRequest**另一个强大的功能就是对请求的数据进行排序，通过使用**NSSortDescriptor**。排序发生在**SQLite**层面，而不是在内存中。这让排序在CoreData中很高效。

	fetchRequest.sortDescriptors = [NSSortDescriptor(key: "warehouse.name", ascending: true)]
	
使用如上代码给**NSFetchRequest**添加排序，支持key path。

# Asynchronous fetching

到目前为止，我们执行的每一个**NSFetchRequest**都会阻塞主线程，阻塞主线程会导致什么结果相信大家都很清楚。

在我们的Demo中没有感受到阻塞是因为我们的数据量太少，做的操作都很简单。

CoreData从一开始就提供了几种不同的技术在后台执行**NSFetchRequest**。在iOS8，CoreData有了一个新的API，能够在后台长时间的执行**NSFetchRequest**，然后在完成时进行回调。
	
    let fetchRequest = NSFetchRequest(entityName: "Good")
    let asyncFetchRequest = NSAsynchronousFetchRequest(fetchRequest: fetchRequest) {
        [unowned self] (result: NSAsynchronousFetchResult! ) -> Void in
        	var goods = result.finalResult as! [Good]
    }
    var error: NSError?
    let results = managedContext.executeRequest(asyncFetchRequest, error: &error)
    if let persistentStoreResults = results {
        //Returns immediately, cancel here if you want
    } else {
        println("Could not fetch \(error), \(error!.userInfo)")
    }
从代码中可以看到**NSAsynchronousFetchRequest**，但是仍然需要**NSFetchRequest**。这里需要注意的是**executeFetchRequest()**被**executeRequest()**方法替换了。
**注意：**你可以使用**NSAsynchronousFetchRequest**的**cancel()**方法来取消数据请求。
除此之外还要修改**NSManagedObjectContext**，在比如之前的Demo，我们需要修改**CoreDataStack**中的**context**，如下：
	context = NSManagedObjectContext(concurrencyType: .MainQueueConcurrencyType)

**NSManagedObjectContext**有三种concurrencyType。如果你不设置，默认的是**.ConfinementConcurrencyType**，已经废弃了。还有一种是**.PrivateQueueConcurrencyType**，后面的章节会讲解。

# Batch updates: no fetching required

我们从CoreData里面获取数据就是为了要修改数据，大部分的操作也是基于此。如果你想要一次更新上百上千条数据怎么办呢？这需要大量的时间和内存开销，当然最普通的方法就是循环需要修改的数据，单着并不是最好的方法。

幸运的是在iOS8，苹果引进了batch updates，一种新的更新CoreData对象的方法，而不需要fetch任何东西到内存里。这种方式大大的减少了时间和内存的开销。代码如下：

    let batchUpdate = NSBatchUpdateRequest(entityName: "Good")
    batchUpdate.propertiesToUpdate = ["price": NSNumber(double: 150)]
    batchUpdate.affectedStores = psc.persistentStores
    batchUpdate.resultType = .UpdatedObjectsCountResultType
    
    var batchError: NSError?
    let batchResult = managedContext.executeRequest(batchUpdate, error: &batchError) as? NSBatchUpdateResult
    
    if let result = batchResult {
        println("Records updated \(result.result)")
    } else {
        println("Could not update \(batchError), \(batchError!.userInfo)")
    }
    
比如上面的代码，我将所有商品的价格都修改成了150。经典的使用场景就是一个邮件或者消息客户端有个“Mark all as read”的功能。

本章没有完整的演示Demo，大都只是提供代码，相信大家都能看懂，毕竟很多代码都是前面几章Demo涉及到的。

PS：最后两小节比较有局限性，只在iOS8中可用。到目前为止，整个App都只有单个的**NSManagedObjectContext**，后面我们会涉及到多个**NSManagedObjectContext**，以及在多线程间的同步问题。不过在学习这些之前还有些基础的知识需要了解学习，以上。
