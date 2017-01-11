---
title: "CoreData: NSFetchedResultsController"
date: 2015-07-23 13:34:02
categories: 
- iOS
tags: 
- CoreData
metaAlignment: center
autoThumbnailImage: no
coverImage: //d1u9biwaxjngwg.cloudfront.net/welcome-to-tranquilpeak/city.jpg

---

使用**UITableView**来展示CoreData的数据已经是标配了，**NSFetchedResultsController**能够简化我们的操作，就像是CoreData与**UITableView**之间的桥梁。
<!--more-->

# begins

在**ViewController**添加了如下所示的属性：

	var fetchedResultsController : NSFetchedResultsController!
	
然后在**viewDidLoad()**里面设置刚才添加的属性：

	let fetchRequest = NSFetchRequest(entityName: "Good")
    
    fetchedResultsController = NSFetchedResultsController(fetchRequest: fetchRequest,
        managedObjectContext: managedContext, sectionNameKeyPath: nil, cacheName: nil)
    
    var error: NSError?
    if !fetchedResultsController.performFetch(&error) {
        println("Error: \(error?.localizedDescription)")
    }
    
如上代码所示，我们还是需要创建**NSFetchRequest**的实例，然后传递给**NSFetchedResultsController**。`sectionNameKeyPath`和`cacheName`先不细说。最后调用**performFetch()**方法。

**NSFetchedResultsController**不仅包装了请求查询，而且它也是请求查询结果的容器。

这里查询获取数据是完成了，接下来我们就要展示数据，所以要实现**UITableViewDataSource**的一些方法，如下所示：

    func numberOfSectionsInTableView(tableView: UITableView) -> Int {
        return fetchedResultsController.sections!.count
    }
    
    func tableView(tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        let sectionInfo = fetchedResultsController.sections![section] as! NSFetchedResultsSectionInfo
        return sectionInfo.numberOfObjects
    }
    
    func tableView(tableView: UITableView, cellForRowAtIndexPath indexPath: NSIndexPath) -> UITableViewCell {
        let identifier = "Cell"
        
        var cell = tableView.dequeueReusableCellWithIdentifier(identifier) as? UITableViewCell
        if cell == nil {
            cell = UITableViewCell(style: .Default, reuseIdentifier: identifier)
        }
        let good = fetchedResultsController.objectAtIndexPath(indexPath) as! Good
        cell?.textLabel?.text = good.name
        return cell!
    }
    
如果你现在运行，程序会崩溃，Xcode打印的原因如下：

	reason: 'An instance of NSFetchedResultsController requires a fetch request with sort descriptors'
	
如果你想这样使用**NSFetchedResultsController**，那么不能将一个基本的**NSFetchRequest**扔给它。因为基本的**NSFetchRequest**不要求sort descriptor。然而**NSFetchedResultsController**至少要求一个sort descriptor，不然怎么知道table view的顺序呢。

所以我回头添加了如下代码：

	fetchRequest.sortDescriptors = [NSSortDescriptor(key: "name", ascending: true)]
	
运行后的截图如下所示：

![alt text](/img/CoreDataNSFetchedResultsController/1.png)

这里我将前面的Demo进行了一些改造，而且已经初始化了一些数据。文章最后会给出源码地址，所以这里就不一一讲解了怎么实现了。

# Grouping results into sections

我们的商品属于不同的分类，比如蔬菜、水果、零食等。所以我们修改下**NSFetchedResultsController**的初始化方法，记得之前提到的`sectionNameKeyPath`参数吗？代码如下：

    fetchedResultsController = NSFetchedResultsController(fetchRequest: fetchRequest,
        managedObjectContext: managedContext, sectionNameKeyPath: "category", cacheName: nil)
        
这里我们只是修改了下`sectionNameKeyPath`参数，能够接收keyPath字符串。比如实体的属性名或者关系，比如**warehouse.name**，所以这里的**category**是**Good**的一个属性。

除此之外，我们还要添加如下所示的方法：

    func tableView(tableView: UITableView, titleForHeaderInSection section: Int) -> String? {
        let sectionInfo = fetchedResultsController.sections![section] as! NSFetchedResultsSectionInfo
        return sectionInfo.name
    }
    
最后运行效果如下：

![alt text](/img/CoreDataNSFetchedResultsController/2.png)

这里有一个**NSFetchedResultsController**的疑难杂症，如果你使用keyPath将请求查询的结果分成了多个section，那么第一个sort descriptor的属性名必须与keyPath的属性名相匹配。

从上面的截图你能看出来，section对应的项是不对的，的确如此，我设置的华夫饼应该在零食里面，而番茄应该在蔬菜里面，全都乱了。所以我重新修改了sort descriptor，如下所示：

	fetchRequest.sortDescriptors = [NSSortDescriptor(key: "category", ascending: true)]
	
