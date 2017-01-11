---
title: "Concurrency Programming Guide - Dispatch Sources"
date: 2016-01-14 13:51:58
categories: 
- 编程指南
tags: 
- 并发编程
metaAlignment: center
autoThumbnailImage: no
coverImage: //d1u9biwaxjngwg.cloudfront.net/welcome-to-tranquilpeak/city.jpg

---

[官方文档](https://developer.apple.com/library/prerelease/mac/documentation/General/Conceptual/ConcurrencyProgrammingGuide/GCDWorkQueues/GCDWorkQueues.html#//apple_ref/doc/uid/TP40008091-CH103-SW1)
<!--more-->

# Dispatch Sources
(调度源)

每当你与底层系统交互，你必须为这项任务准备足够长的时间。调用内核或者其它系统层级与你自己的处理比起来代价更昂贵。因此，许多系统库都提供了异步接口允许你的代码提交请求到系统，在请求处理的时候继续执行其它的工作。Grand Central Dispatch构建在允许你提交请求并通过Block和调度队列将结果返回给你的代码的这一通用行为上。

## About Dispatch Sources
(关于调度源)

一个调度源是一种基础的数据类型，用来协调处理特定的低等级的系统事件。Grand Central Dispatch支持如下几种调度源：

* 定时器调度源，定期产生通知。
* 信号调度源，当**UNIX**信号到达时会通知你。
* 描述源，当文件和套接字变化时通知你，比如：
  * 当数据可读。
  * 当数据可写。
  * 当文件系统中的文件删除、移动、或者重命名。
  * 文件源数据信息改变。
* 进程调度源，通知你进程相关的事件，比如：
  * 当进程退出。
  * 当进程发起**fork**和**exec**等调用。
  * 当信号传递给进程。
* Mach相关的事件通知。
* 你自定义的自己触发的自定义调度源。

调度源替代了异步回调函数来处理系统进程相关的事件。当你配置调度源的时候，你指定想要监控的事件，以及调度队列和处理事件的代码。你可以使用Block对象或者函数来指定你的代码。当你感兴趣的事件到达时，调度源会提交你的Block或者函数到指定的队列进行执行。

不像你手动提交到队列的任务，调度源为你的应用程序持续的提供事件源。一个调度源会保持与调度队列的连接知道你显式的取消。在连接的这段时间，每当对应的事件发生时，就会提交相关的任务代码到调度队列。有些事件，比如定时器事件，都以固定间隔的时间发生，但是大部分都以特定条件零星出现。出于这个原因，调度源会保留与它们关联的调度队列，防止事件在执行的时候队列被过早的释放掉了。

为了防止事件在调度队列上积压，调度源实现了事件合并机制。如果一个新的事件到达时之前的事件已出列在执行，调度源会将新旧事件的数据合并。根据事件类型的不同，合并可能会替换旧事件或者更新旧事件的信息。比如，基于信号的调度源只提供了最近信号的信息，但是也会报道从上次事件处理调用后总共有多少个信号被传递了。

## Creating Dispatch Sources
(创建调度源)

创建一个调度源需要同时创建事件源和调度队列本身。事件源是处理事件所需要的本地数据结构。比如，基于描述符的调度源，你需要打开描述符：基于进程的事件，你需要获得目标程序的进程ID。当你有事件源的时候，你可以使用如下步骤创建对应的调度源：

1. 使用**dispatch_source_create**函数创建调度源。
2. 配置调度源：
  * 给调度源设置一个事件处理器；参见[ Writing and Installing an Event Handler ](https://developer.apple.com/library/prerelease/mac/documentation/General/Conceptual/ConcurrencyProgrammingGuide/GCDWorkQueues/GCDWorkQueues.html#//apple_ref/doc/uid/TP40008091-CH103-SW13)。
  * 对于定时器调度源，关于设置定时器信息使用[ dispatch_source_set_timer ](https://developer.apple.com/library/prerelease/mac/documentation/Performance/Reference/GCD_libdispatch_Ref/index.html#//apple_ref/c/func/dispatch_source_set_timer)函数；参见[ Creating a Timer ](https://developer.apple.com/library/prerelease/mac/documentation/General/Conceptual/ConcurrencyProgrammingGuide/GCDWorkQueues/GCDWorkQueues.html#//apple_ref/doc/uid/TP40008091-CH103-SW2)。
3. 给调度源赋予一个取消处理器，参见[ Installing a Cancellation Handler ](https://developer.apple.com/library/prerelease/mac/documentation/General/Conceptual/ConcurrencyProgrammingGuide/GCDWorkQueues/GCDWorkQueues.html#//apple_ref/doc/uid/TP40008091-CH103-SW14)(可选)。
4. 调用**dispatch_resume**函数开始处理事件，参见[ Suspending and Resuming Dispatch Sources ](https://developer.apple.com/library/prerelease/mac/documentation/General/Conceptual/ConcurrencyProgrammingGuide/GCDWorkQueues/GCDWorkQueues.html#//apple_ref/doc/uid/TP40008091-CH103-SW8)。

因为调度源在使用之前必须进行额外的配置才能被使用，**dispatch_source_create**函数返回一个挂起状态的调度源。在挂起的时候，调度源会接收事件，但是不会处理它们。这时你可以安装事件处理器并执行额外的配置。

接下来的部分将想你展示如何配置调度源的各方面。关于如何配置具体类型的调度源，参见[ Dispatch Source Examples ](https://developer.apple.com/library/prerelease/mac/documentation/General/Conceptual/ConcurrencyProgrammingGuide/GCDWorkQueues/GCDWorkQueues.html#//apple_ref/doc/uid/TP40008091-CH103-SW22)。关于创建和配置调度源的额外的函数，参见[ Grand Central Dispatch (GCD) Reference ](https://developer.apple.com/library/prerelease/mac/documentation/Performance/Reference/GCD_libdispatch_Ref/index.html#//apple_ref/doc/uid/TP40008079)。

### Writing and Installing an Event Handler
(编写和安装一个事件处理器)

为了处理调度源产生的事件，你必须定义一个处理这些事件的事件处理器。一个事件处理器是一个函数或者Block对象，使用**dispatch_source_set_event_handler** 和 **dispatch_source_set_event_handler_f**函数安装是调度源上。当事件到达时，调度源提交你的事件处理器到指定的队列进行处理。

事件处理器的主体负责处理所有到达的事件。如果事件处理器已经在队列中并等待处理已经到达的事件，如果此时又来了一个新事件，调度源会合并这两个事件。事件处理器通常只能看到最新事件的信息，但是取决于调度源的类型，它也可能获得已经合并的事件的信息。如果事件处理器已经开始执行后，又有一个或者多个新的事件到达，调度源会保存这些事件直到当前时间处理器执行完成。在那时，它会使用新的事件将事件处理器再次提交到队列。

基于函数的事件处理器有一个单一的上下文指针，包含调度源对象，没有返回值。基于Block的事件处理器没有参数也没有返回值。

    // Block-based event handler
    void (^dispatch_block_t)(void)

    // Function-based event handler
    void (*dispatch_function_t)(void *)

在事件处理器的内部，你可以从调度源本身获取给定事件的信息。虽然基于函数的事件处理器可以传递指针给调度源当作参数，基于Block的事件处理器必须自己捕获那些指针。一般Block定义时会自动捕获外部定义的变量。比如，如下的代码片段捕获了**source**变量，定义在Block范围之外。

    dispatch_source_t source = dispatch_source_create(DISPATCH_SOURCE_TYPE_READ,
                                                      myDescriptor, 0, myQueue);
    dispatch_source_set_event_handler(source, ^{
        // Get some data from the source variable, which is captured
        // from the parent context.
        size_t estimated = dispatch_source_get_data(source);
        
        // Continue reading the descriptor...
    });
    dispatch_resume(source);

Block内捕获变量通常允许更大的灵活性和动态性。当然，默认捕获的变量是只读的。尽管在一些指定的情形下，Block提供了支持修改捕获变量的功能，但是你不应该在与调度源关联的事件处理器里面这样做。调度源总是异步的执行它们的事件处理器，所以你捕获的变量的定义的范围，在事件处理器执行时已经不存在了。关于Block捕获和使用变量的更多信息，参见[ Blocks Programming Topics ](https://developer.apple.com/library/prerelease/mac/documentation/Cocoa/Conceptual/Blocks/Articles/00_Introduction.html#//apple_ref/doc/uid/TP40007502)。

Table 4-1列出了你可以在事件处理器代码中获取事件信息的函数。

Table 4-1  Getting data from a dispatch source

![alt text](/img/ConcurrencyProgrammingGuideOperationQueues/1.png)

* **dispatch_source_get_handle**：这个函数返回调度源管理的底层系统的数据类型；对于一个描述符的调度队列，这个函数返回一个**int**类型的值，表示调度源关联的描述符；对于信号调度源，这个函数返回一个**int**类型，表示最新事件的信号数量；对于进程调度源，这个函数会返回一个**pid_t**数据结构，表示被监控的进程；对于一个March端口的调度源，这个函数返回一个**mach_port_t**的数据结构；对于其它的调度源，这个函数的返回值是未定义的。
* **dispatch_source_get_data**：这个函数返回与事件相关的所有未决数据；对于一个描述符的调度源，从文件读取数据，这个函数返回可读的字节数量；对于一个描述符的调度源，将数据写入文件，这个函数会返回一个正的整型值表示有空间可以写入；对于一个描述符的调度源，监控文件系统的活动，这个函数会返回一个常量表示发生的事件的类型。关于更多的常量，参见[ dispatch_source_vnode_flags_t ](https://developer.apple.com/library/prerelease/mac/documentation/Performance/Reference/GCD_libdispatch_Ref/index.html#//apple_ref/doc/constant_group/dispatch_source_vnode_flags_t)枚举类型；对于一个进程调度源，这个函数返一个常量指示发生的事件的类型。参见[ dispatch_source_proc_flags_t ](https://developer.apple.com/library/prerelease/mac/documentation/Performance/Reference/GCD_libdispatch_Ref/index.html#//apple_ref/doc/constant_group/dispatch_source_proc_flags_t)枚举类型；对于Mach端口调度源，这个函数返回一个常量指示发生的事件的类型，参见**dispatch_source_machport_flags_t**枚举类型；对于一个自定义的调度源，这个函数返回从现有数据创建的新数据，以及传递给[ dispatch_source_merge_data ](https://developer.apple.com/library/prerelease/mac/documentation/Performance/Reference/GCD_libdispatch_Ref/index.html#//apple_ref/c/func/dispatch_source_merge_data)函数的新数据。
* **dispatch_source_get_mask**：这个函数返回我们用来创建调度源的事件标志；对于进程调度源，这个函数返回调度源接收到的事件掩码，参见[ dispatch_source_proc_flags_t ](https://developer.apple.com/library/prerelease/mac/documentation/Performance/Reference/GCD_libdispatch_Ref/index.html#//apple_ref/doc/constant_group/dispatch_source_proc_flags_t)枚举类型；对于发送权利的Mach端口调度源，这个函数返回了一个期望事件的掩码，参见**dispatch_source_mach_send_flags_t**枚举类型；对于自定义的OR调度源，这个函数返回用来合并数据值的掩码。

关于如何为指定类型的调度源编写和安装事件处理器的一些例子，参见[ Dispatch Source Examples ](https://developer.apple.com/library/prerelease/mac/documentation/General/Conceptual/ConcurrencyProgrammingGuide/GCDWorkQueues/GCDWorkQueues.html#//apple_ref/doc/uid/TP40008091-CH103-SW22)。

### Installing a Cancellation Handler
(安装一个取消处理器)

取消处理器用来在调度源释放之前执行清理操作。对于大多数调度源类型，取消处理器是可选的，除非你对调度源绑定了自定义行为需要在释放时执行。对于描述符和Mach端口调度源必须设置取消处理器，用来关闭描述符或者Mach端口。否则会导致微妙的BUG，这些结构体系被系统其它部分或你的应用在不经意间重用。

你可以在任何时候安装取消处理器，但通常我们在创建调度源的时候就会安装取消处理器。你可以使用**dispatch_source_set_cancel_handler** 或者 **dispatch_source_set_cancel_handler_f**函数来安装取消处理器，取决于你是否想要使用Block对象或者函数来实现。下面的例子展示了一个简单的取消处理器，关闭了一个描述符。这个**fd**变量被描述符捕获。

    dispatch_source_set_cancel_handler(mySource, ^{
        close(fd); // Close a file descriptor opened earlier.
    });

参见[ Reading Data from a Descriptor ](https://developer.apple.com/library/prerelease/mac/documentation/General/Conceptual/ConcurrencyProgrammingGuide/GCDWorkQueues/GCDWorkQueues.html#//apple_ref/doc/uid/TP40008091-CH103-SW6)获取完整的代码事例。

### Changing the Target Queue
(修改目标队列)

尽管你在创建调度源时指定了事件和取消处理器运行的队列，你可以在任何时间使用**dispatch_set_target_queue**函数来修改这个队列。当你需要改变调度源事件优先级的时候你可能会这样做。

改变调度源的队列是一个异步操作而且调度源会竭尽所能的快速的改变。如果一个事件处理器已经在队列中并且等待处理，它会在之前的队列上执行。随后到达的所有事件的处理器都会在后面修改的队列中执行。

### Associating Custom Data with a Dispatch Source
(关联自定义数据到调度源)

类似于Grand Central Dispatch中的其它数据类型，你可以使用**dispatch_set_context**函数给调度源关联自定义数据。你可以使用上下文指针存储任意你事件处理器需要的数据。如果你在上下文指针存储了任何自定义的数据，你也应该安装一个取消处理器，当调度源不再需要这些数据的时候释放这些数据。

如果你使用Block来处理事件处理器，你也可以捕获本地变量然后在基于Block的代码里面使用。这可能会减少你使用上下文指针来给你的调度源存储数据，但是你应该明智的使用Block捕获变量。因为调度源会在你的应用程序中长时间存在，当捕获了包含指针的变量时要特别小心。因为指针指向的数据可能随时被释放，因此你需要复制或者保留防止这种情况发生。不管你用哪种方法，你都应该提供一个取消处理器释放这些数据。

### Memory Management for Dispatch Sources
(调度源的内存管理)

类似于其它的调度对象，调度源也是引用计数的数据类型。初始计数为1，可以使用**dispatch_retain** 和 **dispatch_release**函数来增加和减少引用计数。当引用计数为0时，系统自动释放调度源数据结构。

因为调度源的使用方式，调度源的所有权可以由它自身在内部或者外部进行管理。外部所有权时，另一个对象拥有这个调度源，并负责在不需要时释放它。内部所有权时，调度源自己拥有自己，并负责在合适的时候释放自己。虽然外部所有权很常用，当你想自己创建调度源，并让它自己管理自己的行为时，可以使用内部所有权。比如，一个调度源被设计成为处理单一全局事件，你可以让它自己处理该事件然后立即退出。

## Dispatch Source Examples
(调度源例子)

接下来的部分展示了如何创建和配置一些常用的调度源。更多关于特定类型的调度源，参见[ Grand Central Dispatch (GCD) Reference ](https://developer.apple.com/library/prerelease/mac/documentation/Performance/Reference/GCD_libdispatch_Ref/index.html#//apple_ref/doc/uid/TP40008079)。

### Creating a Timer
(创建一个定时器)

定时器调度源定时产生事件。你可以发起定时执行的任务。比如，游戏和其它的图形应用程序，可以使用定时器来更新屏幕或动画。你也可以设置定时器，并在固定间隔事件中检查服务器的新信息。

所有的调度源定时器都是间隔定时器，一旦创建，会按你指定的间隔定期的传递事件。当你创建了定时器调度源，你必须为调度源指定一个期望的定时器事件精度，也就是**leeway**值。这个值让系统能够灵活的管理电源并唤醒内核。比如，系统可以使用这个值提前或者延迟触发定时器，以及更好的与其它系统事件结合。创建自己的定时器时，你应该尽量指定一个**leeway**值。

**注意**：就算你指定**leeway**值为0，也不要期望定时器能够按照精确的纳秒来触发事件。系统会尽可能地满足你的需求，但是无法保证完全精确的触发时间。

当计算机休眠的时候，所有的定时器调度源都会挂起。当计算机唤醒的时候，这些调度源定时器也会自动唤醒。取决于定时器的配置，暂停定时器可能会影响下一次触发。如果你使用**dispatch_time**函数或者**DISPATCH_TIME_NOW**常量来设置你的调度源定时器，这个调度源定时器会使用默认的系统时钟来决定何时触发。但是，默认的时钟在计算机休眠的时候不会继续。相反，当你使用**dispatch_walltime**函数来设置定时器调度源，定时器调度源追踪其触发时间的挂钟时间。这后一种选择通常适合触发时间间隔比较大的定时器，因为可以防止事件之间的时间出现大多的浮动。

Listing 4-1展示了一个每30秒触发一次，**leeway**值为1秒的例子。因为间隔相对较大，这个调度源使用**dispatch_walltime**函数创建。定时器会立即触发第一次，之后的事件每30秒触发一次。**MyPeriodicTask** 和 **MyStoreTimer**是自定义函数，你用来重写实现定时器的行为，并将定时器存储高你应用程序的数据结构。

Listing 4-1  Creating a timer dispatch source

    dispatch_source_t CreateDispatchTimer(uint64_t interval,
                                          uint64_t leeway,
                                          dispatch_queue_t queue,
                                          dispatch_block_t block)
    {
        dispatch_source_t timer = dispatch_source_create(DISPATCH_SOURCE_TYPE_TIMER,
                                                         0, 0, queue);
        if (timer)
        {
            dispatch_source_set_timer(timer, dispatch_walltime(NULL, 0), interval, leeway);
            dispatch_source_set_event_handler(timer, block);
            dispatch_resume(timer);
        }
        return timer;
    }

    void MyCreateTimer()
    {
        dispatch_source_t aTimer = CreateDispatchTimer(30ull * NSEC_PER_SEC,
                                                       1ull * NSEC_PER_SEC,
                                                       dispatch_get_main_queue(),
                                                       ^{ MyPeriodicTask(); });
        
        // Store it somewhere for later use.
        if (aTimer)
        {
            MyStoreTimer(aTimer);
        }
    }

尽管创建一个定时器调度源是接收基于时间的事件的最主要的方式，当然也有其它可选的选项。如果你想在指定的一个时间间隔后执行一个Block，你可以使用**dispatch_after** 和 **dispatch_after_f**。这个函数的行为和**dispatch_async**函数很像，除了允许你指定一个时间值(时间一到就提交Block到队列中执行)。时间值可以指定为相对或者绝对时间。

### Reading Data from a Descriptor
(从描述符读取数据)

为了从文件或者套接字读取数据，你必须打开文件或者套接字，然后创建一个**DISPATCH_SOURCE_TYPE_READ**类型的调度源。你指定的事件处理器应该能够读取、处理文件描述符的内容。对于文件，需要读取文件数据，并为应用创建适当的数据结构；对于套接字，需要处理最新接收到的网络数据。

读取数据时，你应该配置描述符使用非阻塞的操作。虽然你可以使用**ispatch_source_get_data**函数查看当前有多少数据可读，这个函数返回的数值在函数调用与真实读取时会发生变化。如果底层文件被截断，或者发生了网络错误，从描述符中读取数据会阻塞当前线程，停止在事件处理器中并阻止调度队列去执行其它任务。对于一个串行队列，这可能会死锁你的队列，即使是并发队列，也会减少能够执行的任务数量。

Listing 4-2展示配置一个调度源读取文件数据的例子。在这个例子中，事件处理器读取了指定文件的所有内容到缓冲区，并调用一个自定义函数(定义在你自己的代码中)来处理这些数据。(这个函数的调用者可能会使用返回的调度源来取消，当读取操作完成时。)为了确保调度队列在没有可读数据时发生不必要的阻塞，这个例子使用了**fcntl**函数来配置文件描述符来执行非阻塞的操作。安装给调度源的取消处理器确保文件描述符在数据读取完成后关闭文件描述符。

Listing 4-2  Reading data from a file

    dispatch_source_t ProcessContentsOfFile(const char* filename)
    {
        // Prepare the file for reading.
        int fd = open(filename, O_RDONLY);
        if (fd == -1)
            return NULL;
        fcntl(fd, F_SETFL, O_NONBLOCK);  // Avoid blocking the read operation
        
        dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
        dispatch_source_t readSource = dispatch_source_create(DISPATCH_SOURCE_TYPE_READ,
                                                              fd, 0, queue);
        if (!readSource)
        {
            close(fd);
            return NULL;
        }
        
        // Install the event handler
        dispatch_source_set_event_handler(readSource, ^{
            size_t estimated = dispatch_source_get_data(readSource) + 1;
            // Read the data into a text buffer.
            char* buffer = (char*)malloc(estimated);
            if (buffer)
            {
                ssize_t actual = read(fd, buffer, (estimated));
                Boolean done = MyProcessFileData(buffer, actual);  // Process the data.
                
                // Release the buffer when done.
                free(buffer);
                
                // If there is no more data, cancel the source.
                if (done)
                    dispatch_source_cancel(readSource);
            }
        });
        
        // Install the cancellation handler
        dispatch_source_set_cancel_handler(readSource, ^{close(fd);});
        
        // Start reading the file.
        dispatch_resume(readSource);
        return readSource;
    }

在上面的例子中，自定义的函数**MyProcessFileData**确定读取到足够的数据，调度源可以取消。默认情况下，一个调度源配置的是还有数据可读时，会重复调度事件处理器。如果套接字关闭或者到达文件末尾，调度源自动停止调度事件处理器。如果你自己确定不再需要调度源，也可以手动取消它。

### Writing Data to a Descriptor
(向描述符写入数据)

向文件和套接字写入数据的过程和读取数据的过程非常相似。为写入操作配置了描述符后，你可以创建一个**DISPATCH_SOURCE_TYPE_WRITE**类型的调度源。一旦调度源创建，系统就会调用你的事件处理器，让它有机会开始向文件或者套接字写入数据。当你完成了写入数据，使用**dispatch_source_cancel**函数取消调度源。

写入数据时，你应该总是配置你的描述符是非阻塞操作的。尽管你可以使用**dispatch_source_get_data**函数查看还有多少空间可以写入，这函数的返回值仅供参考，在函数调用与实际写入之间可能会改变。如果发生了错误，写入数据到阻塞的文件描述符会使事件处理器停止在执行中途，并且会阻止调度队列执行其它的任务。对于串行队列，这可能会死锁队列，即使是并发队列，也会减少能够执行的任务数量。

Listing 4-3展示了使用调度源将数据写入文件的基本方法。在创建了新文件后，这个函数传递文件描述符给事件处理器。写入文件的数据由**MyGetData**函数提供，你可以使用自己的代码替换。在写入数据到文件之后，事件处理器取消了调度源防止再次调用。调度源的拥有者负责之后释放调度源。

Listing 4-3  Writing data to a file

    dispatch_source_t WriteDataToFile(const char* filename)
    {
        int fd = open(filename, O_WRONLY | O_CREAT | O_TRUNC,
                      (S_IRUSR | S_IWUSR | S_ISUID | S_ISGID));
        if (fd == -1)
            return NULL;
        fcntl(fd, F_SETFL); // Block during the write.
        
        dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
        dispatch_source_t writeSource = dispatch_source_create(DISPATCH_SOURCE_TYPE_WRITE,
                                                               fd, 0, queue);
        if (!writeSource)
        {
            close(fd);
            return NULL;
        }
        
        dispatch_source_set_event_handler(writeSource, ^{
            size_t bufferSize = MyGetDataSize();
            void* buffer = malloc(bufferSize);
            
            size_t actual = MyGetData(buffer, bufferSize);
            write(fd, buffer, actual);
            
            free(buffer);
            
            // Cancel and release the dispatch source when done.
            dispatch_source_cancel(writeSource);
        });
        
        dispatch_source_set_cancel_handler(writeSource, ^{close(fd);});
        dispatch_resume(writeSource);
        return (writeSource);
    }

### Monitoring a File-System Object
(监控文件系统对象)

如果你想监控文件系统对象的改变，你可以设置一个**DISPATCH_SOURCE_TYPE_VNODE**类型的调度源。当一个文件删除、写入、重命名的时候你可以使用这个调度源接收通知。当文件指定类型的源数据改变时你也能收到提醒。

**注意**：当调度源正在处理事件时，指定给调度源的文件描述符必须保持打开状态。

Listing 4-4展示了一个监控文件名字，当改变时执行了一些自定义行为的例子。(你应该提供你自己的行为，替换例子中**MyUpdateFileName**函数。)因为一个文件描述符专门为调度源打开，调度源安装了取消处理器来关闭文件描述符。这个例子中的文件描述符关联了底层文件系统对象，同一个调度源可以用来检测多次文件名变化。

Listing 4-4  Watching for filename changes

    dispatch_source_t MonitorNameChangesToFile(const char* filename)
    {
        int fd = open(filename, O_EVTONLY);
        if (fd == -1)
            return NULL;
        
        dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
        dispatch_source_t source = dispatch_source_create(DISPATCH_SOURCE_TYPE_VNODE,
                                                          fd, DISPATCH_VNODE_RENAME, queue);
        if (source)
        {
            // Copy the filename for later use.
            int length = strlen(filename);
            char* newString = (char*)malloc(length + 1);
            newString = strcpy(newString, filename);
            dispatch_set_context(source, newString);
            
            // Install the event handler to process the name change
            dispatch_source_set_event_handler(source, ^{
                const char*  oldFilename = (char*)dispatch_get_context(source);
                MyUpdateFileName(oldFilename, fd);
            });
            
            // Install a cancellation handler to free the descriptor
            // and the stored string.
            dispatch_source_set_cancel_handler(source, ^{
                char* fileStr = (char*)dispatch_get_context(source);
                free(fileStr);
                close(fd);
            });
            
            // Start processing events.
            dispatch_resume(source);
        }
        else
            close(fd);
        
        return source;
    }

### Monitoring Signals
(监测信号)

UNIX信号允许来自应用程序域外的操作。一个应用程序可以接受许多不同类型的信号，比如不可恢复的错误(非法指令)，或重要信息的通知(比如子进程退出)。传统编程中，应用程序使用**sigaction**函数安装信号处理器函数，当信号到达时尽可能快的同步处理信号。如果你只是想信号到达时得到通知，并不想实际的处理该信号，你可以使用信号调度源来异步的处理信号。

信号调度源不能替代**sigaction**函数安装的同步信号处理器。同步信号处理器能够捕获一个信号，并阻止它中止应用程序。而信号调度源只允许你监测信号的到达。此外，你不能使用信号调度源获取所有类型的信号。你不能使用信号调度源监测**SIGILL**, **SIGBUS**, 和 **SIGSEGV**信号。

因为信号调度源在队列上是异步执行的，它没有同步信号处理器的一些限制。比如，信号调度源的事件处理器可以调用任何函数。灵活性增大的代价是，信号到达和调度源事件处理器调用的延迟可能会增大。

Listing 4-5向你展示了如何配置一个信号调度源来处理**SIGHUP**信号。这个调度源事件处理器调用**MyProcessSIGHUP**函数，你应该替换为你自己处理信号的代码。

Listing 4-5  Installing a block to monitor signals

    void InstallSignalHandler()
    {
        // Make sure the signal does not terminate the application.
        signal(SIGHUP, SIG_IGN);
        
        dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
        dispatch_source_t source = dispatch_source_create(DISPATCH_SOURCE_TYPE_SIGNAL, SIGHUP, 0, queue);
        
        if (source)
        {
            dispatch_source_set_event_handler(source, ^{
                MyProcessSIGHUP();
            });
            
            // Start processing signals
            dispatch_resume(source);
        }
    }

如果你在开发一个自定义的框架，信号调度源的一种高级用法就是可以监控任何链接的应用程序的信号。信号调度源不是其它调度源的接口或者应用程序安装的同步信号处理器。

更多关于实现同步信号处理器的信息，以及信号名称，参见[ signal ](https://developer.apple.com/library/prerelease/mac/documentation/Darwin/Reference/ManPages/man3/signal.3.html#//apple_ref/doc/man/3/signal)。

### Monitoring a Process
(监控进程)

进程调度源可以监控特定进程的行为，并适当的响应。父进程可以使用这种调度源监控所有子进程的创建。比如，监控子进程的死亡。类似的，子进程也可以使用这种调度源监控父进程，在父进程退出时自己也退出。

Listing 4-6展示了安装一个监控父进程终结的调度源的步骤。当父进程退出时，调度源设置一些内部状态的信息，告知子进程应该退出了。(你自己的应用程序应该实现*8MySetAppExitFlag**函数，设置合适终结的标识。)因为调度源是自主运行的，因此自己拥有自己，在程序关闭时会取消并释放自己。

Listing 4-6  Monitoring the death of a parent process

    void MonitorParentProcess()
    {
        pid_t parentPID = getppid();
        
        dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
        dispatch_source_t source = dispatch_source_create(DISPATCH_SOURCE_TYPE_PROC,
                                                          parentPID, DISPATCH_PROC_EXIT, queue);
        if (source)
        {
            dispatch_source_set_event_handler(source, ^{
                MySetAppExitFlag();
                dispatch_source_cancel(source);
                dispatch_release(source);
            });
            dispatch_resume(source);
        }
		}

## Canceling a Dispatch Source
(取消一个调度源)

调度源会一直保持活动，直到你显式的使用**dispatch_source_cancel**函数。取消一个调度源会停止递送新事件，并且不能撤销。因此，通常你在取消后就立即释放它，如下所示：

    void RemoveDispatchSource(dispatch_source_t mySource)
    {
        dispatch_source_cancel(mySource);
        dispatch_release(mySource);
    }

取消一个调度源是异步操作。调用**dispatch_source_cancel**函数后不会再有新的事件被处理，但是正在被处理的事件会继续被处理完成。在处理完最后的事件之后，调度源会执行自己的取消处理器。

取消处理器是你最后的机会，在那里执行内存或资源的释放工作。如果你的调度源使用了文件描述符或者mach端口，你必须提供取消处理器在取消发生的时候关闭文件描述符或者摧毁端口。其它类型的调度源并不要求取消处理器，尽管如此，你仍然应该提供一个，如果你的调度源关联了内存或者数据。比如，你的调度源上下文指针存储了数据，你就应该提供一个。更多关于取消处理器的信息，参见[ Installing a Cancellation Handler ](https://developer.apple.com/library/prerelease/mac/documentation/General/Conceptual/ConcurrencyProgrammingGuide/GCDWorkQueues/GCDWorkQueues.html#//apple_ref/doc/uid/TP40008091-CH103-SW14)。

## Suspending and Resuming Dispatch Sources
(挂起和恢复调度源)

你可以使用**dispatch_suspend** 和 **dispatch_resume**函数临时的挂起和继续调度源的事件传递。这个函数分别增加和减少调度对象的挂起计数。因此，你必须每次**dispatch_suspend**调用之后，都需要**dispatch_resume**才能继续事件的传递。

当调度源挂起，在挂的时发生的所有事件都会被积压直到调度源恢复。当队列恢复时，不是传递所有的事件，而是传递一个合并的单一事件。比如，如果你在监控文件名的变化，传递的事件只包括最后一次名字的改变。以这种方式合并事件，防止它们建立在队列中，当工作恢复时压倒你的应用程序。
