---
title: "UIButton传递参数"
date: 2015-08-09 16:23:22
categories: 
- Code issue
tags: 
- Swift
metaAlignment: center
autoThumbnailImage: no
coverImage: //d1u9biwaxjngwg.cloudfront.net/welcome-to-tranquilpeak/city.jpg

---

今天开发自己项目的时候有个需求，需要**UIButton**的Action传递多个参数。项目里的实际需求是需要传递**NSIndexPath**，如果只需要section或者row那很好办，设置tag就可以了。但是两个都需要，所以就要传递对象过去了。这里就涉及到了OBJC运行时，使用关联对象就很好办了。
<!--more-->

[ 【iOS】UIButton 传递多个参数的方法 -----使用关联函数 ](http://blog.csdn.net/chenglibin1988/article/details/27549925)
[How to add a method with multiple parameters to the selector of a button](http://stackoverflow.com/questions/13397258/how-to-add-a-method-with-multiple-parameters-to-the-selector-of-a-button)

这里给出两个连接，讲解的很详细，我就不罗嗦了，代码也很简单。

最后贴出我自己的部分代码，Swift版的，仅作参考：

	var AssociatedKey = "IndexPath"
	objc_setAssociatedObject(cell!.checkButton, &AssociatedKey, indexPath, UInt(OBJC_ASSOCIATION_RETAIN_NONATOMIC))
	
	func uncheck(sender: UIButton) {
        var indexPath = objc_getAssociatedObject(sender, &AssociatedKey) as! NSIndexPath
	}
