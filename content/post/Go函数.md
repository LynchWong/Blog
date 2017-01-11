---
title: "Go函数"
date: 2015-06-26 13:56:25
categories: 
- Go
tags: 
- Go语言编程
metaAlignment: center
autoThumbnailImage: no
coverImage: //d1u9biwaxjngwg.cloudfront.net/welcome-to-tranquilpeak/city.jpg

---

<!--more-->

# 函数

函数构成代码执行的逻辑结构。在Go语言中，函数的基本组成为：关键字func、函数名、参数列表、返回值、函数体和返回语句。

## 函数定义

前面我们已经大概介绍过函数，这里我们用一个最简单的加法函数来进行详细说明：

	package mymath
	import "errors"
	
	func Add(a int, b int) (ret int, err error) {
		if a < 0 || b < 0 {// 假设这个函数只支持两个非负数值的加法
			err = errors.New("Should be non-negative numbers!")
			return
		}
		return a + b, nil
	}

<!--more-->

如果参数列表中若干个相邻的参数类型的相同，比如上面例子中的a和b，则可以在参数列表中省略前面变量的类型声明，如下所示：

	func Add(a, b int) (ret int, err error) {
		// ...
	}
	
如果返回值列表中多个返回值的类型相同，也可以用同样的方式合并。如果函数只有一个返回值，也可以这么写：

	func Add(a, b int) int {
		// ...
	}

## 函数调用

函数调用非常方便，只要事先导入了该函数所在的包，就可以直接按照如下所示的方式调用函数：

	import "mymath"// 假设Add被放在一个叫做mymath的包中
		// ...
	c := mymath.Add(1, 2)
	
在Go语言中，函数支持多重返回值，这在之后的内容中会介绍。利用函数的多重返回值和错误处理机制，我们可以很容易地写出优雅美观的Go代码。

Go语言中函数名字的大小写不仅仅是风格，更直接体现了该函数的可见性，这一点尤其需要注意。对于很多注意美感的程序员(尤其是工作在Linux平台上的C程序员)而言，这里的函数名的首字母大写可能会让他们感觉不太适应，在自己练习的时候可能会顺手改成全小写，比如写 成add_xxx这样的Linux风格。很不幸的是，如果这样做了，你可能会遇到莫名其妙的编译错误，比如你明明导入了对应的包，Go编译器还是会告诉你无法找到add_xxx函数。

因此需要先牢记这样的规则：小写字母开头的函数只在本包内可见，大写字母开头的函数才能被其他包使用。

这个规则也适用于类型和变量的可见性。

## 不定参数

在C语言时代大家一般都用过printf()函数，从那个时候开始其实已经在感受不定参数的魅力和价值。如同C语言中的printf()函数，Go语言标准库中的fmt.Println()等函数的实现也严重依赖于语言的不定参数功能。

本节我们将介绍不定参数的用法。合适地使用不定参数,可以让代码简单易用，尤其是输入输出类函数，比如日志函数等。

### 不定参数类型

不定参数是指函数传入的参数个数为不定数量。为了做到这点，首先需要将函数定义为接受不定参数类型：

	func myfunc(args ...int) {
		for _, arg := range args {
			fmt.Println(arg)
		}
	}
	
这段代码的意思是，函数myfunc()接受不定数量的参数，这些参数的类型全部是int，所以它可以用如下方式调用：

	myfunc(2, 3, 4)
	myfunc(1, 3, 7, 13)

形如...type格式的类型只能作为函数的参数类型存在，并且必须是最后一个参数。它是一个语法糖(syntactic sugar)，即这种语法对语言的功能并没有影响，但是更方便程序员使用。通常来说，使用语法糖能够增加程序的可读性，从而减少程序出错的机会。

从内部实现机理上来说，类型...type本质上是一个数组切片，也就是[]type，这也是为什么上面的参数args可以用for循环来获得每个传入的参数。

假如没有...type这样的语法糖，开发者将不得不这么写：
	func myfunc2(args []int) {
		for _, arg := range args {
			fmt.Println(arg)
		}
	}
	
从函数的实现角度来看，这没有任何影响，该怎么写就怎么写。但从调用方来说，情形则完全不同：

	myfunc2([]int{1, 3, 7, 13})
	
你会发现，我们不得不加上[]int{}来构造一个数组切片实例。但是有了...type这个语法糖，我们就不用自己来处理了。

### 不定参数的传递

假设有另一个变参函数叫做myfunc3(args ...int)，下面的例子演示了如何向其传递变参：

func myfunc(args ...int) {
	
	//按原样传递
	myfunc3(args...)
	
	//传递片段，实际上任意的int slice都可以传进去
	myfunc3(args[1:]...)
}

### 任意类型的不定参数