重新运行，截图如下：

![alt text](/img/CoreDataNSFetchedResultsController/3.png)

现在看起来好像对了。

# Cache

你可以指定一个cache name来打开**NSFetchedResultsController**在磁盘上的section cache。这就是你所有要做的工作。需要记住的是这个section cache与CoreData的persistent store是完全分开的。

	    fetchedResultsController = NSFetchedResultsController(fetchRequest: fetchRequest,
        managedObjectContext: managedContext, sectionNameKeyPath: "category", cacheName: "GoodCache")
        
**注意：**NSFetchedResultsController的section cache对fetch request的改变非常敏感。任何改变，比如不同的entity description或者sort descriptors都会给你一个不同的结果集合，完全无效的cache。如果你做了这样的改变，你必须使用deleteCacheWithName:删除已经存在的cache或者使用新的cache name。

在你的应用程序中，当你需要将结果组织到section中或者有大的数据集合或者在老机器上运行时考虑使用NSFetchedResultsController的cache。

# Modifying data

![alt text](/img/CoreDataNSFetchedResultsController/4.png)

如图所示，每种商品都有对应的数量。现在我们要实现一个新的功能，点击对应的商品，然后数量增加1。添加如下代码：

    func tableView(tableView: UITableView, didSelectRowAtIndexPath indexPath: NSIndexPath) {
        tableView.deselectRowAtIndexPath(indexPath, animated: true)
        
        let good = fetchedResultsController.objectAtIndexPath(indexPath) as! Good
        let number = good.number.integerValue
        good.number = NSNumber(int: number + 1)

        var error: NSError?
        if !managedContext.save(&error) {
            println("Could not save: \(error)")
        }
    }
    
运行点击，发现界面并没有变化，重启后发现数据有了变化。为了解决这个问题在方法最后添加如下代码：

	tableView.reloadData()
	
重新运行点击商品会发现界面会跟着变化了。但是这种方法相当的暴力，并不是最优的选择。接下来我们使用更好的方法来实现这个需求。

# Monitoring changes

当数据改变时，我们使用`tableView.reloadData()`来改变界面，这种方式很粗鲁。还有更好的方法，这一次又是**NSFetchedResultsController**拯救了我们。

**NSFetchedResultsController**能够监听结果集的变化，然后通知它的delegate，**NSFetchedResultsControllerDelegate**。你可以使用这个delegate来刷新table view在任何时候，只要底层数据改变了。

首先添加**NSFetchedResultsControllerDelegate**协议，然后设置`fetchedResultsController.delegate = self`。

**注意：**NSFetchedResultsController只能监听初始化时指定的**NSManagedObjectContext**造成的改变。如果你在应用程序的其他地方创建了一个**NSManagedObjectContext**，你的delegate的方法不会被执行，直到这些改变被保存并且被**NSFetchedResultsController**的context合并。

首先，从**tableView(_:didSelectRowAtIndexPath:)**方法中移除`tableView.reloadData()`。

**NSFetchedResultsControllerDelegate**有四个方法，我们实现如下方法：


    func controllerWillChangeContent(controller: NSFetchedResultsController) {
        tableView.beginUpdates()
    }
    
    func controller(controller: NSFetchedResultsController, didChangeObject anObject: AnyObject, atIndexPath indexPath: NSIndexPath?, forChangeType type: NSFetchedResultsChangeType, newIndexPath: NSIndexPath?) {
        switch type {
            case .Insert:
                tableView.insertRowsAtIndexPaths([newIndexPath!], withRowAnimation: .Automatic)
            case .Delete:
                tableView.deleteRowsAtIndexPaths([indexPath!], withRowAnimation: .Automatic)
            case .Update:
                tableView.reloadRowsAtIndexPaths([indexPath!], withRowAnimation: .None)
            case .Move:
                tableView.deleteRowsAtIndexPaths([indexPath!], withRowAnimation: .Automatic)
                tableView.insertRowsAtIndexPaths([newIndexPath!], withRowAnimation: .Automatic)
            default:
                break
        }
    }
    
    func controllerDidChangeContent(controller: NSFetchedResultsController) {
        tableView.endUpdates()
    }
    
重启运行App，点击商品进行测试，实现了同样的需求。

* **controllerWillChangeContent:**这个delegate方法会提醒你改变将要发生了，你使用**beginUpdates()**方法来准备好你的table view。
* **controller(_:didChangeObject...):**这个方法会告诉你哪个对象发生了改变，以及改变的类型，是插入数据、删除、更新、或者重新排序，以及影响的index paths。这个方法会将table view的数据与CoreData进行同步。不管底层数据发生了多大的改变，你的table view都会真实的展现出persistent store的变化。
* **controllerDidChangeContent:**使用**endUpdates()**方法来刷新界面。

