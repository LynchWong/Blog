---
title: "Advanced Memory Management Programming Guide - Using Autorelease Pool Blocks"
date: 2016-02-28 15:03:32
categories: 
- 编程指南
tags: 
- 内存管理
metaAlignment: center
autoThumbnailImage: no
coverImage: //d1u9biwaxjngwg.cloudfront.net/welcome-to-tranquilpeak/city.jpg

---

[官方文档](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/MemoryMgmt/Articles/mmAutoreleasePools.html#//apple_ref/doc/uid/20000047-CJBFBEDI)
<!--more-->

# Using Autorelease Pool Blocks
(使用Autorelease池)

Autorelease池为你提供了延迟释放对象所有权的机制，避免了对象立即被dealloc(比如方法中返回的对象)。通常，你不需要创建你自己的Autorelease池，但个别情况下你必须或者最好这么做。

## About Autorelease Pool Blocks
(关于Autorelease池)

Autorelease池的代码块用@autoreleasepool标识，如下所示：

    @autoreleasepool {
        // Code that creates autoreleased objects.
    }

在Autorelease池代码块的最后，对象会接收autorelease消息。

和其他代码块类似，Autorelease池代码块也可以被嵌套：

    @autoreleasepool {
        // . . .
        @autoreleasepool {
            // . . .
        }
        . . .
    }

(正常情况下你不会看见如上所示的代码，通常一个资源文件中的Autorelease池代码块中的代码会调用其他资源文件中的代码，这些代码可能包含了其他的Autorelease池代码块。)当发送autorelease消息时，发送autorelease消息的对应的Autorelease池的代码块会发送release消息。

Cocoa总是期望代码在Autorelease池代码块中执行，否则autorelease的对象不会被release导致你应用程序内存泄漏。(如果你在Autorelease池代码块之外发送autorelease消息，Cocoa会打印一个适当的错误消息。)AppKit和UIKit框架自动在每个消息循环的开始都创建一个池(比如鼠标按下事件、触摸事件)并在结尾处销毁这个池。正因为如此，你实际上不需要创建Autorelease池，甚至不需要知道创建Autorelease池的代码如何写。下面三种情形，你应该使用自己的Autorelease池：

1. 如果你写的程序，不是基于UI framework，比如命令行程序。
2. 如果你写循环的时候创建了大量的临时对象。你可以在循环体内使用Autorelease池在下一次循环开始前销毁这些对象。这样可以减少你的应用程序对内存的占用峰值。
3. 如果你发送了一个secondary线程(main线程之外的线程)。你必须在线程的最初执行代码中创建Autorelease池；否则你的应用程序就内存泄漏了。

## Use Local Autorelease Pool Blocks to Reduce Peak Memory Footprint
(使用本地Autorelease池减少内存占用峰值)

许多程序使用的临时对象都是autorelease的，这些对象是会占用内存的。要想减少对内存的占用峰值，就应该使用本地的Autorelease池。当池被销毁时，那些对象被release，进而系统占用内存情况得以改善。

下面的例子，在for循环中使用了本地Autorelease池：

    NSArray *urls = <# An array of file URLs #>;
    for (NSURL *url in urls) {
        
        @autoreleasepool {
            NSError *error;
            NSString *fileContents = [NSString stringWithContentsOfURL:url
                                                              encoding:NSUTF8StringEncoding error:&error];
            /* Process the string, creating and autoreleasing more objects. */
        }
    }

for循环每次处理一个文件。任何对象(比如fileContents)在Autorelease池中都会发送一个autorelease消息，存放于池中。当池在单次循环结束时，进行销毁，这些对象也就release了。

在Autorelease池之后，那些曾经收到autorelease消息的对象，只能视为失效，而不要再给他们发消息，或者把他们作为返回值进行返回。如果你必须在Autorelease池代码块之后使用某个对象，你可以先发送一个retain消息，然后在Autorelease池代码块之后发送autorelease消息，如下所示：

    – (id)findMatchingObject:(id)anObject {
        
        id match;
        while (match == nil) {
            @autoreleasepool {
                
                /* Do a search that creates a lot of temporary objects. */
                match = [self expensiveSearchForObject:anObject];
                
                if (match != nil) {
                    [match retain]; /* Keep match around. */
                }
            }
        }
        
        return [match autorelease];   /* Let match go and return it. */
    }

在Autorelease池代码块中给match发送retain，然后在Autorelease池代码块之后发送autorelease，这样延长了match的生命周期。允许match在循环之外接收消息，并且返回给findMatchingObject:方法的调用者。

## Autorelease Pool Blocks and Threads
(Autorelease池和线程)

Cocoa应用程序里的每一个线程都维护的自己的Autorelease池的栈。如果你写的程序仅仅是一个基于Foundation的程序，或者你要detach一个线程，你需要创建你自己的Autorelease池。

如果你的程序或者线程是长期运行，可能会产生大量的临时对象，你应该使用Autorelease池(就像AppKit和UIKit在主线程上做的那样)；否则autorelease对象就会积累并消耗掉大量内存。如果你detach线程不调用Cocoa，你就不必使用Autorelease池。

**注意：**如果你使用POSIX线程API创建secondary线程，而不是NSThread，你不能使用Cocoa，除非Cocoa处于多线程模式。Cocoa只有在detach它的第一个NSThread对象之后，才能进入多线程模式。为了在secondary POSIX线程上使用Cocoa，你的应用程序首先要做的是detach至少一个NSThread对象，然后立刻结束这个线程。你可以使用NSThread的类方法isMultiThreaded来检测Cocoa是否处于多线程模式。