之前的例子中将不定参数类型约束为int，如果你希望传任意类型，可以指定类型为interface{}。下面是Go语言标准库中fmt.Printf()的函数原型：

	func Printf(format string, args ...interface{}) {
		// ...
	}
	
用interface{}传递任意类型数据是Go语言的惯例用法。使用interface{}仍然是类型安全的，这和 C/C++ 不太一样。下面代码示范了如何分派传入interface{}类型的数据。

	package main
	import "fmt"
	
	func MyPrintf(args ...interface{}) {
		for _, arg := range args {
			switch arg.(type) {
				case int:
					fmt.Println(arg, "is an int value.")
				case string:
					fmt.Println(arg, "is an string value.")
				case int64:
					fmt.Println(arg, "is an int64 value.")
				default:
					fmt.Println(arg, "is an unknown value.")
			}
		}
	}
	
	func main() {
		var v1 int = 1
		var v2 int64 = 234
		var v3 string = "hello"
		var v4 float32 = 1.234
		MyPrintf(v1, v2, v3, v4)
	}

## 多返回值

与C、C++和Java等开发语言的一个极大不同在于，Go语言的函数或者成员的方法可以有多个返回值，这个特性能够使我们写出比其他语言更优雅、更简洁的代码，比如File.Read()函数就可以同时返回读取的字节数和错误信息。如果读取文件成功，则返回值中的n为读取的字节数，err为nil，否则err为具体的出错信息：

	func (file *File) Read(b []byte) (n int, err Error)
	
同样，从上面的方法原型可以看到，我们还可以给返回值命名，就像函数的输入参数一样。返回值被命名之后，它们的值在函数开始的时候被自动初始化为空。在函数中执行不带任何参数的return语句时，会返回对应的返回值变量的值。

Go语言并不需要强制命名返回值，但是命名后的返回值可以让代码更清晰，可读性更强，同时也可以用于文档。

如果调用方调用了一个具有多返回值的方法，但是却不想关心其中的某个返回值，可以简单地用一个下划线“_”来跳过这个返回值，比如下面的代码表示调用者在读文件的时候不想关心Read()函数返回的错误码：

	n, _ := f.Read(buf)

## 匿名函数与闭包

匿名函数是指不需要定义函数名的一种函数实现方式，它并不是一个新概念，最早可以回溯到1958年的Lisp语言。但是由于种种原因，C和C++一直都没有对匿名函数给以支持，其他的各种语言，比如JavaScript、C#和Objective-C等语言都提供了匿名函数特性，当然也包含Go语言。

### 匿名函数

在Go里面，函数可以像普通变量一样被传递或使用，这与C语言的回调函数比较类似。不同的是，Go语言支持随时在代码里定义匿名函数。

匿名函数由一个不带函数名的函数声明和函数体组成，如下所示：

	func(a, b int, z float64) bool {
		return a * b < int(z)
	}
	
匿名函数可以直接赋值给一个变量或者直接执行：

	f := func(x, y int) int {
		return x + y
	}
	
	func(ch chan int) {
		ch <- ACK
	} (reply_chan) //花括号后直接跟参数列表表示函数调用

### 闭包

Go的匿名函数是一个闭包，下面我们先来了解一下闭包的概念、价值和应用场景。

#### 基本概念

闭包是可以包含自由(未绑定到特定对象)变量的代码块，这些变量不在这个代码块内或者任何全局上下文中定义，而是在定义代码块的环境中定义。要执行的代码块(由于自由变量包含在代码块中，所以这些自由变量以及它们引用的对象没有被释放)为自由变量提供绑定的计算环境(作用域)。

#### 闭包的价值

闭包的价值在于可以作为函数对象或者匿名函数，对于类型系统而言，这意味着不仅要表示数据还要表示代码。支持闭包的多数语言都将函数作为第一级对象,就是说这些函数可以存储到变量中作为参数传递给其他函数，最重要的是能够被函数动态创建和返回。

#### Go语言中的闭包

Go语言中的闭包同样也会引用到函数外的变量。闭包的实现确保只要闭包还被使用，那么被闭包引用的变量会一直存在，如下所示：

	package main
	import "fmt"
	
	func main() {
		var j int = 5
		
		a := func() (func()) {
			var i int = 10
			return func() {
				fmt.Printf("i, j: %d, %d\n", i, j)
			}
		}()
		
		a()
		j *= 2
		a()
	}
	
上述例子的执行结果是：

	i, j: 10, 5
	i, j: 10, 10
在上面的例子中，变量a指向的闭包函数引用了局部变量i和j，i的值被隔离，在闭包外不能被修改，改变j的值以后，再次调用a，发现结果是修改过的值。

￼在变量a指向的闭包函数中，只有内部的匿名函数才能访问变量i，而无法通过其他途径访问到，因此保证了i的安全性。
