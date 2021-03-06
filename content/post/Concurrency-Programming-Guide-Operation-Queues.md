---
title: "Concurrency Programming Guide - Operation Queues"
date: 2016-01-14 13:51:30
categories: 
- 编程指南
tags: 
- 并发编程

---

[官方文档](https://developer.apple.com/library/prerelease/mac/documentation/General/Conceptual/ConcurrencyProgrammingGuide/OperationObjects/OperationObjects.html#//apple_ref/doc/uid/TP40008091-CH101-SW1)
<!--more-->

# Operation Queues
(操作队列)

Cocoa operations(Cocoa操作)使用了面向对象的方式封装了要异步执行的工作。操作被设计来与操作队列一起使用或是独立使用。因为它们是基于Objective-C的，操作在OS X和iOS中基于Cocoa的应用程序中是最常用的。

本章将展示如何定义和使用操作。

## About Operation Objects
(关于操作对象)

一个operation object(操作对象)是**NSOperation**类(在Foundation框架中)的实例，用来封装你想要应用程序执行的任务。**NSOperation**类是一个抽象基类，必需子类化后才能做一些有用的工作。尽管是抽象的，这个类确实提供了一些基本的功能以减少你子类化的工作量。除此之外，Foundation框架提供了两个具体的子类可供你的代码使用。表2 - 1列出了这两个类，及一些汇总。

Table 2-1  Operation classes of the Foundation framework

![alt text](/img/ConcurrencyProgrammingGuideOperationQueues/1.png)

* **NSInvocationOperation**：基于你应用程序的一个对象和selector来创建一个操作对象。如果你已经有存在的方法执行需要的任务，你可以使用这个类。因为这个类不要求子类化，所以你可以使用这个类动态的创建操作对象。
* **NSBlockOperation**：该类可以并发的执行一个或者多个Block对象。因为可以执行一个以上的Block，所以Block操作对象使用“组”的语义；只有所有这些相关的Block执行完成之后，操作对象才算完成。
* **NSOperation**：用来自定义子类操作对象的基类。子类化**NSOperation**让你有完全的控制权来实现你自己的操作，包括修改操作执行和状态报告的默认方式的能力。

所有的操作对象都支持如下的关键特性：

* 支持在操作对象之间建立依赖图。这些依赖可以阻止某个操作执行，直到它依赖的所有操作完成。
* 支持可选的回调Block，在主要的操作任务完成之后执行。
* 支持使用KVO通知监控操作的执行状态的改变。
* 支持给操作设定优先级，从而影响执行顺序。
* 支持取消语义，允许你停止正在执行的操作。

操作被设计来帮助你提升你应用程序的并发层级。同时也是将你应用程序的行为封装到简单代码块的好方法。比起在主线程上执行一些代码，你可以将一个或者多个操作对象提交到队列上，然后在一个或者多个线程上异步执行这些对应的工作。

## Concurrent Versus Non-concurrent Operations
(并发与非并发操作)

虽然你通常是将操作添加到操作队列来执行操作，但这样做并不是必需的。你也可以手动调用操作对象的**start**方法来执行操作，但是这样做并不能保证你余下代码运行的操作就会并发执行。**NSOperation**类的**isConcurrent**方法会告诉你这个操作相对于调用**start**方法的线程，是同步还是异步执行的。**isConcurrent**方法默认会返回**NO**，表示在调用线程上操作是同步执行的。

如果你想要实现并发操作，也就是说，调用异步运行的线程，你必需编写额外的代码来开始异步操作。比如，你可能会产生一个独立的线程，调用一个异步的系统函数，或者做一些其它的事情确保开始方法执行后这个任务会立即返回，很大的可能性是任务完成之前就返回了。

大多数开发者不需要去实现并发的操作对象。如果你总是将你的操作添加到操作队列当中，你不需要去实现并发的操作。当你把一个非并发的操作提交到操作队列，这个队列会自己创建一个线程来运行你的操作。因此，添加一个非并发的操作到操作队列，仍然会异步执行你的操作对象的代码。只有你不希望使用操作队列来执行操作时，你才需要定义并发操作对象。

更多关于如何创建并发操作，参见[  Configuring Operations for Concurrent Execution ](https://developer.apple.com/library/prerelease/mac/documentation/General/Conceptual/ConcurrencyProgrammingGuide/OperationObjects/OperationObjects.html#//apple_ref/doc/uid/TP40008091-CH101-SW8) 和 [ NSOperation Class Reference ](https://developer.apple.com/library/prerelease/mac/documentation/Cocoa/Reference/NSOperation_class/index.html#//apple_ref/doc/uid/TP40004591)。

## Creating an NSInvocationOperation Object
(创建NSInvocationOperation对象)

**NSInvocationOperation**类是一个**NSOperation**具体的子类，当运行的时候会调用你指定的对象的selector。使用这个类可以避免为每一个任务自定义操作对象；特别是你要修改的应用程序已经有执行必要任务所需的对象和方法。你也可以实现根据周围情况的改变来调用方法。比如，你可以用**NSInvocationOperation**的操作对象来根据用户的输入动态的执行selector。

创建**NSInvocationOperation**操作对象的过程很直观。创建初始化一个新实例，将需要执行的对象的selector方法传递给初始化方法。Listing 2-1展示了在一个自定义类中掩饰创建过程。**taskWithData:**方法创建了一个新的对象，然后提供给另一个方法。

Listing 2-1  Creating an NSInvocationOperation object

    @implementation MyCustomClass
    - (NSOperation*)taskWithData:(id)data {
        NSInvocationOperation* theOp = [[NSInvocationOperation alloc] initWithTarget:self
                                                                            selector:@selector(myTaskMethod:) object:data];
        
        return theOp;
    }

    // This is the method that does the actual work of the task.
    - (void)myTaskMethod:(id)data {
        // Perform the task.
    }
    @end

## Creating an NSBlockOperation Object
(创建NSBlockOperation对象)

**NSBlockOperation**也是**NSOperation**类的子类，包装了一个或者多个Block对象。这个类为那些已经在使用操作队列但是不想使用调度队列的应用程序提供了面向对象的封装。使用**NSBlockOperation**也可以使用操作依赖、KVO通知，以及其它一些调度队列不具有的功能。

当你创建一个**NSBlockOperation**的操作时，你通常在初始化的时候至少添加一个Block；之后需要的话也可以添加更多的Block。当要执行**NSBlockOperation**对象时，会将所有的Block以默认的优先级提交到并发的调度队列。然后**NSBlockOperation**对象会等待直到所有的Block执行完成。当最后一个Block执行完成，**NSBlockOperation**对象标记自己执行完成。因此，你可以使用**NSBlockOperation**操作对象来跟踪一组要执行的Block，就像你使用一个线程合并来自多个线程的结果。不同的就是，**NSBlockOperation**对象的操作运行在分开的线程上，你的应用程序的其它线程可以继续执行它们自己的任务，而不用等待**NSBlockOperation**操作对象完成。

Listing 2-2展示了一个创建简单**NSBlockOperation**对象的例子。这个Block对象没有参数和返回结果。

    NSBlockOperation* theOp = [NSBlockOperation blockOperationWithBlock: ^{
        NSLog(@"Beginning operation.\n");
        // Do some work.
	}];

在创建了一个**NSBlockOperation**操作对象后，你可以使用**addExecutionBlock:**方法来添加更多的Block。如果你需要串行的执行这些Block，你必需将这些Block提交到期望的调度队列上。

## Defining a Custom Operation Object
(自定义操作对象)

如果**NSBlockOperation**和**NSInvocationOperation**操作对象无法满足你应用程序的需求，那么你可以直接子类化**NSOperation**然后添加你自己需要的行为。**NSOperation**提供了一些通用的子类继承点。也提供了大量用来处理依赖和KVO通知的基础设施。然后，有时候你仍然需要实现已经存在的基础设施来确保你的操作行为是正确的。其它的额外工作取决于你实现的是否是并发操作。

定义非并发的操作比定义并发的操作要简单的多。定义非并发的操作，所有你必需做的是执行主要的任务，并对取消事件作出响应；已经存在的基础设施会为你完成剩下的所有工作。对于一个并发操作，你必需使用你的代码替换一些已经存在的基础设施。下面的部分会为你展示如何实现这两种对象。

### Performing the Main Task
(执行主任务)

每一个操作对象应该至少实现如下几个方法：

* 自定义的**initialization**方法。
* 自定义的**main**方法。

你需要一个自定义的**initialization**方法将你的操作对象设置为已知状态，自定义的**main**方法来执行你的任务。如有必要你也可以实现额外的方法，当然，比如如下方法：

* **main**方法中需要调用的其它自定义方法。
* 设置和访问操作对象数据的访问器方法。
* **NSCoding**协议中那些允许你归档和解档操作对象的方法。

Listing 2-3直观展示了子类化**NSOperation**的一个开始模版。(这个实例没有展示如何处理取消，但是展示了通常需要的方法。)这个类的初始化方法接收一个单一的数据对象作为参数，然后将引用存储在操作对象内。**main**方法在将结果返回给应用程序之前表面上看起来在处理这个数据对象。

Listing 2-3  Defining a simple operation object

    @interface MyNonConcurrentOperation : NSOperation
    @property id (strong) myData;
    -(id)initWithData:(id)data;
    @end

    @implementation MyNonConcurrentOperation
    - (id)initWithData:(id)data {
        if (self = [super init])
            myData = data;
        return self;
    }

    -(void)main {
        @try {
            // Do some work on myData and report the results.
        }
        @catch(...) {
            // Do not rethrow exceptions.
        }
    }
    @end

关于如何实现NSOperation的子类化详细例子，参见[ NSOperationSample ](https://developer.apple.com/library/prerelease/mac/samplecode/NSOperationSample/Introduction/Intro.html#//apple_ref/doc/uid/DTS10004184)。

### Responding to Cancellation Events
(响应取消事件)

在一个操作开始执行之后，它将会继续执行任务直到执行完成或者你代码显示的取消操作。取消可能在任何时间发生，甚至可能在操作执行之前。尽管**NSOperation**类为客户提供了一种取消操作的方式，但是识别出取消事件则是你的事情。如果一个操作被直接终止，可能没有办法收回已经分配的资源。因此，操作对象应该在操作执行中检查取消事件，并且优雅的退出执行。

操作对象为了支持取消，所有你需要做的就是你在自定义的代码上定期的调用**isCancelled**方法，如果返回**YES**就立即退出执行。支持取消是很重要的，不管是自定义的**NSOperation**或者是已经存在的两个**NSOperation**子类。**isCancelled**方法非常轻量，可以频繁的调用而不会产生任何显著的性能损失。在设计操作对象时应该在以下代码考虑调用**isCancelled**方法：

* 执行任何实际工作之前。
* 在循环的每次迭代过程中，如果每个迭代相对较长可能需要调用多次。
* 代码中相对比较容易终止操作的任何地方。

Listing 2-4简单展示了在一个操作对象的**main**方法中如何响应取消事件。在这种情形下，**isCancelled**方法每次通过while循环时都会调用，允许在工作开始之前快速退出，并以固定的周期间隔再一次开始。

Listing 2-4  Responding to a cancellation request

    - (void)main {
        @try {
            BOOL isDone = NO;
            
            while (![self isCancelled] && !isDone) {
                // Do some work and set isDone to YES when finished
            }
        }
        @catch(...) {
            // Do not rethrow exceptions.
        }
	}

尽管上面的代码没有包含清理的代码，但是你自己的代码应该确保释放了所有你自定义代码分配的资源。

### Configuring Operations for Concurrent Execution
(为并发执行配置操作对象)

操作对象默认按照同步方式执行，在调用**start**方法的那个线程中执行任务。因为操作队列为非并发的操作提供了线程，多数操作仍然是异步执行的。但是如果你希望手动执行操作，而且希望能够异步操作，那你就应该采取适当的措施。通过定义操作对象为并发的操作来实现。

Table 2-2列出了实现并发操作通常需要重写的方法。

Table 2-2  Methods to override for concurrent operations

![alt text](/img/ConcurrencyProgrammingGuideOperationQueues/2.png)

* **start**：(必须)所有的并发操作都必须重写这个方法，用自定义的实现替换掉默认的行为。通过调用**start**方法来手动执行一个操作。因此，这个方法的实现就是操作开始的起点，设置线程及可执行环境来执行你的任务。你的实现在任何时候都不能调用**super**的方法。
* **main**：(可选)通常这个方法用来实现与操作对象相关的任务。尽管你可以在**start**方法中执行你的任务，用**main**方法来实现任务可以将你设置任务和实现任务的代码分开。
* **isExecuting和isFinished**：(必须)并发操作负责设置它们的可执行环境，并且像外部的客户端报告执行环境的状态。因此，并发操作必需维护一些状态，当任务在执行或者完成时能够知道这些状态信息。必需使用这些方法报告状态信息。你实现的这些方法必需能够在其它线程中同时调用。还需要为相应的**Key paths**的值的改变产生KVO通知。
* **isConcurrent**：(必须)为了标示一个操作是并发操作，重写该方法返回**YES**。

接下来的部分展示了一个实现**MyOperation**类的简单案例，演示了要实现并发操作时所需要的基础代码。**MyOperation**在一个新的线程上简单的执行**main**方法。**main**方法执行的任务事实上是不相关的。这个案例的关键点是要向你演示定义并发操作时所需要提供的基础设施。

Listing 2-5展示了**MyOperation**类的部分接口和部分实现。**MyOperation**类的**isConcurrent**和**isExecuting**和**isFinished**方法的实现比较直观。**isConcurrent**方法应该简单返回**YES**来指示这个操作是并发的。**isExecuting**和**isFinished**方法简单的返回当前类的实例变量。

Listing 2-5  Defining a concurrent operation

    @interface MyOperation : NSOperation {
        BOOL        executing;
        BOOL        finished;
    }
    - (void)completeOperation;
    @end

    @implementation MyOperation
    - (id)init {
        self = [super init];
        if (self) {
            executing = NO;
            finished = NO;
        }
        return self;
    }

    - (BOOL)isConcurrent {
        return YES;
    }

    - (BOOL)isExecuting {
        return executing;
    }

    - (BOOL)isFinished {
        return finished;
    }
    @end

Listing 2-6展示了**MyOperation**的**start**方法。该方法的实现已经最小化工作量了，演示了你必须执行的任务。在这种情形下，方法简单的开启了一个新线程，然后配置它调用**main**方法。同时也更新了**executing**成员变量，然后为**isExecuting**值的改变产生了一个KVO通知。当这些代码执行完后，该方法简单的返回了，让新的线程去执行任务。

Listing 2-6  The start method

    - (void)start {
        // Always check for cancellation before launching the task.
        if ([self isCancelled])
        {
            // Must move the operation to the finished state if it is canceled.
            [self willChangeValueForKey:@"isFinished"];
            finished = YES;
            [self didChangeValueForKey:@"isFinished"];
            return;
        }
        
        // If the operation is not canceled, begin executing the task.
        [self willChangeValueForKey:@"isExecuting"];
        [NSThread detachNewThreadSelector:@selector(main) toTarget:self withObject:nil];
        executing = YES;
        [self didChangeValueForKey:@"isExecuting"];
	}

Listing 2-7展示了实现**MyOperation**类的剩余代码。从上面可知，**main**方法是新线程的入口。它主要的工作是执行与操作对象相关的工作，然后在任务最终完成的时候调用自定义的**completeOperation**方法。**completeOperation**方法在**isExecuting**和**isFinished**的值改变的时候产生KVO通知。

Listing 2-7  Updating an operation at completion time

    - (void)main {
        @try {
            
            // Do the main work of the operation here.
            
            [self completeOperation];
        }
        @catch(...) {
            // Do not rethrow exceptions.
        }
    }

    - (void)completeOperation {
        [self willChangeValueForKey:@"isFinished"];
        [self willChangeValueForKey:@"isExecuting"];
        
        executing = NO;
        finished = YES;
        
        [self didChangeValueForKey:@"isExecuting"];
        [self didChangeValueForKey:@"isFinished"];
	}

即使一个操作是被取消的，你也应该始终通知KVO的观察者你的操作现在已经完成了。当一个操作对象是依赖于其它完成之后的操作对象，它会监听这些操作对象的**isFinished**的key path。只有当所有的对象都报告完成时，依赖的那个操作才会收到准备执行的信号。在你的应用程序中未能产生一个完成的通知会阻止依赖的操作对象执行。

### Maintaining KVO Compliance
(维护KVO)

**NSOperation**类键值观察如下key paths：

* isCancelled
* isConcurrent
* isExecuting
* isFinished
* isReady
* dependencies
* queuePriority
* completionBlock

如果你重写**start**方法，或者对**NSOperation**对象的其它自定义运行(覆盖main除外)，你必须确保你自定义的对象维护了这些关键路径的KVO。当你重写**start**方法，你需要关注**isExecuting**和**isFinished**这两个关键路径。在重新实现方法的时候，这些关键路径是最常被影响的。

如果你希望依赖于其它东西(非操作对象)，你也可以重写**isReady**方法，并强制返回**NO**，直到你等待的依赖满足。(如果你实现自定义的依赖并且仍然要支持**NSOperation**类提供的默认依赖管理，确保在**isReady**方法里面调用了超类的方法。)当你的操作对象的准备状态改变时，应该为关键路径**isReady**生成KVO通知报告这些改变。除非你重写了**addDependency:**或者**removeDependency:**方法，你应该不需要担心为**dependencies**关键路径产生KVO通知。

尽管你可以为**NSOperation**类的其它路径生成KVO通知，但是你不需要这么做。如果你需要取消操作，你可以直接调用已经存在的**cancel**方法。同样的，你也很少需要在操作对象里面修改队列的优先级信息。最后，除非你的操作需要能够动态改变并发状态，你也不需要提供**isConcurrent**关键路径的KVO通知。

更多关于KVO，以及如何在你自定义的对象中使用KVO，参见[ Key-Value Observing Programming Guide ](https://developer.apple.com/library/prerelease/mac/documentation/Cocoa/Conceptual/KeyValueObserving/KeyValueObserving.html#//apple_ref/doc/uid/10000177i)。

## Customizing the Execution Behavior of an Operation Object
(自定义一个操作对象的执行行为)

操作对象的配置发生在创建之后添加到队列之前。接下来的几个部分描述的配置适用于所有的操作对象，不管是你自己子类化的**NSOperation**还是已经存在子类。

### Configuring Interoperation Dependencies
(配置操作之间的依赖)

依赖是在不同的操作对象之间串行执行的一种方法。一个操作依赖于其它操作时必须等到依赖的所有操作执行完成之后才能执行。因此，你可以在两个操作对象之间创建一对一的关系或者构建复杂的对象依赖图。

你可以使用**NSOperation**的**addDependency:**方法在两个操作对象之间建立依赖。这个方法会创建一个单向的依赖，表示当前的操作对象依赖于你参数指定的目标操作对象。这种依赖意味着当前操作对象不能开始执行直到目标对象执行完成。依赖关系并不局限于同一个队列中的操作。操作对象会自己管理依赖，所以完全可以在不同的队列中的操作对象间建立依赖。但是有一件事情是不能接受的，就是在操作之间创建了循环的依赖。这样做就导致了编程错误，没有操作会运行。

当操作的所有依赖都执行完成后，操作对象才会准备执行.(如果你自定义了**isReady**方法的行为，那么操作的准备状态由你设置的决定。)如果操作对象在队列中，那么队列可能在任何时间开始执行。如果你计划手动执行操作，何时执行取决于你何时调用操作的**start**方法。

**重要**：你应该总是在运行你的操作或者添加到队列之前配置依赖。之后添加的依赖可能并没有效果。

依赖依靠于每一个操作对象在状态改变时发出合适的KVO通知。如果你自定义了你操作对象的行为，你需要在你自定义的代码中生成合适的KVO通知来避免依赖出现问题。更多关于KVO通知和操作对象，参见[ Maintaining KVO Compliance ](https://developer.apple.com/library/prerelease/mac/documentation/General/Conceptual/ConcurrencyProgrammingGuide/OperationObjects/OperationObjects.html#//apple_ref/doc/uid/TP40008091-CH101-SW10)。更多关于配置依赖的信息，参见[ NSOperation Class Reference ](https://developer.apple.com/library/prerelease/mac/documentation/Cocoa/Reference/NSOperation_class/index.html#//apple_ref/doc/uid/TP40004591)。

### Changing an Operation’s Execution Priority
(修改操作执行的优先级)

对于添加到队列的操作，执行顺序首先由队列里面操作的准备好的顺序，然后由相关的优先级来确定。准备状态由依赖的其它操作决定，但是优先级是操作对象本身的一个属性。默认情况下，所有的新的操作对象都有默认的优先级，但是你可以根据需要通过调用操作对象的**setQueuePriority:**方法来增加或者减少。

优先级只能应用于相同操作队列里的操作。如果你的应用程序有多个操作队列，每一个队列里面的操作的优先级是独立的。因此，在不同的队列里面，低等级的优先级可能比高等级的优先级先执行。

优先级并不是依赖的替代品。优先级只会用来确定那些队列里面准备开始执行的操作的执行顺序。比如，一个队列里面包含了一个高优先级和一个低优先级的操作，并且都准备好了，这个队列会先执行高优先级的操作。然而，如果高优先级的操作并没有准备好，但是低优先级的准备好了，这个队列会先执行低优先级的操作。如果你想防止一个操作在另一个操作完成之前开始，你必须使用依赖(如[ Configuring Interoperation Dependencies ](https://developer.apple.com/library/prerelease/mac/documentation/General/Conceptual/ConcurrencyProgrammingGuide/OperationObjects/OperationObjects.html#//apple_ref/doc/uid/TP40008091-CH101-SW17)描述的那样)。

### Changing the Underlying Thread Priority
(修改底层线程的优先级)

在OS X v10.6之后，我们可以配置执行操作的底层线程的优先级。线程由系统内核管理，通常高优先级的线程给予更多的执行机会。在操作对象中你使用一个范围是0.0 - 1.0的浮点数来指定线程的优先级。如果你没有显式的指定线程的优先级，操作会运行在默认的线程优先级(0.5)的线程上。

要设置操作的线程的优先级，你必须在将操作对象添加到队列之前(或者手动执行之前)调用操作对象的**setThreadPriority:**方法来设置优先级。当要执行操作时，默认的**start**方法会使用你指定的值来设置当前线程的优先级。不过新的优先级只在操作的**main**方法范围内有效。其它所有的代码(包括操作的完成Block)运行在默认的优先级上。如果你创建了一个并发的操作，因此重写了**start**方法，你必须自己配置线程的优先级

### Setting Up a Completion Block
(设置一个完成Block)

在OS X v10.6及以后，操作可以在完成主任务之后执行一个完成Block。你可以使用这个完成Block来执行任何不属于主任务的工作。比如，你可能会使用这个Block来通知相关的客户端操作已经完成了。一个并发的操作对象可能会使用这个Block生成最终的KVO通知。

调用**NSOperation**的**setCompletionBlock:**方法来设置完成Block。你传递的这个Block应该没有参数列表和返回值。

## Tips for Implementing Operation Objects
(实现操作对象的技巧)

尽管操作对象非常容易实现，当你在写代码的时候还是有几点需要注意的。接下来的部分描述了在你编写操作对象时应该考虑的几点因素。

### Managing Memory in Operation Objects
(操作对象的内存管理)

接下来的几个章节描述了操作对象里面内存管理的几个关键要素。关于Objective-C中一般的内存管理信息，参见[ Advanced Memory Management Programming Guide ](https://developer.apple.com/library/prerelease/mac/documentation/Cocoa/Conceptual/MemoryMgmt/Articles/MemoryMgmt.html#//apple_ref/doc/uid/10000011i)。

#### Avoid Per-Thread Storage
(避免存储每个线程)

虽然大多数操作都在线程上执行，但对于非并发操作，线程通常由队列提供。如果一个操作队列为你提供了线程，你应该认为这个线程的拥有者是队列而你的操作不应该触碰。特别是，你永远不应该关联不是你创建或管理的线程的数据。操作队列管理的线程来来去去，这取决于系统和应用程序的需要。因此，给存储的线程传递数据是不可靠的，很可能会失败。

对于操作对象，没有必要在任何情况下存储线程。当你创建操作对象时，你应该提供对象工作需要的所有数据。所有输入和输出的数据都应该存储在操作对象中，最后再整合到你的应用程序中，或者最终释放掉。

#### Keep References to Your Operation Object As Needed
(根据需要保留操作对象的引用)

因为操作对象是异步运行的，你不应该假设你可以创建它们然后忘记它们。它们仍然是对象，你仍然需要根据你代码的需要来管理它们的引用。特别是如果你要在操作对象完成值后要获取其中的数据。

另一个你应该保留操作的引用的原因是之后你可能没有机会向队列请求操作对象。队列总是尽最大可能的快速调度和执行操作。在大多数情形下，大部分操作在添加到队列之后就开始执行了。通过自己的代码返回到队列以获得操作的引用时，这个操作可能已经执行完成并移除队列了。

### Handling Errors and Exceptions
(处理错误和异常)

因为操作本质上就是应用程序内创建的实体，它们有责任处理出现的任何错误或异常。在OS X v10.6及之后，**NSOperation**类的**start**方法默认不会捕捉异常。(在OS X v10.5，**start**方法会捕捉然后禁止异常。)你自己的代码因该总是捕捉异常然后直接禁止。同时也应该检查错误代码然后通知应用程序的相应部分。如果你替换了**start**方法，你也必须捕获所有异常，阻止它们离开底层线程的范围。

你应该准备好处理以下几种错误情况的类型：

* 检查并处理**UNIX errno**风格的错误代码。
* 明确的检查方法和函数返回的错误码。
* 捕获你自己代码或者其它系统框架抛出的异常。
* 捕获**NSOperation**类自己抛出的异常，会在如下几种情形抛出异常：
  1. 当操作并没有准备好执行，但是调用了**start**方法。
  2. 当操作正在执行或者结束(很有可能是被取消)，**start**再次被调用。
  3. 当你正在给你一个执行的或者结束的操作添加完成Block。
  4. 当你向一个取消的**NSInvocationOperation**对象检索结果。

如果你的自定义的代码会遇到异常或者错误，你应该采取所需的措施将错误传播到应用程序的其它部分。**NSOperation**类没有提供显式的方法传递错误或者异常到应用程序的其它部分。因此，如果这些信息很重要，你必须提供必须的代码。

## Determining an Appropriate Scope for Operation Objects
(为操作对象确定一个适当的范围)

尽管可以为一个操作队列添加任意数量的操作，但是这样做是不切实际的。和任何对象一样，**NSOperation**类的实例都会消耗内存，并且相关的任务执行也会消耗。如果你的操作对象只做很少量的操作，然后你创建了一万个，然后你可能会发现调度花的时间比实际执行任务花的时间多多了。如果你的应用程序已经是内存有限，你可能会发现，仅仅只有数万个的操作对象可能会进一步降低性能。

要想高效的使用操作，关键是要在工作量和保持计算机繁忙之间找到平衡点。确保操作都有一定的工作量要执行。比如，如果你的应用程序创建了100个操作对象执行100次相同的任务，可以考虑换成10个操作对象，每个执行10次。

你同样也应该避免一次给队列添加大量的操作，或者持续快速的向队列添加操作，超过队列的处理的能力。可以分批创建这些对象。当一批执行完成后，使用完成Block高速应用程序创建一批新的。当你有大量工作需要做的时候，你希望足够的操作填满队列，从而使计算机处于繁忙的状态，但是你不想一次创建大量的操作导致应用程序耗尽内存。

当然，你创建的操作对象的数量，以及每个操作要执行的工作量都是可变的，完全依赖于你的应用程序。你应该总是使用像**Instruments**这样的工具来帮助你在效率和速度中找到平衡点。对于**Instruments**和其它的一些用来收集代码性能的性能工具，参见[ Performance Overview ](https://developer.apple.com/library/prerelease/mac/documentation/Performance/Conceptual/PerformanceOverview/Introduction/Introduction.html#//apple_ref/doc/uid/TP40001410)。

## Executing Operations
(执行操作)

最终，你的应用程序需要执行相关的工作。接下来的章节将会学习几种执行操作的方法，以及在运行时如何操纵操作。 

### Adding Operations to an Operation Queue
(添加操作到操作队列)

到目前为止，执行操作的最简单的方法就是使用操作队列，**NSOperationQueue**类的实例。你的应用程序负责创建和维护任何打算使用的队列。一个应用程序可能有任意数量的队列，但也有实际的限制多少操作可以在给定的时间点被执行。操作队列和系统一起工作，限制了并发的操作的数量，使其跟可用的内核以及系统负载合适。因此，创建额外的队列并不意味着你可以执行额外的操作。

使用如下代码创建一个队列：

	NSOperationQueue* aQueue = [[NSOperationQueue alloc] init];、

使用**addOperation:**方法将操作添加到队列。在OS X v10.6及以后，你可以使用**addOperations:waitUntilFinished:**方法将一组操作添加到队列中，或者你可以使用**addOperationWithBlock:**方法直接将Block添加到队列中。所有的这些方法都会将操作进行排队，然后通知队列应该开始处理这些操作了。大部分情形下，操作在添加到队列之后不久就开始执行了，但是操作队列可能因为几个原因延迟执行队列里面的操作。具体而言，一些操作依赖于其它的操作，但是其它的操作还没有完成。或者操作队列被刮起了或者已经在执行最大的并发操作数量的操作了。下面的例子展示了将操作添加到队列的基本语法。

    [aQueue addOperation:anOp]; // Add a single operation
    [aQueue addOperations:anArrayOfOps waitUntilFinished:NO]; // Add multiple operations
    [aQueue addOperationWithBlock:^{
        /* Do something. */
    }];

**重要**：在操作添加到队列以后切勿进行修改。在队列中等待的时候，操作可能在任何时间开始执行，因此改变依赖或者数据会产生不理的影响。如果你想直到操作的状态，你可以使用**NSOperation**类的方法来确定操作是在运行、等待、或者已经完成。

尽管**NSOperationQueue**类被设计来对操作进行并发执行，但是可以强制一个队列在一个时间只运行一个操作。操作队列对象的**setMaxConcurrentOperationCount:**方法允许你配置并发操作的最大数量。传递1这个值会导致队列一次只执行一个操作。尽管一次只执行一个操作，但是执行顺序还是基于其它因素确定，比如每个操作的准备状态以及优先级。因此一个串行的操作队列提供的行为与调度队列的行为并不是一样的。如果操作对象的执行顺序对你很重要，你应该在将操作对象添加到队列之前建立依赖，参见[ Configuring Interoperation Dependencies ](https://developer.apple.com/library/prerelease/mac/documentation/General/Conceptual/ConcurrencyProgrammingGuide/OperationObjects/OperationObjects.html#//apple_ref/doc/uid/TP40008091-CH101-SW17)。

更多关于使用操作队列的信息，参见[ NSOperationQueue Class Reference ](https://developer.apple.com/library/prerelease/mac/documentation/Cocoa/Reference/NSOperationQueue_class/index.html#//apple_ref/doc/uid/TP40004592)。更多关于串行调度队列的信息，参见[ Creating Serial Dispatch Queues ](https://developer.apple.com/library/prerelease/mac/documentation/General/Conceptual/ConcurrencyProgrammingGuide/OperationQueues/OperationQueues.html#//apple_ref/doc/uid/TP40008091-CH102-SW6)。

### Executing Operations Manually
(手动执行操作)

尽管操作队列是执行操作对象最方便的方法，但是不使用队列也可以执行。如果你选择手动执行操作，那么，在你的代码中有一些注意事项要你关注。尤其是这个操作已经准备好了要执行，并且总是调用**start**方法开始。

一个操作并不会考虑执行直到**isReady**方法返回**YES**。**isReady**已经融入了系统管理依赖的方法里面，**NSOperation**类提供操作的依赖的状态。只有当依赖很清晰并且开始执行。

当手动执行一个操作时，你应该总是使用**start**方法开始执行。你使用这个方法替代**main**或者其它的方法，因为**start**方法在执行你的代码之前会执行几项安全检查。特别是，默认的**start**方法会生成操作要求处理它们依赖进度的KVO通知。如果你的操作已经被取消了，或者抛出了异常而没有真正的准备好要执行，这个方法正好也避免了执行你的操作。

如果你的应用程序定义了并发操作对象，你也应该考虑调用**isConcurrent**方法。如果方法返回了**NO**，你本地的代码可以决定是否在当前线程同步执行或者另外创建一个线程。然后，实现这种检查完全取决于你。

Listing 2-8展示了一些在手动执行操作之前要进行的简单检查。如果方法返回**NO**，你可以设置一个定时器然后之后再调用这个方法。你应该不断的重新安排计时器，直到方法返回YES，因为该操作可能会取消。

Listing 2-8  Executing an operation object manually

    - (BOOL)performOperation:(NSOperation*)anOp
    {
        BOOL        ranIt = NO;
        
        if ([anOp isReady] && ![anOp isCancelled])
        {
            if (![anOp isConcurrent])
                [anOp start];
            else
                [NSThread detachNewThreadSelector:@selector(start)
                                         toTarget:anOp withObject:nil];
            ranIt = YES;
        }
        else if ([anOp isCancelled])
        {
            // If it was canceled before it was started,
            //  move the operation to the finished state.
            [self willChangeValueForKey:@"isFinished"];
            [self willChangeValueForKey:@"isExecuting"];
            executing = NO;
            finished = YES;
            [self didChangeValueForKey:@"isExecuting"];
            [self didChangeValueForKey:@"isFinished"];
            
            // Set ranIt to YES to prevent the operation from
            // being passed to this method again in the future.
            ranIt = YES;
        }
        return ranIt;
    }

### Canceling Operations
(取消操作)

一旦添加到操作队列，操作对象就被队列拥有了，并且无法移除。将操作出队列的唯一方法就是取消它。你可以调用操作对象的**cancel**方法移除单个的操作对象或者调用操作队列对象的**cancelAllOperations**方法移除所有的操作。

只有你确定不再需要操作对象时才应该取消它。发出取消命令会将操作对象设置为取消的状态，会阻止它被执行。由于取消也被认为是完成，依赖于它的其它操作对象会收到适当的KVO通知，并清除依赖状态，然后得到执行。有时候为了响应重大的事件时通常会取消队列里面所有的操作，比如应用程序退出或者用户指定取消操作，而不是选择性的进行取消。

### Waiting for Operations to Finish
(等待操作完成)

为了获得最佳的性能，你应该将你的操作尽可能的设计成异步执行的，让你应用程序在执行操作的时候能够自由的处理额外的工作。如果代码创建的操作也要处理操作的结果，你可以使用**NSOperation**的**waitUntilFinished**方法来等待操作的完成。通常我们应该避免写这样的代码。阻塞当前线程可能是简便的解决方法，但是引进了更多的串行代码，限制了应用的并发性降低了用户体验。

**重要**：绝对不要在应用主线程中等待一个操作完成。你只能在其它的线程中或者其它的操作中才能这样做。阻塞主线程将导致应用无法响应用户事件，应用也将表现为无响应。

除了可以等待单个操作完成，你也可以通过调用**NSOperationQueue**的**waitUntilAllOperationsAreFinished**方法来等待队列中的所有的操作完成。当在等待整个队列完成时，注意，你应用程序的其它线程仍然可以添加操作到队列，就能会加长等待的时间。

### Suspending and Resuming Queues
(挂起和恢复队列)

如果你想临时暂停执行的操作，你可以使用**setSuspended:**方法挂起相应的操作队列。挂起一个队列不会导致已经在执行的操作中途放弃任务。只是简单的阻止调度新的操作执行。当用户要求暂停一些正在进行的工作时你可能会挂起一个队列，因为有时候用户可能最终想要恢复那些工作。
