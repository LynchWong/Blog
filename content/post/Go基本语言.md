---
title: "Go基本语法"
date: 2015-06-19 12:30:31
categories: 
- Go
tags: 
- Go语言编程
metaAlignment: center
autoThumbnailImage: no
coverImage: //d1u9biwaxjngwg.cloudfront.net/welcome-to-tranquilpeak/city.jpg

---

<!--more-->

# 变量

## 变量声明

Go语言使用关键字**var**来声明变量，变量类型放在变量名之后，如下所示：

	var v1 int
	var v2 string 
	var v3 [10]int          // 数组
	var v4 []int            // 数组切片
	var v5 struct {
    	f int 
	}
	var v6 *int             // 指针
	var v7 map[string]int   // map,key为string类型,value为int类型
	var v8 func(a int) int 
	
<!--more-->

**var**关键字可以将若干个需要声明的变量放在一起，避免重复书写**var**关键字，如下所示：
	var （
		v1 int
		v2 string
	）
	
## 初始化变量

你可以在声明变量的时候初始化变量，但是这时候**var**关键字不是必须的，如下所示：

	var v1 int = 10 	// 正确的使用方式1
	var v2 = 10			// 正确的使用方式2,编译器可以自动推导出v2的类型
	v3 := 10 			// 正确的使用方式3,编译器可以自动推导出v3的类型
	
这三种使用方式效果是完全一样的。第一种是完整的表达方式，第二种省去了类型说明，编译器会根据赋的值自动推导出v2的类型。第三种的**:=**符号用于明确表达同时进行变量声明和初始化的工作。

指定类型已不再是必需的，Go编译器可以从初始化表达式的右值推导出该变量应该声明为哪种类型，这让Go语言看起来有点像动态类型的语言，但其实Go语言实际上是不折不扣的强类型语言(静态类型语言)。PS：Swift在这点上也是类似的。

出现在**:=**符号左侧的变量不应该是已经被声明过的，否侧会导致编译错误，如下：

	var i int
	i := 2

会导致如下的编译错误：

no new variables on left side of :=

## 变量赋值

在Go语法中，变量初始化和变量赋值是两个不同的概念。下面为声明一个变量之后的赋值过程：

	var v10 int
	v10 = 123
	
多重赋值功能，比如下面这个交换i和j变量的语句：

	i, j = j, i
	
如果大家学过C语言，一定知道需要一个中间变量才能完成如上操作。

## 匿名变量

我们在使用传统的强类型语言编程时，经常会出现这种情况，即在调用函数时为了获取一个值，却因为该函数返回多个值而不得不定义一堆没用的变量。在Go中这种情况可以通过结合使用多重返回和匿名变量来避免这种丑陋的写法，让代码看起来更加优雅。

假设GetName()函数定义如下，它返回3个值，分别为firstName、lastName和nickName：

	func GetName() (firstName, lastName, nickName string) { 
		return "May", "Chan", "Chibi Maruko"
	}
若指向获得nickName，则函数调用语句可以用如下方式编写：
	_, _, nickName := GetName()

# 常量

在Go语言中，常量是指编译期间就已知且不可改变的值。常量可以是数值类型(包括整型、浮点型和复数类型)、布尔类型、字符串类型等。

## 字面常量

所谓字面常量(literal)，是指程序中硬编码的常量，如:

	-12
	3.14159265358979323846	// 浮点类型的常量
	3.2+12i					// 复数类型的常量
	true					// 布尔类型的常量
	"foo"					// 字符串常量
	
在其他语言中，常量通常有特定的类型，比如-12在C语言中会认为是一个int类型的常量。如果要指定一个值为-12的long类型常量，需要写成-12l，这有点违反人们的直观感觉。Go语言的字面常量更接近我们自然语言中的常量概念，它是无类型的。只要这个常量在相应类型的值域范围内，就可以作为该类型的常量，比如上面的常量-12，它可以赋值给int、uint、int32、 int64、float32、float64、complex64、complex128等类型的变量。

