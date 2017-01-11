---
title: "iOS多线程"
date: 2015-04-10 15:33:17
categories: 
- iOS
tags: 
- iOS多线程
metaAlignment: center
autoThumbnailImage: no
coverImage: //d1u9biwaxjngwg.cloudfront.net/welcome-to-tranquilpeak/city.jpg

---

<!--more-->

之前的一个系列全是Swift相关的，基本上也就是做了搬运工的工作。勘误了一部分代码语法错误，相关资料参考自 [这里](https://github.com/CocoaChina-editors/Welcome-to-Swift)，资料算是比较旧的，跟[苹果官方](https://developer.apple.com/library/prerelease/ios/documentation/Swift/Conceptual/Swift_Programming_Language/index.html#//apple_ref/doc/uid/TP40014097)的还是有很多不一样的地方，语法，内容等。最新的Swift 1.2也在近日发布，官方又做了相应的修改、更新。所以后面等到有时间或者Swift相对稳定后再进行勘误，重新整理Swift的这个系列。

在完成上一篇博文之后到现在差不多一个多月的时间，这段时间一直在看**SpriteKit**相关的资料。鄙人虽然是iOS App开发，但是一直都梦想着做独立游戏开发。在iOS5.0、6.0时还没有**SpriteKit**这个东西，就学习过Cocos2d-iPhone，买过不少书进行系统学习，但是一直没有作品出来。后来想想还是先把App开发做好，就搁置了一段时间。再后来2D游戏引擎市场发生了大得改变，由于跨平台的需求，Cocos2d-x的崛起，以及脚本语言lua的简单、热更新特点，Cocos2d-iPhone基本上已经没有多少份额了。而在iOS7.0的时候苹果放出了**SpriteKit**，在8.0的时候更是放出了SceneKit和Metal。随着移动设备硬件的升级，3D游戏也逐渐步入了移动设备，Unity3D就是移动3D游戏引擎中的一匹大黑马。

经过深思熟虑后在这些引擎当中选择了**SpriteKit**，这一个月对**SpriteKit**进行了系统的学习。看的就是这本书 [iOS Games by Tutorials Second Edition](http://www.raywenderlich.com/store/ios-games-by-tutorials)，我想很多iOS开发者都知道这个人的网站，上面有很多很好的教程，国内很多博主都翻译过他得文章。这本书就是他和他得小伙伴一起编辑的，主要讲得就是苹果的**SpriteKit**。是全英文的，但是都是很简单的词汇，想必只要是iOS的开发者都能看懂。后期我会发布一些**SpriteKit**的教程给大家，建议大家都去看原版书籍，支持正版。如果对Cocos2d比较熟悉的话，**SpriteKit**上手是很快的，毕竟很多概念都是相通的。

其实这篇博文是讲iOS多线程的，一不小心就扯远了。

在继续这篇博文之前先推荐一位博主写得iOS多线程的文章，该有的基本都有了，写得很好。[请戳这里](http://blog.csdn.net/totogo2010/article/details/8010231)

写这篇博文的目的就是总结一下，方便自己翻阅，因为前两天开发的时候需要用GCD得时候发现都快忘的差不多了。所以就想起要写这篇博文，也算是系统学习下。除了GCD还有NSThread和NSOperation，让我们从官方文档开始。

## 关于多线程编程

多年来，计算机的最大性能主要受限于它的中心微处理器的速度。由于单个芯片处理速度开始达到性能瓶颈，芯片制造商转向多核设计，让计算机具有了同时执行多个任务的能力。Mac OS X 利用了这些多核de优势，在任何时候都可以执行系统相关的任务，你自己的应用程序也可以通过多线程利用计算机的多核优势。

### 什么是多线程？

多线程是一个比较轻量级的方法来实现单个应用程序内多个代码执行路径。在系统级别，程序并排执行，系统分配到每个程序的执行时间基于它本身所需要的时间和其他程序所需的时间。不过在每一个程序内部，存在一个或多个执行线程，用来同时执行不同的任务。其实系统管理着这些线程的执行，调度他们运行在可用的内核上面，并在其他线程需要执行的时候抢险打断他们。

从技术角度来说，一个线程就是一个需要管理执行代码的内核级和应用级的数据结构的组合。内核级结构协调调度线程事件，并抢险调度线程到可用的内核上。应用级内核结构包含存储函数调用的调用堆栈和应用需要来管理和操纵线程属性和状态的结构。

在一个非并发应用程序，只有一个线程执行。这个线程开始、结束于你应用的main循环，一个个方法和函数实现了你应用的所有行为。相反，一个支持并发的应用程序以一个线程开始执行，在需要额外执行路径的时候创建一个或多个线程。在多线程应用提供了两个潜在的优势：

* 多个线程可以提高应用程序的感知响应。
* 多个线程可以提高应用程序在多核系统上的实时性能。

如果你得应用程序只有一个线程，那么这个线程就需要做所有的事情。它必须响应事件，更新应用程序的界面，并执行所有你应用程序需要实现功能所必需得计算。只有一个线程的问题就在于同一时间只能做一件事情。所以当你的计算需要花费很长时间完成的时候会发生什么？当你的代码忙于计算需要的数值的时候，你得应用程序停止了用户事件的响应，以及界面更新。如果这种情况持续的足够长，用户可能就觉得你得应用程序已经挂了，并且会尝试强行退出。如果你把自定义的计算移到新开的线程，那么你得应用程序就可以自由的、及时的响应用户的交互。

由于当今多核计算机的普及，在某种方式上多线程提供了一种提高程序性能的方式。多线程可以同时在不同的处理器内核上执行不同的任务，让应用程序在给定的某一时间内提升处理工作量成为可能。

当然，多线程不是提升应用程序性能的灵丹妙药。多线程带来好处的同时也伴随着潜在的问题。应用程序内多个代码执行路径给你的代码增加了一定量的复杂性。每一个线程都需要和其他的线程协调它的行为，防止破坏应用程序的状态信息。在应用程序内，线程间共享内存空间，访问相同的数据结构。如果两个线程同时操作相同的数据结构，一个线程有可能覆盖另外线程的改动导致破坏数据结构。即使有适当的保护，你仍然要注意由于编译器的优化导致你的代码产生微妙的Bug。

### 多线程的替代方法

你自己创建多线程代码的最大问题就是它们给你的代码增加了不确定性。多线程是较低层且复杂的方法来支持你的程序并发。如果你不能完全的理解你得设计选择带来的影响，你将会很容易的引发同步或者定时的问题，其范围从细微的行为变化到应用程序崩溃并破坏用户的数据。

你需要考虑的另一个因素是你是否真的需要多线程或并发。多线程解决了如何在同一个进程内并发的执行多路代码路径的问题。然而在很多情况下你是无法保证你所在做的工作是并发的。多线程引入带来大量的开销,包括内存消耗和CPU占用。你会发现这些开销对于你的工作而言实在太大，或者有其他方法会更容易实现。

苹果列出了很多多线程的替代方法，文章最后会贴出原文链接给大家。

### 线程支持

如果你已经有代码用过多线程，OS X和iOS提供了多种技术在你的应用中创建多线程。此外，两个系统都提供了管理和同步你需要在这些线程里面处理的工作。以下几个部分描述了一些你在OS X和iOS上面使用多线程的时候需要 注意的关键技术。

#### 线程包

虽然多线程的底层实现机制是Mach的线程,你很少(即使有)使用Mach级别的线程。相反，你会经常使用到更多易用的POSIX（可移植性操作系统接口）的API或者它的衍生工具。Mach的实现没有提供多线程的基本特征，但是包括抢占式的执行模型和调度线程的能力，所以它们是相互独立的。

列表 1-2 列举你可以在你的应用程序使用的线程技术。

![alt text](/img/iOSMThreading/1.png)

在应用层上，其他平台一样所有线程的行为本质上是相同的。线程启动之后,线 程就进入三个状态中的任何一个：运行(running)、就绪(ready)、阻塞(blocked)。如果一个线程当前没有运行，那么它不是处于阻塞，就是等待外部输入，或者已经准备就绪等待分配 CPU。线程持续在这三个状态之间切换，直到它最终退出或者进入中断 状态。

当你创建一个新的线程，你必须指定该线程的入口点函数(或Cocoa线程时候为入口点方法)。该入口点函数由你想要在该线程上面执行的代码组成。但函数返回的时候，或你显式的中断线程的时候，线程永久停止，且被系统回收。因为线程创建需 要的内存和时间消耗都比较大，因此建议你的入口点函数做相当数量的工作，或建立一个运行循环允许进行经常性的工作。

#### Run Loops
一个 run loop 是用来在线程上管理事件异步到达的基础设施。一个 run loop 为线程监测一个或多个事件源。当事件到达的时候，系统唤醒线程并调度事件到 run loop，然后分配给指定程序。如果没有事件出现和准备处理，run loop 把线程置于休眠状态。
你创建线程的时候不需要使用一个 run loop，但是如果你这么做的话可以给用户带来更好的体验。Run Loops 可以让你使用最小的资源来创建长时间运行线程。因为 run loop 在没有任何事件处理的时候会把它的线程置于休眠状态，它消除了消耗CPU周期轮，并防止处理器本身进入休眠状态并节省电源。

为了配置 run loop，你所需要做的是启动你的线程，获取 run loop 的对象引用，设置你的事件处理程序，并告诉 run loop 运行。Cocoa和Carbon提供的基础设施会自动为你的主线程配置相应的 run loop。如果你打算创建长时间运行的辅助线程，那么你必须为你的线程配置相应的run loop。
关于 run loops 的详细信息和如何使用它们的例子会在“Run Loops”部分介绍。
<!--####同步工具
线程编程的危害之一是在多个线程之间的资源争夺。如果多个线程在同一个时间试图使用或者修改同一个资源，就会出现问题。缓解该问题的方法之一是消除共享资源，并确保每个线程都有在它操作的资源上面的独特设置。因为保持完全独立的资源是不可行的，所以你可能必须使用锁，条件，原子操作和其他技术来同步资源的访问。
锁提供了一次只有一个线程可以执行代码的有效保护形式。最普遍的一种锁是互斥排他锁，也就是我们通常所说的“mutex”。当一个线程试图获取一个当前已经被其他线程占据的互斥锁的时候，它就会被阻塞直到其他线程释放该互斥锁。系统的几个 框架提供了对互斥锁的支持，虽然它们都是基于相同的底层技术。此外 Cocoa 提供了 几个互斥锁的变种来支持不同的行为类型，比如递归。获取更多关于锁的种类的信息，请阅读“锁”部分内容。
除了锁，系统还提供了条件，确保在你的应用程序任务执行的适当顺序。一个条件作为一个看门人，阻塞给定的线程，直到它代表的条件变为真。当发生这种情况的时候，条件释放该线程并允许它继续执行。POSIX 级别和基础框架都直接提供了条件 的支持。(如果你使用操作对象，你可以配置你的操作对象之间的依赖关系的顺序确定任务的执行顺序，这和条件提供的行为非常相似)。

尽管锁和条件在并发设计中使用非常普遍，原子操作也是另外一种保护和同步访问数据的方法。原子操作在以下情况的时候提供了替代锁的轻量级的方法，其中你可以执行标量数据类型的数学或逻辑运算。原子操作使用特殊的硬件设施来保证变量的改变在其他线程可以访问之前完成。

获取更多关于可用同步工具信息,请阅读“同步工具”部分。
####线程间通信
虽然一个良好的设计最大限度地减少所需的通信量，但在某些时候，线程之间的通信显得十分必要。(线程的任务是为你的应用程序工作，但如果从来没有使用过这些工作的结果，那有什么好处呢?)线程可能需要处理新的工作要求，或向你应用程 序的主线程报告其进度情况。在这些情况下，你需要一个方式来从其他线程获取信息。幸运的是，线程共享相同的进程空间，意味着你可以有大量的可选项来进行通信。
###设计技巧
以下各节帮助你实现自己的多线程提供了指导，以确保你代码的正确性。部分指南同时提供如何利用你的线程代码获得更好的性能。任何性能的技巧，你应该在你更改你代码之前、期间、之后总是收集相关的性能统计数据。
####避免显式创建线程

手动编写线程创建代码是乏味的，而且容易出现错误，你应该尽可能避免这样做。 Mac OS X 和 iOS 通过其他 API 接口提供了隐式的并发支持。你可以考虑使用异步 API，GCD 方式，或操作对象来实现并发,而不是自己创建一个线程。这些技术背后为你做了线程相关的工作，并保证是无误的。此外，比如 GCD 和操作对象技术被设计用来管理线程，比通过自己的代码根据当前的负载调整活动线程的数量更高效。 关于更多 GCD 和操作对象的信息，你可以查阅“并发编程指南(Concurrency Programming Guid)”。
####保持你的线程合理的忙
如果你准备人工创建和管理线程，记得多线程消耗系统宝贵的资源。你应该尽最大努力确保任何你分配到线程的任务是运行相当长时间和富有成效的。同时你不应该害怕中断那些消耗最大空闲时间的线程。
####避免共享数据结构
避免造成线程相关资源冲突的最简单最容易的办法是给你应用程序的每个线程 一份它需求的数据的副本。当最小化线程之间的通信和资源争夺时并行代码的效果最好。
创建多线程的应用是很困难的。即使你非常小心，并且在你的代码里面所有正确的地方锁住共享资源，你的代码依然可能语义不安全的。比如，当在一个特定的顺序里面修改共享数据结构的时候，你的代码有可能遇到问题。以原子方式修改你的代码，来弥补可能随后对多线程性能产生损耗的情况。把避免资源争夺放在首位通常可以得到简单的设计同样具有高性能的效果。
####多线程和你的用户界面
如果你的应用程序具有一个图形用户界面，建议你在主线程里面接收和界面相关的事件和初始化更新你的界面。这种方法有助于避免与处理用户事件和窗口绘图相关的同步问题。一些框架，比如 Cocoa，通常需要这样操作，但是它的事件处理可以不这样做，在主线程上保持这种行为的优势在于简化了管理你应用程序用户界面的逻辑。
有几个显著的例外，它有利于在其他线程执行图形操作。比如,QuickTime API 包含了一系列可以在辅助线程执行的操作，包括打开视频文件，渲染视频文件，压缩 视频文件，和导入导出图像。类似的，在 Carbon 和 Cocoa 里面，你可以使用辅助线程来创建和处理图片和其他图片相关的计算。使用辅助线程来执行这些操作可以极大提高性能。如果你不确定一个操作是否和图像处理相关，那么你应该在主线程执行这 些操作。
关于 QuickTime 线程安全的信息,查阅 Technical Note TN2125:“QuickTime 的 线程安全编程”。关于 Cocoa 线程安全的更多信息，查阅“线程安全总结”。关于 Cocoa 绘画信息,查阅 Cocoa 绘画指南(Cocoa Drawing Guide)。
####了解线程退出时的行为
进程一直运行直到所有非独立线程都已经退出为止。默认情况下，只有应用程序的主线程是以非独立的方式创建的，但是你也可以使用同样的方法来创建其他线程。当用户退出程序的时候,通常考虑适当的立即中断所有独立线程，因为通常独立线程所做的工作都是是可选的。如果你的应用程序使用后台线程来保存数据到硬盘或者做其他周期行的工作，那么你可能想把这些线程创建为非独立的来保证程序退出的时候不丢失数据。
以非独立的方式创建线程(又称作为可连接的)你需要做一些额外的工作。因为大部分上层线程封装技术默认情况下并没有提供创建可连接的线程，你必须使用 POSIX API 来创建你想要的线程。此外，你必须在你的主线程添加代码，来当它们最 终退出的时候连接非独立的线程。更多有关创建可连接的线程信息，请查阅“设置线 程的脱离状态”部分。
如果你正在编程 Cocoa 的程序，你也可以通过使用 applicationShouldTerminate： 的委托方法来延迟程序的中断直到一段时间后或者完成取消。当延迟中断的时候，你 的程序需要等待直到任何周期线程已经完成它们的任务且调用了 replyToApplicationShouldTerminate:方法。关于更多这些方法的信息，请查阅NSApplication Class Reference。
####处理异常
当抛出一个异常时，异常的处理机制依赖于当前调用堆栈执行任何必要的清理。 因为每个线程都有它自己的调用堆栈，所以每个线程都负责捕获它自己的异常。如果 在辅助线程里面捕获一个抛出的异常失败，那么你的主线程也同样捕获该异常失败: 它所属的进程就会中断。你无法捕获同一个进程里面其他线程抛出的异常。

如果你需要通知另一个线程(比如主线程)当前线程中的一个特殊情况，你应该捕捉异常，并简单地将消息发送到其他线程告知发生了什么事。根据你的模型和你正在尝试做的事情，引发异常的线程可以继续执行(如果可能的话)，等待指示，或者 干脆退出。
####干净地中断你的线程
线程自然退出的最好方式是让它达到其主入口结束点。虽然有不少函数可以用来立即中断线程，但是这些函数应仅用于作为最后的手段。在线程达到它自然结束点之前中断一个线程阻碍该线程清理完成它自己。如果线程已经分配了内存，打开了文件，或者获取了其他类型资源，你的代码可能没办法回收这些资源，结果造成内存泄漏或者其他潜在的问题。
 
关于更多正确退出线程的信息,请查阅“中断线程”部分。
####线程安全的库
虽然应用程序开发人员控制应用程序是否执行多个线程，类库的开发者则无法这 样控制。当开发类库时，你必须假设调用应用程序是多线程，或者多线程之间可以随时切换。因此你应该总是在你的临界区使用锁功能。
对类库开发者而言，只当应用程序是多线程的时候才创建锁是不明智的。如果你 需要锁定你代码中的某些部分，早期应该创建锁对象给你的类库使用，更好是显式调 用初始化类库。虽然你也可以使用静态库的初始化函数来创建这些锁，但是仅当没有其他方式的才应该这样做。执行初始化函数需要延长加载你类库的时间，且可能对你 程序性能造成不利影响。
如果你真正开发 Cocoa 的类库,那么当你想在应用程序变成多线程的时候收到通知的话，你可以给NSWillBecomeMultiThreadedNotification 注册一个观察者。不 过你不应用依赖于这些收到的通知，因为它们可能在你的类库被调用之前已经被发出 了。-->
官方文档里面还有很多其他的内容，我觉得介绍这些基础的给大家应该够用了（其他难的我表示也看不懂了）。接下来讲讲编码部分了。
iOS实现多线程的方式主要有三种：
1. NSThread
2. NSOperation
3. GCD

### NSThread的使用

#### 创建

有两种方法创建一个线程：

	- (id)initWithTarget:(id)target selector:(SEL)selector object:(id)argument
	+ (void)detachNewThreadSelector:(SEL)aSelector toTarget:(id)aTarget withObject:(id)anArgument
	
第一个方法是实例方法，第二个方法是类方法。

	[NSThread detachNewThreadSelector:@selector(doSomething:) toTarget:self withObject:nil];
	
	NSThread* myThread = [[NSThread alloc] initWithTarget:self  
                                        selector:@selector(doSomething:)  
                                        object:nil];  
	[myThread start]; 

#### 参数的意义

selector ：线程执行的方法，这个selector只能有一个参数，而且不能有返回值。
target   ：selector消息发送的对象
argument ：传输给target的唯一参数，也可以是nil
第一种方式会直接创建线程并且开始运行线程。
第二种方式是先创建线程对象，然后再运行线程操作，在运行线程操作前可以设置线程的优先级等线程信息。

PS：

用NSObject的类方法performSelectorInBackground:withObject:创建一个线程：

	[Obj performSelectorInBackground:@selector(doSomething) withObject:nil];
	
实例：

新建一个Single View Application。将ViewController.m的代码替换为如下代码：

	#import "ViewController.h"

	@interface ViewController ()

	@property (strong, nonatomic) UIImageView *imageVIew;

	@end

	@implementation ViewController

	@synthesize imageVIew = _imageVIew;

	- (void)viewDidLoad {
    	[super viewDidLoad];
    	// Do any additional setup after loading the view, typically from a nib.
    	_imageVIew = [[UIImageView alloc] init];
    	_imageVIew.frame = CGRectMake(0.0, CGRectGetMidY(self.view.frame), self.view.frame.size.width, self.view.frame.size.width / 2.1);
    	[self.view addSubview:_imageVIew];
    
    	[NSThread detachNewThreadSelector:@selector(downloadImage) toTarget:self withObject:nil];
	}

	- (void)downloadImage {
    	NSData *data = [[NSData alloc] initWithContentsOfURL:[NSURL URLWithString:@"http://lynchwong.com//img/iOSControlFlow/1.png"]];
    	UIImage *image = [[UIImage alloc]initWithData:data];
    	if(image == nil){
        
    	}else{
        	[self performSelectorOnMainThread:@selector(updateUI:) withObject:image waitUntilDone:YES];
    	}
	}

	- (void)updateUI:(UIImage*) image {
    	_imageVIew.image = image;
	}

	- (void)didReceiveMemoryWarning {
    	[super didReceiveMemoryWarning];
    	// Dispose of any resources that can be recreated.
	}

	@end
	
模拟器效果如下：启动一小断时间后图片才显示在屏幕上。

![alt text](/img/iOSMThreading/2.png)

### NSOperation的使用

使用NSOperation的方式也有两种：

* 一种是用定义好的两个子类：NSInvocationOperation 和 NSBlockOperation。
* 另一种是继承NSOperation：如果你也熟悉Java，NSOperation就和java.lang.Runnable接口很相似。和Java的Runnable一样，NSOperation也是设计用来扩展的，只需继承重写NSOperation的一个方法main。相当与java 中Runnalbe的Run方法。然后把NSOperation子类的对象放入NSOperationQueue队列中，该队列就会启动并开始处理它。

#### NSInvocationOperation例子：

继续使用之前的实例，只是多线程的实现方式改成NSInvocationOperation，ViewController.m的代码如下：

	#import "ViewController.h"

	@interface ViewController ()

	@property (strong, nonatomic) UIImageView *imageVIew;

	@end

	@implementation ViewController

	@synthesize imageVIew = _imageVIew;

	- (void)viewDidLoad {
    	[super viewDidLoad];
    	// Do any additional setup after loading the view, typically from a nib.
    	_imageVIew = [[UIImageView alloc] init];
    	_imageVIew.frame = CGRectMake(0.0, CGRectGetMidY(self.view.frame), self.view.frame.size.width, self.view.frame.size.width / 2.1);
    	[self.view addSubview:_imageVIew];
    
    	NSInvocationOperation *operation = [[NSInvocationOperation alloc] initWithTarget:self
                                                                            selector:@selector(downloadImage)
                                                                              object:nil];
    	NSOperationQueue *queue = [[NSOperationQueue alloc] init];
    	[queue addOperation:operation];
	}

	- (void)downloadImage {
    	NSData *data = [[NSData alloc] initWithContentsOfURL:[NSURL URLWithString:@"http://lynchwong.com//img/iOSControlFlow/1.png"]];
    	UIImage *image = [[UIImage alloc]initWithData:data];
    	if(image == nil){
        
    	}else{
        	[self performSelectorOnMainThread:@selector(updateUI:) withObject:image waitUntilDone:YES];
    	}
	}

	- (void)updateUI:(UIImage*) image {
    	_imageVIew.image = image;
	}

	- (void)didReceiveMemoryWarning {
    	[super didReceiveMemoryWarning];
    	// Dispose of any resources that can be recreated.
	}

	@end
	
效果和NSThread一样，这里就不贴图了。

#### 第二种方式

继承NSOperation，在.m文件中实现main方法，main方法编写要执行的代码即可。

1. 首先新建一个文件叫作“MyOperation”并且继承自NSOperation。
2. 在.m文件里面重写main方法，完成要执行的代码，即下载图片。
3. 在“MyOperation”定义一个协议叫作“SetImageDelegate”，定义方法“- (void)setImage:(UIImage *)image;”，同时声明属性“@property (strong, nonatomic) id<SetImage> delegate;”。
4. 在main方法中通过delegate调用setImage设置图片。
5. 修改多线程的实现代码，同时设置MyOperation的delegate属性为self。

MyOperation.h代码如下：

	#import <Foundation/Foundation.h>
	#import <UIKit/UIKit.h>

	@protocol SetImage <NSObject>

	- (void)setImage:(UIImage *)image;

	@end

	@interface MyOperation : NSOperation

	@property (strong, nonatomic) id<SetImage> delegate;

	@end
	
MyOperation.m代码如下：

	#import "MyOperation.h"

	@implementation MyOperation

	@synthesize delegate = _delegate;

	- (void)main {
    	NSData *data = [[NSData alloc] initWithContentsOfURL:[NSURL URLWithString:@"http://lynchwong.com//img/iOSControlFlow/1.png"]];
    	UIImage *image = [[UIImage alloc]initWithData:data];
    	if(image == nil){
        
    	}else{
        	if ([_delegate respondsToSelector:@selector(setImage:)]) {
        	    NSLog(@"%@", image);
            	[_delegate setImage:image];
        	}
    	}
	}

	@end
	
最后ViewController.m文件的代码如下：

	#import "ViewController.h"
	#import "MyOperation.h"

	@interface ViewController ()<SetImage>

	@property (strong, nonatomic) UIImageView *imageVIew;

	@end

	@implementation ViewController

	@synthesize imageVIew = _imageVIew;

	- (void)viewDidLoad {
    	[super viewDidLoad];
    	// Do any additional setup after loading the view, typically from a nib.
    	_imageVIew = [[UIImageView alloc] init];
    	_imageVIew.frame = CGRectMake(0.0, CGRectGetMidY(self.view.frame), self.view.frame.size.width, self.view.frame.size.width / 2.1);
    	[self.view addSubview:_imageVIew];
    
    	MyOperation *operation = [[MyOperation alloc] init];
    	operation.delegate = self;
    	NSOperationQueue *queue = [[NSOperationQueue alloc] init];
    	[queue addOperation:operation];
	}

	- (void)setImage:(UIImage *)image {
    	_imageVIew.image = image;
	}

	- (void)didReceiveMemoryWarning {
    	[super didReceiveMemoryWarning];
    	// Dispose of any resources that can be recreated.
	}

	@end
	
启动程序，效果跟之前一样。

#### 如何控制线程池中的线程数？

队列里可以加入很多个NSOperation, 可以把NSOperationQueue看作一个线程池，可往线程池中添加操作（NSOperation）到队列中。线程池中的线程可看作消费者，从队列中取走操作，并执行它。

通过下面的代码设置：

	[queue setMaxConcurrentOperationCount:5];

线程池中的线程数，也就是并发操作数。默认情况下是-1，-1表示没有限制，这样会同时运行队列中的全部的操作。

### GCD（Grand Central Dispatch）

GCD全称就是Grand Central Dispatch，是一个用来执行任务的强大工具。让你根据调用方的需求同步或者异步的执行任意的代码块。你可以用GCD来执行将近所有的用其他线程来执行的任务。而GCD的优势是使用起来更简单，执行任务更高效。

其实GCD相较于NSThread和NSOperation来说更简单，是属于更高层级的API，对开发者来说更简单，也是苹果推荐的多线程方式。

一个调度队列（dispatch queue）是一个对象结构，管理你提交给它的任务。所有的调度队列都是先进先出（FIFO）的数据结构，先添加的任务会先执行。GCD为你自动提供了一些调度队列，但是你也可以创建一些其他具有特定目的的队列。

表 3-1 列出了一些你应用程序可用的调度队列以及你怎样使用它们，如下图：

![alt text](/img/iOSMThreading/3.png)

1. Serial：串行队列（也称作private dispatch queues）按照任务增加到队列的顺序，每次只执行一个任务。当前执行的任务运行在不同的线程，是通过调度队列来管理。串行队列通常用来同步访问特定的资源。你可以创建任意多个串行队列，每一个串行队列相对于其他的串行队列都是并发执行的。换句话说，如果你创建了四个串行队列，每一个队列在同一时间只能执行一个任务，但是最多可以有四个任务执行，每一个任务来自于一个串行队列。
2. Concurrent：并行队列（也称作global dispatch queue）并发的执行一个或多个任务，同样也是按照任务添加到队列的顺序开始执行，任务完成的顺序是随机的。并发执行的任务运行在不同的线程上，通过调度队列来管理。而在任何给定的点，在执行的任务的准确数量是不定的，依赖于系统条件。在iOS5及以后，你可以通过指定“DISPATCH_QUEUE_CONCURRENT”作为队列类型来自己创建并行调度队列。此外，还有四个预先定义好的全局并行队列供你的应用程序使用。
3. Main dispatch queue：它是一个全局可用的串行队列，它在应用程序主线程上执行任务。

#### 常用的方法dispatch_async

为了避免界面在处理耗时的操作时卡死，比如读取网络数据，IO，数据库读写等，我们会在另外一个线程中处理这些操作，然后通知主线程更新界面。

用GCD实现这个流程的操作比前面介绍的NSThread  NSOperation的方法都要简单。代码模板如下：

	dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
       
        dispatch_async(dispatch_get_main_queue(), ^{
            
        });
    });
    
如果这样还不清晰的话，那我们还是用下载图片为例子，代码如下：
    
	dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        NSData *data = [[NSData alloc] initWithContentsOfURL:[NSURL URLWithString:@"http://lynchwong.com//img/iOSControlFlow/1.png"]];
        UIImage *image = [[UIImage alloc]initWithData:data];
        if(image != nil){
            dispatch_async(dispatch_get_main_queue(), ^{
                _imageVIew.image = image;
            });
        }
    });
    
