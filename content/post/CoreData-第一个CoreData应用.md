---
title: "CoreData: 第一个CoreData应用"
date: 2015-07-21 09:14:43
categories: 
- iOS
tags: 
- CoreData
metaAlignment: center
autoThumbnailImage: no
coverImage: //d1u9biwaxjngwg.cloudfront.net/welcome-to-tranquilpeak/city.jpg

---

在开发中使用过几次CoreData，还算是比较熟悉了，觉得有必要整理下。CoreData是非常重要的一个框架，功能丰富，想要完全的掌握还需要花时间。

该系列主要参考[《Core Data by Tutorials: iOS 8 and Swift Edition》](http://www.raywenderlich.com/store/core-data-by-tutorials?source=matthewmorey)，推荐购买正版看英文原版。

<!--more-->

CoreData是什么就不讲解了，最开始接触CoreData的时候感觉像个ORM框架，但是并没有这么简单。搜索自行了解CoreData是什么东西，下面我们就直接开始了。

# 开始

首先我们先做个一个App，这个App很简单，用来显示我们的仓库的列表，我们可以添加仓库。

我们新建一个项目，如下所示，记得勾选**Use Core Data**，如下所示：

![alt text](/img/CoreDataFirstApp/1.png)

新建完成后你可以看见比我们平时不使用Core Data的项目要多了一些东西，而且**AppDelegate.swift**里面也多了不少的代码。这些东西都是Xcode的模板为我们自动生成了，这些配置对于大多数的App都够用了，后面我们会自己创建Core Data Stack，不使用Xcode为我们创建的。

先不讲解CoreData的这些东西，我们先完成这个简单的App。

将**View Controller**包含在**Navigation Controller**里面，如下所示：

![alt text](/img/CoreDataFirstApp/2.png)

给**View Controller**添加如下所示的内容：

![alt text](/img/CoreDataFirstApp/3.png)

请自行给Tableview设置约束。然后设置Tableview的dataSource，如下所示：

![alt text](/img/CoreDataFirstApp/4.png)

这里只需要设置dataSource即可，然后给Tableview设置@IBOutlet，以及设置Bar Button Item的selector，如下所示

![alt text](/img/CoreDataFirstApp/5.png)

![alt text](/img/CoreDataFirstApp/6.png)

**View Controller**的代码如下：

	import UIKit
	
	class ViewController: UIViewController, UITableViewDataSource {
	
	    @IBOutlet weak var tableView: UITableView!
	    
	    var warehouses = [String]()
	    
	    override func viewDidLoad() {
	        super.viewDidLoad()
	        // Do any additional setup after loading the view, typically from a nib.
	        title = "仓库管理"
	        tableView.registerClass(UITableViewCell.self, forCellReuseIdentifier: "Cell")
	    }
	
	    override func didReceiveMemoryWarning() {
	        super.didReceiveMemoryWarning()
	        // Dispose of any resources that can be recreated.
	    }
	
	    @IBAction func add(sender: AnyObject) {
	        var alert = UIAlertController(title: "添加仓库", message: "输入仓库名", preferredStyle: .Alert)
	        let saveAction = UIAlertAction(title: "保存", style: .Default) { (action: UIAlertAction!) -> Void in
	            let textField = alert.textFields![0] as! UITextField
	            self.warehouses.append(textField.text)
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
	    
	    // MARK: UITableViewDataSource
	    func tableView(tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
	        return warehouses.count
	    }
	    func tableView(tableView: UITableView, cellForRowAtIndexPath indexPath: NSIndexPath) -> UITableViewCell {
	        let cell = tableView.dequeueReusableCellWithIdentifier("Cell") as! UITableViewCell
	        cell.textLabel!.text = warehouses[indexPath.row]
	        return cell
	    }
	
	}
	
代码很简单，就不一一讲解了。
	
运行后模拟器截图如下：

![alt text](/img/CoreDataFirstApp/7.png)

点击添加可以添加新的仓库，但是这些数据并没有被保存起来，在程序运行期间存在于内存中。当我们重启设备或者退出(注意这里的退出不是指点击home键退出到主屏幕，而是从后台中清除)，这些数据就不会存在了。所以我们就需要CoreData来持久化数据，当然你也可以用其他的的持久化手段，比如使用SQLite或者读写文件等等。接下来我们就使用CoreData来持久化这些数据。

# CoreData

以前做Web开发的时候，使用过ORM映射框架，比如**NHibernate**，第一步就是映射模型。

## 模型化数据

所以我们先要创建**managed object model**，Xcode自动为我们生成了数据模型文件，叫做**WarehouseList.xcdatamodeld**。

![alt text](/img/CoreDataFirstApp/8.png)

点击**Add Entity**，命名为**Warehouse**，如上所示。

* 一个**entity**就是就是CoreData里面定义的类。典型的例子就是员工和公司。在关系型数据库中，一个**entity**就对应一张表。
* 一个**attribute**是与entity相关的部分信息。比如，员工entity就可能有员工名字，位置，薪水等attribute。在数据库中，一个attribute就对应表中的列。
* **relationship**是多个entity之间的联系。在CoreData中，两个entity之间可以有**to-one relationships**和**to-many relationships**关系

现在我们给**Warehouse**添加一个叫做**name**的**Attributes**，选择String类型，如下所示。

![alt text](/img/CoreDataFirstApp/9.png)

## 保存

使用CoreData需要引入CoreData框架，在**ViewController.swift**中添加如下代码：

	import CoreData
	
修改如下代码：

	var warehouses = [NSManagedObject]()
	
我们需要存储Warehouse的实体，所以我们需要修改为如上所示的代码。然后修改**UITableViewDataSource**相关的方法，如下所示：

	// MARK: UITableViewDataSource
	    func tableView(tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
	        return warehouses.count
	    }
	    
	    func tableView(tableView: UITableView, cellForRowAtIndexPath indexPath: NSIndexPath) -> UITableViewCell {
	        let cell = tableView.dequeueReusableCellWithIdentifier("Cell") as! UITableViewCell
	        let warehouse = warehouses[indexPath.row]
	        cell.textLabel!.text = warehouse.valueForKey("name") as? String
	        return cell
	    }
	    
接下来我们就要修改添加仓库的方法，让CoreData将添加的数据持久化。修改**add**方法中的**saveAction**，如下所示：

	let saveAction = UIAlertAction(title: "保存", style: .Default) {
	    (action: UIAlertAction!) -> Void in
	    let textField = alert.textFields![0] as! UITextField
	    self.saveName(textField.text)
	    self.tableView.reloadData()
	}
        
**saveName**为保存操作，我们将单独提出来：

    func saveName(name: String) {
        //1
        let appDelegate = UIApplication.sharedApplication().delegate as! AppDelegate
        let managedContext = appDelegate.managedObjectContext!
        
        //2
        let entity = NSEntityDescription.entityForName("Warehouse", inManagedObjectContext: managedContext)
        let warehouse = NSManagedObject(entity: entity!, insertIntoManagedObjectContext: managedContext)
        //3
        warehouse.setValue(name, forKey: "name")
        //4
        var error: NSError?
        managedContext.save(&error)
        if error != nil {
            println("Could not save \(error), \(error!.userInfo)") }
        //5
        warehouses.append(warehouse)
    }
    
1. 在你能够从CoreData存储里面保存或者查询很合数据之前，你首先需要一个**NSManagedObjectContext**，你可以将managed object context想象成内存中的一个暂存器。保存一个新的managed object到CoreData中需要两步：第一，你将一个新的managed object插入到managed object context；然后，提交这些改变到managed object context保存到硬盘。因为我们使用了模板，所以我们使用app delegate里面的NSManagedObjectContext。
2. 我们创建了一个新的managed object然后插入到了managed object context中。你可能会奇怪**NSEntityDescription**是什么，**NSManagedObject**就像一个变形者，能够代表任何的entity。**NSEntityDescription**能够在运行时连接你定义在数据模型中实体的实例。
3. 然后我们设置实体的属性。
4. 最后调用managedContext的**save**方法来保存数据，如果有错误则打印出来。
5. 最后将新数据添加到数据源中。

然后我们运行添加一个仓库，如下所示：

![alt text](/img/CoreDataFirstApp/10.png)

但是当我们重启的时候会发现列表是空的，就不截图了，主要原因是因为我们并没有从CoreData中查询数据，所以，接下来我们就来查询数据。

## 查询

我们在**ViewController.swift**中添加如下代码：

    override func viewWillAppear(animated: Bool) {
        super.viewWillAppear(animated)
        
        //1
        let appDelegate = UIApplication.sharedApplication().delegate as! AppDelegate
        let managedContext = appDelegate.managedObjectContext!
        
        //2
        let fetchRequest = NSFetchRequest(entityName: "Warehouse")
        
        //3
        var error: NSError?
        
        let fetchedResults = managedContext.executeFetchRequest(fetchRequest, error: &error) as? [NSManagedObject]
        
        if let results = fetchedResults {
            warehouses = results
        } else {
            println("Could not fetch \(error), \(error!.userInfo)")
        }
    }
    
1. 我们仍然需要**managed object context**，查询也不列外。
2. 顾名思义**NSFetchRequest**负责从CoreData中查询数据。功能强大且很灵活，你可以查询一些符合特定条件的数据集合(比如查询居住在威斯康辛州并且工龄至少3年的所有员工)等等。
3. 然后managed object context执行**executeFetchRequest(_:error:)**方法，返回了一个可选的数组。

**注意：**如果没有符合查询条件的数据，那么该方法就会返回一个没有数据的空的可选数组。如果发生了错误，这个方法就会返回nil。
   
现在让我们运行，截图如下所示，列表中就有了我之前添加的数据了：

![alt text](/img/CoreDataFirstApp/11.png)

[ 程序完整源码 ](https://github.com/LynchWong/WarehouseList)
