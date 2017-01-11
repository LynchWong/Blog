---
title: "storyboard、xib、AutoLayout实现UILabel自动换行"
date: 2015-10-22 22:23:20
categories: 
- Code issue
tags: 
- UILabel自动换行
metaAlignment: center
autoThumbnailImage: no
coverImage: //d1u9biwaxjngwg.cloudfront.net/welcome-to-tranquilpeak/city.jpg

---

UILabel自动换行的这种需求随处可见的，基本上大家都会，几行代码就能解决。

大多数开发者都是使用代码实现这种需求的，当然使用自动布局，xib，storyboard也能实现，接下来就来看看这两种方法。
<!--more-->

新建一个项目，然后在SB中拖放一个UILabel，如下所示：

![alt text](/img/AXSUILabelHuanHang/1.png)

布局如图中所示，居中在ViewController中，然后我们@IBOutlet到ViewController。

在**viewDidLoad()**方法中设置UILabel的文本信息，尽量设置超出长度。运行后如下所示：

![alt text](/img/AXSUILabelHuanHang/2.png)

# 自动布局换行 #

现在我们来实现这以功能：

1. 设置Lines为0；
2. 设置UILabel的高为0；
3. 设置高度的约束小于等于1200；

截图如下：

![alt text](/img/AXSUILabelHuanHang/3.png)
![alt text](/img/AXSUILabelHuanHang/4.png)
![alt text](/img/AXSUILabelHuanHang/5.png)

运行后截图如下：

![alt text](/img/AXSUILabelHuanHang/6.png)

# 代码实现换行 #

在**ViewController**的**viewDidLoad()**方法中添加如下代码：

    let anotherLabel = UILabel(frame: CGRect(x: 0.0, y: 20.0, width: 192, height: 21))
    anotherLabel.text = "Do any additional setup after loading the view, typically from a nib."
    anotherLabel.numberOfLines = 0
    anotherLabel.textAlignment = NSTextAlignment.Center
    view.addSubview(anotherLabel)
    
    let attributesDic = [NSFontAttributeName: anotherLabel.font]
    let labelSize = (anotherLabel.text! as NSString).boundingRectWithSize(CGSize(width: 192, height: CGFloat.max),
        options: NSStringDrawingOptions.UsesLineFragmentOrigin, attributes: attributesDic, context: nil).size
    anotherLabel.frame = CGRect(x: 0.0, y: 20.0, width: labelSize.width, height: labelSize.height)

运行后效果如下所示：

![alt text](/img/AXSUILabelHuanHang/7.png)

[ 完整源码 ](https://github.com/LynchWong/UILabelDemo)
