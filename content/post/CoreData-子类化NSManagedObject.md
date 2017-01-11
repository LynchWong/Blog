---
title: "CoreData: 子类化NSManagedObject"
date: 2015-07-21 15:08:30
categories: 
- iOS
tags: 
- CoreData
metaAlignment: center
autoThumbnailImage: no
coverImage: //d1u9biwaxjngwg.cloudfront.net/welcome-to-tranquilpeak/city.jpg

---

在之前我们设置或者获取实体的属性时时通过KVC来实现的，其实你可以使用KVC来直接操纵NSManagedObject的所有东西，但不意味着你必须这样做。使用KVC时，涉及到大量的字符串，所以可能经常导致错误的拼写。

最好的替代方法就是为数据模型实体创建**NSManagedObject**的子类。你可以像访问属性一样来访问或者设置实体的属性。
<!--more-->

# 数据类型

我们继续使用之前的项目，[ 程序完整源码 ](https://github.com/LynchWong/WarehouseList)。

我们设置**Warehouse**实体的**name**属性的类型为**String**，可以浏览下支持的类型。

Integer有不同长度可选，区别如下，根据需求来选择。

> Range for 16-bit integer: -32768 to 32767
> Range for 32-bit integer: –2147483648 to 2147483647
> Range for 64-bit integer: –9223372036854775808 to 9223372036854775807
**Binary Data**类型可以存储二进制数据，比如图片，音乐文件等等。如果存储的二进制数据太大，会影响应用程序的性能。这就意味着你每次访问实体时都会将这个实体加载到内存中，包括这些二进制数据，即使你只是想要访问实体的其他属性，比如**name**。
幸运的是CoreData预计到了这个问题。你可以在**Attributes**查看器里面选上**Allows External Storage**，如下所示：
![alt text](/img/CoreDataNSManagedObjectSub/1.png)
勾选上了**Allows External Storage**之后，CoreData会依据每次的值来决定是直接存储到数据库还是存储分离文件的URI。
**注意：Allows External Storage**选项只在**Binary Data**类型下可用。而且，如果打开了这个选项，CoreData就不能使用这个属性来进行查询。
# 任意类型的数据
你可以使用**Transformable**来存储任意的数据类型，即使是你自定义的类型。只要该类型适配了**NSCoding**协议。如果你想要保存你自定义的对象，你首先应该实现**NSCoding**协议。
# Managed object subclasses
如下所示我们让Xcode帮我们创建了**NSManagedObject**的子类**Warehouse**。

![alt text](/img/CoreDataNSManagedObjectSub/2.png)
语言我们选择Swift，生成之后截图如下：
![alt text](/img/CoreDataNSManagedObjectSub/3.png)
如果你之前在OC中使用过CoreData，你会发现有些不一样。与OC中**@dynamic**属性是类似的，**@NSManaged**属性通知Swift的编译器属性的实现和存储在运行时提供，而不是在编译时。正常模式下一个实例变量的属性存储在内存中。而managed object的属性不一样，由managed object context存储，所以源数据在编译时是不知道的。
* String maps to String
* Integer 16/32/64, Float, Double and Boolean map to NSNumber 
* Decimal maps to NSDecimalNumber
* Date maps to NSDate
* Binary data maps to NSData
* Transformable maps to AnyObject
CoreData会将对象图持久到磁盘，所以默认是和对象工作。所以像integers，doubles以及Booleans封装成了NSNumber。如果你想直接使用Double或者Int32这些基本类型，你可以在创建的时候勾选上**Use scalar properties for primitive data types**，如下所示：
![alt text](/img/CoreDataNSManagedObjectSub/4.png)
基于面向对象，建议还是不要勾选。
接下来我们修改**ViewController.swift**里面的代码，如下：
	var warehouses = [Warehouse]()
	let fetchedResults = managedContext.executeFetchRequest(fetchRequest, error: &error) as? [Warehouse]
	let warehouse = Warehouse(entity: entity!, insertIntoManagedObjectContext: managedContext)

主要是涉及到**NSManagedObject**以及和KVC相关的一些代码，先清除掉与App相关的数据，删除App，然后Command+Shift+K清除数据。在运行之前还需要如下设置，在实体的查看器里面设置**Class**，如下所示：
	
![alt text](/img/CoreDataNSManagedObjectSub/5.png)
如果不这样设置，会出现警告，说找不到**Warehouse**然后使用**NSManagedObject**代替。
# 数据验证
CoreData能够进行数据验证，比如我设置了一个**number**的属性，类型是Integer32，然后设置了最大值和最小值，如下所示：
![alt text](/img/CoreDataNSManagedObjectSub/6.png)
当你调用了managed object context的save操作后，验证会立即执行。managed object context会检查新值是否冲突了验证规则。如果验证出现错误，save操作就会失败。记得之前我们传递了**NSError**的指针给save方法。发生错误时并没有做其他的事情，验证操作改变了这种情况。

如果save操作失败了，CoreData会将错误信息填充到你传递过来的引用里面，然后你要决定当错误发生时做什么。比如我们执行save操作时，实体的number属性设置的值大于了10，那么就会发生错误。如果我们将错误代码和信息打印出来就能看到具体的内容。
	var error: NSError?
	if !self.managedContext.save(&error) {
	    if error!.code == NSValidationNumberTooLargeError || error!.code == NSValidationNumberTooSmallError {
	        //错误处理
	    }
	} else {
	    
	}
所以我们应该做如上代码所示的处理。**NSValidationNumberTooLargeError**这些是与错误码1610对应错误的枚举值。完整的CoreData错误码的定义，参见**CoreDataErrors.h**，Xcode中Cmd点击**NSValidationNumberTooLargeError**。
