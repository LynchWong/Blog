---
title: "自定义iOS控件"
date: 2014-12-17 11:40:47
categories: 
- iOS
tags: 
- iOS自定义控件
metaAlignment: center
autoThumbnailImage: no
coverImage: //d1u9biwaxjngwg.cloudfront.net/welcome-to-tranquilpeak/city.jpg

---

刚开始学习iOS开发的时候接触了很多Cocoa Touch的标准控件，那时就觉得这些控件好用是好用，但是可定制性不高。往往我们的需求又比较变态，特别是控件外观方面基本达不到需求，所以很多时候我们需要自己做控件。
<!--more-->

不过很多需求我们可以通过组合已经存在的原生控件来达到，但是还是有一些需求不能实现。比如我们要实现一个类似汽车驾驶室的界面，这个界面的功能不要太复杂：

* 能控制汽车的方向，我们可以做一个方向盘的控件；
* 能控制汽车前进、后退、停止，我们还需要一个油门的控件；这个我们可以把油门和变速器用一个控件实现。

这个界面的实现思路：

* 首先可以给这个界面一个汽车驾驶室的图片，当作背景，图片内容主要就是驾驶室方向盘那部分，可以让美工设计下；
* 其次我们把方向盘放在界面右边，如果背景图片有仪表盘，最好方向盘和仪表盘有对应。界面右边我们放油门这个空间；
* 操作方式可以这样来设定，油门上有个按钮，往前拖就是前进，越拖到上面速度越快；然后有个N档，拖到这里就是停止；有个R档，拖到这里就后退。方向盘就是根据手指在控件上的滑动来计算出角度，然后根角度进行旋转，我们还可以给方向盘加上重力感应来控制方向盘。

思路大概就是上面这样，具体的实现和细节还要再讨论。这里我只做“油门”这个控件的自定义，接下来我们就一步一步的来实现。方向盘这个控件大家可以自行实现一下，后面可能会贴出源码（可能贴出来）。

