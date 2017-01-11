---
title: "Advanced Memory Management Programming Guide - Practical Memory Management"
date: 2016-02-28 15:03:11
categories: 
- 编程指南
tags: 
- 内存管理
metaAlignment: center
autoThumbnailImage: no
coverImage: //d1u9biwaxjngwg.cloudfront.net/welcome-to-tranquilpeak/city.jpg

---

[官方文档](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/MemoryMgmt/Articles/mmPractical.html#//apple_ref/doc/uid/TP40004447-SW1)
<!--more-->

# Practical Memory Management
(内存管理实战)

我们前面介绍了内存管理策略的一些基本概念。这些概念非常简单直接，但有些实战方面的做法可以使得内存管理更加容易，进而保证你的应用稳定而健壮，减少对系统资源的占用。

## Use Accessor Methods to Make Memory Management Easier
(使用访问方法使得内存管理更加容易)

如果你的类的属性是对象，你必须保证这个对象的值在你使用它的时候没有被dealloc。所以你必须在设置值的时候获得该对象的所有权。你还必须保证对这些对象所有权的放弃。

这个工作看似非常枯燥乏味，如果你坚持用访问方法，那么内存管理出现问题的几率就大大减少了。如果你在代码中对成员变量到处用retain和release，那么你基本上是做错了。

我们假定有一个Counter对象，你用它来实现计数。

    @interface Counter : NSObject
    @property (nonatomic, retain) NSNumber *count;
    @end;

这个属性声明了两个访问方法。通常而言，这些方法由编译器来合成，但如果你知道这些方法是如何实现的，还是会帮助你理解问题的。

在get访问器中，就是返回实例变量，所以没有retain或者release。

    - (NSNumber *)count {
        return _count;
    }

在set访问器中，如果其他的人和我们使用相同的规则，那么你必须假设新的count可能会随时被dealloc。因此你必须通过发送retain消息来取得新的count的所有权，进而确保dealloc不会发生。你还必须对旧值发送release消息。在Objective-C中给nil发送消息是没有问题的，因此就算没有设置_count，这个方法的实现仍旧可以工作。你必须在[newCount retain]之后对旧值发送release。如果新值和旧值是同一个对象，如果你先release了，然后再retain，恐怕为时已晚。因为对象已经dealloc了，retain没用了。

    - (void)setCount:(NSNumber *)newCount {
        [newCount retain];
        [_count release];
        // Make the new assignment.
        _count = newCount;
    }

### Use Accessor Methods to Set Property Values
(使用访问器方法来设置属性的值)

假设你要实现一个方法来复位计数器。有好几个可行的方法。第一种方法就是使用alloc创建NSNumber实例，然后调用release，使引用计数平衡。

    - (void)reset {
        NSNumber *zero = [[NSNumber alloc] initWithInteger:0];
        [self setCount:zero];
        [zero release];
    }

第二种方法是使用便利方法创建一个新的NSNumber对象。所以就不需要release和retain消息。

    - (void)reset {
        NSNumber *zero = [NSNumber numberWithInteger:0];
        [self setCount:zero];
    }

注意上面都用了set访问器。

下面的做法对于简单的情况而言肯定是没问题的。但是，它的实现绕开了set方法，这样做在特定情况下会导致内存泄漏(比如，当你忘了retain或者release，或者这个变量的内存管理发生了变化)。

    - (void)reset {
        NSNumber *zero = [[NSNumber alloc] initWithInteger:0];
        [_count release];
        _count = zero;
    }

还需要注意的是，如果你使用key-value observing，那么这种对于值的复位就跟KVO不兼容了。

### Don’t Use Accessor Methods in Initializer Methods and dealloc
(不要在初始化方法和dealloc方法中使用访问器方法)

不允许使用访问器方法来设置实例变量的地方就是初始化方法和dealloc方法。我们使用一个表示零的number对象来初始化一个counter对象，你可能会如下实现init方法：

    - init {
        self = [super init];
        if (self) {
            _count = [[NSNumber alloc] initWithInteger:0];
        }
        return self;
    }

为了让counter初始化为非0值，你可以实现一个initWithCount:方法：

    - initWithCount:(NSNumber *)startingCount {
        self = [super init];
        if (self) {
            _count = [startingCount copy];
        }
        return self;
    }

因为Counter类有一个成员对象实例，你就必须实现dealloc方法。这个方法通过发release消息来放弃对于所有其他对象的所有权，然后，在最后时刻调用super的实现：

    - (void)dealloc {
        [_count release];
        [super dealloc];
    }

## Use Weak References to Avoid Retain Cycles
(使用弱引用来避免retain环)

Retain一个对象实际上是对一个对象的强引用。一个对象在所有的强引用解除之前是不能被dealloc的，这导致一个“环形持有”的问题：两个对象相互强引用(可能是直接引用，也可能是通过其他对象间接地引用)。

下图的所示的对象关系就构成了一个环形持有。Document对象持有多个Page对象，每个Page对象又具有一个Document引用来指示它归属的文档。如果Document对象有Page的强引用，Page对象有Document的强引用，这样每个对象都不能被dealloc。在Page对象release之前Document对象的引用计数不能变为0，而如果Document对象存在，Page对象也无法被release。

Figure 1  An illustration of cyclical references

![alt text](/img/PracticalMemoryManagement/1.png)

解决环形持有的问题就是使用弱引用。所谓的弱引用，就是一种非持有的关系，源对象不会对它引用的对象进行持有。

为了实现上面的对象图，肯定是需要强引用的(如果只有弱引用，那么Page和Paragraph就没有了拥有者，就会被dealloc)。Cocoa设定了一个规则，那就是：父对象建立对子对象的强引用，而子对象只对父对象建立弱引用。

所以，在Figure 1中，document对象有对page对象的强引用，但是page对象有对document对象的弱引用。

Cocoa中弱引用的例子包括但不局限于：table data sources，outline view items，notification observers，miscellaneous targets和delegates。

当给弱引用的对象发送消息时需要小心。如果你给一个dealloc的弱引用对象发送消息，应用程序会崩溃。因此你必须判断对象是否有效。多数情况下，被弱引用的对象是知道其他对象对它的弱引用(比如环形持有的情形)，所以需要通知其他对象它自己的dealloc。比如，当你向Notification Center注册一个对象时，Notification Center对这个对象是弱引用的，并且在有消息需要通知到这个对象时，就发送消息给这个对象。当这个对象dealloc的时候，你必须向Notification Center取消这个对象的注册。这样，这个Notification Center就不会再发送消息给这个不存在的对象了。同样，当一个delegate对象被dealloc的时候，必须向其他对象发送一个setDelegate:消息，并传递nil参数，从而将代理的关系撤销。这些消息通常在对象的dealloc方法中发出。

## Avoid Causing Deallocation of Objects You’re Using
(避免dealloc你正在使用的对象)

Cocoa的所有权策略时这么说的：返回的对象，在调用者的调用方法中，始终保持有效。所以说，在当前方法内部，不必担心你收到的返回对象会被dealloc。对于你的代码而言，通过getter方法收到的返回对象是一个被缓存的实例，还是计算出来的实例。这并不重要，重要的是这个对象在你使用它的时候会一直有效。

这个策略并非放之四海而皆准，在有些情况下是特列，特别是下面两种情况：

1. 当一个对象从fundamental的集合类中删除的时候。

		heisenObject = [array objectAtIndex:n];
		[array removeObjectAtIndex:n];
		// heisenObject could now be invalid.

当一个对象从集合中删除的时候，系统会立刻调用对象的release方法(而不是autorelease)。如果这时候，这个集合是该对象的唯一属主，那么这个对象(本例中的heisenObject)会立刻被dealloc。

2. 当父对象被dealloc的时候：

		id parent = <#create a parent object#>;
		// ...
		heisenObject = [parent child] ;
		[parent release]; // Or, for example: self.parent = nil;
		// heisenObject could now be invalid.

有时候你从一个对象获取另外一个对象，然后直接或者间接的release了父对象。如果对父对象的release造成了它被dealloc，且这时候该父对象恰好是子对象的唯一属主，那么子对象(本例中的heisenObject)会同时也被dealloc(这里我们假定在父对象的dealloc方法中，对子对象发送的是release消息，而非autorelease)。

在这些情况下，你retain接收的heisenObject对象，然后完成工作后release。比如：

    heisenObject = [[array objectAtIndex:n] retain];
    [array removeObjectAtIndex:n];
    // Use heisenObject...
    [heisenObject release];

## Don’t Use dealloc to Manage Scarce Resources
(不要使用dealloc来管理稀缺资源)

通常，你不应该在dealloc方法中管理稀缺资源，比如文件句柄、网络连接、缓存等。更具体的说，你不可能设计出一个类，你想让系统什么时候调用dealloc，系统就什么时候调用(因为你能做的就是release，至于release是否会导致系统一定调用dealloc，还要看这个对象有没有其他属主)。因为系统性能的下降、系统自身的BUG，有可能dealloc的调用被推迟搁置。

正确的做法是，如果你的对象管理了稀缺资源，它就必须知道它什么时候不再需要这些资源，并在此时立即释放资源。通常情况下，此时你会调用release来dealloc，但是因为此前你已经释放了资源，这里就不会遇到任何问题。

如果你把资源管理的职能交给dealloc，那么会暴露很多问题，比如：

1. 对象图的拆除顺序。
实际上，对象图的拆除是没有任何顺序保证的。也许你认为、你希望有一个具体明确的顺序，但事实是没有。如果对象被放到了autorelease池，这个拆除的过程也会发生变化，并导致你无法预见的后果。
2. 系统稀缺资源不能回收。
内存泄漏是BUG，应该被修复，但这个问题不是立即就暴露出来的。如果没有保留的时候，你认为某个资源已经释放，而实际上没有释放，你就会面临更加严重的问题了。比如，如果文件句柄被用光了，其结果将是你无法保存数据。
3. 清除逻辑在错误的线程上执行。
如果对象在一个不确定的时刻被放到了autorelease池中，它将被线程池中的线程来dealloc。这对于有些只能由单一线程来访问的资源而言，是致命的错误。

## Collections Own the Objects They Contain
(集合拥有包含对象的所有权)

当你把一个对象添加到集合容器中(比如数组array、字典dictionary、集合set)，容器就会取得对象的所有权。当容器自己release的时候，或者对象从容器中删除时，容器会放弃该对象的所有权。比如，如果要建一个存放数字的数组，你可以按下面方法来做：

    NSMutableArray *array = <#Get a mutable array#>;
    NSUInteger i;
    // ...
    for (i = 0; i < 10; i++) {
        NSNumber *convenienceNumber = [NSNumber numberWithInteger:i];
        [array addObject:convenienceNumber];
    }

这时你并没有使用alloc，因此你也不必使用release。你不需要调用新数值的retain，因为数组会这么做。

    NSMutableArray *array = <#Get a mutable array#>;
    NSUInteger i;
    // ...
    for (i = 0; i < 10; i++) {
        NSNumber *allocedNumber = [[NSNumber alloc] initWithInteger:i];
        [array addObject:allocedNumber];
        [allocedNumber release];
    }

这种做法，我们在for循环内部向allocedNumber发送了与alloc相对应的release消息。因为Array的addObject:方法实际上对这个对象做了retain处理，那么这个对象(allocedNumber)不会因此而被dealloc。

我们换位思考一下，假定你自己就是这个集合类的作者。你要确保加入的对象只要继续存在于集合里，就不应该被dealloc，因此你在添加这个对象时，向它发送了retain消息，删除这个对象时，向它发送了release消息。当你这个集合类自己dealloc时，对容器内所有的对象发送release消息。

## Ownership Policy Is Implemented Using Retain Counts
(所有权策略通过引用计数来实现)

所有权策略是通过引用计数来实现的，通常称之为“retain count”。每个对象都有一个retain count。

* 当你创建一个对象时，它的retain count为1。
* 当你给对象发送retain消息，它的retain count增加1。
* 当你给对象发送release消息，它的retain count减少1。当你给对象发送autorelease消息，它的retain count将在未来某个时候减1。
* 如果对象的retain count减少到0，它就会被dealloc。

**重要：**其实你应该没有理由想知道一个对象的retain count。这个数值有时候会对你造成误导：实际上你不知道哪些框架retain了你感兴趣的对象。在调试内存问题的时候，你只需要遵守所有权规则就行了。