代码比NSThread和NSOperation简洁很多，而且GCD会自动根据任务在多核处理器上分配资源，优化程序。

系统给每一个应用程序提供了四个并行调度队列。

	DISPATCH_QUEUE_PRIORITY_DEFAULT
	DISPATCH_QUEUE_PRIORITY_BACKGROUND
	DISPATCH_QUEUE_PRIORITY_HIGH
	DISPATCH_QUEUE_PRIORITY_LOW

这四个并行调度队列是全局的，它们只有优先级的不同。因为是全局的，我们不需要去创建。我们只需要通过使用函数dispath_get_global_queue去得到队列，如下：

	dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
	
#### dispatch_group_async的使用

	dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);  
	dispatch_group_t group = dispatch_group_create();  
	dispatch_group_async(group, queue, ^{  
    	[NSThread sleepForTimeInterval:1];  
    	NSLog(@"group1");  
	});  
	dispatch_group_async(group, queue, ^{  
    	[NSThread sleepForTimeInterval:2];  
    	NSLog(@"group2");  
	});  
	dispatch_group_async(group, queue, ^{  
    	[NSThread sleepForTimeInterval:3];  
    	NSLog(@"group3");  
	});  
	dispatch_group_notify(group, dispatch_get_main_queue(), ^{  
    	NSLog(@"updateUi");  
	});  
	dispatch_release(group);
	
