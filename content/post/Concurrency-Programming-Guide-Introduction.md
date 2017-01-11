---
title: "Concurrency Programming Guide - Introduction"
date: 2016-01-14 13:44:09
categories: 
- 编程指南
tags: 
- 并发编程
metaAlignment: center
autoThumbnailImage: no
coverImage: //d1u9biwaxjngwg.cloudfront.net/welcome-to-tranquilpeak/city.jpg

---

[官方文档](https://developer.apple.com/library/prerelease/mac/documentation/General/Conceptual/ConcurrencyProgrammingGuide/Introduction/Introduction.html#//apple_ref/doc/uid/TP40008091-CH1-SW1)
<!--more-->

之前的编程指南我都是写在一篇博客里的，发现有的太长了。感觉上不利于编辑也不利于查找，所以我现在就分成几篇来完成，每个大章节单独用一篇来。

# 简介

重要提示：这是一份正在开发中的API或者技术的初步文档。虽然这个文档已经被技术准确性审核，但是这并不是最终的。

并发的概念就是多个事务发生在同一时间。随着多核CPU的增值以及意识到每个处理器中的内核数量只会增加，所以软件设计者需要新的方法来利用这些优势。尽管像OS X和iOS这样的操作系统都能够并行运行多个程序，大部分这些程序都运行在后台，执行任务时都需要处理器的一小段连续的时间。而前台的应用程序捕捉了用户的注意力，并且使计算机处于繁忙的状态。如果一个应用程序有很多工作要做，但是这些工作只保持在部分的内核上运行，那么这些额外的处理器资源(其它的内核)就浪费了。

在过去，在应用程序中引进并发编程要求创建一个或者更多的额外线程。不幸的是编写线程代码很有挑战性。线程属于底层工具，必须手动管理。鉴于一个应用程序的线程数量会根据当前系统负载和底层硬件动态变化，实现一个正确的线程解决方案极其困难，几乎不可能实现。除此之外，用于线程的同步机制通常会给软件设计带来额外的复杂性和风险，这些并不能保证会提升性能。

OS X和iOS都采用更异步的方式来执行这些任务，而不是像传统的基于线程的系统和应用程序那样。应用程序只需要定义特定的任务，然后让系统执行它们，而不是去直接创建线程。我们让系统来管理线程，应用程序从而获得了一个数量级的可拓展性，而这些在编写线程代码时是不可能的。应用程序的开发者也获得了简单以及高效的编程模型。

本文档介绍的技术，你应该用在你的应用程序中来实现并发。这些技术适用于OS X和iOS。

## 本文档的结构

本文档包含如下章节：

* [ Concurrency and Application Design ](http://lynchwong.com/2016/01/14/Concurrency-Programming-Guide-Concurrency-and-Application-Design/)介绍了异步应用程序设计的基础，以及异步执行你自定义操作的技术的基础。
* [ Operation Queues ](http://lynchwong.com/2016/01/14/Concurrency-Programming-Guide-Operation-Queues/)介绍了如何使用Objective-C对象来封装和执行任务。
* [ Dispatch Queues ](http://lynchwong.com/2016/01/14/Concurrency-Programming-Guide-Dispatch-Queues/)介绍了基于C的应用程序如何并发执行任务。
* [ Dispatch Sources ](http://lynchwong.com/2016/01/14/Concurrency-Programming-Guide-Dispatch-Sources/)展示了如何异步处理系统事件。
* [ Migrating Away from Threads ](http://lynchwong.com/2016/01/14/Concurrency-Programming-Guide-Migrating-Away-from-Threads/)提供如何将你基于线程的代码进行迁移的方法和技术，从而使用更新的技术。

本文档也包含了定义相关条款的一些术语。

## 术语

在进行讨论并发之前，有必要来定义一些相关的术语来避免困惑。一些熟悉**UNIX**和**OS X**系统的开发者可能发现了“task”, “process”, 和 “thread”在文档中的使用不一样方式不太一样。本文档以如下方式来使用这些术语：

* 术语线程用来指执行代码的一个单独的路径。在OS X中的线程的底层实现是基于POSIX线程的API。
* 术语进程用于指正在运行的可执行的，它可以包括多个线程。
* 术语任务用来指需要被执行的工作的抽象概念。

关于完整的术语的定义，以及本文档中使用到的术语，参见[ Glossary ](https://developer.apple.com/library/prerelease/mac/documentation/General/Conceptual/ConcurrencyProgrammingGuide/Glossary/Glossary.html#//apple_ref/doc/uid/TP40008091-CH104-SW2)。

## 参见

本文档专注于如何在你的应用程序中实现并发编程，而没有覆盖线程的使用。如果你需要关于使用线程的信息和其它线程相关的技术，参见[ Threading Programming Guide ](https://developer.apple.com/library/prerelease/mac/documentation/Cocoa/Conceptual/Multithreading/Introduction/Introduction.html#//apple_ref/doc/uid/10000057i)。