这里有一篇很好的[博文](http://beyondvincent.com/blog/2014/01/20/how-to-build-a-custom-control-in-ios/)，就是讲解怎么在iOS中自定义控件的，所以我们根据这篇博文来实现我们的“油门”控件。

现在我们就一步一步的开始实现“油门”控件，OS X是10.10.1，Xcode是6.6.1。我们新建一个项目，名字叫做“AcceleratorView”，我们使用OC开发。建好后的项目截图如下所示：

![alt text](/img/iOSSelfDefine/1.png)

新建一个文件`Accelerator`，继承自`UIControl`。

然后我们打开`Main.storyboard`文件，拖拽一个`View`控件到`View Controller`里面。如下图所示：

![alt text](/img/iOSSelfDefine/2.png)

然后我们将`Class`属性改成我们刚创建的，如下图所示：

![alt text](/img/iOSSelfDefine/3.png)

接下来就是为`View`控件添加约束，我们大小设置为200 x 580，x 和 y分别是5 和 20。首先设置长宽的约束，如下图所示：

![alt text](/img/iOSSelfDefine/4.png)

设置完成之后Xcode会有红色的警示线，表示添加的约束无法确定`View`的位置，如下图所示：

![alt text](/img/iOSSelfDefine/5.png)

所以我们还要继续添加约束，添加x和y的约束，如下图所示：

![alt text](/img/iOSSelfDefine/6.png)

添加完成后红色的虚线消失，如下图所示：

![alt text](/img/iOSSelfDefine/7.png)

大家可以试下不为`View`添加约束，然后运行会发现视图错乱，那是因为iOS8、Xcode6新建的项目使用了`Size Classes`，是为多种设备做适配用的。iOS的设备越来越多，所以适配是必须要做的，所以`AutoLayout`和`Size Classes`是必备技能了。这里就不说了，以后有时间也要搞一篇。这时候运行模拟器还没有什么东西，让我们继续。

在`ViewController.m`文件里面添加如下三个方法：

        - (BOOL)shouldAutorotate
        {
            return YES;
        }

        - (NSUInteger)supportedInterfaceOrientations
        {
            return UIInterfaceOrientationMaskLandscapeLeft | UIInterfaceOrientationMaskLandscapeRight;
        }
        
        - (BOOL)prefersStatusBarHidden
		{
    		return YES; //返回NO表示要显示，返回YES将hiden
		}
        
前两个方法会让当前显示的Controller支持横屏显示。若我要跳转到下一个Controller，而下一个Controller要求竖屏显示，那么我们同样实现上述两个方法，只不过`- (NSUInteger)supportedInterfaceOrientations`方法返回`UIInterfaceOrientationMaskPortrait`。或者完全不实现这两个方法，默认就会竖屏显示。第三个方法会隐藏状态栏。

接下来我们的主要工作就是实现`Accelerator`。

    @property (strong, nonatomic) UIImage *accButtonImage;
    @property (assign, nonatomic) CGPoint accButtonPosition;
    
新建两个属性，第一个属性表示按钮图片，第二个属性就是按钮的位置。至于为什么要有这两个属性，一开始我做的时候也不是就想好了要有这两个属性，一开始做的时候只有个大概的思路，很多东西和细节都是在做的过程中想到或者遇到的，经过不断修改而来的。所以有时候最重要的就是要动手开始做，去做了才知道。

这个“油门”控件要做成什么样的，大家都没看过，所以说有思路是不可能的，所以我们先看下效果图，然后来分析分析，应该怎么做，有个大体的思路。

![alt text](/img/iOSSelfDefine/8.png)

上图就是完成后的效果图，当然不是很美观，跟我项目里完整的还是有很大的差别，这里我们只实现这一部分。首先控件的按钮是一张图片，其他的部分全都是在`- (void)drawRect:(CGRect)rect`方法中绘图绘上去的。N、和R分别标示不同的档位，即空档和倒档，效果分别是制动和后退。上面的档位表示前进，档位越高表示速度越快。拖动按钮来变换档位，而且标示档位的线的颜色也会改变。需求就是这些了，看到效果图，实现也应该是比较简单的。

* 所以在控件里面应该有监测手指在控件上触摸和滑动的方法。
* 当手指在控件上滑动时，按钮要不停的改变位置，而且我们还要计算按钮的位置来判断是否要给标示档位的档位线上色。而且按钮这张图片，不是`addSubview:`到控件里面的，而是在`- (void)drawRect:(CGRect)rect`方法里面使用`UIKit` `UIImage`自带的方法`[self.accButtonImage drawAtPoint:self.accButtonPosition];`绘制到屏幕上的。同理，档位线的R、N字幕也是用`NSString`的`drawAtPoint:withFont:`绘制的。
* 当方法检测到控件上有手指滑动时，每次都需要重绘，大家都知道不能直接调用`- (void)drawRect:(CGRect)rect`方法，所以要调用`setNeedsDisplay`方法来重绘。

下面我们就直接上代码了。

    - (id)initWithCoder:(NSCoder *)aDecoder
    {
        if ((self = [super initWithCoder:aDecoder])) {
            [self initialize];
        }
        return self;
    }

    - (void)initialize
    {
        self.backgroundColor = [UIColor clearColor];
        self.accButtonImage = [UIImage imageNamed:@"accelerator_but"];
        self.accButtonPosition = CGPointMake((self.bounds.size.width - 54.0f) / 2, 224.0f);
    }
    
这是控件初始化的代码，很简单，就不讲解了。控件大小是固定的，所以我很多位置的代码都是写固定位置的，简单一点。

    - (BOOL)beginTrackingWithTouch:(UITouch *)touch withEvent:(UIEvent *)event
    {
        [super beginTrackingWithTouch:touch withEvent:event];
        
        CGPoint touchLocation = [touch locationInView:self];
        
        if (CGRectContainsPoint(CGRectMake(self.accButtonPosition.x - 73.0f, self.accButtonPosition.y, 200.0f, 32.0f), touchLocation)) {
            NSLog(@"YES");
            return YES;
        } else {
            NSLog(@"NO");
            [self calculatorButtonPosition:touchLocation];
            return YES;
        }
    }

    - (BOOL)continueTrackingWithTouch:(UITouch *)touch withEvent:(UIEvent *)event
    {
        [super continueTrackingWithTouch:touch withEvent:event];
        CGPoint touchLocation = [touch locationInView:self];
        [self calculatorButtonPosition:touchLocation];
        return YES;
    }

    - (void)endTrackingWithTouch:(UITouch *)touch withEvent:(UIEvent *)event
    {
    }
    
上面三个方法就是用来检测触摸和滑动的，只有前面的方法返回`YES`的时候才会调用下一个方法。方法首先调用父类的方法，然后获取手指当前在控件上的位置。第一个方法中`if`和`else`分支，一开始只有`if`分支，即手指在按钮内滑动的时候才会返回`YES`，然后调用`continueTrackingWithTouch`方法，既而计算按钮的位置，在`calculatorButtonPosition:touchLocation`中，我们计算出位置后，然后再调用`setNeedsDisplay`方法来重绘。`else`分支是为了满足项目需求加上的，即当手指接触到控件，即使没有在按钮内，也会马上计算出按钮位置，然后重绘。下面我们就来看看`calculatorButtonPosition:touchLocation`方法。

    - (void)calculatorButtonPosition:(CGPoint)touchLocation
    {
        CGFloat y = touchLocation.y;
        
        if (y <= 30.0f) {
            self.accButtonPosition = CGPointMake(self.accButtonPosition.x, 4.0f);
        } else if (y > 30.0f && y <= 50.0f) {
            self.accButtonPosition = CGPointMake(self.accButtonPosition.x, 24.0f);
        } else if (y > 50.0f && y <= 70.0f) {
            self.accButtonPosition = CGPointMake(self.accButtonPosition.x, 44.0f);
        } else if (y > 70.0f && y <= 90.0f) {
            self.accButtonPosition = CGPointMake(self.accButtonPosition.x, 64.0f);
        } else if (y > 90.0f && y <= 110.0f) {
            self.accButtonPosition = CGPointMake(self.accButtonPosition.x, 84.0f);
        } else if (y > 110.0f && y <= 130.0f) {
            self.accButtonPosition = CGPointMake(self.accButtonPosition.x, 104.0f);
        } else if (y > 130.0f && y <= 150.0f) {
            self.accButtonPosition = CGPointMake(self.accButtonPosition.x, 124.0f);
        } else if (y > 150.0f && y <= 170.0f) {
            self.accButtonPosition = CGPointMake(self.accButtonPosition.x, 144.0f);
        } else if (y > 170.0f && y <= 190.0f) {
            self.accButtonPosition = CGPointMake(self.accButtonPosition.x, 164.0f);
        } else if (y > 190.0f && y <= 210.0f) {
            self.accButtonPosition = CGPointMake(self.accButtonPosition.x, 184.0f);
        } else if (y > 210.0f && y <= 270.0f) {
            self.accButtonPosition = CGPointMake(self.accButtonPosition.x, 224.0f);
        } else if (y > 270.0f) {
            self.accButtonPosition = CGPointMake(self.accButtonPosition.x, 264.0f);
        }
        
        [self setNeedsDisplay];
    }
    
这个方法也很简单，主要就是根据手指触摸的位置的y值和档位线的y值比较来计算按钮的位置。

    - (int)calculatorLineIndex:(CGPoint)buttonPosition
    {
        CGFloat y = buttonPosition.y + 16.0f;
        int index = 0;
        
        if (y <= 30.0f) {
            index = 0;
        } else if (y > 30.0f && y <= 50.0f) {
            index = 1;
        } else if (y > 50.0f && y <= 70.0f) {
            index = 2;
        } else if (y > 70.0f && y <= 90.0f) {
            index = 3;
        } else if (y > 90.0f && y <= 110.0f) {
            index = 4;
        } else if (y > 110.0f && y <= 130.0f) {
            index = 5;
        } else if (y > 130.0f && y <= 150.0f) {
            index = 6;
        } else if (y > 150.0f && y <= 170.0f) {
            index = 7;
        } else if (y > 170.0f && y <= 190.0f) {
            index = 8;
        } else if (y > 190.0f && y <= 210.0f) {
            index = 9;
        } else if (y > 210.0f && y <= 270.0f) {
            index = 10;
        } else if (y > 270.0f) {
            index = 11;
        }
        
        return index;
    }
    
类似的，上面的方法是用来计算当前按钮在哪个档位线，然后在重绘方法里面决定那些档位线需要上色。

    - (void)drawRect:(CGRect)rect
    {
        [super drawRect:rect];
        NSLog(@"drawRect");
        int buttonIndex = [self calculatorLineIndex:self.accButtonPosition];
        //NSLog(@"buttonIndex : %d", buttonIndex);
        
        [[UIColor whiteColor] set];
        CGContextRef currentContext = UIGraphicsGetCurrentContext();
        CGContextSetLineWidth(currentContext, 1.0f);
        
        for (int i = 0; i < 12; i++) {
            
            if (i < 10) {
                
                if (i >= buttonIndex) {
                    if (i == 0) {
                        [[UIColor colorWithHexString:@"#ff0000"] set];
                    } else if (i == 1) {
                        [[UIColor colorWithHexString:@"#ed5002"] set];
                    } else if (i == 2) {
                        [[UIColor colorWithHexString:@"#f25102"] set];
                    } else if (i == 3) {
                        [[UIColor colorWithHexString:@"#ed5d03"] set];
                    } else if (i == 4) {
                        [[UIColor colorWithHexString:@"#f27202"] set];
                    } else if (i == 5) {
                        [[UIColor colorWithHexString:@"#f18a02"] set];
                    } else if (i == 6) {
                        [[UIColor colorWithHexString:@"#f2a102"] set];
                    } else if (i == 7) {
                        [[UIColor colorWithHexString:@"#f2b702"] set];
                    } else if (i == 8) {
                        [[UIColor colorWithHexString:@"#ecc202"] set];
                    } else if (i == 9) {
                        [[UIColor colorWithHexString:@"#f2c602"] set];
                    }
                } else {
                    [[UIColor whiteColor] set];
                }
                
                CGFloat leftStartX = 0.0f;
                CGFloat rightEndX = 0.0f;
                
                if (i != 9) {
                    leftStartX = 42.5f;
                    rightEndX = 157.5f;
                } else {
                    leftStartX = 2.5f;
                    rightEndX = 197.5f;
                }
                
                CGContextMoveToPoint(currentContext, leftStartX, 20.0f * (i + 1));
                CGContextAddLineToPoint(currentContext, 87.5f, 20.0f * (i + 1));
                CGContextStrokePath(currentContext);
                
                CGContextMoveToPoint(currentContext, 112.5f, 20.0f * (i + 1));
                CGContextAddLineToPoint(currentContext, rightEndX, 20.0f * (i + 1));
                CGContextStrokePath(currentContext);
            } else {
                if (i == 10) {
                    
                    if (i == buttonIndex) {
                        [[UIColor colorWithHexString:@"#ff0000"] set];
                    } else {
                        [[UIColor whiteColor] set];
                    }
                    NSString *string = @"N";
                    [string drawAtPoint:CGPointMake(20.0f, 10.0f + 20.0f * (i  + 1)) withFont:[UIFont systemFontOfSize:20.0f]];
                    [string drawAtPoint:CGPointMake(165.5f, 10.0f + 20.0f * (i  + 1)) withFont:[UIFont systemFontOfSize:20.0f]];
                    
                    CGContextMoveToPoint(currentContext, 42.5f, 20.0f * (i  + 2));
                    CGContextAddLineToPoint(currentContext, 87.5f, 20.0f * (i  + 2));
                    CGContextStrokePath(currentContext);
                    
                    CGContextMoveToPoint(currentContext, 112.5f, 20.0f * (i  + 2));
                    CGContextAddLineToPoint(currentContext, 157.5f, 20.0f * (i  + 2));
                    CGContextStrokePath(currentContext);
                }
                if (i == 11) {
                    
                    if (i == buttonIndex) {
                        [[UIColor colorWithHexString:@"#00ff00"] set];
                    } else {
                        [[UIColor whiteColor] set];
                    }
                    
                    NSString *string = @"R";
                    [string drawAtPoint:CGPointMake(20.0f, 10.0f + 20.0f * (i  + 2)) withFont:[UIFont systemFontOfSize:20.0f]];
                    [string drawAtPoint:CGPointMake(165.5f, 10.0f + 20.0f * (i  + 2)) withFont:[UIFont systemFontOfSize:20.0f]];
                    
                    CGContextMoveToPoint(currentContext, 42.5f, 20.0f * (i  + 3));
                    CGContextAddLineToPoint(currentContext, 87.5f, 20.0f * (i  + 3));
                    CGContextStrokePath(currentContext);
                    
                    CGContextMoveToPoint(currentContext, 112.5f, 20.0f * (i  + 3));
                    CGContextAddLineToPoint(currentContext, 157.5f, 20.0f * (i  + 3));
                    CGContextStrokePath(currentContext);
                }
            }
        }
        [self.accButtonImage drawAtPoint:self.accButtonPosition];
    }

`- (void)drawRect:(CGRect)rect`方法也很简单，主要复杂的就是业务逻辑，也没多大好讲解的。这里有个比较坑的地方就是`- (void)drawRect:(CGRect)rect`方法何时调用的问题。之前有个需求，需要重绘控件，调用`setNeedsDisplay`方法后并没有立即重绘。而且我在一个10次循环里面调用了10次`setNeedsDisplay`方法，但是最终只调用了一次`- (void)drawRect:(CGRect)rect`方法。最后在Stack Overflow上找到了方法：

    [self setNeedsDisplay];
    [[NSRunLoop currentRunLoop] runMode:NSDefaultRunLoopMode beforeDate:[NSDate date]];
    
上面的方法会立即重绘。

项目代码：[AcceleratorView](https://github.com/LynchWong/AcceleratorView)
