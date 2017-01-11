---
title: "Concurrency Programming Guide - Migrating Away from Threads"
date: 2016-01-14 13:52:12
categories: 
- 编程指南
tags: 
- 并发编程
metaAlignment: center
autoThumbnailImage: no
coverImage: //d1u9biwaxjngwg.cloudfront.net/welcome-to-tranquilpeak/city.jpg

---

[官方文档](https://developer.apple.com/library/prerelease/mac/documentation/General/Conceptual/ConcurrencyProgrammingGuide/ThreadMigration/ThreadMigration.html#//apple_ref/doc/uid/TP40008091-CH105-SW1)
<!--more-->

# Migrating Away from Threads
(从线程迁移)

将现有线程代码迁移到GCD和操作对象有很多种方法。尽管不是所有的线程代码都能够迁移，但是迁移可能提升性能，并简化你的代码。使用调度队列和操作队列来代替线程拥有许多有点：

* 不再需要存储线程堆栈到应用程序的内存空间。
* 消除了创建和配置线程的代码。
* 消除了管理和调度线程工作的代码。
* 简化了你要编写的代码。

本章提供了一些技巧和指引，如何使用调度队列和操作队列替换基于线程的代码，并且实现相同的行为。

## Replacing Threads with Dispatch Queues
(使用调度队列替换线程)

首先考虑应用可能使用线程的几种方式：

* 单一任务线程：创建一个线程执行一个单一的任务，当任务完成时释放线程。
* 工作线程：创建一个或者多个工作线程执行特定的任务，定期分配任务给每个线程。
* 创建一个通用线程池，为每一个线程设置**run loops**。当你有任务要执行的时候，从线程池中抓取一个线程，并分配任务给它。如果没有空闲线程可用，将任务入队列，等待可用的线程。

尽管这些看起来像是不同的技术，但实际上只是相同原理的变种。这些线程都是应用程序用来执行任务的。唯一的区别就是管理线程和任务排队的代码。使用调度队列和操作队列，你可以消除线程和线程间通信的代码，让你专注于要执行的任务。

如果你使用了上面其中一种线程模型，你应该非常了解你应用程序要执行的任务类型。不再是将任务提交到一个你自定义的线程，而是封装在一个操作对象里面或者Block对象里面，然后调度到合适的队列。对于那些没有争议，不需要使用锁的任务，你可以直接使用一下方法进行迁移：

* 单一任务线程，将任务封装在Block或者操作对象里面，然后提交给并发队列。
* 对于工作线程，你需要决定是使用串行的队列还是并发的队列。如果你需要工作线程将特定任务集同步的执行，使用串行队列。如果你使用工作线程执行具体的没有相互依赖的任务，使用并发队列。
* 对于线程池，将任务封装到Block或者操作对象，调度给并发队列执行。

当然，这种简单的替换并不适合所有的情形。如果你要执行的任务会争夺共享资源，理想的解决方案就是消除或者最小化资源的争夺。如果你可以重构代码或者重新架构代码来消除共享资源的依赖，这是最完美的。如果不能这样做，或者没有效果，你仍然可以使用队列。队列的一大优势就是提供了可供预测的代码执行。这里的可预测意味着你在不使用锁或者其它重量级的锁机制的情形下仍然可以同步的执行代码。你可以使用队列来替代锁执行一下任务：

* 如果你的任务必须按照指定顺序执行，提交到串行调度队列。如果你更喜欢使用操作队列，请使用操作对象的依赖确保按照指定的顺序执行。
* 如果你使用锁保护共享资源，使用串行队列来执行任何会修改资源的任务。串行队列替代的同步的锁的机制。关于更多摆脱锁的信息，参见[ Eliminating Lock-Based Code ](https://developer.apple.com/library/prerelease/mac/documentation/General/Conceptual/ConcurrencyProgrammingGuide/ThreadMigration/ThreadMigration.html#//apple_ref/doc/uid/TP40008091-CH105-SW3)。
* 如果你使用**thread joins**来等待后台任务的完成，你可以考虑使用**dispatch groups**替代。你也可以使用一个**NSBlockOperation**对象或者操作对象依赖来实现这些行为。更多关于如何跟踪组的任务执行，参见[ Replacing Thread Joins ](https://developer.apple.com/library/prerelease/mac/documentation/General/Conceptual/ConcurrencyProgrammingGuide/ThreadMigration/ThreadMigration.html#//apple_ref/doc/uid/TP40008091-CH105-SW6)。
* 如果你使用生产者－消费者算法来管理有限资源池，请考虑改变实现，参考[ Changing Producer-Consumer Implementations ](https://developer.apple.com/library/prerelease/mac/documentation/General/Conceptual/ConcurrencyProgrammingGuide/ThreadMigration/ThreadMigration.html#//apple_ref/doc/uid/TP40008091-CH105-SW7)。
* 如果你使用线程从描述符读写数据，或者监控文件操作，使用**dispatch sources**，参见[ Dispatch Sources ](https://developer.apple.com/library/prerelease/mac/documentation/General/Conceptual/ConcurrencyProgrammingGuide/GCDWorkQueues/GCDWorkQueues.html#//apple_ref/doc/uid/TP40008091-CH103-SW1)。

记住队列不是替代线程的万能药这点很重要。异步编程模型提供的队列适合延迟无关紧要的场合。虽然队列提供配置任务执行的优先级的方法，但更高的优先级也不能确保任务一定能在特定时间得到执行。因此，线程仍然是实现最小延迟的适当选择，例如音频和视频的播放。

## Eliminating Lock-Based Code
(消除基于锁的代码)

对于线程代码，锁不同线程间同步访问共享资源的一种方式。然而，锁会带来开销。即使不考虑开销的情形下，使用锁也会由性能损失。在竞争的情形下，一个或者多个线程会为了等待锁的释放而阻塞。

使用队列替代锁，消除了锁带来的开销，并且简化了现有代码。你可以创建队列来串行的访问资源，而不是使用锁来保护共享资源。队列的开销和锁不一样。比如，将任务入队列并不会到内核中去获取互斥锁。

当把任务入队列时，你做的主要决定就是同步还是异步。异步提交任务使当前线程在任务执行时继续执行。同步提交任务会阻塞当前线程直到任务完成。两种机制各有用途，不过异步优于同步，尽可能的异步提交。

接下来展示了如何使用基于队列的代码来等价的替换已经存在的基于锁的代码。

### Implementing an Asynchronous Lock
(实现异步锁)

异步锁保护共享资源，也不会阻塞任何修改资源的代码。当你代码的部分工作需要修改一个数据结构的时候你可能会使用异步锁。使用传统的线程，你的实现：获得共享资源的锁，做必要的修改，释放锁，然后继续任务的其它部分的工作。但是，使用调度队列，调用代码可以异步修改，而不需要等待这些修改完成。

Listing 5-1展示了一个异步锁的实现。在这个例子中，受保护的资源定义了自己的串行队列。调用代码提交了一个Block对象到队列，Block包含了对资源必要的修改。因为这个队列是串行执行的，这个资源的修改可以确保按顺序进行；而且任务是异步执行的，调用线程不会阻塞。

Listing 5-1  Modifying protected resources asynchronously

    dispatch_async(obj->serial_queue, ^{
        // Critical section
    });

### Executing Critical Sections Synchronously
(同步执行临界区)

如果当前的代码无法继续直到给定的任务完成，你可以使用**dispatch_sync**函数同步提交任务。这个函数将任务添加到调度队列，然后阻塞当前线程，直到任务执行完成。根据你的需要这个队列可以是串行的或者并发的。因为这个函数会阻塞当前线程，所以只在需要的时候使用这个函数。Listing 5-2展示了使用**dispatch_sync**实现临界区的例子。

Listing 5-2  Running critical sections synchronously

    dispatch_sync(my_queue, ^{
        // Critical section
    });

如果你已经使用串行队列来保护共享资源，同步提交并不会比异步提交更好的保护资源。同步提交的唯一理由就是阻止当前代码运行直到临界区执行完成。如果你想要马上使用共享资源的一些值，你可能会同步提交。如果当前代码不需要等待临界区执行完成，或者接下来可以简单的提交额外的任务到这个相同的串行队列，通常异步提交更好。

## Improving on Loop Code
(改进循环代码)

如果你的代码中有循环，每次循环迭代执行的任务是相互独立的，你可以考虑使用**dispatch_apply** 或者 **dispatch_apply_f**函数来替换循环代码。这些函数将循环的迭代分开，然后提交给调度队列处理。当使用并发队列时，可以并发的执行循环中的多个迭代。

调用**dispatch_apply** 和 **dispatch_apply_f**函数是同步的，会阻塞当前线程直到循环所有的迭代都完成。当提交给并发队列时，循坏迭代的顺序是不确定的。运行每个迭代的线程可能会阻塞导致一个迭代在其它迭代之前或者之后完成。因此，你用于每个循环迭代的Block对象或者函数必须是可重入的。

Listing 5-3展示了如何使用GCD等价的替换**for**循环。你传递给**dispatch_apply** 或 **dispatch_apply_f**函数的Block或者函数必须接收一个整型值的参数，用来指示当前的循环迭代。在这个例子中，简单的将循环的次数打印到控制台。

Listing 5-3  Replacing a for loop without striding

    queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
    dispatch_apply(count, queue, ^(size_t i) {
        printf("%u\n", i);
    });

尽管之前的例子很简单，演示了使用调度队列替换**for**循环的基本技术。虽然这是优化基于循环代码的好方法，但是你仍然需要明智的使用这项技术。虽然调度队列的开销非常小，但是在线程上调度每个循环迭代仍然是有开销的。因此，你的循环迭代必须由一定的工作量才能忽略这些开销。你应该使用性能工具找出到底需要执行多大的工作量。

提升每次循环迭代的工作量的方法就是使用跨步。使用跨步时，重写原来的Block代码，执行多个循环迭代。从而减少了你指定给**dispatch_apply**函数的count值。Listing 5-4展示了使用跨步实现Listing 5-3的代码。Block仍然会打印出当前迭代循环的索引，当前跨步是137。(真实的跨步值应该由你基于工作量自己配置。)因为有余值，任何剩余的迭代内联执行。

Listing 5-4  Adding a stride to a dispatched for loop

    int stride = 137;
    dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);

    dispatch_apply(count / stride, queue, ^(size_t idx){
        size_t j = idx * stride;
        size_t j_stop = j + stride;
        do {
            printf("%u\n", (unsigned int)j++);
        }while (j < j_stop);
    });

    size_t i;
    for (i = count - (count % stride); i < count; i++)
    printf("%u\n", (unsigned int)i);

使用跨步有明确的性能优势，特别是当循环迭代的次数很大的时候。调度少量的Block并发，意味着执行代码花的时间比调度它们的时间更多。正如任何性能指标，你应该多试几次跨步值，找到你代码最高效的那个跨步值。

## Replacing Thread Joins
(替换Thread Joins)

Thread Joins允许你生成一个或者多个线程，然后让当前线程等待这些线程完成。为了实现一个Thread Joins，父线程创建子线程时作为joinable的线程。如果父线程没有子线程的结果无法继续处理，就可以join子线程。会阻塞父线程直到子线程的任务完成后退出，这时候父线程可以获得子线程的结果然后继续它的工作。父线程可以一次join多个子线程。

**Dispatch groups**提供了类似于Thread Joins的语义，但是拥有一些额外的优势。和Thread Joins类似，**Dispatch groups**也能够阻塞一个线程直到一个或者多个子任务执行完成。但是不同于Thread Joins，**Dispatch groups**同时等待所有子任务。因为**Dispatch groups**使用调度队列执行任务，非常高效。

使用**Dispatch groups**替换Thread Joins，步骤如下：

1. 使用**dispatch_group_create**函数创建一个新的**dispatch group**。
2. 使用**dispatch_group_async** 和 **dispatch_group_async_f**函数给组添加任务。每个提交到组的任务表示你使用joinable线程执行的普通任务。
3. 如果当前线程不能继续任何工作，调用**dispatch_group_wait**函数等待组。这个函数会阻塞当前线程直到组里所有的任务执行完成。

如果你使用操作对象实现你的任务，你也可以使用依赖实现Thread Joins。不过这时候不是让父线程等待一个或者多个任务完成，而是将父线程的代码移到操作对象中。然后你就可以设置父的操作对象和任意数量的子操作对象之间的依赖，执行joinable线程执行的普通任务。因为父的操作对象在其它操作对象上有依赖，阻止了父操作对象在其它操作完成之前执行。

关于如何使用**dispatch groups**，参见[ Waiting on Groups of Queued Tasks ](https://developer.apple.com/library/prerelease/mac/documentation/General/Conceptual/ConcurrencyProgrammingGuide/OperationQueues/OperationQueues.html#//apple_ref/doc/uid/TP40008091-CH102-SW25)。关于操作对象之间设置依赖，参见[ Configuring Interoperation Dependencies ](https://developer.apple.com/library/prerelease/mac/documentation/General/Conceptual/ConcurrencyProgrammingGuide/OperationObjects/OperationObjects.html#//apple_ref/doc/uid/TP40008091-CH101-SW17)。

## Changing Producer-Consumer Implementations
(修改生产者－消费者实现)

生产者－消费者模型可以让你管理有限的动态产生的资源。当生产者创建了一个新的资源，一个或者多个消费者等待并消耗这些资源。通常实现生产者－消费者的典型机制是条件或者信号量。

使用条件时，生产者线程通常如下：

1. 使用**pthread_mutex_lock**锁住与条件相关联的mutex。
2. 产生要消费的资源或者工作。
3. 使用信号提醒条件变量有资源可以消费了(使用**pthread_cond_signal**)。
4. 使用**pthread_mutex_unlock**解锁mutex。

对应的消费者线程如下：

1. 使用**pthread_mutex_lock**锁住条件相关的mutex。
2. 设置一个**while**循环做如下工作：
   * 检查是否有真的有工作在做。
   * 如果没有工作做(或者没有资源可用)，调用**pthread_cond_wait**阻塞当前线程直到相应的信号触发。
3. 获得生产者提供的工作或资源。
4. 使用**pthread_mutex_unlock**解锁mutex。
5. 处理工作。

使用调度队列，你只需要一个调用就可以实现生产者和消费者模型：

    dispatch_async(queue, ^{
        // Process a work item.
    });

当你的生产者有工作要做时，只需要将工作添加到队列，并让队列去处理该工作。唯一需要确定的就是队列的类型。如果生产者生成的任务需要特定的执行顺序。你可以使用串行队列。如果生产者生成的任务并发执行，使用并发队列，然后让系统同时尽可能多的执行。

## Replacing Semaphore Code
(替换信号量代码)

如果你正在使用信号量来限制共享资源的访问，你应该考虑使用调度信号量来替换。传统的信号量总是需要调用内核来测试信号量。与此相反，调度信号量在用户空间快速测试信号状态，只有测试失败的时候才会调用内核，调用线程需要阻塞。在没有竞争的情形下，调度信号量的行为比传统信号量要快的多。在其它方面，两者的行为是类似的。

关于如何使用调度信号量，参见[ Using Dispatch Semaphores to Regulate the Use of Finite Resources ](https://developer.apple.com/library/prerelease/mac/documentation/General/Conceptual/ConcurrencyProgrammingGuide/OperationQueues/OperationQueues.html#//apple_ref/doc/uid/TP40008091-CH102-SW24)。

## Replacing Run-Loop Code
(替换Run-Loop代码)

如果你使用Run-Loop来管理一个或多个线程执行的工作，你会发现使用队列来实现和维护任务要简单许多。设置一个自定义的Run-Loop需要同时设置底层线程和Run-Loop本身。Run-Loop代码需要包含设置一个或者多个run-loop sources然后编写回调处理到达这些sources上的事件。你可以简单的创建一个串行队列然后调度任务给它。你可以使用一行代码来替换你所有的小城和Run-Loop创建的代码：

	dispatch_queue_t myNewRunLoop = dispatch_queue_create("com.apple.MyQueue", NULL);

因为队列自动执行添加进来的任务，不需要额外的代码来管理队列。你不需要创建或者配置一个线程，也不需要创建或者附加任何的run-loop sources。此外，你还可以通过简单的添加任务到这个队列上来执行其它类型的任务。使用Run-Loop来实现这一点，你需要修改现有的run loop source或者创建一个新的处理新的数据。

Run-Loop的一个常用配置就是处理网络套接字异步到达的数据。为了替代这种行为类型的Run-Loop，你可以附加一个调度源到期望的队列上。调度源同样也提供了比传统run loop sources更多的选择来处理数据。除了处理定时器和网络端口的事件，你也可以使用调度源读写文件。监控文件系统对象，监控进程，以及信号。你甚至可以自定义调度源然后由你部分的异步代码来触发。关于设置调度源的更多信息，参见[ Dispatch Sources ]()。

## Compatibility with POSIX Threads
(与POSIX线程的兼容性)

因为Grand Central Dispatch管理你提供的任务之间的关系以及运行任务的线程，通常你应该避免在任务代码中调用POSIX线程事务。如果由于一些原因你必须调用，你应该非常小心你调用的事务。接下来的部分给你提供了那些事务在你任务代码中调用是安全的，哪些是不安全的。这个列表是未完成的，但是应该提示你哪些调用是安全的，哪些不是。

通常来说，你的应用程序不能删除或者改变不是自己创建的对象和数据结构。因此，调度队列执行的Block对象不能调用如下函数：

    pthread_detach
    pthread_cancel
    pthread_join
    pthread_kill
    pthread_exit

虽然在任务运行的时候修改线程的状态是可以的，你必须在任务返回之前还原线程原来的状态。只要你还原了线程原来的状态，调用如下的函数就是安全的：

    pthread_setcancelstate
    pthread_setcanceltype
    pthread_setschedparam
    pthread_sigmask
    pthread_setspecific

用来执行给定Block的底层线程在多次调用间会发生改变。因此，你的应用程序不应该依赖于如下函数与调用Block之间的返回值信息：

    pthread_self
    pthread_getschedparam
    pthread_get_stacksize_np
    pthread_get_stackaddr_np
    pthread_mach_thread_np
    pthread_from_mach_thread_np
    pthread_getspecific

**重要**：Block必须捕获和禁止任何在其中抛出的语言级别的异常。Block执行期间的其它错误也应该由Block处理或者通知应用程序。

更多关于POSIX线程的信息以及这里提到的函数，参见[ pthread ](https://developer.apple.com/library/prerelease/mac/documentation/Darwin/Reference/ManPages/man3/pthread.3.html#//apple_ref/doc/man/3/pthread)主页。

================= 结束 =================

到这里就结束了，官方文档最后一章是名词解释，这里就不翻译了。[ Glossary ](https://developer.apple.com/library/prerelease/mac/documentation/General/Conceptual/ConcurrencyProgrammingGuide/Glossary/Glossary.html#//apple_ref/doc/uid/TP40008091-CH104-SW2)
