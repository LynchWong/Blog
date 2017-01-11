---
title: "Debug/Release等编译模式"
date: 2015-10-25 00:07:53
categories: 
- Code issue
tags: 
- 编译模式
metaAlignment: center
autoThumbnailImage: no
coverImage: //d1u9biwaxjngwg.cloudfront.net/welcome-to-tranquilpeak/city.jpg

---

通常我们的App有多种不同的编译模式，比如**Debug**、**Release**。我们在开发的时候大都是**Debug**模式，发布的时候会选择**Release**模式。通常我们的服务请求地址也会根据编译模式来进行不同的选择。当然系统已经给我们定义好了这两种编译模式，但有时候我们不止需要两种，我可以自己添加额外的编译模式。
<!--more-->

假设我们有3种环境，开发、测试、发布。开发是开发人员使用的环境；测试是测试人员使用的环境；发布是App正式上线之后使用的环境。那么我们就需要3种编译模式，测试对应**Debug**、发布对应**Release**，那么还需要添加另外一种编译模式。这里我们假设就需要添加一种叫做**Develop**的编译模式。

我们新建一个项目，语言选择OC。然后我们替换**ViewController.m**文件内容如下：

    #import "ViewController.h"

    @interface ViewController ()

    @end

    @implementation ViewController

    #ifdef DEBUG
        #define kWidth 100.0f
    #elif RELEASE
        #define kWidth 200.0f
    #else
        #define kWidth 300.0f
    #endif

    - (void)viewDidLoad {
        [super viewDidLoad];
        // Do any additional setup after loading the view, typically from a nib.
        NSLog(@"kWidht : %f", kWidth);
    }

    - (void)didReceiveMemoryWarning {
        [super didReceiveMemoryWarning];
        // Dispose of any resources that can be recreated.
    }

    @end

这里我们在3种不同的环境下定义了**kWidth**常量，然后我们打印这个常量，运行之后输出如下结果：

	2015-10-25 13:48:01.217 DebugRelease[1193:76047] kWidht : 100.000000

项目新建之后我们没有对Scheme做任何更改，默认Scheme截图如下，模拟器运行是在**DEBUG**模式下：

![alt text](/img/iOSDebugReleas/1.png)

修改Scheme，将**Build Configuration**修改为Release，截图如下；

![alt text](/img/iOSDebugReleas/2.png)

运行之后控制台输出：

	2015-10-25 14:01:37.186 DebugRelease[1360:87151] kWidht : 300.000000

跟我们预想的有些不一样，按照逻辑来说应该输出：

	2015-10-25 14:05:10.454 DebugRelease[1430:90460] kWidht : 200.000000

接下来我们做下修改就好了，截图如下：

![alt text](/img/iOSDebugReleas/3.png)

如图中所示，我们在**Build Settings**中搜索macro，然后发现**Release**这一项是没值的，所以到了另外一个分支里面，输出了其他结果。**Debug**已经默认设置好了，所以我们这里设置一下值就好了，设置为**RELEASE=1 $(inherited)**，运行之后输出结果就对了。

这里我们还需要添加一种编译模式，如最开始的时候说的**Develop**模式。

![alt text](/img/iOSDebugReleas/4.png)

我们点击**+**号来添加一个编译模式，叫**Develop**。截图如下：

![alt text](/img/iOSDebugReleas/5.png)

然后我们修改Scheme为**Develop**，运行后输出：

	2015-10-25 14:19:55.663 DebugRelease[1555:100946] kWidht : 200.000000

这是因为我是按照**Release**模式来设置**Develop**的，如果我们去**Build Settings**里面去看就会发现**Develop**模式的值和**Release**的值是一样的，所以输出的结果也是一样的。如果你是按照按照**Debug**模式来设置**Develop**，那么输出的结果就是和**Debug**的值一样。所以我们这里修改下**Develop**的值，修改为**DEVELOP=1 $(inherited)**。

运行后的结果：

	2015-10-25 14:25:07.919 DebugRelease[1584:104028] kWidht : 300.000000

判断编译模式的代码也可以修改为如下代码：

    #ifdef DEBUG
        #define kWidth 100.0f
    #elif RELEASE
        #define kWidth 200.0f
    #elif DEVELOP
        #define kWidth 300.0f
    #endif

这种方式可能会导致没有**kWidth**这个常量，当其他的同事定义使用了其他的编译模式时就会出现这个问题。原来的代码始终会有一个**kWidth**常量，只是值可能不是你期望的。