**注意：**上面的这些代码都是由苹果的NSFetchedResultsControllerDelegate文档提供。请注意这些方法的顺序和性质。使用“begin updates- make changes-end updates”这种模式并不是一种巧合。

还剩一个**NSFetchedResultsControllerDelegate**的方法：

    func controller(controller: NSFetchedResultsController, didChangeSection sectionInfo: NSFetchedResultsSectionInfo, atIndex sectionIndex: Int, forChangeType type: NSFetchedResultsChangeType) {
                    
        let indexSet = NSIndexSet(index: sectionIndex)
                    
        switch type {
            case .Insert:
                tableView.insertSections(indexSet, withRowAnimation: .Automatic)
            case .Delete:
                tableView.deleteSections(indexSet, withRowAnimation: .Automatic)
            default :
                break
        }
    }
    
这个方法和**didChangeObject...**类似，只不过是提醒你sections的改变，而不是具体的对象的改变。上面方法只有创建新的section或者删除section的时候才会触发。

现在我们来思考下什么样的改变会触发这些通知。也许我新添加了一种类型的商品，比如服装。相反的情况是我删除了一个商品，而这个商品所属的类型的所有商品只有它自己，所以我就删除了这个section。接下来的小节让我们来实现这些需求。

# Inserting data

在**ViewController**里重写如下方法：

    override func motionEnded(motion: UIEventSubtype, withEvent event: UIEvent) {
        if motion == UIEventSubtype.MotionShake {
            addButton.enabled = true
        }
    }
    
当摇晃手机的时候，“+” bar button item就变成可用的。接下来我们就要实现添加商品的方法：

    @IBAction func add(sender: AnyObject) {
        var alert = UIAlertController(title: "添加商品", message: "新增商品", preferredStyle: UIAlertControllerStyle.Alert)
        
        alert.addTextFieldWithConfigurationHandler {
            (textField: UITextField!) in
            textField.placeholder = "名字"
        }
        alert.addTextFieldWithConfigurationHandler {
            (textField: UITextField!) in
            textField.placeholder = "数量"
        }
        alert.addTextFieldWithConfigurationHandler {
            (textField: UITextField!) in
            textField.placeholder = "类型"
        }
        
        alert.addAction(UIAlertAction(title: "保存", style: .Default, handler: {
            (action: UIAlertAction!) -> Void in
                let nameTextField = alert.textFields![0] as! UITextField
                let numberTextField = alert.textFields![1] as! UITextField
                let categoryTextField = alert.textFields![2] as! UITextField
            
                let entity = NSEntityDescription.entityForName("Good", inManagedObjectContext: self.managedContext)!
                let good = Good(entity: entity, insertIntoManagedObjectContext: self.managedContext)
                good.name = nameTextField.text
                good.number = NSNumber(int: Int32(NSString(string: numberTextField.text).integerValue))
                good.category = categoryTextField.text
                
                self.managedContext.save(nil)
        }))
        
        alert.addAction(UIAlertAction(title: "取消", style: .Default, handler: {
            (action: UIAlertAction!) -> Void in
                println("Cancel")
        }))
        
        presentViewController(alert, animated: true, completion: nil)
    }
    
代码很简单，这里就不讲解了。运行之后**Command+Control+Z**来模拟摇晃事件来让“+” bar button item可用：

![alt text](/img/CoreDataNSFetchedResultsController/5.png)

我新添加了一种类型的商品，最后效果如下所示：

![alt text](/img/CoreDataNSFetchedResultsController/6.png)

# Deleting data

现在让我们来删除数据，添加或者修改如下方法：

    func tableView(tableView: UITableView, commitEditingStyle editingStyle: UITableViewCellEditingStyle, forRowAtIndexPath indexPath: NSIndexPath) {
        if editingStyle == UITableViewCellEditingStyle.Delete {
            let good = fetchedResultsController.objectAtIndexPath(indexPath) as! Good
            managedContext.deleteObject(good)
            
            var error: NSError?
            if !managedContext.save(&error) {
                println("Could not save: \(error)")
            }
        }
    }
    
代码也很简单，不讲解了。运行后最后效果如下图所示：

![alt text](/img/CoreDataNSFetchedResultsController/7.png)

如图所示，我删除了卫衣这个商品，由于服装只有一个商品，所以这个section也被删除了，如下所示：

![alt text](/img/CoreDataNSFetchedResultsController/8.png)

[ 完整源码 ](https://github.com/LynchWong/OwnCoreDataStack/tree/NSFetchedResultsController)。

这些源码由之前的Demo修改而来，也上传到了之前Demo的NSFetchedResultsController分支。
