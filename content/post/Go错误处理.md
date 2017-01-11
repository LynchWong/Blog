---
title: "Go错误处理"
date: 2015-06-26 15:49:50
categories: 
- Go
tags: 
- Go语言编程
metaAlignment: center
autoThumbnailImage: no
coverImage: //d1u9biwaxjngwg.cloudfront.net/welcome-to-tranquilpeak/city.jpg

---

<!--more-->

# 错误处理

错误处理是学习任何编程语言都需要考虑的一个重要话题。在早期的语言中，错误处理不是语言规范的一部分，通常只作为一种编程范式存在，比如C语言中的errno。但自C++语言以来，语言层面上会增加错误处理的支持，比如异常(exception)的概念和try-catch关键字的引入。Go语言在此功能上考虑得更为深远。漂亮的错误处理规范是Go语言最大的亮点之一。

## error接口

Go语言引入了一个关于错误处理的标准模式，即error接口，该接口的定义如下：

	type error interface {
		Error() string
	}
	
对于大多数函数，如果要返回错误，大致上都可以定义为如下模式，将error作为多种返回值中的最后一个，但这并非是强制要求：

	func Foo(param int) (n int, err error) {
		 // ...
	}
	
调用时的代码建议按如下方式处理错误情况：

	n, err := Foo(0)
	
	if err != nil {
		// 错误处理
	} else {
		// 使用返回值n
	}

下面我用Go库中的实际代码来示范如何使用自定义的error类型。

首先，定义一个用于承载错误信息的类型。因为Go语言中接口的灵活性，你根本不需要从error接口继承或者像Java一样需要使用implements来明确指定类型和接口之间的关系，具体代码如下：

	type PathError struct {
		Op string
		Path string
		Err error
	}
	
如果这样的话，编译器又怎能知道PathError可以当一个error来传递呢?关键在于下面的代码实现了Error()方法：

	func (e *PathError) Error() string {
		return e.Op + " " + e.Path + ": " + e.Err.Error()
	}

关于接口的更多细节，后面再介绍。之后就可以直接返回PathError变量了，比如在下面的代码中，当syscall.Stat()失败返回err时，将该err包装到一个PathError对象中返回:

	func Stat(name string) (fi FileInfo, err error) {
		var stat syscall.Stat_t
		
		err = syscall.Stat(name, &stat)
		
		if err != nil {
			return nil, &PathError{"stat", name, err}
		}
		
		return fileInfoFromStat(&stat, name), nil
	}

如果在处理错误时获取详细信息，而不仅仅满足于打印一句错误信息，那就需要用到类型转换知识了：

	fi, err := os.Stat("a.txt")
	
	if err != nil {
		if e, ok := err.(*os.PathError); ok && e.Err != nil {
			// 获取PathError类型变量e中的其他信息并处理
		}
	}
	
这就是Go中error类型的使用方法。与其他语言中的异常相比，Go的处理相对比较直观、简单。 

关于类型转换的更多知识，在第3章中也会有更进一步的阐述。

## defer

	func CopyFile(dst, src string) (w int64, err error) { 
		srcFile, err := os.Open(src)
		if err != nil {
			return
		}
		
		defer srcFile.Close()
	
		dstFile, err := os.Create(dstName) 
		if err != nil {
			return
		}
	
		defer dstFile.Close()
		return io.Copy(dstFile, srcFile) 
	}
	
即使其中的CopyFile()函数抛出异常，Go仍然会保证dstFile和srcFile会被正常关闭。 

如果觉得一句话干不完清理的工作，也可以使用在defer后加一个匿名函数的做法：

	defer func() {
		// 做你复杂的清理工作
	} ()

另外，一个函数中可以存在多个defer语句，因此需要注意的是，defer语句的调用是遵照￼￼先进后出的原则，即最后一个defer语句将最先被执行。只不过，当你需要为defer语句到底哪个先执行这种细节而烦恼的时候，说明你的代码架构可能需要调整一下了。
	
## panic()和recover()

Go语言引入了两个内置函数panic()和recover()以报告和处理运行时错误和程序中的错误场景：

	func panic(interface{}) 
	func recover() interface{}
	
当在一个函数执行过程中调用panic()函数时，正常的函数执行流程将立即终止，但函数中之前使用defer关键字延迟执行的语句将正常展开执行，之后该函数将返回到调用函数，并导致逐层向上执行panic流程，直至所属的goroutine中所有正在执行的函数被终止。错误信息将被报告，包括在调用panic()函数时传入的参数，这个过程称为错误处理流程。

从panic()的参数类型interface{}我们可以得知，该函数接收任意类型的数据，比如整 型、字符串、对象等。调用方法很简单，下面为几个例子：

	panic(404)
	panic("network broken") 
	panic(Error("file not exists"))
	
recover()函数用于终止错误处理流程。一般情况下，recover()应该在一个使用defer关键字的函数中执行以有效截取错误处理流程。如果没有在发生异常的goroutine中明确调用恢复过程(使用recover关键字)，会导致该goroutine所属的进程打印异常信息后直接退出。

以下为一个常见的场景。
我们对于foo()函数的执行要么心里没底感觉可能会触发错误处理，或者自己在其中明确加入了按特定条件触发错误处理的语句，那么可以用如下方式在调用代码中截取recover()：
	defer func() {
		if r := recover(); r != nil {
			log.Printf("Runtime error caught: %v", r)
		}
	}()
	
	foo()
无论foo()中是否触发了错误处理流程，该匿名defer函数都将在函数退出时得到执行。假如foo()中触发了错误处理流程，recover()函数执行将使得该错误处理过程终止。如果错误处理流程被触发时,程序传给panic函数的参数不为nil，则该函数还会打印详细的错误信息。
