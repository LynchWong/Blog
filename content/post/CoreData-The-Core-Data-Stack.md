---
title: "CoreData: The Core Data Stack"
date: 2015-07-21 21:48:02
categories: 
- iOS
tags: 
- CoreData
metaAlignment: center
autoThumbnailImage: no
coverImage: //d1u9biwaxjngwg.cloudfront.net/welcome-to-tranquilpeak/city.jpg

---

到现在为止我们都是使用的Xcode生成的CoreData模板。使用Xcode来帮助我们并没有什么错，Xcode就是用来帮助我们提升开发效率的，而且在这些细节之处也体现了Xcode的优秀。一般情形下使用Xcode生成的CoreData模板就够用了，但是如果你想知道CoreData是如何工作的，那么你就必须创建自己的Core Data Stack。
<!--more-->

这个Stack由四个Core Data类组成：

* NSManagedObjectModel
* NSManagedObjectModel
* NSPersistentStoreCoordinator
* NSManagedObjectContext

之前我们已经接触过**NSManagedObjectContext**了。但是其他的三个类都在场景后面给予**NSManagedObjectContext**提供支持。

# 准备工作

本章的示例不想弄的太复杂，我们还是使用最开始的那个仓库管理的例子，在这基础上再改进一下，比如仓库中有商品等。只不过这次我们不使用Xcode为我们生成的模板，而是创建自己的Stack。所以我们应该新开一个项目，只是在新建的时候不要勾选**Use Core Data**。

在开始编码前，我们先来了解上面提及到的类。

# The managed object model

**NSManagedObjectModel**表示你数据模型中的所有对象类型，它们能够拥有的属性，以及它们之间的关系。Stack使用这个模型去创建对象，存储属性以及保存数据。

你可以将**NSManagedObjectModel**想象成数据库中的表视图。如果你的Stack在底层使用SQLite，那么**NSManagedObjectModel**代表什么就很明显。

然而，SQLite只是CoreData中很多持久化存储类型之一，所以应该在一般情形下思考**NSManagedObjectModel**。

**注意：**你可能对**NSManagedObjectModel**是怎样与数据模型编辑器联系起来的。可视的编辑器创建和编辑**xcdatamodel**文件。那里有一个特殊的编译器(**momc**)，这个编译器会将模型文件在**momd**文件夹中编译成一个文件集合。就像Swift代码能够编译优化然后运行在设备上，编译后的模型能够在运行时被高效的访问。CoreData在运行时使用**momd**文件夹中编译后的内容来初始化生成**NSManagedObjectModel**。

# The persistent store

**NSPersistentStore**会读取和写入数据，无论你决定使用哪个存储方法。CoreData提供了四种**NSPersistentStore**的类型：3种原子的和1种非原子的。

原子的**NSPersistentStore**在做任何读写操作前需要完全的反序列化然后加载到内存中。相反，非原子的**NSPersistentStore**可以根据需要将自己的部分加载到内存中。

下面简述了这四种内置类型：

1. **NSQLiteStoreType**使用SQLite数据库存储。这是CoreData支持的唯一一种非原子的，轻量级的操作及高效的内存使用。让它成为了大部分iOS项目的最佳选择。Xcode的模板默认使用该类型。
2. **NSXMLStoreType**使用XML文件存储，是所有类型中最具可读性的。该类型是原子的，所以会有大量的内存使用。该类型只在OS X中可用。
3. **NSBinaryStoreType**使用二进制数据文件存储。同样也是原子的，所以在操作之前你必须将所有的二进制数据加载到内存中。在现实世界中你基本很难找到使用该类型的应用程序。
4. **NSInMemoryStoreType**在某种程度上来说，这种类型并不是真正的持久化。如果你终止应用程序或者关掉了你的手机，这些在内存中的数据就会消失。感觉上这违背了CoreData的目的，但是该类型在单元测试或者某种缓存中是很有用的。