## 常量定义

通过**const**关键字可以给常量指定一个友好的名字：

	const Pi float64 = 3.14159265358979323846
	const zero = 0.0 				// 无类型浮点常量
	const (
		size int64 = 1024
		eof = -1					// 无类型整型常量
	)
	const u, v float32 = 0, 3		// u = 0.0, v = 3.0, 常量多重赋值
	const a, b, c = 3, 4, "foo"		//a=3,b=4,c="foo", 无类型整型和字符串常量
	
Go的常量定义可以限定常量类型，但不是必需的。如果定义常量时没有指定类型，那么它与字面常量一样，是无类型常量。
 
常量定义的右值也可以是一个在编译期运算的常量表达式，比如
	const mask = 1 << 3
由于常量的赋值是一个编译期行为，所以右值不能出现任何需要运行期才能得出结果的表达式，比如试图以如下方式定义常量就会导致编译错误：
	const Home = os.GetEnv("HOME")
原因很简单，os.GetEnv()只有在运行期才能知道返回结果，在编译期并不能确定，所以无法作为常量定义的右值。
## 预定义常量
Go语言预定义了这些常量：true、false和iota。
iota比较特殊，可以被认为是一个可被编译器修改的常量，在每一个const关键字出现时被重置为0，然后在下一个const出现之前，每出现一次iota，其所代表的数字会自动增1。
从以下的例子可以基本理解iota的用法：
	const (			// iota被重设为0
		c0 = iota	// c0 == 0
		c1 = iota	// c1 == 1
		c2 = iota	// c2 == 2
	)
	
	const (
		a = 1 << iota	// a == 1 (iota在每个const开头被重设为0)
		b = 1 << iota	// b == 2
		c = 1 << iota	// c == 4
	)
	
	const (
		u 		  = iota * 42	// u == 0
		v float64 = iota * 42	//v==42.0
		w 		  = iota * 42	//w==84
	)
	const x = iota
	const y = iota
	
如果两个const的赋值语句的表达式是一样的，那么可以省略后一个复制表达式。因此，上面的前两个const语句可简写为：

	const (			// iota被重设为0
		c0 = iota	// c0 == 0
		c1 			// c1 == 1
		c2			// c2 == 2
	)
	
	const (
		a = 1 << iota	// a == 1 (iota在每个const开头被重设为0)
		b				// b == 2
		c 				// c == 4
	)
	
## 枚举

枚举指一系列相关的常量，比如下面关于一个星期中每天的定义。通过上一节的例子，我们看到可以用在const后跟一对圆括号的方式定义一组常量，这种定义法在Go语言中通常用于定义枚举值。Go语言并不支持众多其他语言明确支持的enum关键字。

