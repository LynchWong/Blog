---
title: "Go流程控制"
date: 2015-06-21 01:23:08
categories: 
- Go
tags: 
- Go语言编程
metaAlignment: center
autoThumbnailImage: no
coverImage: //d1u9biwaxjngwg.cloudfront.net/welcome-to-tranquilpeak/city.jpg

---

<!--more-->

# 流程控制

流程控制决定了程序代码的执行路径，建立程序的逻辑结构。可以说，流程控制语句是整个程序骨架。

从根本上讲，流程控制只是为了控制程序语句的执行顺序，一般需要与各种条件配合，因此，在各种流程中，会加入条件判断语句。流程控制语句一般起以下3个作用：

* **选择**，即根据条件跳转到不同的执行序列；
* **循环**，即根据条件反复执行某个序列，当然每一次循环执行的输入输出可能会发生变化；
* **跳转**，即根据条件返回到某执行序列。

Go语言支持如下的几种流程控制语句：

* **条件语句**，对应的关键字为if、else和else if；
* **选择语句**，对应的关键字为switch、case和select(将在介绍channel的时候细说)；
* **循环语句**，对应的关键字为for和range；
* **跳转语句**，对应的关键字为goto。

在具体的应用场景中，为了满足更丰富的控制需求，Go语言还添加了如下关键字：break、 continue和fallthrough。在实际的使用中，需要根据具体的逻辑目标、程序执行的时间和空间限制、代码的可读性、编译器的代码优化设定等多种因素，灵活组合。

## 条件语句

关于条件语句的样例代码如下：

	if a < 5 { 
		return 0
	} else { 
		return 1
	}
关于条件语句,需要注意以下几点：
* 条件语句不需要使用括号将条件包含起来()：
* 无论语句体内有几条语句，花括号{}都是必须存在的：
* 左花括号{必须与if或者else处于同一行：
* 在if之后，条件语句之前，可以添加变量初始化语句，使用;间隔：
* 在有返回值的函数中，不允许将“最终的”return语句包含在if...else...结构中，否则会编译失败：function ends without a return statement。
失败的原因在于，Go编译器无法找到终止该函数的return语句。编译失败的案例如下：

	func example(x int) int { 
		if x == 0 {
			return 5 
		} else {
			return x 
		}
	}
## 选择语句
根据传入条件的不同，选择语句会执行不同的语句。下面的例子根据传入的整型变量i的不同而打印不同的内容：
	switch i { 
		case 0:
			fmt.Printf("0") 
		case 1:
			fmt.Printf("1") 
		case 2:
			fallthrough 
		case 3:
			fmt.Printf("3") 
		case 4, 5, 6:
			fmt.Printf("4, 5, 6") 
		default:
			fmt.Printf("Default")
	}
运行上面的案例，将会得到如下结果：
* i = 0时，输出0;
* i = 1时，输出1;
* i = 2时，输出3;
* i = 3时，输出3;
* i = 4时，输出4, 5, 6;
* i = 5时，输出4, 5, 6;
* i = 6时，输出4, 5, 6;
* i = 其他任意值时，输出Default。
比较有意思的是，switch后面的表达式甚至不是必需的，比如下面的例子：
	switch {
		case 0 <= Num && Num <= 3:
			fmt.Printf("0-3")
		case 4 <= Num && Num <= 6:
			fmt.Printf("4-6")
		case 7 <= Num && Num <= 9:
			fmt.Printf("7-9")
	}
在使用switch结构时，我们需要注意以下几点：
* 左花括号{必须与switch处于同一行；
* 条件表达式不限制为常量或者整数；
* 单个case中,可以出现多个结果选项；
* 与C语言等规则相反，Go语言不需要用break来明确退出一个case；
* 只有在case中明确添加fallthrough关键字，才会继续执行紧跟的下一个case；
* 可以不设定switch之后的条件表达式，在此种情况下，整个switch结构与多个if...else...的逻辑作用等同。
## 循环语句
与多数语言不同的是，Go语言中的循环语句只支持for关键字，而不支持while和do-while结构。关键字for的基本使用方法与C和C++中非常接近：
	sum := 0
	for i := 0; i < 10; i++ {
		sum += i 
	}
	
可以看到比较大的一个不同在于for后面的条件表达式不需要用圆括号()包含起来。Go语言还进一步考虑到无限循环的场景，让开发者不用写无聊的for (;;) {} 和 do {} while(1);，而直接简化为如下的写法：

	sum := 0 
	for {
		sum++
		if sum > 100 {
			break
		} 
	}
	
在条件表达式中也支持多重赋值，如下所示：

	a := []int{1, 2, 3, 4, 5, 6}
	for i, j := 0, len(a) – 1; i < j; i, j = i + 1, j – 1 {
	    a[i], a[j] = a[j], a[i]
	}
使用循环语句时，需要注意的有以下几点。
* 左花括号{必须与for处于同一行。
* Go语言中的for循环与C语言一样，都允许在循环条件中定义和初始化变量，唯一的区别是，Go语言不支持以逗号为间隔的多个赋值语句，必须使用平行赋值的方式来初始化多个变量。
* Go语言的for循环同样支持continue和break来控制循环，但是它提供了一个更高级的￼break，可以选择中断哪一个循环，如下例：

本例中，break语句终止的是JLoop标签处的外层循环。

	for j := 0; j < 5; j++ {
		for i := 0; i < 10; i++ {
			if i > 5 { 
				break JLoop
	        }
			fmt.Println(i)
		} 
	}
	JLoop: 
	// ...

## 跳转语句
goto语句被多数语言学者所反对，谆谆告诫不要使用。但对于Go语言这样一个惜关键字如金的语言来说，居然仍然支持goto关键字，无疑让某些人跌破眼镜。但就个人一年多来的Go语言编程经验来说，goto还是会在一些场合下被证明是最合适的。
goto语句的语义非常简单，就是跳转到本函数内的某个标签，如：
	func myfunc() { 
		i := 0
	    HERE:
	    fmt.Println(i)
	    i++
	    if i < 10 {
	￼￼		goto HERE 
		}
	}
