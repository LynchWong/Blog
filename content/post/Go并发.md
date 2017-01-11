---
title: "Go并发"
date: 2015-06-30 14:06:56
categories: 
- Go
tags: 
- Go语言编程
metaAlignment: center
autoThumbnailImage: no
coverImage: //d1u9biwaxjngwg.cloudfront.net/welcome-to-tranquilpeak/city.jpg

---

先留个坑，后面再填。

因为之前做iOS很少会有大并发的操作，顶多多线程操作。对并发了解的并不多，等之后有了一定了解的时候再记录。
<!--more-->

<!--#并发

有人把Go比作21世纪的C语言，第一是因为Go语言设计简单，第二，21世纪最重要的就是并行程序设计，而Go从语言层面就支持了并行。


##goroutine

goroutine是Go并行设计的核心。goroutine说到底其实就是线程，但是它比线程更小，十几个goroutine可能体现在底层就是五六个线程，Go语言内部帮你实现了这些goroutine之间的内存共享。执行goroutine只需极少的栈内存(大概是4~5KB)，当然会根据相应的数据伸缩。也正因为如此，可同时运行成千上万个并发任务。goroutine比thread更易用、更高效、更轻便。

goroutine是通过Go的runtime管理的一个线程管理器。goroutine通过`go`关键字实现了，其实就是一个普通的函数。

	go hello(a, b, c)
	
通过关键字go就启动了一个goroutine。我们来看一个例子：

	package main

	import (
		"fmt"
		"runtime"
	)

	func say(s string) {
		for i := 0; i < 5; i++ {
			runtime.Gosched()
			fmt.Println(s)
		}
	}

	func main() {
		go say("world") //开一个新的Goroutines执行
		say("hello") //当前Goroutines执行
	}

	// 以上程序执行后将输出：
	// hello
	// world
	// hello
	// world
	// hello
	// world
	// hello
	// world
	// hello
	
我们可以看到go关键字很方便的就实现了并发编程。
上面的多个goroutine运行在同一个进程里面，共享内存数据，不过设计上我们要遵循：不要通过共享来通信，而要通过通信来共享。

> runtime.Gosched()表示让CPU把时间片让给别人，下次某个时候继续恢复执行该goroutine。

>默认情况下，调度器仅使用单线程，也就是说只实现了并发。想要发挥多核处理器的并行，需要在我们的程序中显式调用 runtime.GOMAXPROCS(n) 告诉调度器同时使用多个线程。GOMAXPROCS 设置了同时运行逻辑代码的系统线程的最大数量，并返回之前的设置。如果n < 1，不会改变当前设置。以后Go的新版本中调度得到改进后，这将被移除。这里有一篇Rob介绍的关于并发和并行的文章：http://concur.rspace.googlecode.com/hg/talk/concur.html#landing-slide

##channels

goroutine运行在相同的地址空间，因此访问共享内存必须做好同步。那么goroutine之间如何进行数据的通信呢，Go提供了一个很好的通信机制channel。channel可以与Unix shell 中的双向管道做类比：可以通过它发送或者接收值。这些值只能是特定的类型：channel类型。定义一个channel时，也需要定义发送到channel的值的类型。注意，必须使用make 创建channel：

	ci := make(chan int)
	cs := make(chan string)
	cf := make(chan interface{})

channel通过操作符`<-`来接收和发送数据

	ch <- v    // 发送v到channel ch.
	v := <-ch  // 从ch中接收数据，并赋值给v

我们把这些应用到我们的例子中来：

	package main

	import "fmt"

	func sum(a []int, c chan int) {
		total := 0
		for _, v := range a {
			total += v
		}
		c <- total  // send total to c
	}

	func main() {
		a := []int{7, 2, 8, -9, 4, 0}

		c := make(chan int)
		go sum(a[:len(a)/2], c)
		go sum(a[len(a)/2:], c)
		x, y := <-c, <-c  // receive from c

		fmt.Println(x, y, x + y)
	}
	
默认情况下，channel接收和发送数据都是阻塞的，除非另一端已经准备好，这样就使得Goroutines同步变的更加的简单，而不需要显式的lock。所谓阻塞，也就是如果读取（value := <-ch）它将会被阻塞，直到有数据接收。其次，任何发送（ch<-5）将会被阻塞，直到数据被读出。无缓冲channel是在多个goroutine之间同步很棒的工具。

##Buffered Channels

上面我们介绍了默认的非缓存类型的channel，不过Go也允许指定channel的缓冲大小，很简单，就是channel可以存储多少元素。ch:= make(chan bool, 4)，创建了可以存储4个元素的bool 型channel。在这个channel 中，前4个元素可以无阻塞的写入。当写入第5个元素时，代码将会阻塞，直到其他goroutine从channel 中读取一些元素，腾出空间。

	ch := make(chan type, value)

	value == 0 ! 无缓冲（阻塞）
	value > 0 ! 缓冲（非阻塞，直到value 个元素）

我们看一下下面这个例子，你可以在自己本机测试一下，修改相应的value值


	package main

	import "fmt"

	func main() {
		c := make(chan int, 2)//修改2为1就报错，修改2为3可以正常运行
		c <- 1
		c <- 2
		fmt.Println(<-c)
		fmt.Println(<-c)
	}
        //修改为1报如下的错误:
        //fatal error: all goroutines are asleep - deadlock!

##Range和Close

##Select

##超时

##runtime goroutine-->