下面是一个常规的枚举表示法，其中定义了一系列整型常量：
	const (
		Sunday = iota
        Monday
        Tuesday
        Wednesday
        Thursday
        Friday
        Saturday
        numberOfDays	// 这个常量没有导出
	）
	
同Go语言的其他符号(symbol)一样，以大写字母开头的常量在包外可见。以上例子中numberOfDays为包内私有，其他符号则可被其他包访问。

# 类型

Go语言内置以下这些基础类型：

* 布尔类型：bool。
* 整型：int8、byte、int16、int、uint、uintptr等。 
* 浮点类型：float32、float64。
* 复数类型：complex64、complex128。 
* 字符串：string。
* 字符类型：rune。
* 错误类型：error。
此外，Go语言也支持以下这些复合类型：
* 指针(pointer)
* 数组(array)
* 切片(slice)
* 字典(map)
* 通道(chan)
* 结构体(struct) 
* 接口(interface)
## 布尔类型
Go语言中的布尔类型与其他语言基本一致，关键字也为bool，可赋值为预定义的true和false示例代码如下：
	var v1 bool
	v1 = true
	v2 := (1 == 2) // v2也会被推导为bool类型
布尔类型不能接受其他类型的赋值，不支持自动或强制的类型转换。以下的示例是一些错误的用法，会导致编译错误：
	var b bool
	b=1// 编译错误
	b = bool(1) // 编译错误
以下的用法才是正确的：
	var b bool
	b = (1!=0) // 编译正确
	fmt.Println("Result:", b) // 打印结果为Result: true

## 整型

整型是所有编程语言里最基础的数据类型。Go语言支持表2-1所示的这些整型类型。

![alt text](/img/GoBasicSyn/1.png)

### 类型表示

需要注意的是，int和int32在Go语言里被认为是两种不同的类型，编译器也不会帮你自动做类型转换，比如以下的例子会有编译错误：

	var value2 int32
	value1 := 64 // value1将会被自动推导为int类型 
	value2 = value1 // 编译错误
编译错误类似于：
	cannot use value1 (type int) as type int32 in assignment。
使用强制类型转换可以解决这个编译错误：
	value2 = int32(value1) // 编译通过
当然，开发者在做强制类型转换时，需要注意数据长度被截短而发生的数据精度损失(比如 将浮点数强制转为整数)和值溢出(值超过转换的目标类型的值范围时)问题。
### 数值运算
Go语言支持下面的常规整数运算:+、-、*、/和%。加减乘除就不详细解释了，需要说下的是，% 和在C语言中一样是求余运算，比如：
	5%3// 结果为:2
### 比较运算
Go语言支持以下的几种比较运算符：>、<、==、>=、<=和!=。这一点与大多数其他语言相同，与C语言完全一致。

下面为条件判断语句的例子：
	i, j := 1, 2 
	if i == j {
		fmt.Println("i and j are equal.")
	}

两个不同类型的整型数不能直接比较，比如int8类型的数和int类型的数不能直接比较，但各种类型的整型变量都可以直接与字面常量(literal)进行比较，比如：

	var i int32
	var j int64
	
	i, j = 1, 2
	
	if i == j { // 编译错误
		fmt.Println("i and j are equal.")
	}
	
	if i == 1 || j == 2 {// 编译通过
		fmt.Println("i and j are equal.")
	}
	
### 位运算

Go语言支持表2-2所示的位运算符。

![alt text](/img/GoBasicSyn/2.png)

Go语言的大多数位运算符与C语言都比较类似，除了取反在C语言中是~x，而在Go语言中是^x。

## 浮点型

浮点型用于表示包含小数点的数据，比如1.234就是一个浮点型数据。Go语言中的浮点类型采用IEEE-754标准的表达方式。

### 浮点数表示

Go语言定义了两个类型float32和float64，其中float32等价于C语言的float类型，float64等价于C语言的double类型。
￼在Go语言里，定义一个浮点数变量的代码如下：
	var fvalue1 float32
	
	fvalue1 = 12
	fvalue2 := 12.0 // 如果不加小数点,fvalue2会被推导为整型而不是浮点型
对于以上例子中类型被自动推导的fvalue2，需要注意的是其类型将被自动设为float64(Swift的类型推断也是推断精度更高的类型)，而不管赋给它的数字是否是用32位长度表示的。因此，对于以上的例子，下面的赋值将导致编译错误：
	fvalue1 = fvalue2
而必须使用这样的强制类型转换：
	fvalue1 = float32(fvalue2)
### 浮点数比较
因为浮点数不是一种精确的表达方式，所以像整型那样直接用==来判断两个浮点数是否相等是不可行的，这可能会导致不稳定的结果。
  
下面是一种推荐的替代方案：
	import "math"
	
	// p为用户自定义的比较精度,比如0.00001 
	func IsEqual(f1, f2, p float64) bool {
		return math.Fdim(f1, f2) < p 
	}
	
## 复数类型

复数实际上由两个实数(在计算机中用浮点数表示)构成，一个表示实部(real)，一个表示 虚部(imag)。如果了解了数学上的复数是怎么回事，那么Go语言的复数就非常容易理解了。

### 复数表示

复数表示的示例如下：

	var value1 complex64			// 由2个float32构成的复数类型
	
	value1 = 3.2 + 12i
	value2 := 3.2 + 12i				// value2是complex128类型
	value3 := complex(3.2, 12)		// value3结果同 value2

### 实部与虚部
对于一个复数z = complex(x, y)，就可以通过Go语言内置函数real(z)获得该复数的实部，也就是x，通过imag(z)获得该复数的虚部，也就是y。

更多关于复数的函数，请查阅math/cmplx标准库的文档。
## 字符串
在Go语言中，字符串也是一种基本类型。相比之下，C/C++语言中并不存在原生的字符串类型，通常使用字符数组来表示，并以字符指针来传递。
Go语言中字符串的声明和初始化非常简单，如下：
	var str string // 声明一个字符串变量
	str = "Hello world" // 字符串赋值
	ch := str[0] // 取字符串的第一个字符
	fmt.Printf("The length of \"%s\" is %d \n", str, len(str)) 
	fmt.Printf("The first character of \"%s\" is %c.\n", str, ch)
输出结果为：
	The length of "Hello world" is 11
    The first character of "Hello world" is H.

字符串的内容可以用类似于数组下标的方式获取，但与数组不同，字符串的内容不能在初始化后被修改，比如以下的例子：

	str := "Hello world" // 字符串也支持声明时进行初始化的做法 
	str[0] = 'X' // 编译错误
	
编译器会报类似如下的错误：

	cannot assign to str[0]
	
在这个例子中我们使用了一个Go语言内置的函数len()来取字符串的长度。这个函数非常有用，我们在实际开发过程中处理字符串、数组和切片时将会经常用到。

本节中我们还顺便示范了Printf()函数的用法。有C语言基础的读者会发现，Printf()函数的用法与C语言运行库中的printf()函数如出一辙。读者在以后学习更多的Go语言特性时，可以配合使用Println()和Printf()来打印各种自己感兴趣的信息，从而让学习过程更加直观、有趣。

Go编译器支持UTF-8的源代码文件格式。这意味着源代码中的字符串可以包含非ANSI的字符，比如“Hello world. 你好,世界!”可以出现在Go代码中。但需要注意的是，如果你的Go代码需要包含非ANSI字符，保存源文件时请注意编码格式必须选择UTF-8。特别是在Windows下一般编辑器都默认存为本地编码，比如中国地区可能是GBK编码而不是UTF-8，如果没注意这点在编译和运行时就会出现一些意料之外的情况。

字符串的编码转换是处理文本文档(比如TXT、XML、HTML等)非常常见的需求，不过可惜的是Go语言仅支持UTF-8和Unicode编码。对于其他编码，Go语言标准库并没有内置的编码转换支持。不过，所幸的是我们可以很容易基于iconv库用Cgo包装一个。这里有一个开源项目: https://github.com/xushiwei/go-iconv。

### 字符串操作

平时常用的字符串操作如表2-3所示。

![alt text](/img/GoBasicSyn/3.png)

更多的字符串操作，请参考标准库strings包。

### 字符串遍历

Go语言支持两种方式遍历字符串。一种是以字节数组的方式遍历：

	str := "Hello,世界"
	n := len(str)
	for i := 0; i < n; i++ {
		ch := str[i] // 依据下标取字符串中的字符,类型为byte
		fmt.Println(i, ch)
	}

这个例子的输出结果为：
	0 72
    1 101
    2 108
    3 108
    4 111
    5 44
    6 32
    7 228
    8 184
    9 150
    10 231
    11 149
    12 140
可以看出，这个字符串长度为13。尽管从直观上来说，这个字符串应该只有9个字符。这是因为每个中文字符在UTF-8中占3个字节，而不是1个字节。

另一种是以Unicode字符遍历：
	str := "Hello,世界"
	for i, ch := range str {
		fmt.Println(i, ch)//ch的类型为rune 
	}

输出结果为：
	0 72
    1 101
    2 108
    3 108
    4 111
    5 44
    6 32
    7 19990
    10 30028

以Unicode字符方式遍历时，每个字符的类型是rune(早期的Go语言用int类型表示Unicode 字符)，而不是byte。

## 字符类型
在Go语言中支持两个字符类型，一个是byte(实际上是uint8的别名)，代表UTF-8字符串的单个字节的值；另一个是rune，代表单个Unicode字符。
关于rune相关的操作，可查阅Go标准库的unicode包。另外unicode/utf8包也提供了UTF8和Unicode之间的转换。
出于简化语言的考虑，Go语言的多数API都假设字符串为UTF-8编码。尽管Unicode字符在标准库中有支持，但实际上较少使用。

## 数组
数组是Go语言编程中最常用的数据结构之一。顾名思义，数组就是指一系列同一类型数据的集合。数组中包含的每个数据被称为数组元素(element)，一个数组包含的元素个数被称为数组的长度。
以下为一些常规的数组声明方法：
	[32]byte 						// 长度为32的数组,每个元素为一个字节 
	[2*N] struct { x, y int32 } 	// 复杂类型数组
	[1000]*float64 					// 指针数组
	[3][5]int 						// 二维数组
	[2][2][2]float64				// 等同于[2]([2]([2]float64))

从以上类型也可以看出，数组可以是多维的，比如[3][5]int就表达了一个3行5列的二维整 型数组，总共可以存放15个整型元素。
在Go语言中,数组长度在定义后就不可更改，在声明时长度可以为一个常量或者一个常量表达式(常量表达式是指在编译期即可计算结果的表达式)。数组的长度是该数组类型的一个内置常量，可以用Go语言的内置函数len()来获取。下面是一个获取数组arr元素个数的写法：
	arrLength := len(arr)
### 元素访问
可以使用数组下标来访问数组中的元素。与C语言相同，数组下标从0开始，len(array)-1则表示最后一个元素的下标。下面的示例遍历整型数组并逐个打印元素内容：
	for i := 0; i < len(array); i++ {
		fmt.Println("Element", i, "of array is", array[i])
	}
	
Go语言还提供了一个关键字range，用于便捷地遍历容器中的元素。当然，数组也是range的支持范围。上面的遍历过程可以简化为如下的写法:

	for i, v := range array {
		fmt.Println("Array element[", i, "]=", v)
	￼}
在上面的例子里可以看到，range具有两个返回值，第一个返回值是元素的数组下标，第二个返回值是元素的值。
### 值类型
需要特别注意的是，在Go语言中数组是一个值类型(value type)。所有的值类型变量在赋值和作为参数传递时都将产生一次复制动作。如果将数组作为函数的参数类型，则在函数调用时该参数将发生数据复制。因此，在函数体中无法修改传入的数组的内容，因为函数内操作的只是所传入数组的一个副本。
下面用例子来说明这一特点:
	package main
	
	import (
		"fmt"
	)
	
	func modify(array [5]int) {
		array[0] = 10	// 试图修改数组的第一个元素
		fmt.Println("In modify(), array values:", array)
	}
	
	func main() {
		array := [5]int{1,2,3,4,5}	// 定义并初始化一个数组
		modify(array)	// 传递给一个函数,并试图在函数体内修改这个数组内容
		fmt.Println("In main(), array values:", array)
	}
	
该程序的执行结果为：

	In modify(), array values: [10 2 3 4 5]
	In main(), array values: [1 2 3 4 5]
	
从执行结果可以看出，函数modify()内操作的那个数组跟main()中传入的数组是两个不同的实例。那么，如何才能在函数内操作外部的数据结构呢?用数组切 片功能来达成这个目标。

## 数组切片

在前一节里我们已经提过数组的特点：数组的长度在定义之后无法再次修改；数组是值类型，每次传递都将产生一份副本。显然这种数据结构无法完全满足开发者的真实需求。

不用失望，Go语言提供了数组切片(slice)这个非常酷的功能来弥补数组的不足。

初看起来，数组切片就像一个指向数组的指针，实际上它拥有自己的数据结构，而不仅仅是个指针。数组切片的数据结构可以抽象为以下3个变量：

* 一个指向原生数组的指针；
* 数组切片中的元素个数；
* 数组切片已分配的存储空间。
从底层实现的角度来看，数组切片实际上仍然使用数组来管理元素，因此它们之间的关系让 C++程序员们很容易联想起STL中std::vector和数组的关系。基于数组，数组切片加了一系列管理功能，可以随时动态扩充存放空间，并且可以被随意传递而不会导致所管理的元素被重复复制。
### 创建数组切片
创建数组切片的方法主要有两种--基于数组和直接创建,下面我们来简要介绍一下这两种方法。
#### 基于数组

数组切片可以基于一个已存在的数组创建。数组切片可以只使用数组的一部分元素或者整个数组来创建，甚至可以创建一个比所基于的数组还要大的数组切片。如下代码演示了如何基于一个数组的前5个元素创建一个数组切片：

	package main
	
	import "fmt"
	
	func main() {
		// 先定义一个数组
		var myArray [10]int = [10]int{1, 2, 3, 4, 5, 6, 7, 8, 9, 10} 
		
		// 基于数组创建一个数组切片
		var mySlice []int = myArray[:5]
		fmt.Println("Elements of myArray: ") 
		
		for _, v := range myArray {
			fmt.Print(v, " ")
		}
		
		fmt.Println("\nElements of mySlice: ")
		
		for _, v := range mySlice { 
			fmt.Print(v, " ")
		}
		fmt.Println()
	}
运行结果为：
	Elements of myArray: 
	1 2 3 4 5 6 7 8 9 10 
	Elements of mySlice: 
	1 2 3 4 5

读者应该已经注意到，Go语言支持用myArray[first:last]这样的方式来基于数组生成一个数组切片，而且这个用法还很灵活，比如下面几种都是合法的。

基于myArray的所有元素创建数组切片：
	mySlice = myArray[:]
￼基于myArray的前5个元素创建数组切片：
	mySlice = myArray[:5]
基于从第5个元素开始的所有元素创建数组切片：
    mySlice = myArray[5:]
#### 直接创建
并非一定要事先准备一个数组才能创建数组切片。Go语言提供的内置函数make()可以用于灵活地创建数组切片。下面的例子示范了直接创建数组切片的各种方法。
创建一个初始元素个数为5的数组切片，元素初始值为0：
	mySlice1 := make([]int, 5)
创建一个初始元素个数为5的数组切片，元素初始值为0，并预留10个元素的存储空间：
	mySlice2 := make([]int, 5, 10)
直接创建并初始化包含5个元素的数组切片：
	mySlice3 := []int{1, 2, 3, 4, 5}
当然，事实上还会有一个匿名数组被创建出来，只是不需要我们来操心而已。
### 元素遍历
操作数组元素的所有方法都适用于数组切片，比如数组切片也可以按下标读写元素，用len()函数获取元素个数，并支持使用range关键字来快速遍历所有元素。
传统的元素遍历方法如下：
	for i := 0; i <len(mySlice); i++ { 
		fmt.Println("mySlice[", i, "] =", mySlice[i])
	}
使用range关键字可以让遍历代码显得更整洁。range表达式有两个返回值，第一个是索引，第二个是元素的值：
	for i, v := range mySlice { 
		fmt.Println("mySlice[", i, "] =", v)
	}
对比上面的两个方法，我们可以很容易地看出使用range的代码更简单易懂。
### 动态增减元素
可动态增减元素是数组切片比数组更为强大的功能。与数组相比，数组切片多了一个存储能力(capacity)的概念，即元素个数和分配的空间可以是两个不同的值。合理地设置存储能力的值，可以大幅降低数组切片内部重新分配内存和搬送内存块的频率，从而大大提高程序性能。
假如你明确知道当前创建的数组切片最多可能需要存储的元素个数为50，那么如果你设置的存储能力小于50，比如20，那么在元素超过20时，底层将会发生至少一次这样的动作--重新分配一块“够大”的内存，并且需要把内容从原来的内存块复制到新分配的内存块，这会产生比较明显的开销。给“够大”这两个字加上引号的原因是系统并不知道多大才是够大，所以只是一个简单的猜测。比如，将原有的内存空间扩大两倍，但两倍并不一定够，所以之前提到的内存重新分配和内容复制的过程很有可能发生多次，从而明显降低系统的整体性能。但如果你知道最大是50并且一开始就设置存储能力为50,那么之后就不会发生这样非常耗费CPU的动作,从而达到空间换时间的效果。
数组切片支持Go语言内置的cap()函数和len()函数，如下简单示范了这两个内置函数的用法。可以看出，cap()函数返回的是数组切片分配的空间大小，而len()函数返回的是数组切片中当前所存储的元素个数。
	package main
	
	import "fmt" 
	
	func main() {
		mySlice := make([]int, 5, 10)
		fmt.Println("len(mySlice):", len(mySlice))
		fmt.Println("cap(mySlice):", cap(mySlice))
	}
	
该程序的输出结果为：

	len(mySlice): 5
    cap(mySlice): 10
如果需要往上例中mySlice已包含的5个元素后面继续新增元素，可以使用append()函数。下面的代码可以从尾端给mySlice加上3个元素，从而生成一个新的数组切片：
	mySlice = append(mySlice, 1, 2, 3)
函数append()的第二个参数其实是一个不定参数，我们可以按自己需求添加若干个元素，甚至直接将一个数组切片追加到另一个数组切片的末尾：
	mySlice2 := []int{8, 9, 10}
	// 给mySlice后面添加另一个数组切片
	mySlice = append(mySlice, mySlice2...)
需要注意的是,我们在第二个参数mySlice2后面加了三个点，即一个省略号，如果没有这个省略号的话，会有编译错误，因为按append()的语义，从第二个参数起的所有参数都是待附加的元素。因为mySlice中的元素类型为int，所以直接传递mySlice2是行不通的。加上省略号相当于把mySlice2包含的所有元素打散后传入。
上述调用等同于:
	mySlice = append(mySlice, 8, 9, 10)
数组切片会自动处理存储空间不足的问题。如果追加的内容长度超过当前已分配的存储空间(即cap()调用返回的信息)，数组切片会自动分配一块足够大的内存。
### 基于数组切片创建数组切片
类似于数组切片可以基于一个数组创建，数组切片也可以基于另一个数组切片创建。下面的例子基于一个已有数组切片创建新数组切片：
	oldSlice := []int{1, 2, 3, 4, 5}
	newSlice := oldSlice[:3] // 基于oldSlice的前3个元素构建新数组切片
有意思的是，选择的oldSlicef元素范围甚至可以超过所包含的元素个数，比如newSlice可以基于oldSlice的前6个元素创建，虽然oldSlice只包含5个元素。只要这个选择的范围不超过oldSlice存储能力(即cap()返回的值)，那么这个创建程序就是合法的。newSlice中超出oldSlice元素的部分都会填上0。
### 内容赋值
数组切片支持Go语言的另一个内置函数copy()，用于将内容从一个数组切片复制到另一个数组切片。如果加入的两个数组切片不一样大，就会按其中较小的那个数组切片的元素个数进行复制。下面的示例展示了copy()函数的行为：
slice1 := []int{1, 2, 3, 4, 5} 
slice2 := []int{5, 4, 3}
copy(slice2, slice1) // 只会复制slice1的前3个元素到slice2中
copy(slice1, slice2) // 只会复制slice2的3个元素到slice1的前3个位置

## map

在C++/Java中，map一般都以库的方式提供，比如在C++中是STL的std::map<>，在C#中是Dictionary<>，在Java中是Hashmap<>，在这些语言中，如果要使用map，事先要引用相应的库。而在Go中，使用map不需要引入任何库，并且用起来也更加方便。
map是一堆键值对的未排序集合。比如以身份证号作为唯一键来标识一个人的信息，如下所示：
	package main
	import "fmt"
	
	//PersonInfo是一个包含个人详细信息的类型
	type PersonInfo struct {
		ID string
		Name string
		Address string
	}
	
	func main() {
		var personDB map[string] PersonInfo
		personDB = make(map[string] PersonInfo)
		
		//往这个map里插入几条数据
		personDB["12345"] = PersonInfo{"12345", "Tom", "Room 203,..."}
		personDB["1"] = PersonInfo{"1", "Jack", "Room 101,..."}
		
		//从这个map查找键为"1234"的信息
		person, ok := personDB["1234"]
		//ok是一个返回的bool类型，返回true表示查找到了对应的数据
		if ok {
			fmt.Println("Found person", person.Name, "with ID 1234.")
		} else {
			fmt.Println("Did not find person with ID 1234.")
		}
	}

上面这个简单的例子基本上已经覆盖了map的主要用法，下面对其中的关键点进行细述。

### 变量声明

map的声明基本上没有多余的元素，比如：

	var myMap map[string] PersonInfo
	
其中，myMap是声明的map变量名，string是键的类型，PersonInfo则是其中所存放的值类型。

### 创建

我们可以使用Go语言内置的函数make()来创建一个新map。下面的这个例子创建了一个键类型为string、值类型为PersonInfo的map：

	myMap = make(map[string] PersonInfo)
	
也可以选择是否在创建时指定该map的初始存储能力，下面的例子创建了一个初始存储能力为100的map：

	myMap = make(map[string] PersonInfo, 100)
	
创建并初始化map的代码如下：

	myMap = map[string] PersonInfo{
		"1234": PersonInfo{"1", "Jack", "Room 101,..."},
	}
	
### 元素赋值

赋值过程非常简单明了，就是将键和值用下面的方式对应起来即可：

	myMap["1234"] = PersonInfo{"1", "Jack", "Room 101,..."}
	
### 元素删除

Go语言提供了一个内置函数delete()，用于删除容器内的元素。下面我们简单介绍一下如何用delete()函数删除map内的元素：

	delete(myMap, "1234")
	
上面的代码将从myMap中删除键为“1234”的键值对。如果“1234”这个键不存在，那么这个调用将什么都不发生，也不会有什么副作用。但是如果传入的map变量的值是nil，该调用将导致程序抛出异常(panic)。

### 元素查找

在Go语言中，map的查找功能设计得比较精巧。而在其他语言中，我们要判断能否获取到一个值不是件容易的事情。判断能否从map中获取一个值的常规做法是：

1. 声明并初始化一个变量为空；
2. 试图从map中获取相应键的值到该变量中；
3. 判断该变量是否依旧为空，如果为空则表示map中没有包含该变量。

这种用法比较啰嗦，而且判断变量是否为空这条语句并不能真正表意(是否成功取到对应的值)，从而影响代码的可读性和可维护性。有些库甚至会设计为因为一个键不存在而抛出异常，让开发者用起来胆战心惊，不得不一层层嵌套try-catch语句，这更是不人性化的设计。在Go语言中，要从map中查找一个特定的键，可以通过下面的代码来实现：

	value, ok := myMap["1234"]
	if ok { // 找到了
		// 处理找到的value
	}
	
判断是否成功找到特定的键，不需要检查取到的值是否为nil，只需查看第二个返回值ok，这让表意清晰很多。配合:=操作符，让你的代码没有多余成分，看起来非常清晰易懂。