**注意：**是否有JSON文件或者CSV文件来作为持久存储类型呢？好消息是你可以通过子类化**NSIncrementalStore**来创建你自己的持久存储类型。[ 详见 ](https://developer.apple.com/library/ios/documentation/DataManagement/Conceptual/IncrementalStorePG/Introduction/Introduction.html)。

# The persistent store coordinator

**NSPersistentStoreCoordinator**是**NSManagedObjectModel**与**NSPersistentStore**的桥梁。在CoreData中它负责使用model和persistent stores来完成大部分繁重的工作。它明白**NSManagedObjectModel**，知道怎样给它发送消息，以及从**NSPersistentStore**获取信息。

**NSPersistentStoreCoordinator**也隐藏了怎样配置persistent store或者stores的实现细节。这样很有用，主要有两个原因：

1. **NSManagedObjectContext**不需要知道它保存的数据是保存在SQLite数据库，XML文件或者iCloud。
2. 如果你有多个persistent stores，对于**NSManagedObjectContext**来说，**NSPersistentStoreCoordinator**提供了一个统一的接口。只要**NSManagedObjectContext**关心，它总会表现成一个单一的聚合的persistent store。

# The managed object context

大多数情况下我们都是在和**NSManagedObjectContext**打交道。基本上你只会在设置你的Stack或者做迁移的时候才会看见其他三个组件。

因此，明白它是如何工作的就很重要。下面这几点可能是到目前为止你学到的：

* 一个context是存储managed objects的内存里面的暂存器。
* 使用context来处理所有CoreData对象的工作。
* 所有你做的改变并不会影响磁盘上的数据，直到你调用了context的save()方法。

这里还有另外的五点在之前没有提及，这些很重要，所以请注意：

1. context管理它创建或者请求的对象的生命周期。这个生命周期管理包括一些强大的功能，比如断层，逆向关系处理以及验证。
2. 一个managed object不能在context之外独立存在。事实上，一个managed object和它的context是紧密耦合的，所以所有的managed object都有它context的引用，可以像这样访问`let employeeContext = employee.managedObjectContext`。
3. context是非常有领域性的。一旦一个managed object与一个特定的context关联后，那么在它的生命周期中会一直保持这种连接。
4. 一个应用程序可以使用不止一个的context，大部分比较复杂的CoreData应用程序都是这一类的。由于context是磁盘内容在内存中的暂存器，事实上你可以在同一时间加载相同的CoreData对象到两个不同的context。
5. context不是线程安全的，managed object也是。你可以在创建它们的那个线程上与context和managed objects进行交互。苹果提供了多种方式在多线程应用程序中与context交互(后面详解)。

# Creating your stack object

我新建了一个项目，只是没有勾选要使用CoreData，这里就不详细讲解了。然后新建了一个Swift源文件，叫做`CoreDataStack`，将内容替换成了如下代码：

	import CoreData
	
	class CoreDataStack {
	    let context: NSManagedObjectContext
	    let psc: NSPersistentStoreCoordinator
	    let model: NSManagedObjectModel
	    var store: NSPersistentStore?
	}
	
这是编译器会报错，说没有Initializers，我们先暂时不管。这里的四个属性对应了之前的组件。

我们继续添加如下方法：

    func applicationDocumentsDirectory() -> NSURL {
        let fileManager = NSFileManager.defaultManager()
        
        let urls = fileManager.URLsForDirectory(.DocumentDirectory, inDomains: .UserDomainMask) as! [NSURL]
        return urls[0]
    }
    
这个方法返回了应用程序documents目录的URL。我们需要将SQLite数据库(就是一个文件)存放在这个documents目录内。这里是用来存放用户数据推荐的目录，不管你是否使用CoreData。

现在我们增加Initializer，如下所示：

    init() {
        //1
        let bundle = NSBundle.mainBundle()
        let modelURL = bundle.URLForResource("", withExtension: "momd")!
        model = NSManagedObjectModel(contentsOfURL: modelURL)!
        
        //2
        psc = NSPersistentStoreCoordinator(managedObjectModel: model)
        
        //3
        context = NSManagedObjectContext()
        context.persistentStoreCoordinator = psc
        
        //4
        let documentsURL = applicationDocumentsDirectory()
        let storeURL = documentsURL.URLByAppendingPathComponent("")
        
        let options = [NSMigratePersistentStoresAutomaticallyOption: true]
        var error: NSError?
        store = psc.addPersistentStoreWithType(NSSQLiteStoreType, configuration: nil,
            URL: storeURL, options: options, error: &error)
        
        if store == nil {
            println("Error adding persistent store: \(error)")
            abort()
        }
    }
    
Initializer负责对Stack中每一个独立的组件进行配置：

1. 第一步是从磁盘加载managed object model到**NSManagedObjectModel**对象。你通过得到包含**.xcdatamodeld**文件编译后的版本的**momd**目录的URL来完成这些操作。
2. 一旦你初始化了**NSManagedObjectModel**，下一步就是创建**NSPersistentStoreCoordinator**。记住persistent store coordinator斡旋于persistent store与NSManagedObjectModel之间。
3. **NSManagedObjectContext**的初始化器并不接受参数。然后在连接**NSPersistentStoreCoordinator**之后并没有多少用。你可以设置context的persistentStoreCoordinator属性。
4. 你并没有直接初始化创建一个persistent store。persistent store coordinator帮你处理了NSPersistentStore对象。你只需要简单的指定store type，store file的URL location以及一些配置选项。

上面代码中的一些URL并没有设置，稍后设置。

继续添加如下方法：

    func saveContext() {
        var error: NSError?
        if context.hasChanges && !context.save(&error) {
            println("Could not save: \(error), \(error?.userInfo)")
        }
    }
    
这个方法保存了Stack的managed object context，当错误发生时进行处理。

接下来到**ViewController.swift**，替换为如下内容：

	import UIKit
	import CoreData
	
	class ViewController: UIViewController {
	    
	    var managedContext: NSManagedObjectContext!
	
	    override func viewDidLoad() {
	        super.viewDidLoad()
	        // Do any additional setup after loading the view, typically from a nib.
	    }
	
	    override func didReceiveMemoryWarning() {
	        super.didReceiveMemoryWarning()
	        // Dispose of any resources that can be recreated.
	    }
	
	
	}
	
然后到**AppDelegate.swift**进行设置，引入CoreData框架`import CoreData`。然后在`var window: UIWindow?`下面添加如下代码：

	lazy var coreDataStack = CoreDataStack()
	
我们将coreDataStack属性设置为延迟属性，表示Stack在第一次被访问时才会被初始化。然后在application(_:didFinishLaunchingWithOptions:)方法中添加如下代码，主要就是设置**ViewController**的**managedContext**属性：

    let navigationController = window?.rootViewController as! UINavigationController
    
    let viewController = navigationController.topViewController as! ViewController
    viewController.managedContext = coreDataStack.context
    
修改如下方法：

    func applicationDidEnterBackground(application: UIApplication) {
        coreDataStack.saveContext()
    }
    
    func applicationWillTerminate(application: UIApplication) {
        coreDataStack.saveContext()
    }
    
这些方法保证了程序不管什么原因导致的进入后台或者被终结，都会将挂起的改变进行保存。

# Modeling your data

现在我们就来创建数据模型，**File\New\File...**，选择**iOS\Core Data\Data Model**，这里我使用了默认的名字，所以我的Data Model就叫做**Model.xcdatamodeld**。现在我们回头去设置之前没有设置的一些URL。

	let modelURL = bundle.URLForResource("Model", withExtension: "momd")!
	let storeURL = documentsURL.URLByAppendingPathComponent("Model")
	
主要就是这两处代码。

现在我们就来设置实体，如之前就提到的，我们应该有Warehouse和Good这两个实体，那么我就一一设置。

![alt text](/img/CoreDataStack/1.png)

如上所示，我设置了一个**Warehouse**实体，有一个**name**属性，是**String**类型。

![alt text](/img/CoreDataStack/2.png)

同样的，设置了**Good**这个实体，也只有**name**属性，**String**类型。

其实在现实世界中，这些实体之间都是有关系的，比如我们仓库中可能存放着许多的商品。那么我应该在**Warehouse**实体中设置一个goods的属性，类型应该是数组或者集合的，但是我们发现属性类型里面根本就没有数组或者集合类型。其实这里我们不应该设置属性了，应该设置实体之间的关系。

如下图所示，我在**Warehouse**实体的**Relationships**这一栏设置了一个叫做**goods**的关系，**Destination**设置为了**Good**实体。**Destination**就可以想象成关系的末端。

![alt text](/img/CoreDataStack/3.png)

设置完成后，Xcode有一个警告，如下图，先不用管，还没设置完：

![alt text](/img/CoreDataStack/4.png)

每种关系刚开始都被默认设置成了**to-one**(即一对一的关系)，在上面的示例中表示我们的仓库一次只能追踪一个商品，其实这是不对的，因为我们的仓库肯定不止一个商品。仓库和商品应该是一对多的关系，所以我们应该进行修改，如下所示：

![alt text](/img/CoreDataStack/5.png)

我们选择**Good**实体，同样也需要设置关系，设置了一个叫做**warehouse**的关系，**Destination**为**Warehouse**，设置**Inverse**为**goods**。如下所示：

![alt text](/img/CoreDataStack/6.png)

这里默认的**to-one**关系是正确的，因为一个商品不可能存在于多个仓库中。

**Inverse**能够让模型进行逆向的寻找。比如给定了一条Good记录，你可以根据关系找到该商品的仓库。

![alt text](/img/CoreDataStack/7.png)

你可以切换Editor Style，如上图所示，你可以清晰的看出实体之间的关系。

# Adding managed object subclasses

之前我们已经创建过managed object subclasses了，到**Editor\Create NSManagedObject Subclass...**，勾选要需要创建的实体。

如下所示：

	import Foundation
	import CoreData
	
	class Warehouse: NSManagedObject {
	
	    @NSManaged var name: String
	    @NSManaged var goods: NSOrderedSet
	
	}
	
如前面一样，name使用了String类型，但是goods关系使用了集合来表示。CoreData使表示一对多的关系时使用集合，而不是数组。因为我们让goods关系有序，所以是有序的集合。

**注意：NSSet**看起来像是奇怪的选择。与数组不同，集合不是通过下标来访问它们的成员。事实上，集合是无序的。CoreData使用NSSet是因为集合保证了成员的唯一性。相同的对象在一对多的关系中不会存在多次。如果你想使用下标来访问独立的对象，你应该勾选上**Ordered**，就如我们之前已经勾选上了一样。CoreData就会使用NSOrderedSet来表示这个关系。

**Good**看起来如下所示：

	import Foundation
	import CoreData
	
	class Good: NSManagedObject {
	
	    @NSManaged var name: String
	    @NSManaged var warehouse: Warehouse
	
	}
	
也许你的代码看起来不是这样的，而是`@NSManaged var warehouse: NSManagedObject`，你可以重新执行一遍生成操作即可。

还要进行如下所示的设置，**Good**也是类似：

![alt text](/img/CoreDataStack/8.png)

# A good persistence lane

尽量让实例程序简单点，在**ViewController**的**viewDidLoad**方法里面，我们根据仓库name来查找是否有叫`L`的仓库，如果有就赋值给`currentWarehouse`属性，没有我们就新建一个。如下：

    override func viewDidLoad() {
        super.viewDidLoad()
        // Do any additional setup after loading the view, typically from a nib.
        
        let warehouseEntity = NSEntityDescription.entityForName("Warehouse", inManagedObjectContext: managedContext)!
        
        let warehouseName = "L"
        let fetchRequest = NSFetchRequest(entityName: "Warehouse")
        fetchRequest.predicate = NSPredicate(format: "name == %@", warehouseName)
        
        var error: NSError?
        let result = managedContext.executeFetchRequest(fetchRequest, error: &error) as? [Warehouse]
        if let warehouses = result {
            if warehouses.count == 0 {
                currentWarehouse = Warehouse(entity: warehouseEntity, insertIntoManagedObjectContext: managedContext)
                currentWarehouse.name = warehouseName
                
                if !managedContext.save(&error) {
                    println("Could not save: \(error)")
                }
            } else {
                currentWarehouse = warehouses[0]
            }
        } else {
            println("Could not fetch: \(error)")
        }
        
        navigationItem.title = currentWarehouse.name
    }
    
代码很简单，就不讲解了。

和最开始的仓库管理类似，对ViewController进行了如下的设置，添加了UITableView以及UIBarButtonItem，设置了@IBOutlet和selector，这些就不详细讲解怎么弄了。

![alt text](/img/CoreDataStack/9.png)

运行之后，模拟器截图如下：

![alt text](/img/CoreDataStack/10.png)

要在列表中显示这个仓库的商品，但是现在还没有商品，接下来就来添加商品。

    @IBAction func add(sender: AnyObject) {
        var alert = UIAlertController(title: "添加仓库", message: "输入仓库名", preferredStyle: .Alert)
        let saveAction = UIAlertAction(title: "保存", style: .Default) {
            (action: UIAlertAction!) -> Void in
            let textField = alert.textFields![0] as! UITextField
            self.saveName(textField.text)
            self.tableView.reloadData()
        }
        let cancelAction = UIAlertAction(title: "取消", style: .Default) {
            (action: UIAlertAction!) -> Void in
        }
        
        alert.addTextFieldWithConfigurationHandler {
            (textField: UITextField!) -> Void in
        }
        
        alert.addAction(saveAction)
        alert.addAction(cancelAction)
        
        presentViewController(alert, animated: true, completion: nil)
    }
    
    func saveName(name: String) {
        let entity = NSEntityDescription.entityForName("Good", inManagedObjectContext: managedContext)!
        let good = Good(entity: entity, insertIntoManagedObjectContext: managedContext)
        good.name = name
        
        var goods = currentWarehouse.goods.mutableCopy() as! NSMutableOrderedSet
        goods.addObject(good)
        
        currentWarehouse.goods = goods.copy() as! NSOrderedSet
        
        var error: NSError?
        if !managedContext.save(&error) {
            println("Could not save: \(error)")
        }
        
        tableView.reloadData()
    }
    
@IBAction方法就是之前的，没有变动。**saveName()**方法也很简单，主要就是**NSOrderedSet**的可变和不可变。

这里看起来好像设置一对多关系时还是有些复杂。如果你的关系是没有ordered的，那么你可以在**one**这一端进行设置，比如`good.warehouse = currentWarehouse`。然后CoreData能够根据Inverse关系来设置goods。

运行之后截图如下：

![alt text](/img/CoreDataStack/11.png)

你可以重启App进行检查，查看数据是否被持久化了。

PS：这里代码并没有给全，结束时会给出完整的代码。

# Deleting objects from Core Data

在**ViewController**里面添加如下方法：

    func tableView(tableView: UITableView, commitEditingStyle editingStyle: UITableViewCellEditingStyle, forRowAtIndexPath indexPath: NSIndexPath) {
        if editingStyle == UITableViewCellEditingStyle.Delete {
            let goodRemove = currentWarehouse.goods[indexPath.row] as! Good
            let goods = currentWarehouse.goods.mutableCopy() as! NSMutableOrderedSet
            goods.removeObjectAtIndex(indexPath.row)
            currentWarehouse.goods = goods.copy() as! NSOrderedSet
            
            managedContext.deleteObject(goodRemove)
            
            var error: NSError?
            if !managedContext.save(&error) {
                println("Could not save: \(error)")
            }
            
            tableView.deleteRowsAtIndexPaths([indexPath], withRowAnimation: .Automatic)
        }
    }
    
运行模拟器之后向左滑动Cell就会出现删除按钮，点击删除，如下所示删除了`water`：

![alt text](/img/CoreDataStack/12.png)

同样你也可以重启App进行检查。

[ 完整源码 ](https://github.com/LynchWong/OwnCoreDataStack)。