dispatch_group_async是异步的方法，运行后可以看到打印结果：

	2012-09-25 16:04:16.737 gcdTest[43328:11303] group1
	2012-09-25 16:04:17.738 gcdTest[43328:12a1b] group2
	2012-09-25 16:04:18.738 gcdTest[43328:13003] group3
	2012-09-25 16:04:18.739 gcdTest[43328:f803] updateUi
	
每个一秒打印一个，当第三个任务执行后，upadteUi被打印。

#### dispatch_barrier_async的使用

dispatch_barrier_async是在前面的任务执行结束后它才执行，而且它后面的任务等它执行完成之后才会执行例子代码如下：

	dispatch_queue_t queue = dispatch_queue_create("gcdtest.rongfzh.yc", DISPATCH_QUEUE_CONCURRENT);  
	dispatch_async(queue, ^{  
    	[NSThread sleepForTimeInterval:2];  
    	NSLog(@"dispatch_async1");  
	});  
	dispatch_async(queue, ^{  
    	[NSThread sleepForTimeInterval:4];  
    	NSLog(@"dispatch_async2");  
	});  
	dispatch_barrier_async(queue, ^{  
    	NSLog(@"dispatch_barrier_async");  
    	[NSThread sleepForTimeInterval:4];  
	});  
	dispatch_async(queue, ^{  
    	[NSThread sleepForTimeInterval:1];  
    	NSLog(@"dispatch_async3");  
	});
	
打印结果：

	2012-09-25 16:20:33.967 gcdTest[45547:11203] dispatch_async1
	2012-09-25 16:20:35.967 gcdTest[45547:11303] dispatch_async2
	2012-09-25 16:20:35.967 gcdTest[45547:11303] dispatch_barrier_async
	2012-09-25 16:20:40.970 gcdTest[45547:11303] dispatch_async3
	
请注意执行的时间，可以看到执行的顺序如上所述。

#### dispatch_apply 

执行某个代码片段N次。

	dispatch_apply(5, globalQ, ^(size_t index) {
    	// 执行5次
	});
	
#### dispatch_once

可用于实现单列模式

	// 一次性执行：
	static dispatch_once_t onceToken;
	dispatch_once(&onceToken, ^{
     	// code to be executed once
	});
	
#### dispatch_after

延迟执行

	// 延迟2秒执行：
	double delayInSeconds = 2.0;
	dispatch_time_t popTime = dispatch_time(DISPATCH_TIME_NOW, delayInSeconds * NSEC_PER_SEC);
	dispatch_after(popTime, dispatch_get_main_queue(), ^(void){
     	// code to be executed on the main queue after delay
	});
