---
title: "Programming with Objective-C"
date: 2015-04-27 14:41:32
categories: 
- 编程指南
tags: 
- Objective-C
metaAlignment: center
autoThumbnailImage: no
coverImage: //d1u9biwaxjngwg.cloudfront.net/welcome-to-tranquilpeak/city.jpg

---

[官方文档](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/Introduction/Introduction.html#//apple_ref/doc/uid/TP40011210-CH1-SW1)
<!--more-->

# 简介

## 关于Objective-C

Objective-C是你在编写OS X 和 iOS应用程序时使用的主要编程语言。它是C语言的一个超集，同时提供了面向对象的能力和一个动态的运行时。Objective-C继承了C语言的语法，主要的类型和控制流语句，增加了定义类和方法的语法。也在语言层面支持了对象图的管理和对象字面量，同时提供了动态类型和动态绑定，将编译时的许多责任推迟到了运行时。

### 概览

本文档介绍Objective-C语言，提供了大量如何使用该语言的列子。你将学会怎样创建自己的类来描述自定义的对象，以及如何与Cocoa 和 Cocoa Touch框架提供的类协作。尽管框架的类和语言是分开的，这些类的使用与Objective-C编写的这些框架代码息息相关，同时很多语言层面的功能都依赖于这些框架类的行为。

#### 应用程序构建自对象网

当我们为OS X 和 iOS构建应用程序时，大部分的时间我们都在处理对象。这些对象是Objective-C类的实例，一部分实例由Cocoa 和 Cocoa Touch类提供给你，一部分由你自己创建的类提供。

如果你创建你自己的类，那么就从描述类的公共接口的细节开始。这个接口包括封装数据的公共属性，以及方法列表。方法声明了这个对象能够接收的消息，包括方法调用时需要的参数。定义完接口之后，你需要提供类的实现，包括为接口中定义的方法提供可执行的代码。

#### 类别(Category)拓展已经存在的类

通过定义类别来为已经存在的类添加自定义的行为完全是可以的，而不是在已有类的基础上创建一个全新的类来提供微小的额外功能。你可以用类别为任何类添加方法，包括那些你没有源码的类，比如框架类`NSString`。

如果你有类的原始源代码，你可以使用拓展(class extension)来添加新的属性，或者调整已经存在属性的描述属性。拓展通常用来在单个源文件或者自定义框架私有实现的内部来隐藏私有行为。

#### 协议(Protocol)定义了消息契约

在Objective-C程序内，大部分的工作都是对象之间的消息传递。通常，这些消息由类的接口中的方法显式的定义。然而，有时定义一个没有与指定的类相关联的方法集也是很有用的。

Objective-C用协议来定义一个相关方法的合集，比如一个对象的方向可以通过`delegate`来调用，这个合集的方法可以是可选的或者是必须的。任何类都可以直接适配一个协议，意味着该类必须实现协议里面必须的方法。

#### 通常使用Objective-C对象来表示值和容器类型

在Objective-C里面使用Cocoa 或者 Cocoa Touch的类来表示值是很常见的。`NSString`类用来表示字符串和字符，`NSNumber`类用来表示不同类型的数字，比如整型或者浮点型，`NSValue`类用来表示其他的值，比如C结构体。同时你也可以使用C语言中定义的任意主要类型，比如`int`，`float`，或者`char`。

容器通常用`NSArray`, `NSSet`, 或者 `NSDictionary`的实例来表示，通常用来作为其他Objective-C对象的容器。

#### Block简化了常见的任务

Block是C，Objective-C，C++引进的语言层面的功能，用来表示一个简单单元的任务；他们封装一个代码块来捕获状态，跟其他语言里面的闭包是类似的。Block通常用来简化常见的一些任务，比如容器的枚举，排序，测试等等。同时在Grand Central Dispatch (GCD)等技术中用来简化并发或者异步执行的任务调度。

#### Error对象用来处理运行时问题

尽管Objective-C包含异常捕获的语法，Cocoa 和 Cocoa Touch只在编程错误的时候使用异常(比如数组越界)，必须在应用发布前修复.

所有其他的错误，包括运行时的问题，比如硬盘空间不足或者无法访问Web服务，都使用`NSError`类的实例来表示。你的应用程序需要有处理这些错误的方案，当出现错误时你需要决定如何最优的处理这些错误从而达到最好得用户体验。

#### Objective-C代码遵循确定的约定(惯例)

在编写Objective-C代码的时候，你需要在脑海中谨记一些已经确定的编程约定(惯例)。方法名，小写字母开始，多个单词时使用驼峰的形式；比如`doSomething`或者`doSomethingElse`。你应当让你的代码有尽可能高的可读性，意味着方法名需要极具表现力，但是不要太冗长。

另外，如果你希望使用语言和框架的一些高级功能，有些约定是必须的。属性的访问方法，比如，必须遵循严格的命名约定从而使用KVC或者KVO这些技术。

### 预备知识

如果你是OS X 或者 iOS的开发新手，你需要在阅读本文档之前先了解Start Developing iOS Apps Today 或者[Start Developing Mac Apps Today](https://developer.apple.com/library/mac/referencelibrary/GettingStarted/RoadMapOSX/index.html#//apple_ref/doc/uid/TP40012262)，对iOS和OS X的应用开发过程有个大致的了解。另外，你需要对Xcode很熟悉，在文档的大部分章节的最后都会有联系。Xcode是开发iOS和OS X的应用程序的IDE；你将使用它来编写代码，设计你得应用程序的用户界面，测试你得应用程序，调试问题。

尽管Objective-C与C语言或者其他以C为基础的语言，比如Java或者C#，有很多类似的地方，本文档包含了大量的以C语言功能为基础的列子，比如控制流语句。如果你有其他高级编程语言的知识，比如Ruby或者Python，你应该能跟上这些内容。

合理的覆盖被当做是通用的面向对象的准则，特别的是当他们应用在Objective-C中时，但是，那是假设你熟悉一些基本的面向对象的概念。如果你不熟悉这些概念，你应该先阅读这些相关章节[Concepts in Objective-C Programming](https://developer.apple.com/library/mac/documentation/General/Conceptual/CocoaEncyclopedia/Introduction/Introduction.html#//apple_ref/doc/uid/TP40010810)。
     
### 另请参阅

该文档中的内容适用于Xcode 4.4及以后，假定你开发的应用的目标是OS X v10.7及以后或者iOS5及以后。关于Xcode更多信息，参见[Xcode Overview](https://developer.apple.com/library/mac/documentation/ToolsLanguages/Conceptual/Xcode_Overview/index.html#//apple_ref/doc/uid/TP40010215)。关于语言可用功能的信息，参见[Objective-C Feature Availability Index](https://developer.apple.com/library/mac/releasenotes/ObjectiveC/ObjCAvailabilityIndex/index.html#//apple_ref/doc/uid/TP40012243)。

Objective-C的应用使用引用的计数来决定对象的生命周期。在极大程度上，由编译器的Automatic Reference Counting (ARC)功能帮你处理。如果你无法使用ARC，或者需要维护之前手动管理内存的代码，你需要阅读[Advanced Memory Management Programming Guide](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/MemoryMgmt/Articles/MemoryMgmt.html#//apple_ref/doc/uid/10000011i)。

除了编译器之外，Objective-C语言使用了一个运行时系统来实现它的动态特性和面向对象的功能。尽管大多时候你不需要担心Objective-C是怎么样工作的，但是和运行时系统直接交互也是可能的，如[Objective-C Runtime Programming Guide](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Introduction/Introduction.html#//apple_ref/doc/uid/TP40008048) 和 [Objective-C Runtime Reference](https://developer.apple.com/library/mac/documentation/Cocoa/Reference/ObjCRuntimeRef/index.html#//apple_ref/doc/uid/TP40001418)中描述的一样。

# 定义类

当你在编写iOS或者OS X的软件时，大部分时间都花在了处理对象上。Objective-C里面的对象和其他的面向对象语言的对象是类似的：封装数据及与数据相关的行为。

一个应用程序作为一个对象的大型生态系统构建，对象之间相互通讯解决特定的问题，比如显示可视界面，响应用户的输入，或者存储信息。对于OS X或者iOS开发，你不需要从头创建对象来解决每一个你想到的问题；相反的，Cocoa (for OS X) 和 Cocoa Touch (for iOS)提供了大量的库，库中有很多已经存在的对象可供你使用。

这些对象中的一部分是可以直接使用的，比如strings 和 numbers这些基础的数据类型，或者像buttons(按钮) 和 table(表)视图这些用户界面元素。一部分对象设计的目的是让你用自己代码来自定义，让对象表现出你需要的行为。应用程序的开发过程牵涉到决定怎样最优的自定义对象，以及把底层框架提供的对象和你自己的对象相结合使你的应用具有唯一的功能和特征。

在面向对象编程方面，一个对象是一个类的实例。本章将演示如何在Objective-C里面来定义类，通过声明接口来表明你希望怎样使用类和类的实例。这个接口包括了该类能够接收的一个消息列表，所以你也需要提供类的实现，包括响应每条消息的可执行代码。

PS：其实很多初学者肯定都很困惑Objective-C里面的消息，所以这里说明一下。首先我们先理清几个专有名词的区别，比如函数和方法，我们可以简单的理解为与类相关联的函数我们就叫做方法。在Objective-C里面调用一个对象的方法，即调用类的实例方法，我们叫做给对象发送消息。同样的，在Objective-C里面调用一个类的方法，即调用类的方法，那就是给这个类发送消息。后面会讲到实例方法和类方法的区别，现在可简单理解为实例方法“-”开头，类方法“+”开头。所以你可以简单的理解为发送消息就是调用相应的方法；如果你要深入的理解、研究，这里就涉及到Objective-C的运行时系统，研究运行时系统是怎样实现面向对象的，怎样处理消息的等等。

## 类是对象的蓝图

一个类描述了特定类型对象的常见行为和属性。对于一个string对象来说(在Objective-C中是`NSString`类的实例)，这个类提供了不同的方法去检查、转换它所表示字符串的内部字符。相似的，用来描述number对象(`NSNumber`)的类围绕内部数值提供了不同的功能，比如转换一个值到不同的数值类型。

由类的不同构造器构造的实例具有相同的结构，每一个类的实例都具有相同的属性和行为。每一个`NSString`的实例具有相同的行为，不管它的实例内部拥有的字符串是怎样的。

任何对象都被设计成有特定的用途。你也许知道一个string对象用来表示字符串，但是你不需要知道内部存储字符的机制。你不清楚对象内部的任何行为，你通过对象来直接处理字符，但是你需要知道怎样和对象交互，比如要求对象返回指定的字符或者要求返回一个将所有原来字符都转换成大写的新的对象。

在Objective-C里，类的接口指定了该类期望其他的对象如何使用该类。换言之就是类在实例和外部之间定义了公共接口。

### 可变性决定了值能否被改变

一些类规定对象是不可变的。这就意味着在对象创建的时候内部的内容必须被设置，而且之后不能被其他的对象改变。在Objective-C里，所有`NSString` 和 `NSNumber`的对象都是不可变的。如果你要表示其他的不同的数值，你需要使用一个新的`NSNumber`实例。

一些不可变的类同样提供了可变的版本。如果你明确的需要在运行时改变一个字符串的内容，比如附加通过网络连接接收的字符，你可以使用`NSMutableString`类的实例。这个类的实例和`NSString`对象具有相同的行为，除了提供了可以改变对象持有的字符的功能外。

尽管`NSString` 和 `NSMutableString`是不同的类，但它们有很多相似之处。比起从头创建两个具有相似行为的类来说，使用继承更合理。

### 类的继承

在自然界，分类法将动物分为团体，诸如种、属、族。这些团体是有层级结构的，比如多个种属于一个属，多个属属于一个族。

例如,大猩猩,人类和猩猩有一些明显的相似之处。虽然他们属于不同的种类，甚至是不同的属，部落，和亚科,但他们是分类学相关的因为他们都属于同一个族(称为人科),如图1 - 1所示。

Figure 1-1  Taxonomic relationships between species
![alt text](/img/ProgrammingwithObjectiveC/humansgorillas.png)

在面向对象编程的世界里，对象也被分类到不同层级的团体中。对象简单的组织成类，而不是使用不同的层级，比如种和属。人类作为人科的一员继承了一些确定的特征，一个类也可以继承父类的功能。

当一个类继承自另一个类时，子类继承了父类定义的所有的行为和属性。子类也可以定义自己的行为和属性，或者从写父类的行为。

比如在Objective-C中的字符串这个类，`NSMutableString`继承自`NSString`，如图1 - 2所示。所有`NSString`类提供的功能，`NSMutableString`都可使用，比如查询指定的字符或者要求新的小写得字符串，但是`NSMutableString`类增加了附加、插入、替换或者删除子字符串和独立字符的方法。

Figure 1-2  NSMutableString class inheritance
![alt text](/img/ProgrammingwithObjectiveC/nsstringmutablestring.png)

### 基类提供了基本的功能

与许多有生命的生物体有部分相同的基本生命特征一样，Objective-C里所有的对象都有一些相同的、常见的功能。

当一个Objective-C对象需要和其他类的实例协作的时候，我们希望其他的类提供了一些确定的、基本的特征和行为。基于这个原因，Objective-C定义了一个基类，绝大多数的其他类都继承了该类，叫做`NSObject`。当一个对象和另外一个对象交互的时候，我们期望至少可以使用基类描述定义的一些基本行为来交互。

当你定义自己的类时，你应该继承`NSObject`。通常而言，你需要在Cocoa 和 Cocoa Touch对象中找到那些提供了与你需要的功能最靠近的类，然后继承它。

如果你想自定义一个按钮，比如，`UIButton`类没有提供足够的自定义属性来满足你得需求，那么创建一个继承自`UIButton`的类比继承`NSObject`更合理。如果你简单的继承自`NSObject`，你需要做复制大量`UIButton`已经定义实现的很多功能，比如复杂的可视化交互操作和通讯以确保你的按钮表现的和用户期望的一样。通过继承`UIButton`，你的子类自动获得了`UIButton`所有的行为和功能，这些应用于`UIButton`行为的功能在未来会被增强、以及修复的Bug。

`UIButton`类本身定义继承`UIControl`，`UIControl`描述了iOS中所有用户界面控件的基本行为。`UIControl`继承自`UIView`，`UIView`让对象显示在屏幕上。`UIView`继承自`UIResponder`，`UIResponder`能响应用户的点击，手势或者晃动。最终到达继承树的顶端，`UIResponder`继承自`NSObject`，如图1 - 3所示。

Figure 1-3  UIButton class inheritance
![alt text](/img/ProgrammingwithObjectiveC/buttoninheritance.png)

## 类的接口定义了期望的交互操作

面向对象编程的好处之一就是前面所提到的—-所有你需要知道的就是如何与类的实例交互。更具体的说，应该设计一个隐藏了内部实现细节的对象。

如果你在使用一个标准的`UIButton`类，比如，你不需要担心怎样操作像素将按钮显示在屏幕上。所有你需要知道的是你可以改变确定的属性，比如按钮的标题、颜色，当你把按钮添加到你的可视界面时，相信这个按钮如你期望的显示在正确位置、具有正确的行为。

当你定义你自己的类时，一开始你需要弄清楚哪些是公开的属性和行为。哪些属性是可以公开访问的？是否允许这些属性被改变？其他的对象与你的类的实例怎么通讯？

这些信息都体现在了类的接口里面--定义了你类的实例与其他对象交互的方式。这个公开的接口描述的类的行为将内部的行为分开，在Objective-C里面，接口和实现通常放在分开的文件里面，所以接口是公开的。

### 基本语法

Objective-C用来声明类的接口的语法如下所示：

	@interface SimpleClass : NSObject
 
	@end
	
这个例子声明了一个叫作`SimpleClass`的类，继承自`NSObject`。

公开的属性和行为定义在`@interface`声明中。在这里例子中，没有其他的内容，所以`SimpleClass`类的实例只有继承自`NSObject`类的功能。

### 属性控制对象值的访问

对象通常有可以公开访问的属性。如果你在一个记录保持的应用里面定义一个类来代表人类，比如，你可能决定你需要strings属性来表示一个人的first和last names。

声明这些属性应该添加到接口里面，如下：

	@interface Person : NSObject
 
	@property NSString *firstName;
	@property NSString *lastName;
 
	@end
	
在这个例子中，`Person`类声明了两个公开属性，两个都是`NSString`类的实例。

这两个属性都是Objective-C对象，所以使用星号(*)来指示他们是C指针。就像在C语言中声明其他变量的语句一样，语句最后需要分号。

你可能决定增加一个属性来表示出生的年份，来进行排序，而不仅仅是根据名称来排序。你可以使用一个number对象属性。

	@property NSNumber *yearOfBirth;
	
但是，用来存储简单数字值有点过了。一种替代的方法就是使用C语言的类型，比如整数类型：

	@property int yearOfBirth;
	
#### 属性的描述属性指示了数据的可访问性和存储注意事项

实例到现在为止声明的属性都有完整的公开的访问权限。这意味着其他对象能够读取和修改属性的值。

在有些情况下，你可能决定声明不需要改变值得属性。在现实世界中，一个人必须填写大量的文书工作来修改记录的first 和 last name。如果你在开发一个官方的纪录保持应用，你可能决定人名的这个公开属性是只读的。

Objective-C的属性声明都可以包含描述属性，用来指示，一个属性是否是只读等。官方的纪录保持应用中，`Person`类的接口如下所示：

	@interface Person : NSObject
	@property (readonly) NSString *firstName;
	@property (readonly) NSString *lastName;
	@end
	
属性的描述属性在`@property`关键字之后指定，包含在括号中，在[ Declare Public Properties for Exposed Data ](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/EncapsulatingData/EncapsulatingData.html#//apple_ref/doc/uid/TP40011210-CH5-SW4)中有完整的描述。

### 方法声明了对象可以直接接收的消息

例子到现在涉及到了典型的模型类，主要被设计来封装数据。在例子中，`Person`类可能除了访问声明的属性之外不需要其他的功能。大部分的类，除了声明属性之外还包括一些其他的行为。

鉴于Objective-C软件构建自一张大型的对象网，这些对象通过发送消息来进行交互。在Objective-C里，通过调用这个对象的方法来给这个对象发送消息。

Objective-C方法在概念上类似于C和其他编程语言的标准函数,虽然语法有很大的不同。一个C函数声明如下：

	void SomeFunction();
	
等价的Objective-C方法声明如下：

	- (void)someMethod;
	
这个方法没有参数。这个C语言的关键字`void`放在括号里在方法声明的开头表示这个方法在结束后不会返回任何值。减号表示该方法是一个实例方法，只能由类的实例的来调用。这与类方法不同，类方法由类调用，详见[ Objective-C Classes Are also Objects ](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/DefiningClasses/DefiningClasses.html#//apple_ref/doc/uid/TP40011210-CH3-SW18)

与C函数原型一样，在Objective-C类的接口里面声明一个方法需要以分号结束。

#### 方法可以带参数

如果你需要定义带有一个或多个参数的方法，语法和典型的C函数完全不同。

对于一个典型的C函数来说，参数在括号里面指定，如：

	void SomeFunction(SomeType value);
	
一个Objective-C方法的声明将参数作为了方法名的一部分，用分号来分割，如下：

	- (void)someMethodWithValue:(SomeType)value;
	
和返回值类型一样，参数类型也在括号里面指定。

如果你需要支持多个参数，那么语法与C语言还是有很大得区别。C函数还是在括号里面指定多个参数，用逗号分开；在Objective-C里，声明一个带有两个参数的方法如下：

	- (void)someMethodWithFirstValue:(SomeType)value1 secondValue:(AnotherType)value2;
	
在这里例子中，value1和value2是访问方法调用时提供的值的变量名，类似于变量。

有些编程语言允许函数定义所谓的命名参数(命名参数可按照不同顺序传递实参)；在Objective-C中是不允许的。方法调用时的参数顺序必须与定义时相匹配，事实上`secondValue:`是方法名的一部分：

	someMethodWithFirstValue:secondValue:
	
这些特征帮助Objective-C是一门可读性很高的语言，因为方法调用时传递的值是内联指定，接下来就是方法名相关的部分，详见[ You Can Pass Objects for Method Parameters ](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/WorkingwithObjects/WorkingwithObjects.html#//apple_ref/doc/uid/TP40011210-CH4-SW13)

*注意：*上面使用的value1和value2严格的来说不是方法声明的一部分，这意味着在接口声明与在实现时不需要完全一样。唯一的要求就是签名匹配，即方法名以及参数和返回值类型与声明时一致。

如下方法与上面所示的方法有相同的签名：

	- (void)someMethodWithFirstValue:(SomeType)info1 secondValue:(AnotherType)info2;

这些方法就与上面所示的方法签名不同：

	- (void)someMethodWithFirstValue:(SomeType)info1 anotherValue:(AnotherType)info2;
	- (void)someMethodWithFirstValue:(SomeType)info1 secondValue:(YetAnotherType)info2;

### 类名必须是唯一

需要注意每个类的名字必须是唯一的在一个应用程序里面，甚至包括库或框架。如果你试图在项目里创建一个与已经存在的类的相同名字的类，你将会收到编译器的警告。

为此，在定义你的类的名称的时候使用前缀是明智的，使用3个或者跟多的字母。这些字母与你当前开发的应用相关，或可重用的代码框架的名称，或者只是你的名字的首字母。

文档中剩下的例子都会使用类名的前缀，如下：

	@interface XYZPerson : NSObject
	@property (readonly) NSString *firstName;
	@property (readonly) NSString *lastName;
	@end
	
PS：因为Cocoa 和 Cocoa Touch框架里面有大量的带有前缀的类，比如UI，CA，NS，等等。而这些前缀都跟跟很多历史有关，大家自行了解即可。

方法和属性名只需要在定义的类中是唯一的即可。尽管每一个C函数在一个应用里面都必须是唯一的名称，但是在不同的Objective-C类中定义相同的方法名是可接受的。你不能在同一个类中你不能多次声明同一个方法，如果你希望从写继承自父类的方法，你需要使用与声明是完全一样的方法名。

和方法一样，对象的属性和实例变量(详见[ Most Properties Are Backed by Instance Variable](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/EncapsulatingData/EncapsulatingData.html#//apple_ref/doc/uid/TP40011210-CH5-SW6))必须在定义他们的类中是唯一的。如果你使用全局变量，这些变量名在项目里必须是唯一的。

更多的命名约定和建议详见[ Conventions ](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/Conventions/Conventions.html#//apple_ref/doc/uid/TP40011210-CH10-SW1)

## 类的实现提供内部行为

一旦你定义了类的接口，包括可公开访问的属性和方法，你就需要编写实现类的行为的代码。

如前所述，类的接口通常放在专用的文件中，通过引用头文件来使用接口，即那些生成`.h`拓展名的文件。在具有`.m`拓展名得源文件中编写接口的实现。

每当在头文件中定义了接口，你都需要在编译实现源文件之前告诉编译器读取接口头文件。Objective-C提了预处理器指令`#import`来实现这个目的。更C语言的`#include`指令类似，但是会确保在编译期间只会引入头文件一次。

注意那些预处理器指令跟传统的C语言的语句不一样，不需要使用分号来结束语句。

### 基本语法

接口实现的语法如下所示：

	#import "XYZPerson.h"
 
	@implementation XYZPerson
 
	@end
	
如果你在接口中定义了方法，你需要在这个文件中实现这些方法。

### 实现方法

如下展示了一个简单的类，只有一个方法：

	@interface XYZPerson : NSObject
	- (void)sayHello;
	@end
	
接口的实现如下所示：

	#import "XYZPerson.h"
 
	@implementation XYZPerson
	- (void)sayHello {
    	NSLog(@"Hello, World!");
	}
	@end
	
这个例子使用了`NSLog()`函数来打印信息到控制台。跟标准C语言库的`printf()`函数类似，可以携带不同数量的参数，第一个参数必须是Objective-C的string类型。

方法的实现和C函数很类似，使用大括号来包含相关的代码。这方法的名字必须和原型是相同的，参数和返回值类型必须完全一致。

Objective-C与C语言一样，都是大小写敏感，所以这个方法：

	- (void)sayhello {
	}
	
编译器会当做与`sayHello`完全不同的方法。

通常，方法名应该以小写字母开始。跟典型的C语言相比，Objective-C约定给方法名使用更具描述性的名字。如果一个方法的名字有多个词组成，使用驼峰的形式更具可读性。

在Objective-C中需要注意空格是很灵活的。通常我们会习惯性的使用`tabs`和`spaces`键让每个代码块缩进，你将经常在新的一行看到左开的大括号，如下所示:

	- (void)sayHello
	{
    	NSLog(@"Hello, World!");
	}

Xcode，苹果集成开发环境(IDE)用来开发OS X和iOS的软件，会根据你的偏好设置来进行自动缩进。你可以在偏好设置中进行更改。

在下一章中，你将会看到大量的方法实现的例子，详见[ Working with Objects ](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/WorkingwithObjects/WorkingwithObjects.html#//apple_ref/doc/uid/TP40011210-CH4-SW1)

## Objective-C中类也是对象

在Objective-C中，类本身就是一个不透明类型叫做`Class`的对象。类不能像实例一样定义属性，但是可以接收消息。

类方法的典型应用是作为工厂方法，当做对象的内存分配和初始化过程的可替代方法，[ Objects Are Created Dynamically ](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/WorkingwithObjects/WorkingwithObjects.html#//apple_ref/doc/uid/TP40011210-CH4-SW7)。比如，`NSString`类有大量可用的工厂方法，不仅可以用来创建空得字符串对象，也可以使用指定的字符值来初始化，包括如下方法：

	+ (id)string;
	+ (id)stringWithString:(NSString *)aString;
	+ (id)stringWithFormat:(NSString *)format, …;
	+ (id)stringWithContentsOfFile:(NSString *)path encoding:(NSStringEncoding)enc error:(NSError **)error;
	+ (id)stringWithCString:(const char *)cString encoding:(NSStringEncoding)enc;
	
如上展示的这些例子，类方法使用 + 号，跟实例方法的 - 号不一样。

类方法的原型与实例方法的原型一样包含在类的接口里。方法的实现和类方法也类似，包含在类的`@implementation`块中。

原文档中这里是一些关于类的概念的编程练习，这里不再翻译，也不做解答，请参考原文档。

# 对象协作

Objective-C应用程序中大部分的工作都是对象网中对象之间消息的来回传递。一些对象是Cocoa 或者 Cocoa Touch库的类的实例，一些是你自己的类的实例。

上一个章描述了定义类接口和实现的语法，响应消息的方法实现。本章将解释怎么给一个对象发送消息，包括覆盖了一些Objective-C的动态特性，包括动态类型，以及在运行时决定哪个方法是可以被调用的能力。

当一个对象可以被使用之前，这个对象必须使用给属性分配内存和初始化必要的内部值的两段式构造来进行正确的创建。本章描述了怎么嵌套方法调用内存分配和初始化来保证对象正确的创建。

## 对象发送及接收消息

尽管Objective-C里有多重不同方式可以在对象间发送消息，最常用的还是方括号，如下：

	[someObject doSomething];
	
接收者在左边，`someObject`是消息的接收者。消息在右边，`doSomething`，是接收者调用的方法名。换言之，当上面的这行代码执行的时候，`someObject`就会发送`doSomething`消息。

上一个章节描述了如何创建类的接口，如下：

	@interface XYZPerson : NSObject
	- (void)sayHello;
	@end
	
以及实现：

	@implementation XYZPerson
	- (void)sayHello {
	    NSLog(@"Hello, world!");
	}
	@end
	
*注意：*这个示例使用了Objective-C的字符串字面量，`@"Hello, world!"`。字符串是Objective-C里面的几个类类型之一，允许使用字符串字面量来创建。指定一个字符串`@"Hello, world!"`概念上讲相当于“一个表示字符串`Hello, world!`的Objective-C字符串对象”。
字面量和对象的创建，更多信息参见[ Objects Are Created Dynamically ](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/WorkingwithObjects/WorkingwithObjects.html#//apple_ref/doc/uid/TP40011210-CH4-SW7)

将设你已经占有一个`XYZPerson`对象，你可以给它这样发送`sayHello`消息：

	[somePerson sayHello];
	
发送一个Objective-C消息概念上讲跟调用C函数非常类似。如图2 - 1展示了 `sayHello` 消息的程序流程。

Figure 2-1  Basic messaging program flow
![alt text](/img/ProgrammingwithObjectiveC/programflow1.png)

为了指定消息的接收者，要明白在Objective-C里面怎样使用指针来引用对象。

### 使用指针跟踪对象

C 和 Objective-C使用变量来跟踪值，就像大部分其他编程语言一样。

标准C语言中定义了许多基本类型，包括整型，浮点型和字符型，声明赋值如下：

	int someInteger = 42;
    float someFloatingPointNumber = 3.14f;
    
本地变量，声明在方法或者函数体内，如下：

	- (void)myMethod {
	    int someInteger = 42;
	}
	
本地变量只在定义他们的作用域中有效。

在这里例子中，`someInteger`就是定义在方法`myMethod`的本地变量；一旦执行到方法的终止大括号，`someInteger`将不能访问。当一个本地基本类型变量(如 `int`或者`float`)消失时，他们的值也会消失。

相比之下，Objective-C对象的分配略有不同。对象通常比方法作用域有更长的生命周期。在一些特殊情形中，一个对象需要存在的时间往往比创建跟踪它的原始变量存在的时间更久，所以一个对象的内存是动态分配和释放的。

*注意：*本地变量是在栈上分配内存的，对象在堆上分配内存。

所以对象要求你使用C指针(保存内存地址)来跟踪对象在内存中得地址，如下：

	- (void)myMethod {
	    NSString *myString = // get a string from somewhere...
	    [...]
	}
	
尽管指针变量`myString`(星号表示它是指针)的作用域限制在方法`myMethod`的作用域内，实际上指向内存中的这个字符串对象比方法作用域存在更长的时间。它可能还存在，或者你需要在方法调用之后传递这个对象。

### 对象可以当做方法参数

如果你需要在发送消息的时候传递一个对象，你需要在方法中提供支持。上一章描述了声明带有单个参数的方法:

	- (void)someMethodWithValue:(SomeType)value;
	
带有一个字符串对象作为参数的方法如下：

	- (void)saySomething:(NSString *)greeting;
	
你可能像这样实现`saySomething:`方法：

	- (void)saySomething:(NSString *)greeting {
	    NSLog(@"%@", greeting);
	}

这里的`greeting`指针的行为和本地变量一样，作用域被限制在`saySomething`方法内，尽管如此，这个字符串对象在方法调用之前就已经存在，在方法调用之后任然会继续存在。

### 方法可以返回值

和通过参数传递值一样，方法也可以返回一个值。到目前为止，所有方法的返回类型都是`void`。C关键字`void`意味这个方法不返回任何值。

指定返回类型为`int`意味着这个方法会返回一个整型值：

	- (int)magicNumber;
	
在方法的实现中，使用C语句的`return`语句来指示在方法执行完成时应该返回的值，如下所示：

	- (int)magicNumber {
	    return 42;
	}
	
你完全可以忽略方法返回的值。`magicNumber`方法除了仅仅返回一个值之外没有做任何有用的事情，像下面这样调用这个方法是完全正确的：

	[someObject magicNumber];
	
如果你需要跟踪方法的返回值，你可以声明一个变量，然后再将方法的返回值赋值给变量，如下：

	int interestingNumber = [someObject magicNumber];
	
类似的，方法也可以返回一个对象。`NSString`类，比如，提供一个`uppercaseString`方法：

	- (NSString *)uppercaseString;

跟返回基本类型类似的，只不过你需要使用指针来跟踪方法的返回值：

	NSString *testString = @"Hello, world!";
    NSString *revisedString = [testString uppercaseString];
    
当方法调用返回值时，`revisedString`将会指向一个表示字符串`HELLO WORLD!`的`NSString`对象。

实现一个返回对象的方法如下：

	- (NSString *)magicString {
	    NSString *stringToReturn = // create an interesting string...
	 
	    return stringToReturn;
	}
	
`stringToReturn`指针指向的这个字符串对象被当做返回值传递后会继续存在，即使`stringToReturn`指针已不在作用域内。

在这种情形下有一些内存管理注意事项：方法的返回值是一个对象(在堆上创建)的时候，我们需要这个对象存在足够长的时间，以便于方法的调用者可以使用该对象，但并不需要这个对象是永久存在于内存中，因为那样会导致内存泄露。Objective-C编译器的ARC功能在极大程度上帮你处理了这些注意事项。

### 对象可以给自己发送消息

当你在编写方法实现的时候，你都访问了一个非常重要的隐藏值，`self`。`self`相当于是接收消息的对象的引用。它是一个指针，就像上面的`greeting`一样，可以在当前消息接收的对象上调用方法。

你可能决定重构`XYZPerson`的实现，通过使用`saySomething:`方法来调整`sayHello`方法，将NSLog()的调用移到另外的方法中。你可以添加其他的方法，比如`sayGoodbye`，我们通过调用`saySomething:`方法来处理具体的问候过程。如果你之后想要在用户界面的文本框中展示每个问候，你只需要修改`saySomething:`方法，而不用单独调整`sayHello`方法。

	@implementation XYZPerson
	- (void)sayHello {
	    [self saySomething:@"Hello, world!"];
	}
	- (void)saySomething:(NSString *)greeting {
	    NSLog(@"%@", greeting);
	}
	@end
	
如果你给`XYZPerson`对象发送`sayHello`消息，有效的程序流程如图2 - 2所示：

Figure 2-2  Program flow when messaging self
![alt text](/img/ProgrammingwithObjectiveC/programflow2.png)

### 对象可以调用父类方法的实现

在Objective-C中有另外一个重要的关键字可以使用，叫做`super`。给`super`发送消息是通过继承链调用父类实现方法的一种方式。在方法重写时常会使用`super`。

现在创建一个person类的新的类型，“shouting person”类，每次问候都使用大写字母显示。你可以创建一个`XYZPerson`类的副本，然后修改字符串为大写的，但是最简单的方式是新建一个继承自`XYZPerson`的类，然后重写`saySomething:`方法来让显示的问候语是大写的，如下：

	@interface XYZShoutingPerson : XYZPerson
	@end
	
	@implementation XYZShoutingPerson
	- (void)saySomething:(NSString *)greeting {
	    NSString *uppercaseGreeting = [greeting uppercaseString];
	    NSLog(@"%@", uppercaseGreeting);
	}
	@end
	
在这个例子中，我们声明了一个额外的字符串指针`uppercaseGreeting`，然后将`greeting`对象发送`uppercaseString`消息的返回值赋值给这个指针。如你之前所见，这是一个新的字符串对象。

因为`sayHello`是由`XYZPerson`实现的，`XYZShoutingPerson`继承自`XYZPerson`，所以你也可以在`XYZShoutingPerson`类的对象上调用`sayHello`方法。当你在`XYZShoutingPerson`类的对象上调用`sayHello`方法，`[self saySomething:...] `将会使用`XYZShoutingPerson`类重写的版本，然后显示大写的问候语句，如图2 - 3展示了有效的程序流程：

Figure 2-3  Program flow for an overridden method
![alt text](/img/ProgrammingwithObjectiveC/programflow3.png)

`XYZShoutingPerson`类的实现并不理想，假如你之后决定修改`XYZPerson`类的`saySomething`方法将问候语显示在用户界面的元素中，而不是通过`NSLog()`打印到控制台中，同样你也需要修改`XYZShoutingPerson`的实现。

一种较好的解决方案就是改变当前的`XYZShoutingPerson`的`saySomething:`版本，通过调用父类`XYZPerson`的实现来处理上述的情形：

	@implementation XYZShoutingPerson
	- (void)saySomething:(NSString *)greeting {
	    NSString *uppercaseGreeting = [greeting uppercaseString];
	    [super saySomething:uppercaseGreeting];
	}
	@end
	
如下图2 - 4展示了给`XYZShoutingPerson`对象发送`sayHello`消息的流程图。

Figure 2-4  Program flow when messaging super
![alt text](/img/ProgrammingwithObjectiveC/programflow4.png)

## 对象是动态创建的

如本章之前所述，内存是动态分配给Objective-C对象的。创建对象的第一步就是不仅要为对象的类定义的属性分配足够的内存，而且也要为继承链中的父类定义的属性分配足够的内存。

根类`NSObject`提供了一个`alloc`类方法来为你处理上述过程：

	+ (id)alloc;
	
注意这个方法的返回值类型是`id`。这是Objective-C里一个特别的关键字，用来表示某种类型的对象。是一个指向对象的指正，就如(NSObject *)，特别的地方就是不需要使用星号。本章后面会详细讲解，详见[ Objective-C Is a Dynamic Language ](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/WorkingwithObjects/WorkingwithObjects.html#//apple_ref/doc/uid/TP40011210-CH4-SW18)

`alloc`方法还有另外一个很重要的任务，通过设置对象的属性为"零"(这里的零是象征性的说法)清理分配的内存。这样就避免了在使用的时候因为之前存储过其他的信息导致内存里面有垃圾信息的问题，但是这些并没有完全的初始化一个对象，只是完成了内存分配而已。

你需要将`alloc`方法和`NSObject`的`init`方法组合起来：

	- (id)init;
	
`init`方法保证对象的属性在创建时拥有合适的初始值，下一章包含了更多细节。注意`init`方法的返回值也是`id`类型。

如果一个方法返回一个对象指针，那么就是嵌套成为另一个方法的接收者，通过组合在一起可以在一条语句中发送多条消息。正确的初始化对象的方式是将`alloc`方法嵌套到`init`方法中，如下：

	NSObject *newObject = [[NSObject alloc] init];
	
这个例子设置变量`newObject`指向新创建的`NSObject`类的实例。

最内层的方法最先执行，所以给`NSObject`类发送`alloc`消息，然后返回了一个分配了内存的`NSObject`实例。返回的这个对象会被当做`init`消息的接收者，返回的对象会被赋值给`newObject`指针，如图2 - 5所示：

Figure 2-5  Nesting the alloc and init message
![alt text](/img/ProgrammingwithObjectiveC/nestedallocinit.png)

*注意：*`init`方法可能会返回一个与`alloc`方法创建的不同的对象，所以最佳实践就是嵌套调用，如上所示。
绝对不要初始化一个对象之后而没有重新赋值指向这个对象的指针，比如，不要像如下所示这样：

	NSObject *someObject = [NSObject alloc];
    [someObject init];
    
如果调用`init`返回了其他的对象，那么你将会留下一个只分配了内存但是从来没有初始化的指向对象的指针。

### 初始化器方法可以带参数

有些对象需要指定的值来完成初始化。比如`NSNumber`类的对象，需要数值来初始化。

`NSNumber`类定义了几个初始化器，如下：

	- (id)initWithBool:(BOOL)value;
	- (id)initWithFloat:(float)value;
	- (id)initWithInt:(int)value;
	- (id)initWithLong:(long)value;
	
调用带参数的初始化方法跟调用简洁的`init`方法一样，一个`NSNumber`类的对象的内存分配和初始化如下所示：

	NSNumber *magicNumber = [[NSNumber alloc] initWithInt:42];
	
### 类的工厂方法是内存分配和初始化的替代方案

如前一章所述，类可以定义工厂方法。工厂方法提供了传统初始化过程`alloc] init]`的替代方案，不再嵌套两个方法。

`NSNumber`类定义了和初始化器方法对应的类工厂方法，如下：

	+ (NSNumber *)numberWithBool:(BOOL)value;
	+ (NSNumber *)numberWithFloat:(float)value;
	+ (NSNumber *)numberWithInt:(int)value;
	+ (NSNumber *)numberWithLong:(long)value;
	
使用方式如下：

	NSNumber *magicNumber = [NSNumber numberWithInt:42];
	
如上所示的代码效果与之前` alloc] initWithInt:]`的效果是一样的。类的工厂方法通常直接调用`alloc`方法和相关的`init`方法，非常的方便。

### 可以使用`new`来创建不需要参数的初始化

同样也可以使用类方法`new`来创建类的实例。这个方法由`NSObject`类提供，你自己的子类不需要重写。

如下所示两种写法效果完全一样：

	XYZObject *object = [XYZObject new];
    // is effectively the same as:
    XYZObject *object = [[XYZObject alloc] init];
    
### 字面量提供了一种简洁的对象创建语法

有些类允许你使用更简洁的字面量语法来创建实例。

你可以使用指定的字面符号来创建一个`NSString`类的实例，如下：

	NSString *someString = @"Hello, World!";
	
如上所示代码的效果跟使用两段式构造或者类的工厂方法的效果是一样的：

	NSString *someString = [NSString stringWithCString:"Hello, World!"
                                              encoding:NSUTF8StringEncoding];
                                              
`NSNumber`允许使用不同的字面量：

	NSNumber *myBOOL = @YES;
    NSNumber *myFloat = @3.14f;
    NSNumber *myInt = @42;
    NSNumber *myLong = @42L;
    
你也可以使用表达式来创建`NSNumber`类的实例：

	NSNumber *myInt = @(84 / 2);
	
如上所示，表达式会被求值，结果用来创建`NSNumber`类的实例。

Objective-C同样支持字面量来创建不可变的`NSArray`和`NSDictionary`的对象，详见[ Values and Collections ](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/FoundationTypesandCollections/FoundationTypesandCollections.html#//apple_ref/doc/uid/TP40011210-CH7-SW1)

## Objective-C是动态语言

如之前提到的，你需要一个指针来跟踪内存中的对象。因为Objective-C的动态特性，它并不在意指针所指向的类型--当你给相关的对象发送消息时，总会调用正确的方法。

`id`类型定义了一个通用的对象指针。完全可以使用`id`来声明变量，但是但编译时期就无法获得关于对象的一些信息。

考虑以下代码：

	id someObject = @"Hello, World!";
    [someObject removeAllObjects];
    
如上所示，`someObject`是指向`NSString`实例的指针，但是编译器并不知道这些信息，编译器仅仅知道`someObject`是某一种类型的对象。Cocoa 或者 Cocoa Touch的部分类定义了`removeAllObjects`消息，比如`NSMutableArray`，所以编译器这个并没有警告，即使这些代码在运行时会产生异常，因为`NSString`的对象是无法响应`removeAllObjects`消息的。

使用一个静态类型重写上面的代码：

	NSString *someObject = @"Hello, World!";
    [someObject removeAllObjects];
    
现在编译器就产生了一个错误，因为编译器知道`NSString`类的公开接口里面没有定义`removeAllObjects`消息。

因为对象的类型是在运行时才决定的，当你将创建或者处理的实例赋值给一个变量的时候，使用什么类型并没有什么区别。如下所示：

	XYZPerson *firstPerson = [[XYZPerson alloc] init];
    XYZPerson *secondPerson = [[XYZShoutingPerson alloc] init];
    [firstPerson sayHello];
    [secondPerson sayHello];
    
尽管`firstPerson` 和 `secondPerson`都是静态类型`XYZPerson`的对象，`secondPerson`会在运行时指向`XYZShoutingPerson`对象。当调用`sayHello`方法的时候，会调用方法正确的实现；对`secondPerson`来说，就是调用`XYZShoutingPerson`的版本。

### 判断对象相同

如果你需要决定一个对象是否与另外一个对象一样，最重要的一点就是要记住你是在处理指针。

标准的C的相等运算符 == 用来测试两个变量的值是否相等，如下所示：

	if (someInteger == 42) {
        // someInteger has the value 42
    }
    
当处理对象的时候，== 运算符用来测试两个不同的指针是否是指向同一个对象：

	if (firstPerson == secondPerson) {
        // firstPerson is the same object as secondPerson
    }
    
如果你需要测试两个对象是否有相同的数据，你需要调用`isEqual:`方法，由`NSObject`提供：

	if ([firstPerson isEqual:secondPerson]) {
        // firstPerson is identical to secondPerson
    }
    
如果你需要比较两个对象的大小，你不能使用标准C的比较运算符 > 和 <。基础的Foundation类型，比如`NSNumber`, `NSString` 和 `NSDate`，提供了`compare:`方法：

	if ([someDate compare:anotherDate] == NSOrderedAscending) {
        // someDate is earlier than anotherDate
    }
    
### 使用nil

在声明变量时进行复制是明智的，否则就会包含上次存储内容的垃圾信息：

	BOOL success = NO;
    int magicNumber = 42;
    
对于对象指针来说就是不必要的，因为编译器会自动设置变量为`nil`在没有其他初始化值的时候：

	XYZPerson *somePerson;
    // somePerson is automatically set to nil
    
当你没有其他的可用初始化值得时候，`nil`值是一种安全的初始化对象的方式，因为在Objective-C中给`nil`发送消息是完全可接受的。如果你给`nil`发送消息，显然什么都不会发生。

*注意：*如果你给`nil`发送的消息有返回值，如果返回值是对象类型就会返回`nil`，如果是数值类型就会返回0，如果是布尔类型就会返回`NO`。如果是结构体，会将结构体所有成员设置成默认值。

如果你像检查确保一个对象不为`nil`，你也可以使用标准C的不等运算符：

	if (somePerson != nil) {
        // somePerson points to an object
    }
    
或者就是简单的提供这个变量：

	if (somePerson) {
        // somePerson points to an object
    }
    
如果`somePerson`变量是`nil`，那么逻辑值就是0(假)，如果有地址，就不是0，所以计算结果就是真。

类似的，如果你需要检查一个`nil`的变量，你也可以使用相等运算符：

	if (somePerson == nil) {
        // somePerson does not point to an object
    }
    
或者使用逻辑否运算符：

	if (!somePerson) {
        // somePerson does not point to an object
    }
    
# 封装数据

对象通过属性来封装数据。

本章将会描述Objective-C为对象属性声明的语法，并会讲解这些属性通过合成的访问器方法和实例变量是怎样默认实现的。如果一个属性被实例变量支持，这个变量必须在任何初始化方法中正确的设置。

如果一个对象通过属性需要与另外一个对象来保持连接，很重要的一点就是考虑两个对象之间关系的性质。尽管Objective-C中对象内存管理的工作大部分通过Automatic Reference Counting (ARC)来处理，但是，知道怎样避免强引用环导致的内存泄露问题也很重要。本章将会讲解对象的生命周期，以及如何思考对象间的关系来管理对象图。

## 属性封装了对象的值

大部分的对象都需要持续跟踪信息来执行任务。有些对象被设计成具有一个或多个值的模型，比如`Cocoa`的类`NSNumber`就有一个数字值，或者自定义类`XYZPerson`模型就有人的姓名。有些对象在一定范围内更通用，比如处理用户界面与信息之间的交互，但是这些对象也需要持续跟踪用户界面元素或者相关的模型对象。

### 声明公共属性来暴露数据

Objective-C的属性提供了一种定义类想要封装的信息的方式。如你所见[ Properties Control Access to an Object’s Values ](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/DefiningClasses/DefiningClasses.html#//apple_ref/doc/uid/TP40011210-CH3-SW7)，属性定义在类的接口中，所示：

	@interface XYZPerson : NSObject
	@property NSString *firstName;
	@property NSString *lastName;
	@end
	
在这里示例中，`XYZPerson`类声明了字符串属性来存储人得姓和名。

在面向对象编程的主要原则之一就是把对象的内部实现细节隐藏在公共接口之后，通过类暴露出来的行为来获取对象的属性，这一点很重要，而不是试图直接获取内部的值。

### 使用访问器方法访问或设置属性的值

通过访问器方法来获取或者设置对象的属性：

	NSString *firstName = [somePerson firstName];
    [somePerson setFirstName:@"Johnny"];
    
默认情况下，编译器自动为你合成了访问器方法，除了使用`@property`在类接口文件中声明属性之外不需要再做其他任何事情。

合成的方法遵循特定的命名约定：

* 用来获取值的方法(`getter`方法)和属性有相同的名字。叫做`firstName`的属性的`getter`方法名就叫做`firstName`
* 用来设置值的方法(`setter`方法)的名称由“set”开始，然后用首字母大写的属性名。叫做`firstName`的属性的`setter`方法名就是`setFirstName:`。

如果你想要属性的值不能被修改，你可以在声明属性的时候指定属性的描述属性为`readonly`：

	@property (readonly) NSString *fullName;

属性的描述属性除了向别的对象展示了该如何和属性交互，也告诉了编译器应该怎样合成相关的访问器方法。

在上述示例中，编译器只会合成`fullName`的getter方法，但不会合成`setFullName:`方法。

*注意：*`readonly`相反就是`readwrite`。不需要显式的特别指定`readwrite`，为默认值。

如果你想使用不同名字的访问器方法，你可以在给属性增加描述属性的时候指定你自定义的名字。当有布尔属性(属性有`YES`或者`NO`值)时这种情况比较常见，`getter`方法通常使用自定义的“is”开始。比如，一个属性的getter方法叫做`finished`，应该叫做`isFinished`。

	@property (getter=isFinished) BOOL finished;
	
如果你想指定多个属性的描述属性，用逗号分割，如下：

	@property (readonly, getter=isFinished) BOOL finished;
	
如上所示，编译器只会合成`isFinished`方法，不会合成`setFinished:`方法。

*注意：*通常来说，访问器方法应该兼容键值编码，也就是说遵循着明确的命名约定。详见[ Key-Value Coding Programming Guide ](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/KeyValueCoding/Articles/KeyValueCoding.html#//apple_ref/doc/uid/10000107i)

### 点操作符是调用访问方法的替代方式

除了显式的调用访问器方法，Objective-C也提供了点语法来获取对象的属性作为替代方案。

如下：

	NSString *firstName = somePerson.firstName;
    somePerson.firstName = @"Johnny";
    
点语法知识单纯的包装了调用访问器方法。当你使用点语法，属性任然是使用`getter` 和 `setter`方法来获取、设置的：

* 使用`somePerson.firstName`获取值，跟使用`[somePerson firstName]`是一样的
* 使用`somePerson.firstName = @"Johnny"`设置值跟使用`[somePerson setFirstName:@"Johnny"]`是一样的效果。

所以这意味着使用点语法来获取属性任然是被属性的描述属性控制着的。如果一个属性的描述属性是`readonly`，那么你使用点语法来设置值，编译器就会报错。

### 大部分属性都被实例变量支持

默认的，`readwrite`的属性会被一个实例变量支持，实例变量也是被编译器自动合成。

实例变量就是一个变量，存在于对象的生命周期之中，用来保存值。所使用的内存在对象创建的时候分配(通过`alloc`方法)，在对象释放的时候释放内存。

除非你指定，否则合成的实例变量和属性有相同的名字，但是有下划线前缀。一个叫做`firstName`的属性，合成的实例变量叫做`_firstName`。

尽管对于一个对象获取自己属性的最佳实践是使用访问器方法或者点语法，但是在类的实现中，所有的实例方法都可以直接获取实例变量。下划线前缀很清楚的表示你获取的是一个实例变量，而不是本地变量，比如：

	- (void)someMethod {
	    NSString *myString = @"An interesting string";
	 
	    _someString = myString;
	}
	
如上所示，`myString`是本地变量，`_someString`是一个实例变量。

通常来说，你应该使用访问器方法或者点语法来获取属性，即使你在该对象的实现里，你应该使用`self`：

	- (void)someMethod {
	    NSString *myString = @"An interesting string";
	 
	    self.someString = myString;
	  // or
	    [self setSomeString:myString];
	}

这条规则的例外情况就是当你在编写初始化、释放、或者自定义的访问器方法时。

#### 你可以自定义合成的实例变量名

如之前所提，对于可写的属性，编译器默认合成的实例变量名叫做`_propertyName`。

如果你想为实例变量使用不同的名字，那么你需要指示编译器在合成变量时使用遵循如下所示的语法，在类的实现文件中：

	@implementation YourClass
	@synthesize propertyName = instanceVariableName;
	...
	@end
	
比如：

	@synthesize firstName = ivar_firstName;
	
如上所示，属性名仍然叫做`firstName`，可以通过`firstName`和`setFirstName:`以及点语法来获取属性，但是该属性会被一个叫做`ivar_firstName`的实例变量所支持。

*重要：*如果你使用`@synthesize`时没有指定实例变量名，如下：

	@synthesize firstName;
	
这个实例变量名会和属性名一样。

比如，这个实例变量名也是`firstName`，没有下划线。

#### 你可以在没有属性的情况下定义实例变量

当你任何时候想要保持跟踪一个对象的值或者另一个对象的时候，使用属性是最佳选择。

如果你需要在没有声明属性的情况下定义自己的实例变量，你可以把它们添加到类的接口和实现的大括号中，如下：

	@interface SomeClass : NSObject {
	    NSString *_myNonPropertyInstanceVariable;
	}
	...
	@end
	 
	@implementation SomeClass {
	    NSString *_anotherCustomInstanceVariable;
	}
	...
	@end
	
*注意：*你也可以在类的拓展的顶部添加实例变量，详见[ Class Extensions Extend the Internal Implementation ](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/CustomizingExistingClasses/CustomizingExistingClasses.html#//apple_ref/doc/uid/TP40011210-CH6-SW3)。

### 直接从初始化器方法存取实例变量

`setter`方法可能会有额外的副作用。它们可能会触发KVC的通知，或者自定义的方法执行了额外的任务。

你应该总是在初始化方法里面直接存取实例变量，因为那时属性被设置，对象可能还没完全初始化完成。即使你没有提供自定义的访问器方法或者知道自己类中得副作用，将来的一个子类可能很好的覆盖这些行为。

典型的`init`方法如下所示：

	- (id)init {
	    self = [super init];
	 
	    if (self) {
	        // initialize instance variables here
	    }
	 
	    return self;
	}
	
一个	`init`方法应该在执行自己的初始化任务之前将调用父类初始化方法的返回值赋值给`self`。父类可能不能正确的初始化这个对象而返回`nil`，所以你需要总是检查，以确保在执行你自己的初始化时`self`不是`nil`。

通过调用`[super init]`方法，对象从基类的初始化开始，向下通过每一个子类的`init`方法实现。如图3 - 1展示了初始化`XYZShoutingPerson`对象的过程。

Figure 3-1  The initialization process
![alt text](/img/ProgrammingwithObjectiveC/initflow.png)

对象不仅可以调用`init`来初始化，也可以通过调用带有指定参数值的初始化方法来初始化。

在`XYZPerson`类中，提供一个设置姓和名的初始化方法完全合理：

	- (id)initWithFirstName:(NSString *)aFirstName lastName:(NSString *)aLastName;
	
方法实现如下：

	- (id)initWithFirstName:(NSString *)aFirstName lastName:(NSString *)aLastName {
	    self = [super init];
	 
	    if (self) {
	        _firstName = aFirstName;
	        _lastName = aLastName;
	    }
	 
	    return self;
	}
	
#### 指定初始化器是主要的初始化方法

如果一个对象定义了一个或者多个初始化方法，你应该决定哪一个方法是指定的初始化器。通常来说是提供了最多选择性的初始化方法(最多的参数)，会被其他的便利初始化器调用。你应该代表性的重写`init`方法使用合适的值来调用你自己的指定初始化器。

如果`XYZPerson`类还有一个生日日期的属性，这指定初始化器如下：

	- (id)initWithFirstName:(NSString *)aFirstName lastName:(NSString *)aLastName
                                            dateOfBirth:(NSDate *)aDOB;
                                            
应该在这个方法里面设置相关的一些实例变量，如上所示。如果你仍旧希望提供一个只需设置姓和名的便利初始化器，在实现这个方法的时候你应该调用指定初始化器，如下：

	- (id)initWithFirstName:(NSString *)aFirstName lastName:(NSString *)aLastName {
	    return [self initWithFirstName:aFirstName lastName:aLastName dateOfBirth:nil];
	}
	
你可以使用合适的默认值调用指定初始化器实现标准的`init`方法：

	- (id)init {
	    return [self initWithFirstName:@"John" lastName:@"Doe" dateOfBirth:nil];
	}
	
当你需要为子类写初始化方法时，但父类又有多个初始化方法，你应该重写父类的指定初始化器方法来执行自己的初始化，或者添加你自己的初始化器。不管是哪一种方法，你都应该在这里`[super init];`调用父类的指定初始化器在执行自己的初始化任务之前。

### 自定义访问器方法

属性不是总是需要被自己的实例变量支持。

比如，`XYZPerson`类也许定义了一个`read-only`的属性：

	@property (readonly) NSString *fullName;
	
跟不得不在first 或者 last name改变时更新`fullName`属性比起来，在自定义的访问器方法中根据要求构建`fullName`更简单：

	- (NSString *)fullName {
	    return [NSString stringWithFormat:@"%@ %@", self.firstName, self.lastName];
	}
	
如上，这个简单的例子利用格式化字符串来构建`fullName`。

*注意：*虽然这个例子很简单、方便，但是并不适用于所有国家。

如果你使用实例变量为属性编写自定义的访问器方法，你必须在方法里面直接存取实例变量。比如，推迟属性的初始化直到第一次使用的时候很常见，使用延迟加载，如下：

	- (XYZObject *)someImportantObject {
	    if (!_someImportantObject) {
	        _someImportantObject = [[XYZObject alloc] init];
	    }
	 
	    return _someImportantObject;
	}
	
在返回值之前，该方法首先检查了`_someImportantObject`实例变量是否是`nil`；如果是，就初始化。

*注意：*编译器在所有情况下将会自动合成实例变量，也会至少合成一个访问器方法。如果你为`readwrite`属性实现了`getter`和`setter`方法，或者为`readonly`属性实现了`getter`方法，那么编译器将会假定你想自己控制属性的实现，就不会自动合成实例变量。

如果你仍然需要一个实例变量，你通过如下代码要求编译器合成一个：

	@synthesize property = _property;

### 属性默认是原子的

Objective-C属性默认是原子操作的：

	@interface XYZObject : NSObject
	@property NSObject *implicitAtomicObject;          // atomic by default
	@property (atomic) NSObject *explicitAtomicObject; // explicitly marked atomic
	@end

这意味着合成的访问器确保值通过`getter`和`setter`方法会完整的获取到或者被设置，即使在不同的线程中同时调用访问器方法。

因为在内部实现里面合成的`atomic`访问器方法是私有的，所以你不可能将自己实现的访问器方法和合成的访问器方法组合到一起。如果你这样做将会收到编译器的警告，比如，比如你为`atomic`，`readwrite`的属性提供了自定义的`setter`方法，然后将剩下的`getter`方法让编译器合成。

你可以使用`nonatomic`描述属性来指定合成的访问器只是简单的设置或者返回一个值，并不保证在不同线程里同时存取同一个值时会发生什么情况。基于这个原因，存取`nonatomic`的属性比`atomic`属性更快，同时你也可以将合成的`setter`方法和自己实现的`getter`方法组合起来：

	@interface XYZObject : NSObject
	@property (nonatomic) NSObject *nonatomicObject;
	@end
	
	@implementation XYZObject
	- (NSObject *)nonatomicObject {
	    return _nonatomicObject;
	}
	// setter will be synthesized automatically
	@end
	
*注意：*属性的原子性并不等同于对象的线程安全。

考虑如下情况，`XYZPerson`类的对象在一个线程里面通过`atomic`访问器方法修改了first 和 last names。如果另外一个线程在同一时间存取first 和 last names，`atomic`的`getter`方法会返回一个完整的字符串(不会崩溃)，但是并不会保证返回的这些值是正确的。如果first name在改变之前就存取了，但是last name在改变之后存取的，那么你得到的结果就是不一致的，错误匹配的names。

## 通过所有权和责任管理对象图

Objective-C对象的内存都是动态分配在堆上的，所以你需要使用指针来记录一个对象的地址。不同于基本类型，不能根据指针变量的作用域来决定一个对象的生命周期。相反，如果其他的对象还需要使用这个对象，那么这个对象就应该在内存中保留足够长的时间。

比起手动管理每一个对象的生命周期，你更应该考虑对象之间的关系。

比如，`XYZPerson`的对象，`firstName` 和 `lastName`属性被`XYZPerson`的实例拥有。意味着只要`XYZPerson`的对象还存在于内存中，那么这两个对象也应该存在在内存中。

当一个对象依赖于其他对象时，即这个对象拥有其他对象的所有权，就说第一个对象对其他对象强引用(strong references)。在Objective-C中，只要一个对象至少还有一个其他的对象强引用它，它就不会从内存中被清除。`XYZPerson`实例与`NSString`对象的关系如图3 - 2所示。

Figure 3-2  Strong Relationships
![alt text](/img/ProgrammingwithObjectiveC/strongpersonproperties.png)

当`XYZPerson`的对象从内存中被释放时，这两个string对象也被释放了，假如没有其他的强引用引用至它们。

为了增加一点难度，考虑如图3 - 3所示情形：

Figure 3-3  The Name Badge Maker application
![alt text](/img/ProgrammingwithObjectiveC/namebadgemaker.png)

当用户点击Update按钮时，显示的徽章被相关的信息更新了。

首先，person的细节被输入，然后点击update按钮，简单的对象图如图3 - 4所示。

Figure 3-4  Simplified object graph for initial XYZPerson creation
![alt text](/img/ProgrammingwithObjectiveC/simplifiedobjectgraph1.png)

当用户修改person的first name时，对象图如图3 - 5所示。

Figure 3-5  Simplified object graph while changing the person’s first name
![alt text](/img/ProgrammingwithObjectiveC/simplifiedobjectgraph2.png)

徽章视图仍旧保留了对`@"John"`字符串对象的强引用，即使`XYZPerson`对象已经时不同的`firstName`。意味着`@"John"`对象仍旧保留在内存中，被徽章视图用来打印名称。

一旦用户第二次点击Update按钮，徽章视图将会被告知更新内部的属性来匹配person对象，所以对象图如图3 - 6所示。

Figure 3-6  Simplified object graph after updating the badge view
![alt text](/img/ProgrammingwithObjectiveC/simplifiedobjectgraph3.png)

现在，没有强引用引用至`@"John"`对象了，将会从内存中被移除。

默认情况下，Objective-C的属性和变量都会保持对象的强引用。大多数情况下没有问题，但是这回引起一个潜在的问题：强引用环。

### 避免强引用环

尽管强引用在单向的对象关系中工作的很好，但是在一组内部关联的对象中就需要特别小心了。如果一组内部关联的对象形成了强引用环的关系，即使外部没有了引用至它们的强引用，它们仍然会被保留在内存中。

很可能就形成引用环的示例就是table view对象(`UITableView` for iOS and `NSTableView` for OS X)和它的代理。为了使table view类在多种情形下有更好的通用性，它的代理在对象的外部做一部分事情。意味着它依赖于另外一个对象来决定它的显示内容，或者用户和table view交互时做些什么工作。

常见的场景就是table view拥有它的代理的引用，然后代理也拥有table view的引用，如图3 - 7所示：

Figure 3-7  Strong references between a table view and its delegate
![alt text](/img/ProgrammingwithObjectiveC/strongreferencecycle1.png)

当没有强引用引用至table view 和 delegate时就会引发问题，如图3 - 8所示。

Figure 3-8  A strong reference cycle
![alt text](/img/ProgrammingwithObjectiveC/strongreferencecycle2.png)

即使不再需要它们保留在内存中--也没有外部其他的强引用引用至table view 或者 delegate，除了它们自己强引用至对方--但是它们依然保留在内存中。这就是强引用环。

解决该问题的方法就是将其中的一个强引用替换为弱引用(weak reference)。弱引用并不暗指两个对象之间拥有所有权或者责任，并不会持有一个对象。

如果table view使用弱引用引用至它的代理(`UITableView` 和 `NSTableView`就是这样解决强引用环问题的)，对象对如图3 - 9所示。

Figure 3-9  The correct relationship between a table view and its delegate
![alt text](/img/ProgrammingwithObjectiveC/strongreferencecycle3.png)

当其他的对象放弃对table view 和 delegate的强引用时，两个对象之间就不会存在强引用环，如图3 - 10.

Figure 3-10  Avoiding a strong reference cycle
![alt text](/img/ProgrammingwithObjectiveC/strongreferencecycle4.png)

这意味着delegate对象将会被释放，从而释放了table view上得强引用，如图3 - 11所示。

Figure 3-11  Deallocating the delegate
![alt text](/img/ProgrammingwithObjectiveC/strongreferencecycle5.png)

一旦delegate对象被释放，那就没有引用至table view的强引用了，所以table view也释放了。

### 使用`Strong`和`Weak`声明管理所有权

对象属性默认声明如下：

	@property id delegate;
	
使用强引用合成实例变量。如下所示定义一个弱引用的属性：

	@property (weak) id delegate;
	
*注意：*`weak`的对立就是`strong`。不需要显式的指定`strong`，因为它使默认的。

本地变量(非属性的实例变量)默认情况下也保持对象的强引用。这意味着如下代码正如你期望的那样：

	NSDate *originalDate = self.lastModificationDate;
    self.lastModificationDate = [NSDate date];
    NSLog(@"Last modification date changed from %@ to %@",
                        originalDate, self.lastModificationDate);

在这里实例中，本地变量`originalDate`保持了对`lastModificationDate`对象原始日期的强引用。当`lastModificationDate`属性改变的时候，该属性就不再保持对原始日期的强引用了，但是原始日期仍然在内存中，因为`originalDate`保持了对它的强引用。

*注意：*一个变量只在它所在的作用域内保持对一个对象的强引用，直到被重新赋值或者被设置为`nil`。

如果你不想一个变量保持强引用，你可以声明的时候使用`__weak`，如下所示：

	NSObject * __weak weakVariable;
	
因为弱引用不会持有一个对象，所以当引用在使用的时候，被引用的对象可能被释放。为了避免由释放对象引起的这个危险的悬空(大概意思就是这个指针并不指向任何对象了，类似野指针的概念)的指针引发问题，当对象被释放时弱引用会被自动设置为`nil`。

如果你使用弱引用编写上面的日期例子：

	NSDate * __weak originalDate = self.lastModificationDate;
    self.lastModificationDate = [NSDate date];
    
`originalDate`可能被设置为`nil`了。当`self.lastModificationDate`被重新赋值，该属性就不再保持对原始日期的强引用了。如果没有强引用至原始日期的对象，那么该对象将会被释放，`originalDate`被设置成`nil`。

`Weak`的变量可能是霍乱之源，特别是如下所示代码：

	NSObject * __weak someObject = [[NSObject alloc] init];
	
如上所示，这个新的初始化的对象没有强引用至它，所以它直接被释放了，`someObject`被设置为`nil`。

*注意：*`__weak`的对立就是`__strong`。同样的，`__strong`是默认值。

考虑如下情况，有时我们在一个方法中可能会存取`weak`属性多次，如下：

	- (void)someMethod {
	    [self.weakProperty doSomething];
	    ...
	    [self.weakProperty doSomethingElse];
	}

在这种情形下，你可能想要缓存`weak`属性通过强引用变量，保证在内存中保留足够长的时间：

	- (void)someMethod {
	    NSObject *cachedObject = self.weakProperty;
	    [cachedObject doSomething];
	    ...
	    [cachedObject doSomethingElse];
	}
	
在该示例中，`cachedObject`保持了对原来弱引用值的一个强引用，所以，只要`cachedObject`还在作用域内(也没有重新赋值其他的值)，就不会被释放。

如果你需要在使用之前确保弱引用属性不是`nil`，如上所示的方法很有用。只进行如下所示的测试是不够的：

	if (self.someWeakProperty) {
        [someObject doSomethingImportantWith:self.someWeakProperty];
    }
    
因为在一个多线程的应用程序中，这个属性可能在测试与方法调用之间就被释放了，所以这个测试其实是没有用的。相反，你需要声明一个强引用的本地变量来缓存这个值，如下：

	NSObject *cachedObject = self.someWeakProperty;           // 1
    if (cachedObject) {                                       // 2
        [someObject doSomethingImportantWith:cachedObject];   // 3
    }                                                         // 4
    cachedObject = nil;                                       // 5
    
如上所示的示例，第一行创建了一个强引用的变量，保证对象在测试和方法调用的时候是可用的。在第五行，`cachedObject`被设置成了`nil`，所以放弃了强引用。如果没有其他的强引用引用至最初的对象，该对象就会被释放，`someWeakProperty`将会被设置为`nil`。

### 使用`Unsafe Unretained References`

Cocoa 和 Cocoa Touch框架中有些类还不支持弱引用，所以你不能定义`weak`属性或者本地变量。这些类包括`NSTextView`，`NSFont` 和 `NSColorSpace`，完整列表详见 [ Transitioning to ARC Release Notes ](https://developer.apple.com/library/mac/releasenotes/ObjectiveC/RN-TransitioningToARC/Introduction/Introduction.html#//apple_ref/doc/uid/TP40011226)。

如果你想使用这些类的弱引用，你必须使用unsafe referenc。对于属性，使用`unsafe_unretained`描述：

	@property (unsafe_unretained) NSObject *unsafeProperty;
	
对于变量，你需要使用`__unsafe_unretained`：

	NSObject * __unsafe_unretained unsafeReference;
	
unsafe reference和weak reference一样，不持有关联的对象，但是当对象被释放的时候，它不会被设置为`nil`。这个词“unsafe(不安全)”，因此给这个指针发送消息时会导致程序崩溃。

### `copy`属性持有它们自己的拷贝

在某些情形下，对象可能希望保持自己对象属性的副本。

比如，`XYZBadgeView`类的接口如下所示：

	@interface XYZBadgeView : NSView
	@property NSString *firstName;
	@property NSString *lastName;
	@end
	
声明了两个`NSString`属性，都是它们的对象强引用。

如下所示，我们创建一个string对象，被设置为badge view的属性：

	NSMutableString *nameString = [NSMutableString stringWithString:@"John"];
    self.badgeView.firstName = nameString;
    
这完全是可行的，因为`NSMutableString`类是`NSString`的子类。尽管badge view认为处理的是`NSString`实例，但实际处理的是`NSMutableString`的实例。

这意味着这个string是可以改变的：

	[nameString appendString:@"ny"];
	
尽管最开始的时候设置给badge view的`firstName`属性的值是“John”，但是现在的值是“Johnny”了因为string的改变。

你也许会选择badge view应该保留任何设置给`firstName` 和 `lastName`属性值得副本，通过在声明属性的时候添加`copy`描述属性：

	@interface XYZBadgeView : NSView
	@property (copy) NSString *firstName;
	@property (copy) NSString *lastName;
	@end
	
这个对象现在有了这两个strings的副本。即使这个可变的string设置之后在之后改变，这个badge view只会捕获设置时的值。如下：

	NSMutableString *nameString = [NSMutableString stringWithString:@"John"];
    self.badgeView.firstName = nameString;
    [nameString appendString:@"ny"];
    
这一次，badge view持有的`firstName`属性的值将不会再受影响。

`copy`描述属性意味着属性将使用强引用，因为必须持有新创建的对象。

*注意：*任何你希望设置给`copy`属性的对象都必须支持`NSCopying`，意味着你必须适配`NSCopying`协议。详见协议[ Protocols Define Messaging Contracts ](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/WorkingwithProtocols/WorkingwithProtocols.html#//apple_ref/doc/uid/TP40011210-CH11-SW2)。关于`NSCopying`更多信息，详见[ NSCopying ](https://developer.apple.com/library/mac/documentation/Cocoa/Reference/Foundation/Protocols/NSCopying_Protocol/index.html#//apple_ref/occ/intf/NSCopying)，或者[ Advanced Memory Management Programming Guide ](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/MemoryMgmt/Articles/MemoryMgmt.html#//apple_ref/doc/uid/10000011i)。

如果你需要在初始化方法中直接设置`copy`属性的实例变量，比如，不要忘了使用原始对象的副本来设置：

	- (id)initWithSomeOriginalString:(NSString *)aString {
	    self = [super init];
	    if (self) {
	        _instanceVariableForCopyProperty = [aString copy];
	    }
	    return self;
	}

# 自定义已经存在的类

对象应该明确定义自己的任务，比如模型化信息，显示可视内容或者控制信息流。类的接口定义了其他对象期望与它交互的方式，并帮助这些对象完成任务。

有时，你可能会发现你希望通过增加功能来拓展现有的类，使其在特定情况下更有用。比如，你发现你的应用程序经常需要在可视界面上显示字符串。比起每次在需要显示的时候创建绘制字符串的对象，然而让`NSString`类具有将自己绘制到屏幕上的能力或许更合理。

在这种情况下，将一些工具类的行为添加到原始、主要的类的接口也不总是有意义的。在应用程序中大部分时间使用string对象时并不需要绘制的能力，在`NSString`的这个情形中，你并不能修改原始的接口和实现，因为它是框架里面的类。

而且，子类化已经存在的类也许也不合理，因为你不仅仅希望只有`NSString`类具有绘制的能力，同样也希望它的子类也有绘制的能力，比如`NSMutableString`。因为`NSString`在OS X 和 iOS都可用，所以在不同平台需要不同的绘制代码，你也需要在不同平台上使用不同的子类。

相反，Objective-C允许你使用类别和拓展向已经存在的类添加你自己的方法。

## 使用类别(category)向已经存在的类添加方法

如果你需要向一个已经存在的类添加方法，最简单的方法就是使用类别。

使用*@interface*关键字来定义类别，就像定义标准的Objective-C类一样，但是不要继承任何类。相反，指定类别的名字在圆括号里，如下：

	@interface ClassName (CategoryName)
 
	@end

你可以为任何类添加类别，即使你没有原是实现的源代码(比如Cocoa 或者 Cocoa Touch框架里面的类)。你在类别里面声明的任何方法，类的实例都是可以使用的，包括该类的子类。在运行时，类别添加的方法和原来的方法并没有区别。

考虑之前定义的*XYZPerson*类，有first 和 last name属性。如果你正在编写一个记录的App，你发现你需要频繁的现实last name的列表，如下：

	Appleseed, John
	Doe, Jane
	Smith, Bob
	Warwick, Kate
	
比起每次在需要显示时编写合成字符串的代码，你可以给*XYZPerson*添加类别，如下：

	#import "XYZPerson.h"
	
	@interface XYZPerson (XYZPersonNameDisplayAdditions)
		- (NSString *)lastNameFirstNameString;
	@end
	
在这个例子中，*XYZPersonNameDisplayAdditions*类别声明了一个额外的方法来返回一个需要的字符串。

类别通常声明、实现在分开的原头文件和实现文件中。比如上面声明的类别声明在名字是*XYZPerson+XYZPersonNameDisplayAdditions.h*的头文件中。

虽然你添加的任何类别，类和类的子类的实例都可以使用，但是你在使用的时候还是需要引入头文件的，否则编译器在编译的时候会警告、报错。

类别的实现如下所示：

	#import "XYZPerson+XYZPersonNameDisplayAdditions.h"
	 
	@implementation XYZPerson (XYZPersonNameDisplayAdditions)
	- (NSString *)lastNameFirstNameString {
	    return [NSString stringWithFormat:@"%@, %@", self.lastName, self.firstName];
	}
	@end
	
一旦你声明实现了类别添加的方法，这个类和子类的实例都可以使用这些方法了，就如类接口的一部分一样：

	#import "XYZPerson+XYZPersonNameDisplayAdditions.h"
	@implementation SomeObject
	- (void)someMethod {
	    XYZPerson *person = [[XYZPerson alloc] initWithFirstName:@"John"
	                                                    lastName:@"Doe"];
	    XYZShoutingPerson *shoutingPerson =
	                        [[XYZShoutingPerson alloc] initWithFirstName:@"Monica"
	                                                            lastName:@"Robinson"];
	 
	    NSLog(@"The two people are %@ and %@",
	         [person lastNameFirstNameString], [shoutingPerson lastNameFirstNameString]);
	}
	@end
	
除了向已经存在的类添加方法，你也可以使用类别将和复杂的类拆分到多个源文件中。比如，将绘制自定义用户界面的代码放在分开的实现文件中，剩下的实现如几何计算，颜色，渐变等，这些特别难复杂的。或者，你也可以为类别的方法提供不同的实现，这些都视情况而定。

类别可以用来声明实例方法或者类方法，但是并不适合声明额外的属性。在类别声明时包含属性的声明是合法的，但是在类别中并不能声明实例变量。这就意味着编译器并不会合成任何实例变量，任何属性的访问器方法。你可以在类别的实现里写自己的访问器方法，但是你并不追踪那个属性的值，除非被原始类保存。

向已经存在的类添加额外的属性(同时还有备份的实例变量)的唯一方法就是使用类的拓展。

*注意：*Cocoa 和 Cocoa Touch包含了大量主要框架类的类别。比如之前提到的字符串绘制自己的能力已经由*NSString*的*NSStringDrawing*类别向OS X提供了，包括*drawAtPoint:withAttributes:*和*drawInRect:withAttributes:*方法。对于iOS，由*UIStringDrawing*类别提供，包括*drawAtPoint:withFont:*和*drawInRect:withFont:*方法。

### 避免类别的方法名冲突

因为类别声明的方法时添加给已经存在的类的，所以对于方法名你需要非常小心。

如果类别中得方法名和原类的相同，或者与该类另外一个类别的方法名相同(或者该类的父类)，在运行时到底使用哪个方法并不确定。当你给自己的类添加类别时不会发生这样的问题，但是给Cocoa 或 Cocoa Touch框架类添加方法时就不一样了。

比如你的应用程序需要和远程的web service交互，你需要使用一种简单的方法用Base64来编码字符串的字符。给*NSString*定义一个类别，添加一个实例方法，返回一个经过Base64编码的字符串版本，所以你添加了一个叫做*base64EncodedString*的方法。

当你接入了另外一个框架，正好那个框架也定义了自己的*NSString*的类别，也有个方法叫做*base64EncodedString*。在运行时，只有一个方法会被添加到*NSString*，但是是哪一个并不确定。

还可能引起另外一个问题，比如你给Cocoa 或 Cocoa Touch框架类增加了一个便捷方法，后来框架类也发布了这个方法。比如，*NSSortDescriptor*类，描述了一组对象应该怎样排序，该类有*initWithKey:ascending:*初始化方法，但是在旧的OS X 和 iOS 版本中并没有提供对应的工厂方法。

为了方便，这个工厂方法应该叫做*sortDescriptorWithKey:ascending:*，所以你可能通过类别来提供这个工厂方法。在旧的OS X 和 iOS 版本中如你预期的工作的很好，但是随着Mac OS X 10.6 and iOS 4.0的发布，方法*sortDescriptorWithKey:ascending:*被添加到了原始类*NSSortDescriptor*中，意味着出现了命名冲突。

为了避免这些不确定的行为，最好的实践就是给框架类定义类别时给方法名加上前缀，就像给类名添加前缀一样。你可能选择和类名一样的前缀，但是使用小写，然后下划线，*NSSortDescriptor*示例，如下所示：

	@interface NSSortDescriptor (XYZAdditions)
	+ (id)xyz_sortDescriptorWithKey:(NSString *)key ascending:(BOOL)ascending;
	@end
	
意味着你能确定你的方法在运行时会被调用。歧义也消除了，因为你的代码看起来像这样：

	NSSortDescriptor *descriptor = [NSSortDescriptor xyz_sortDescriptorWithKey:@"name" ascending:YES];

## 拓展(extension)可以拓展类的内部实现

拓展和类别很相似，但是拓展只能在编译时添加给有源码的类(类和拓展同时编译)。拓展声明定义的方法在原始类的*@implementation*块里实现，所以你不能给框架类定义拓展，比如Cocoa 或 Cocoa Touch的*NSString*类。

定义拓展的语法和类别很像，如下：

	@interface ClassName ()
	 
	@end
	
因为圆括号里没有给定名字，所以拓展通常当做匿名类别。

和普通的类别不一样，拓展可以给类添加自己的属性和实例变量。如果你在类别里面定义了属性，如下：

	@interface XYZPerson ()
	@property NSObject *extraProperty;
	@end
	
在类的主要实现里面，编译器会自动合成相关的访问器方法，也包括实例变量。

如果你在类别里面添加了任何方法，这些方法必须在类的主要实现里面实现(即之前说的*@implementation*块)。

你也可以使用拓展来添加自定义的实例变量。定义在拓展接口的花括号里面：

	@interface XYZPerson () {
	    id _someCustomInstanceVariable;
	}
	...
	@end
	
### 使用拓展来隐藏私有信息

类的主要接口用来定义与它交互的方式。换言之，即是一个公共的接口。

类的拓展经常用来添加实现文件需要的私有方法和属性。比如，在接口文件里面定义了一个*readonly*的属性，但是在拓展里面定义的是*readwrite*，为了能在方法内部直接修改属性的值。

比如，*XYZPerson*类增加了一个*uniqueIdentifier*属性，用来保存社保账号。

通常需要大量的文书工作才能获得一个唯一的identifier，所以*XYZPerson*接口定义了*readonly*的属性，然后提供了请求赋值的方法，如下：

	@interface XYZPerson : NSObject
	...
	@property (readonly) NSString *uniqueIdentifier;
	- (void)assignUniqueIdentifier;
	@end
	
这意味着别的对象不可能直接修改*uniqueIdentifier*。如果一个人还没有*uniqueIdentifier*，那么必须通过调用*assignUniqueIdentifier*方法来赋值一个。

为了能在*XYZPerson*内部直接修改，所以在类的拓展里面重新定义这个属性就很合理：

	@interface XYZPerson ()
	@property (readwrite) NSString *uniqueIdentifier;
	@end
	 
	@implementation XYZPerson
	...
	@end
	
*注意：readwrite*是可选的，因为它使默认的。为了更好的表达意图最好还是写上。

这就意味着编译器也将合成*setter*方法，所以*XYZPerson*实现文件里面的任何方法都可以直接修改属性的值了，不管是访问器方法还是点语法。

通过在*XYZPerson*类的实现文件里面定义拓展，这些信息是*XYZPerson*类的私有信息。如果其他类型的对象视图设置这个属性，编译器将会生成错误。

*注意：*如上所示的实例，通过添加拓展，重新定义了*uniqueIdentifier*属性为*readwrite*，所以在运行时所有的*XYZPerson*对象都会有*setUniqueIdentifier:*方法，不管其他源文件是否意识到了个拓展。

如果在依他的源文件里面视图调用私有方法或者设置*readonly*的属性，编译器会抱怨，但是可以避免编译器错误，利用动态运行时的其他方法来调用这些方法，比如*NSObject*提供的*performSelector:...*方法。

如果你视图让私有方法和属性能被其他的类选择，比如同框架里面相关的其他类，你也可以在分开的头文件中定义拓展，然后在需要的源文件中引入进来。对一个类来说有两个头文件并不常见，比如，*XYZPerson.h* 和 *XYZPersonPrivate.h*。当你发布框架的时候，你只发布公共的*XYZPerson.h*头文件。

## 其他自定义类的替代方案

类别和拓展可以轻易的向已经存在的类直接添加行为，但是有时并不是最优的选择。

面向对象编程的一条主要原则就是编写可重用的代码，这就意味着在大多数情形下类应该是可重用的，不管是否可能。如果你创建了一个视图类来描述一个显示在屏幕上的对象的信息，比如，就应该考虑在多种情形下的可用性。

比起每次通过硬编码的方式来决定视图的布局和内容，另一种可替代的方法就是指定这些方法给子类重写。尽管这些并没有让这个类有多容易重用，你仍然需要每次在使用的时候子类化。

另一种对类来说的可替代方式就是使用delegate对象。任何可能限制可重用性的决定都委托给另外一个对象，让这个对象在运行时的时候决定。很常见的一个列子就是table view类(NSTableView for OS X and UITableView for iOS)。为了让table view更通用，让另一个对象在运行时的时候再决定table view的内容。

### 直接和Objective-C运行时交互

Objective-C通过运行时系统来提供动态特性。

很多决定都不是在编译时决定的，比如当消息发送的时候调用哪一个方法，而是在应用运行的时候才决定的。Objective-C不仅仅是编译成了底层的机器语言。相反，它要求一套运行时系统来执行这些代码。

可以直接和这套运行时系统进行交互，比如给对象增加关联引用。不像拓展，关联引用并不影响原始类的声明和实现，意味着你可以使用它来关联你没有源码的框架类。

一个关联引用连接一个对象到另一个对象，与属性或者实例变量很相似。运行时系统详见[Objective-C Runtime Programming Guide](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Introduction/Introduction.html#//apple_ref/doc/uid/TP40008048)

# 使用协议

在现实世界中，人们在正式场合处理特定事情的时候需要遵循严格的操作程序。比如，执法人员，进行询问或者手机证据的时候要求遵守协议。

能够定义一组行为，期望在给定的情形下对象具有这样的行为，在面向对象编程的世界里很重要。比如，tableview期望能够跟datasource通信找出它需要显示的内容。这就意味着datasource对象能够响应一组特定的、由tableview发送的消息。

datasource可以使任何类的实例，比如viewcontroller或者是一个仅仅继承自*NSObject*的类。为了让tableview知道这个对象是否适合作为datasource，重要的是声明这个对象实现了协议必须得方法。

Objective-C允许你定义协议，也就是定义了一组期望在特定情形下使用的方法集合。本章将描述定义协议的语法，以及适配协议，也就是类必须实现协议里面必须得方法。

## 协议定义了消息合约

类的接口定义了和类关联的属性和方法。一个协议，对比来说，是用来声明独立于任何指定类的方法和属性。语法如下：

	@protocol ProtocolName
	// list of methods and properties
	@end
	
协议可以声明实例方法、类方法，也包括属性。

假设我们自定义了一个视图类来用显示饼图，如图Figure 5-1所示

Figure 5-1  A Custom Pie Chart View
![alt text](/img/ProgrammingwithObjectiveC/piechartsim.png)

为了能让这个类尽可能的重用，所有的饼图的数据信息都应该交由另外一个对象来决定，一个datasource对象。这就意味着这个视图类的多个实例通过和不同的datasource通信可以显示不同的信息。

这个视图类需要的信息至少应该包括饼图应该有几段，每段的大小，以及每段的标题。这个饼图的datasource协议可能如下：

	@protocol XYZPieChartViewDataSource
	- (NSUInteger)numberOfSegments;
	- (CGFloat)sizeOfSegmentAtIndex:(NSUInteger)segmentIndex;
	- (NSString *)titleForSegmentAtIndex:(NSUInteger)segmentIndex;
	@end
	
*注意：*这个协议使用*NSUInteger*值来用来表示无符号的整型值。下章节将详细讨论该类型。

这个饼图类的借口需要一个属性来保持datasource对象。这个对象可以是任何类，所以这个属性的类型应该是*id*。对这个对象唯一了解的事情就是适配了相关的协议。

可能像如下定义饼图类的datasource属性：

	@interface XYZPieChartView : UIView
	@property (weak) id <XYZPieChartViewDataSource> dataSource;
	...
	@end
	
Objective-C使用尖括号来指示适配的协议。该示例定义了一个*weak*的属性，一个*id*类型，适配了*XYZPieChartViewDataSource*协议。

*注意：*delegate 和 datasource属性通常被标记为*weak*，原因参考前面的内容，避免强引用环。

为了保证定义了适配协议的属性的一致性，如果你设置给这个属性的对象没有适配协议，将会收到编译器的警告。即使这个属性的类型是*id*。这个对象是*UIViewController*或者*NSObject*并没有关系。重要的是适配了那个协议，意味着饼图类可以向它获取信息。

### 协议可以包含可选方法

默认情况下，定义在协议里的方法都是必须得。意味着任何适配该协议的类都应该实现这些方法。

也可以指定可选的方法。这些方法只在类需要的时候才实现。

比如，你将决定饼图标题的方法定义为了可选。如果datasource对象没有实现这个*titleForSegmentAtIndex:*方法，那么就没有标题显示在视图里。

你可以使用*@optional*来定义可选的方法，如下所示：

	@protocol XYZPieChartViewDataSource
	- (NSUInteger)numberOfSegments;
	- (CGFloat)sizeOfSegmentAtIndex:(NSUInteger)segmentIndex;
	@optional
	- (NSString *)titleForSegmentAtIndex:(NSUInteger)segmentIndex;
	@end
	
如上所示，只有*titleForSegmentAtIndex:*方法被标记为可选。其他的方法时必须得。

*@optional*指令将应用于它之后的所有方法，直到协议定义结束，或者直到另一个指令出现，比如*@required*。你可能定义更多的方法，如下所示：

	@protocol XYZPieChartViewDataSource
	- (NSUInteger)numberOfSegments;
	- (CGFloat)sizeOfSegmentAtIndex:(NSUInteger)segmentIndex;
	@optional
	- (NSString *)titleForSegmentAtIndex:(NSUInteger)segmentIndex;
	- (BOOL)shouldExplodeSegmentAtIndex:(NSUInteger)segmentIndex;
	@required
	- (UIColor *)colorForSegmentAtIndex:(NSUInteger)segmentIndex;
	@end
	
这个例子定义了3个必需方法、2个可选方法的协议。

#### 在运行时检查可选方法是否实现

如果一个方法被标记为了可选，你必须在调用这个方法之前检查这个对象是否实现了这个方法。

比如，可以像如下一样检查分段标题的方法：

	NSString *thisSegmentTitle;
    if ([self.dataSource respondsToSelector:@selector(titleForSegmentAtIndex:)]) {
        thisSegmentTitle = [self.dataSource titleForSegmentAtIndex:index];
    }
    
这个*respondsToSelector:*方法使用了一个*selector*，即方法的标识。你可以使用*@selector()*指令指定方法的名字。

如果datasource在上面的列子中实现了方法，标题方法就会被使用，没有标题就会使nil。

*记住：*本地对象变量总是自动被初始化为nil。

如果你试图调用*respondsToSelector:*，如上面所示，你将会得到编译器的错误*there’s no known instance method for it*。一旦你用协议限制了一个*id*类型，所有静态类型检查就随之而来；如果你调用没有定义在协议里面的方法，那么就会得到错误。避免这种错误的方法就是让自定义的协议适配*NSObject*协议。

### 协议可以继承其他的协议

如Objective-C类可以继承其他类一样，你也可以指定一个协议适配另一个协议。

比如，自定义协议的最佳实践就是适配*NSObject*协议(*NSObject*的一些行为从类接口分到了协议里面，然后*NSObject*类适配了*NSObject*协议)。

通过指定你自己的协议适配*NSObject*协议，意味着任何适配你协议的对象也要提供*NSObject*协议方法的实现。因为你使用的是*NSObject*的子类，所以你不需要担心自己实现*NSObject*协议的方法。协议的适配很有用，然后，仅限上述所描述的情形中。

指定一个协议适配另一个，如下所示：

	@protocol MyProtocol <NSObject>
	...
	@end
	
在这个示例中，适配了*MyProtocol*协议的对象同样也适配了*NSObject*协议。

## 适配协议

类适配协议的语法如下：

	@interface MyClass : NSObject <MyProtocol>
	...
	@end
	
这意味着*MyClass*类的实例不仅会响应定义在接口里面的方法，也包括提供了现实的*MyProtocol*协议的方法。没有必要再在接口文件里面定义协议的方法，协议的适配就足够了。

*注意：*编译器不会自动合成适配协议里面的属性。

如果你需要你的类适配多个协议，语法如下：

	@interface MyClass : NSObject <MyProtocol, AnotherProtocol, YetAnotherProtocol>
	...
	@end

*提示：*如果你发现一个类适配了大量的协议，也许这就标志着你需要重构这个类，将这个类分解成多个类似的类，每个类都定义了明确的责任。

同于新手开发者来说一个易犯的共同错误就是使用一个单一的程序代理类来提供大多数的应用程序功能(比如管理底层的数据结构，给多个用户界面提供数据，响应手势，处理其他的用户交互)。随着复杂性的增加，这个类变得难以维护。

一旦你适配了一个协议，这个类至少应该提供协议必须方法的实现，包括你选择的可选方法。如果你没有实现所有的必须方法，你将会收到编译器的警告。

*注意：*定义在协议里面的方法。在实现时方法的名称和参数类型必须和协议里面定义的相匹配。

### Cocoa 和 Cocoa Touch定义了大量的协议

Cocoa 和 Cocoa Touch 对象在大量不同的情形下都是用了协议。比如tableview类使用datasource对象来提供需要的信息。定义了datasource协议，如*XYZPieChartViewDataSource*协议一样。tableview类也允许你设置delegate对象，必须适配相关的*NSTableViewDelegate*和*UITableViewDelegate*协议。这个delegate负责处理用户的交互，或者自定义某些条目的显示。

一些协议用来指示两个类之间是没有层级关系的。比起关联到指定的类，一些协议在Cocoa 或者 Cocoa Touch里面作为通信机制被多个不同的不相关的类适配。

比如，许多框架的模型对象(比如容器类像*NSArray*和*NSDictionary*)支持*NSCoding*协议，意味着它们可以编码解码它们的属性归档为原始数据。*NSCoding*协议可以将任何适配了的对象很容易的归档到硬盘。

一些Objective-C语言层面的功能也依赖于协议。为了能枚举容器，比如，容器必须适配*NSFastEnumeration*协议。另外，一些对象可以被复制，如前所述的属性的*copy*。任何你尝试复制的对象都需要适配*NSCopying*协议，否则会导致运行时异常。

## 协议可以匿名使用

协议也可以用在当一个类的对象不知道的情形中，或者这个对象需要影藏。

比如，一个框架的开发者选择没有公开类的接口。由于不知道这个类的名字，所以框架的使用者不能直接创建这个类的实例。框架里面的其他对象被设计成返回一个已经准备好得实例，如下：

	id utility = [frameworkObject anonymousUtility];
	
为了让这个对象有用，框架的开发者可以发布一个协议。即使原始类的接口并没有提供，意味着这个类是匿名的，但是这个对象仍然在一定程度上可以使用：

	id <XYZFrameworkUtility> utility = [frameworkObject anonymousUtility];
	
如果你使用了Core Data框架来编写iOS的应用程序，比如，你很可能用到了*NSFetchedResultsController*类。这个类帮助*UITableView*的datasource对象来存储数据，更容易提供信息。

如果你tableview的内容被分解成多个sections，你可以向fetched results controller请求相关section的信息。比起返回一个包含这些信息的指定的类，*NSFetchedResultsController*返回了一个适配了*NSFetchedResultsSectionInfo*协议的匿名对象。这意味着你仍然可以查询你需要的信息，比如行数等。

	NSInteger sectionNumber = ...
    id <NSFetchedResultsSectionInfo> sectionInfo =
            [self.fetchedResultsController.sections objectAtIndex:sectionNumber];
    NSInteger numberOfRowsInSection = [sectionInfo numberOfObjects];
    
尽管你不知道*sectionInfo*对象的类型，但是*NSFetchedResultsSectionInfo*协议指示它可以响应*numberOfObjects*消息。

# 值和容器

尽管Objective-C是一门面向对象的编程语言，是C语言的超集，这就意味着你可以在Objective-C代码里面使用所有的C语言的类型，诸如*int*, *float* 和 *char*。除此之外，Cocoa 和 Cocoa Touch的应用程序中还有其他的类型，比如*NSInteger*, *NSUInteger* 和 *CGFloat*，根据不同的目标平台他们有不同的定义。

标量类型通常用在不需要对象来表示一个值的时候。字符串通常使用*NSString*类的额实例来表示，数值通常保存在本地变量或者属性中。

Objective-C中完全可以定义C风格的数组，但是你会发现容器在Cocoa 和 Cocoa Touch应用程序通常使用诸如*NSArray* 或者 *NSDictionary*来表示。这些容器类只能用来包含Objective-C的对象，也就是说你必须在使用诸如*NSValue*，*NSNumber* 或者 *NSString*这些对象来表示值，然后再添加到容器。

前一章节使用了*NSString*类，有使用初始化方法或者工厂方法创建字符串，也使用了*@"string"*字面量创建字符串对象。本章节将解释怎样创建*NSValue* 和 *NSNumber*对象，以及使用字面量创建。

## C语言的简单类型在Objective-C中可用

C语言类型：

	int someInteger = 42;
    float someFloatingPointNumber = 3.1415;
    double someDoublePrecisionFloatingPointNumber = 6.02214199e23;
    
C操作符：

	int someInteger = 42;
    someInteger++;            // someInteger == 43
 
    int anotherInteger = 64;
    anotherInteger--;         // anotherInteger == 63
 
    anotherInteger *= 2;      // anotherInteger == 126
    
在Objective-C的属性中使用值类型：

	@interface XYZCalculator : NSObject
	@property double currentValue;
	@end
	
在使用点语法访问属性时也可使用C操作符：

	@implementation XYZCalculator
	- (void)increment {
	    self.currentValue++;
	}
	- (void)decrement {
	    self.currentValue--;
	}
	- (void)multiplyBy:(double)factor {
	    self.currentValue *= factor;
	}
	@end
	
点语法实际就是访问器方法的包装，所以在上面的示例中，会先使用访问器方法的get方法得到值之后再操作，然后再使用set方法赋值结果。

### Objective-C定义了额外的简单类型

Objective-C中定义*BOOL*类型来表示布尔值，即*YES* 或者 *NO*。*YES*表示逻辑真和1，*NO*表示逻辑假和0.

在Cocoa 和 Cocoa Touch对象的许多方法的参数中都使用了特别的数值类型，如*NSInteger* 或者 *CGFloat*。

比如，*NSTableViewDataSource* 和 *UITableViewDataSource*的协议都需要返回现实行数的方法：

	@protocol NSTableViewDataSource <NSObject>
	- (NSInteger)numberOfRowsInTableView:(NSTableView *)tableView;
	...
	@end
	
这些类型，像*NSInteger* 和 *NSUInteger*，根据不同的平台架构有不同的定义。当在32位环境构建时，它们分别是32位的有符号和无符号的整型，当在64位的环境构件时，它们分别是64位的有符号和无符号的整型。

对于本地变量，比如循环的计数，可以使用基本的C类型如果你知道值是在标准范围内。

### 结构体能拥有简单值

一些Cocoa 和 Cocoa Touch API使用C结构体来保存它们的值。比如字符串对象可以请求子串的范围，如下：

	NSString *mainString = @"This is a long string";
    NSRange substringRange = [mainString rangeOfString:@"long"];
    
*NSRange*结构体保存范围的起始地址和长度。在上面的实例中*substringRange*的range就是*{10,4}*，10就是子串开始的位置，从0开始数，4就是子串的长度。

类似的，如果你需要编写自己绘制的代码，你需要和*Quartz*交互，需要处理一些基于*CGFloat*类型的结构体，比如*NSPoint* 和 *NSSize* 在OS X中， *CGPoint* 和 *CGSize* 在iOS中。同样*CGFloat*也与不同目标平台的架构有关。

更多关于Quartz 2D 绘引擎的信息，参见[Quartz 2D Programming Guide](https://developer.apple.com/library/mac/documentation/GraphicsImaging/Conceptual/drawingwithquartz2d/Introduction/Introduction.html#//apple_ref/doc/uid/TP30001066)

## 对象能够表示值

如果你需要将一个值类型像对象一样表示，比如添加到容器当中，你可以使用Cocoa 和 Cocoa Touch提供的基本值的类。

### 字符串由NSString类的实例表示

*NSString*用来表示字符串，像*Hello World*。有很多种方法来创建*NSString*对象，包括初始化器方法，工厂方法，或者字面量：

	NSString *firstString = [[NSString alloc] initWithCString:"Hello World!"
                                                     encoding:NSUTF8StringEncoding];
    NSString *secondString = [NSString stringWithCString:"Hello World!"
                                                encoding:NSUTF8StringEncoding];
    NSString *thirdString = @"Hello World!";
    
如上所示，这三个示例都高效的完成了相同的事情，根据提供的字符创建了字符串对象。

*NSString*类是不可变的，这意味着一旦在创建的时候设置了随后就不能被改变。如果你需要表示一个不同的字符串，那么你必须创建一个新的对象，如下：

	NSString *name = @"John";
    name = [name stringByAppendingString:@"ny"];    // returns a new string object
    
*NSMutableString*类是可变的，是*NSString*的子类，允许你在运行时通过*appendString:* 或者 *appendFormat:*方法来改变，如下：

	NSMutableString *name = [NSMutableString stringWithString:@"John"];
    [name appendString:@"ny"];   // same object, but now represents "Johnny"
    
#### 格式化字符串从其他值和对象创建字符串

如果你需要创建一个包含变量值的字符串，你可以使用格式化字符串。如下：

	int magicNumber = ...
    NSString *magicString = [NSString stringWithFormat:@"The magic number is %i", magicNumber];

### 数字由NSNumber类的实例表示

*NSNumber*用来表示C语言的简单值类型，包括*char*，*double*，*float*，*int*，*long*，*short*，以及*unsigned*和Objective-C的布尔类型*BOOL*。

如同*NSString*，你有大量的选择可以创建*NSNumber*的实例，如下：

	NSNumber *magicNumber = [[NSNumber alloc] initWithInt:42];
    NSNumber *unsignedNumber = [[NSNumber alloc] initWithUnsignedInt:42u];
    NSNumber *longNumber = [[NSNumber alloc] initWithLong:42l];
 
    NSNumber *boolNumber = [[NSNumber alloc] initWithBOOL:YES];
 
    NSNumber *simpleFloat = [NSNumber numberWithFloat:3.14f];
    NSNumber *betterDouble = [NSNumber numberWithDouble:3.1415926535];
 
    NSNumber *someChar = [NSNumber numberWithChar:'T'];
    
同时，也可以使用字面量来创建*NSNumber*实例，如下：

	NSNumber *magicNumber = @42;
    NSNumber *unsignedNumber = @42u;
    NSNumber *longNumber = @42l;
 
    NSNumber *boolNumber = @YES;
 
    NSNumber *simpleFloat = @3.14f;
    NSNumber *betterDouble = @3.1415926535;
 
    NSNumber *someChar = @'T';
    
一旦你创建了*NSNumber*的实例，就可以通过访问器方法来请求值类型的值。如下：

	int scalarMagic = [magicNumber intValue];
    unsigned int scalarUnsigned = [unsignedNumber unsignedIntValue];
    long scalarLong = [longNumber longValue];
 
    BOOL scalarBool = [boolNumber boolValue];
 
    float scalarSimpleFloat = [simpleFloat floatValue];
    double scalarBetterDouble = [betterDouble doubleValue];
 
    char scalarChar = [someChar charValue];
    
*NSNumber*类也提供了额外处理Objective-C简单类型的方法。如果你需要创建表示*NSInteger* 和 *NSUInteger*的对象，比如，请确保你用对了方法：

	NSInteger anInteger = 64;
    NSUInteger anUnsignedInteger = 100;
 
    NSNumber *firstInteger = [[NSNumber alloc] initWithInteger:anInteger];
    NSNumber *secondInteger = [NSNumber numberWithUnsignedInteger:anUnsignedInteger];
 
    NSInteger integerCheck = [firstInteger integerValue];
    NSUInteger unsignedCheck = [secondInteger unsignedIntegerValue];
    
所有的*NSNumber*实例都是不可变的，也没有可变的子类；如果你需要不同的*NSNumber*，请创建一个新的*NSNumber*实例。

*注意：**NSNumber*实际上是一个类的族群。这就意味着当你在运行时创建实例的时候，你会得到一个合适的子类来表示值。就当做是*NSNumber*的实例就好了。

### 用NSValue类的实例表示其他值

*NSNumber*是*NSValue*类的子类，提供了一个封装了单一值或者数据项的对象。除此基本的C的值类型之外，*NSValue*也能用来表示指针和结构体。

*NSValue*提供了根据给定结构体创建实例的工厂方法，比如，*NSRange*，如之前的例子：

	NSString *mainString = @"This is a long string";
    NSRange substringRange = [mainString rangeOfString:@"long"];
    NSValue *rangeValue = [NSValue valueWithRange:substringRange];
    
你也可以根据自定义的结构体来创建*NSValue*对象。如果你特别需要结构体而不是对象来存储信息：

	typedef struct {
	    int i;
	    float f;
	} MyIntegerFloatStruct;
	
你可以通过提供结构体的指针和编码的Objective-C类型来创建*NSValue*的实例。这个*@encode()*便以其指令是用来创建正确的Objective-C类型的，如下：

	struct MyIntegerFloatStruct aStruct;
    aStruct.i = 42;
    aStruct.f = 3.14;
 
    NSValue *structValue = [NSValue value:&aStruct
                             withObjCType:@encode(MyIntegerFloatStruct)];
                             
引用运算符(&)是用来得到*aStruct*的地址的，提供给*value*参数。

## 大多数容器都是对象类型

尽管可以使用C风格的数组来保存值类型或者对象的指针，大部分Objective-C代码里的容器都是Cocoa 和 Cocoa Touch框架里的容器类的实例，比如*NSArray*， *NSSet* 和 *NSDictionary*。

这些类用来管理一组对象，意味着你添加到容器里面的任何项都必须是Objective-C类的实例。如果你需要添加简单值，你必须先创建合适的*NSNumber* 或者 *NSValue*实例来表示。

容器类并不会创建对象的副本，而是使用强引用来保持容器内的对象。这就意味着，只要容器类还在，任何添加到容器里面的对象都会存在。

除此之外，每个Cocoa 和 Cocoa Touch的容器类都能很容易的执行特定的任务，比如枚举，访问特定的项，或者找出特定的对象是否是容器类的一部分。

*NSArray*， *NSSet*，和 *NSDictionary*是不可变的，他们也有可变的子类版本。

### 数组是有序容器

*NSArray*类用来表示有序的容器。唯一的要求就是里面的每一项都是Objective-C对象，并没有要求添加到里面的项是要相同类的实例。

为了维护顺序，元素都从下0开始存储，如果6 - 1所示：

Figure 6-1  An Array of Objective-C Objects
![alt text](/img/ProgrammingwithObjectiveC/orderedarrayofobjects.png)

#### 创建数组

你可以通过初始化方法，工厂方法，字面量来创建数组。

根据对象的数量不同有很多初始化方法和工厂方法，如下：

	+ (id)arrayWithObject:(id)anObject;
	+ (id)arrayWithObjects:(id)firstObject, ...;
	- (id)initWithObjects:(id)firstObject, ...;
	
*arrayWithObjects:* 和 *initWithObjects:*方法都以nil结束，可变的参数个数，意味着你必须把nil当做最后一个值，如下：

	 NSArray *someArray = [NSArray arrayWithObjects:someObject, someString, someNumber, someValue, nil];
	 
这个实例创建了如图6 - 1所示的的数组。第一个对象，*someObject*，数组下标为0；最后一个对象，*someValue*，下标为3.

如果提供的一个值是nil，完全可能会截断数组，如下：

	id firstObject = @"someString";
    id secondObject = nil;
    id thirdObject = @"anotherString";
    NSArray *someArray = [NSArray arrayWithObjects:firstObject, secondObject, thirdObject, nil];
    
在这个示例中，*someArray*只会包含*firstObject*对象，因为*nil secondObject*会被当做列表的最后一项。

#### 字面量创建数组

可以使用Objective-C字面量来创建数组，如下所示：

	NSArray *someArray = @[firstObject, secondObject, thirdObject];
	
当使用字面量创建数组的时候你不需要使用nil来结束列表，事实上nil在这里是不合法的值。如果你试图执行下面的代码就会导致运行时异常，比如：

	id firstObject = @"someString";
    id secondObject = nil;
    NSArray *someArray = @[firstObject, secondObject];
    // exception: "attempt to insert nil object"
    
如果你需要在容器类中表示nil值，你应该使用*NSNull*单例类。

#### 查询数组对象

一旦你创建了一个数组，你就可以查询信息，或者判断数组是否包含给定的项：

	NSUInteger numberOfItems = [someArray count];
 
    if ([someArray containsObject:someString]) {
        ...
    }
    
你也可以查找给定下标的数组项。如果你运行时请求了非法下标的数组项，你就会导致越界异常，所以你需要始终检查数组项的数量：

	if ([someArray count] > 0) {
        NSLog(@"First item is: %@", [someArray objectAtIndex:0]);
    }
    
这个示例首先检查数组项的数量是否大于0。如果大于0，就打印出第一个数组项。

你也可以使用下标语法来替代*objectAtIndex:*，跟C语言的数组一样，如下：

	if ([someArray count] > 0) {
        NSLog(@"First item is: %@", someArray[0]);
    }
    
#### 数组排序

*NSArray*类同样也提供了大量的排序的方。因为*NSArray*是不可变的，所以所有这些方法都会返回一个新的排序好的数组对象。

比如，你又一个字符串的数组，你可以在每一个字符串对象上调用*compare:*方法来排序，如下所示;

	NSArray *unsortedStrings = @[@"gammaString", @"alphaString", @"betaString"];
    NSArray *sortedStrings =
                 [unsortedStrings sortedArrayUsingSelector:@selector(compare:)];
                 
#### 可变性

尽管*NSArray*类是不可变的。如果你将一个可变的字符串添加到不可变的数组，如下：

	NSMutableString *mutableString = [NSMutableString stringWithString:@"Hello"];
	NSArray *immutableArray = @[mutableString];
	
没有什么能够阻止你改变这个字符串：

	if ([immutableArray count] > 0) {
        id string = immutableArray[0];
        if ([string isKindOfClass:[NSMutableString class]]) {
            [string appendString:@" World!"];
        }
    }
    
如果你需要在创建数组后添加或者删除对象，你需要使用*NSMutableArray*，有大量添加或者删除对象的方法：

	NSMutableArray *mutableArray = [NSMutableArray array];
    [mutableArray addObject:@"gamma"];
    [mutableArray addObject:@"alpha"];
    [mutableArray addObject:@"beta"];
 
    [mutableArray replaceObjectAtIndex:0 withObject:@"epsilon"];
    
这个示例创建了一个以*@"epsilon"*，*@"alpha"*，*@"beta"*结束的数组。

同样也可以给可变数组排序，而不需要创建第二个新的数组，如下：

	[mutableArray sortUsingSelector:@selector(caseInsensitiveCompare:)];
	
在这个示例中，将会按照升序排列。*@"alpha"*，*@"beta"*，*@"epsilon"*。

### 集合是无序的容器

一个集合跟数组很像，但是保存的是无序的不同的对象，如图6 - 2所示。

Figure 6-2  A Set of Objects
![alt text](/img/ProgrammingwithObjectiveC/unorderedsetofobjects.png)

因为集合是无序的，所以比数组有了性能上得提升。

*NSSet*类是不可变的，所以在初始化的时候就需要指定，同样的你也可以使用初始化方法或者工厂方法：

	NSSet *simpleSet =
      [NSSet setWithObjects:@"Hello, World!", @42, aValue, anObject, nil];
      
和*NSArray*一样，*initWithObjects:* 和 *setWithObjects:*方法都以nil结束，可变的参数数量。*NSSet*的子类*NSMutableSet*是可变的。

对于每个独立的对象集合只会保存一份引用，即使你添加这个对象多次：

	NSNumber *number = @42;
    NSSet *numberSet =
      [NSSet setWithObjects:number, number, number, number, nil];
    // numberSet only contains one object

### 字典是键值对的容器

比起只是有序或者无序的存储单一的对象，*NSDictionary*通过给定的键存储对应的对象，键可以用来查询。

最佳实践是使用字符串对象作为键，如图6 - 3所示。

Figure 6-3  A Dictionary of Objects
![alt text](/img/ProgrammingwithObjectiveC/dictionaryofobjects.png)

*注意：*完全可以使用其他的对象作为键，重要的是需要意识到*NSDictionary*会复制键，所以必须支持*NSCopying*协议。

如果你需要使用Key-Value Coding(键值编码)，然而，如[Key-Value Coding Programming Guide](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/KeyValueCoding/Articles/KeyValueCoding.html#//apple_ref/doc/uid/10000107i)描述的一样，你必须使用字符串作为*NSDictionary*的键。

#### 创建NSDictionary

你可以用初始化方法或者工厂方法来创建字典，如下：

	NSDictionary *dictionary = [NSDictionary dictionaryWithObjectsAndKeys:
                   someObject, @"anObject",
             @"Hello, World!", @"helloString",
                          @42, @"magicNumber",
                    someValue, @"aValue",
                             nil];
                             
注意*dictionaryWithObjectsAndKeys:* 和 *initWithObjectsAndKeys:*方法，每个对象都在键之后指定，同样的都以nil结束。

#### 字面量

你也可以使用字面量创建字典，如下：

	NSDictionary *dictionary = @{
                  @"anObject" : someObject,
               @"helloString" : @"Hello, World!",
               @"magicNumber" : @42,
                    @"aValue" : someValue
    };
    
注意键在对象之前指定，而且也不以nil结束。

#### 查询字典

一旦你创建了字典，你可以通过给定的键来查询值，如下：

	NSNumber *storedNumber = [dictionary objectForKey:@"magicNumber"];
	
如果没有找到值，那么*objectForKey:*方法就会返回nil。

你也可以使用下标语法来替代*objectForKey:*，如下：

	NSNumber *storedNumber = dictionary[@"magicNumber"];
	
#### 可变性

如果你需要在创建一个字典以后添加或者删除键值对，你需要使用*NSDictionary*的子类*NSMutableDictionary*，如下：

	[dictionary setObject:@"another string" forKey:@"secondString"];
    [dictionary removeObjectForKey:@"anObject"];

### 使用NSNull表示空

不能将nil添加到容器类中，因为nil在Objective-C中意味着没有对象。如果你需要在容器类中表示没有对象，你可以使用*NSNull*类：

	NSArray *array = @[ @"string", @42, [NSNull null] ];
	
*NSNull*类是一个单例类，意味着*null*方法始终会返回同一个实例。意味着你可以检查数组中得一个对象是否是*NSNull*的实例：

	for (id object in array) {
        if (object == [NSNull null]) {
            NSLog(@"Found a null object");
        }
    }

## 使用容器来持久化你的对象图

*NSArray*和*NSDictionary*类都能轻易的将自己的内容直接写入到硬盘，如下：

	NSURL *fileURL = ...
    NSArray *array = @[@"first", @"second", @"third"];
 
    BOOL success = [array writeToURL:fileURL atomically:YES];
    if (!success) {
        // an error occured...
    }
    
如果包含的每一个对象都是属性列表的类型(*NSArray*，*NSDictionary*，*NSString*，*NSData*，*NSDate* and *NSNumber*)，那么就可以从硬盘上从新构建整个层级，如下：

	NSURL *fileURL = ...
    NSArray *array = [NSArray arrayWithContentsOfURL:fileURL];
    if (!array) {
        // an error occurred...
    }
    
更所属性列表信息查看[Property List Programming Guide](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/PropertyLists/Introduction/Introduction.html#//apple_ref/doc/uid/10000048i)。

如果你需要持久化其他的对象，而不是上述描述的标准的属性列表的对象，你可以使用归档对象，比如*NSKeyedArchiver*，用来创建集合对象的归档。

对于创建归档的唯一条件就是每一个对象必须支持*NSCoding*协议。这就意味着每一个对象必须知道怎么编码自己来归档(通过实现*encodeWithCoder:*方法)以及如何从已经存在的归档里面解码自己(通过实现*initWithCoder:*方法)。

*NSArray*，*NSSet* 和 *NSDictionary*以及它们的可变子类，都支持*NSCoding*协议，意味着你可以使用归档对象来持久化拥有复杂层级的对象。如果你使用Interface Builder来布局视图，比如，*nib*文件其实就是你可视化编辑对象的归档。在运行时，使用相关的类解档成为了对象层级。

更多归档信息，参见[Archives and Serializations Programming Guide](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/Archiving/Archiving.html#//apple_ref/doc/uid/10000047i)。

## 使用高效的容器枚举技术

Objective-C 和 Cocoa or Cocoa Touch提供了大量的方法来枚举容器的内容。尽管可以使用传统的C循环来遍历内容，如下：

	int count = [array count];
    for (int index = 0; index < count; index++) {
        id eachObject = [array objectAtIndex:index];
        ...
    }
    
最佳实践是本章将会描述的另一个方法。

### 快速枚举容器

许多容器类都适配了*NSFastEnumeration*协议，包括*NSArray*，*NSSet* 和 *NSDictionary*。这意味着你可以使用快速枚举，一个Objective-C 语言层面的功能。

语法如下：

	for (<Type> <variable> in <collection>) {
        ...
    }
    
比如，你可能想使用快速枚举来打印数组里面的对象，如下：

	for (id eachObject in array) {
        NSLog(@"Object: %@", eachObject);
    }
    
*eachObject*在每一轮循环被自动设置，所以每个对象都会有个log。

如果你对字典使用快速枚举，你遍历字典的键，如下:

	for (NSString *eachKey in dictionary) {
        id object = dictionary[eachKey];
        NSLog(@"Object: %@ for key: %@", object, eachKey);
    }
    
快速枚举的行为就很像C语言的循环，所以你可以使用*break*关键字来打断迭代，或者使用*continue*跳到下轮循环。

如果你在枚举有顺序的容器，这枚举过程也将是有序的。比如枚举一个数组，最先传递过来的对象就是下标为0的，第二个就是下标为1的，等等。如果你需要追踪当前的下标，在每次迭代的时候计数即可：

	int index = 0;
    for (id eachObject in array) {
        NSLog(@"Object at index %i is: %@", index, eachObject);
        index++;
    }
    
在快速枚举的时候你不恩能够改变容器，即使这个容器是可变的，如果你尝试添加或者删除容器里的对象，将会导致运行时异常。

### 大部分容器也支持枚举器

你也可以是使用*NSEnumerator*对象来枚举大部分Cocoa 和 Cocoa Touch的容器。

比如，你可以向数组请求一个*objectEnumerator*或者*reverseObjectEnumerator*。你可以在快速枚举中使用这些对象，如下：

	for (id eachObject in [array reverseObjectEnumerator]) {
        ...
    }
    
在这个示例中，将会按照数组的逆序来迭代循环数组，所以第一个对象就是数组下标为0的对象，等等。

你也可以通过重复调用枚举器的nextObject方法来迭代，如下：

	id eachObject;
    while ( (eachObject = [enumerator nextObject]) ) {
        NSLog(@"Current object is: %@", eachObject);
    }
    
在这个示例中，*while*循环设置eachObject来保存循环传递过来的对象。当再没有对象传递过来时，*eachObject*方法将会返回nil，将会逻辑计算为假，所以停止了循环。

*注意：*因为使用C的赋值操作符(=)当做等等操作符(==)是常见的编程错误，当你在条件分支或者循环的时候设置变量编译器将会警告你，如下：

	if (someVariable = YES) {
	    ...
	}
	
如果你真的是要指定一个变量(所有赋值的逻辑值都将是左边的最终值)，你可以将赋值操作放在圆括号里，如下：

	if ( (someVariable = YES) ) {
	    ...
	}

快速枚举，在枚举的时候你不能改变容器。从名字来看，快速枚举比手动使用枚举器要快。

### 大多数容器也支持基于Block的枚举

你也可以使用blocks来枚举*NSArra*，*NSSet* 和 *NSDictionary*。下章节涵盖了Blocks的细节。

# 使用Blocks

一个Objective-C的类定义了对象的数据和相关的行为。有时，只需要代表一个单一的任务或者一组行为单元，而不是方法的集合。

Blocks是增加给C，Objective-C 和 C++的语言层面的功能，允许你创建像值一样可以传递给函数或者方法的不同的代码块。Blocks是Objective-C对象，意味着它可以添加到容器中，如*NSArray*和*NSDictionary*。它们也能捕获封闭环境内的值，跟其他编程语言的闭包或者lambdas表达式很像。

本章将讲述声明定义blocks的语法，以及如何使用blocks来简化常见任务，比如枚举。

## Block语法

我们使用插入符号(^)定义block，如下：

	^{
         NSLog(@"This is a block");
    }
    
比方法和函数的定义相比，花括号指示了block的开始和结束。在这个示例中，block没有返回任何值，也没有任何参数。

我们可以使用函数指针指向一个C函数，类似的，你也可以定义变量来保存block，如下：

	void (^simpleBlock)(void);
	
如果你没有使用过C函数指针，这种语法看起来可能很奇怪。在这个示例中，定义了一个block变量*simpleBlock*，这个block没有参数也没有返回值，也就是说可以把block赋值给这个变量，如下：

	simpleBlock = ^{
        NSLog(@"This is a block");
    };
    
就像其他变量赋值一样，所以必须以分号结束语句。你也可以将声明和赋值结合在一起，如下：

	void (^simpleBlock)(void) = ^{
        NSLog(@"This is a block");
    };
    
一旦你声明赋值了一个block变量，你就可以调用它：

	simpleBlock();

*注意：*如果你试图调用一个没有赋值的block变量(nil)，你程序将会崩溃。

### Block有参数和返回值

block可以像方法和函数一样携带参数和返回值。

假设一个变量引用了一个block，这个block返回两个数的乘积：

	double (^multiplyTwoValues)(double, double);
	
相应的block代码如下：

	^ (double firstValue, double secondValue) {
        return firstValue * secondValue;
    }
    
*firstValue* 和 *secondValue*在block调用时接收参数，就如函数一样。在这个示例中，返回类型根据最后的返回语句推断。

如果你乐意，你也可以在插入符号和参数列表之间显示的指定返回类型：

	^ double (double firstValue, double secondValue) {
        return firstValue * secondValue;
    }
    
一旦你声明定义了一个block，你可以像函数一样调用：

	double (^multiplyTwoValues)(double, double) =
                              ^(double firstValue, double secondValue) {
                                  return firstValue * secondValue;
                              };
 
    double result = multiplyTwoValues(2,4);
 
    NSLog(@"The result is %f", result);

### Block能够捕获封闭范围内的值

除了包含可执行的代码，block也有可以捕获封闭作用域内的状态。

比如，你在一个方法里定义了一个block，你可以捕获这个方法作用域内的任何值，如下：

	- (void)testMethod {
	    int anInteger = 42;
	 
	    void (^testBlock)(void) = ^{
	        NSLog(@"Integer is: %i", anInteger);
	    };
	 
	    testBlock();
	}
	
在这个示例中，*anInteger*定义在block外面，但是它的值在block定义的时候就被捕获了。

只有值被捕获，除非你特别指定。这就意味着如果你在定义block与调用之间改变了外部变量的值，如下：

	int anInteger = 42;
 
    void (^testBlock)(void) = ^{
        NSLog(@"Integer is: %i", anInteger);
    };
 
    anInteger = 84;
 
    testBlock();
    
那么block捕获的值将不受影响，意味着log会输出：

	Integer is: 42
	
这也意味着block不能改变原始变量的值，或者捕获的值(捕获的变量被当作常量)。

#### 使用__block变量共享存储

如果你需要改变block捕获的变量，你可以使用*__block*来声明需要改变的变量。这就意味着这个变量会在定义变量的作用域与定义在该作用域内的block共享存储：

比如，改写之前的示例：

	__block int anInteger = 42;
 
    void (^testBlock)(void) = ^{
        NSLog(@"Integer is: %i", anInteger);
    };
 
    anInteger = 84;
 
    testBlock();
    
因为使用*__block*定义了*anInteger*变量，所以和定义的block共享存储。这就意味将会输出：

	Integer is: 84
	
同时也意味着修改了原始变量：

	__block int anInteger = 42;
 
    void (^testBlock)(void) = ^{
        NSLog(@"Integer is: %i", anInteger);
        anInteger = 100;
    };
 
    testBlock();
    NSLog(@"Value of original variable is now: %i", anInteger);
    
将会输出：

	Integer is: 42
	Value of original variable is now: 100

### Block可以当做方法或者函数的参数

之前的示例中，我们在定义完block之后就立即调用了block。实际上，大多时候将block当作函数或者方法的参数传递过去，在别处调用。你可能使用GCD在后台调用了一个block，比如，定义了一个循环执行的任务，比如枚举容器类。并发以及之后讲到的枚举。

Blocks也用于回调，定义任务完成后需要执行的代码。比如，你的应用程序需要在完成用户的任务后回应用户，比如从web service请求信息，因为这些任务可能花费较长的实践，你需要显示一些进度指示当任务正在进行时，然后隐藏这些指示当任务完成后。

你可能会使用委托来完成这些任务，你需要创建合适的协议，实现必须得方法，设置你的对象为任务的delegate，然后等待任务完成时调用代理的方法。

Blocks简化了这种操作，你需要在开始任务的时候定义回调的行为，如下：

	- (IBAction)fetchRemoteInformation:(id)sender {
	    [self showProgressIndicator];
	 
	    XYZWebTask *task = ...
	 
	    [task beginTaskWithCallbackBlock:^{
	        [self hideProgressIndicator];
	    }];
	}
	
首先调用了显示进度指示器的方法，然后创建了任务并开始了任务。回调的block指定了任务完成时要执行的代码，这个例子中，简单的调用隐藏进度指示器的方法。注意这个回调的block捕获了*self*来调用*hideProgressIndicator*。当捕获*self*的时候需要非常小心，因为会引起强引用环。

同时block也有很好的代码可读性，能够轻易的看出任务执行前后发生了什么，而不像delegate需要根据方法来判断发生了什么。

*beginTaskWithCallbackBlock:*方法的定义可能如下所示：

	- (void)beginTaskWithCallbackBlock:(void (^)(void))callbackBlock;
	
*(void (^)(void))*指定了参数是一个block，没有返回值和参数，方法的实现如下：

	- (void)beginTaskWithCallbackBlock:(void (^)(void))callbackBlock {
	    ...
	    callbackBlock();
	}
	
如果期望block携带两个参数，如下所示：

	- (void)doSomethingWithBlock:(void (^)(double, double))block {
	    ...
	    block(21.0, 2.0);
	}
	
#### block应该始终是方法的最后一个参数

如果方法除了block之外还有其他的参数，那么block应该放在参数列表的最后：

	- (void)beginTaskWithName:(NSString *)name completion:(void(^)(void))callback;
	
这让内嵌block的方法更容易读，如下：

	[self beginTaskWithName:@"MyTask" completion:^{
        NSLog(@"The task is complete");
    }];

### 使用类型定义来简化Block

如果你需要使用block来定义多个变量，那么你可能需要将这个block定义为自己的类型。

比如，你可以定义一个简单的没有参数和返回值的block类型，如下：

	typedef void (^XYZSimpleBlock)(void);
	
你可以在方法参数或者定义block变量时使用这个自定义的类型：

	XYZSimpleBlock anotherBlock = ^{
        ...
    };
    
	- (void)beginFetchWithCallbackBlock:(XYZSimpleBlock)callbackBlock {
	    ...
	    callbackBlock();
	}
	
当我们把block当做函数参数或者返回值的时候，自定义的block类型特别有用，考虑如下例子：

	void (^(^complexBlock)(void (^)(void)))(void) = ^ (void (^aBlock)(void)) {
	    ...
	    return ^{
	        ...
	    };
	};
	
*complexBlock*变量时一个block变量，这个变量使用另外另外一个block(aBlock)当做参数，然后返回另外一个block。

使用类型定义重写这段代码，使其更具可读性：

	XYZSimpleBlock (^betterBlock)(XYZSimpleBlock) = ^ (XYZSimpleBlock aBlock) {
	    ...
	    return ^{
	        ...
	    };
	};

### 对象可以使用属性来保持Block

定义block属性和变量类似：

	@interface XYZObject : NSObject
	@property (copy) void (^blockProperty)(void);
	@end
	
*注意：*你应该指定属性为*copy*，因为block需要赋值一份来保存它捕获的外部状态。当你使用ARC的时候，你并不需要担心这些，这些都是自动发生的，最好还是显式的指定。

block属性就像其他block变量一样被设置或者调用：

	self.blockProperty = ^{
        ...
    };
    self.blockProperty();
    
你也可以使用类型定义来定义block属性，如下：

	typedef void (^XYZSimpleBlock)(void);
	 
	@interface XYZObject : NSObject
	@property (copy) XYZSimpleBlock blockProperty;
	@end

### 当捕获self的时候避免强引用环

如果你需要在block里面捕获*self*，比如定义了回调block，那么内存管理就很重要了。

block将保留任何捕获对象的强引用，包括*self*，那就意味着很容易引起强引用环，比如，一个对象拥有捕获了自己的block属性：

	@interface XYZBlockKeeper : NSObject
	@property (copy) void (^block)(void);
	@end
	
	@implementation XYZBlockKeeper
	- (void)configureBlock {
	    self.block = ^{
	        [self doSomething];    // capturing a strong reference to self
	                               // creates a strong reference cycle
	    };
	}
	...
	@end

对于这个简单点的示例，编译器将会给出警告，但是对于对象之间复杂的强引用，很难诊断是否形成了强引用环：

为了避免这个问题，最佳实践就是捕获一个弱引用的*self*，如下：

	- (void)configureBlock {
	    XYZBlockKeeper * __weak weakSelf = self;
	    self.block = ^{
	        [weakSelf doSomething];   // capture the weak reference
	                                  // to avoid the reference cycle
	    }
	}
	
通过捕获了弱引用的*self*，这个block将不会对*XYZBlockKeeper*的对象持有强引用。如果在block调用之前释放了这个对象，那么*weakSelf*指针将会被设为nil。

## Block能够简化枚举

除了作为处理句柄外，许多Cocoa 和 Cocoa Touch API使用block来简化常见的任务，比如容器的枚举。比如，*NSArray*类，提供了3个基于block的方法：

	- (void)enumerateObjectsUsingBlock:(void (^)(id obj, NSUInteger idx, BOOL *stop))block;
	
这个方法只接受一个block参数，数组里的每一项都会调用这个block：

	NSArray *array = ...
    [array enumerateObjectsUsingBlock:^ (id obj, NSUInteger idx, BOOL *stop) {
        NSLog(@"Object at index %lu is %@", idx, obj);
    }];
    
这个block接收3个参数，头两个参数和数组当前项相关，即当前的对象和下标。第三个参数用来是否停止枚举：

	[array enumerateObjectsUsingBlock:^ (id obj, NSUInteger idx, BOOL *stop) {
        if (...) {
            *stop = YES;
        }
    }];
    
你也可以使用*enumerateObjectsWithOptions:usingBlock:*方法来自定义枚举。指定*NSEnumerationReverse*选项，比如，将会逆序枚举。

如果枚举block的并发执行是安全的，那么你可以使用*NSEnumerationConcurrent*选项：

	[array enumerateObjectsWithOptions:NSEnumerationConcurrent
                            usingBlock:^ (id obj, NSUInteger idx, BOOL *stop) {
        ...
    }];
    
这个选项表示枚举block会被不同的线程调用，提升了性能，注意使用这个选项时枚举的顺序将是不确定的。

*NSDictionary*类也提供了基于block的方法，包括：

	NSDictionary *dictionary = ...
    [dictionary enumerateKeysAndObjectsUsingBlock:^ (id key, id obj, BOOL *stop) {
        NSLog(@"key: %@, value: %@", key, obj);
    }];
    
比起传统的循环，通过枚举每一个键值对更方便。

## Block能够简化并发任务

一个block表示不同的任务单元，在作用域范围内组合可执行的代码以及捕获可选的状态。这让block成为iOS或者OS X并发任务中异步调用的一种选项。比如搞清楚底层线程的工作机制，你可以简单的使用block来定义任务，然后让系统在处理器资源可用的时候执行这些任务。

OS X和iOS提供了多种并发技术，包括任务调度机制：操作队列和GCD。这些机制解决了任务队列等待调用的问题。你可以将你的block添加到队列里面，，然后系统在处理器时间和资源可用的时候让这些任务出列执行。

串行队列同一时间只允许一个任务执行，下一个任务在任务执行完成时不会出列被执行。并行队列可以执行多个任务，并不需要等待前面的任务完成。

### 操作队列使用Block

操作队列是Cocoa 和 Cocoa Touch的任务调用方式。你创建*NSOperation*的实例来封装任务和必须得数据，然后再操作添加到*NSOperationQueue*队列里面执行。

尽管你可以自定义自己的*NSOperation*子类来实现复杂的任务，你也可以用*NSBlockOperation*使用block创建任务，如下：

	NSBlockOperation *operation = [NSBlockOperation blockOperationWithBlock:^{
	    ...
	}];
	
你完全可以手动执行这些操作，但是通常将操作添加到已经存在的操作队列中，后者是你自己创建的操作队列中准备执行：

	// schedule task on main queue:
	NSOperationQueue *mainQueue = [NSOperationQueue mainQueue];
	[mainQueue addOperation:operation];
	 
	// schedule task on background queue:
	NSOperationQueue *queue = [[NSOperationQueue alloc] init];
	[queue addOperation:operation];
	
如果你使用操作队列，你可以配置操作的优先权以及操作之间的依赖性，比如制定一个操作在其他一组操作完成之后再执行。你也可以使用key-value observing(键值观察)来检测操作状态的变化，然后更新进度指示器。

更多操作及操作队列信息详见[Operation Queues](https://developer.apple.com/library/mac/documentation/General/Conceptual/ConcurrencyProgrammingGuide/OperationObjects/OperationObjects.html#//apple_ref/doc/uid/TP40008091-CH101)

### GCD中的Block

你也可以使用GCD队列来执行同步或者异步的操作，按照先进先出的顺序执行任务。

你可以自己创建GCD队列，或者使用GCD提供的队列。如果你需要调度并发执行的任务，你可以使用*dispatch_get_global_queue()*函数指定队列优先级来得到一个队列，如下：

	dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
	
为了将block添加到队列，你可以使用*dispatch_async()*或者*dispatch_sync()*函数。*dispatch_async()*函数会立即返回，不必等待block被调用：

	dispatch_async(queue, ^{
    	NSLog(@"Block for asynchronous execution");
	});
	
*dispatch_sync()*方法并不会返回，直到block执行完成；你可能在这种情形下需要使用这个方法，比如，你的并发的block需要等待主线程上另外一个任务完成之后才能继续。

GCD更多信息，参见[Dispatch Queues](https://developer.apple.com/library/mac/documentation/General/Conceptual/ConcurrencyProgrammingGuide/OperationQueues/OperationQueues.html#//apple_ref/doc/uid/TP40008091-CH102)

# 处理错误

几乎所有的应用程序都会遇到错误。有些错误是不受你控制的，比如没有磁盘空间或者网络连接断开。有些错误是可以恢复的，比如非法的用户输入。即使开发人员尽善尽美，也可能会引起偶然的编程错误。

如果你是从其他平台或者语言转过来的，那么你大部分的错误处理都是在处理异常。当你在编写Objective-C代码时，异常被单独用来处理编程错误，比如数组越界或者非法的方法参数。这些问题应该是你在发布应用程序之前就应该测试发现并修复的。

所有的错误都是由*NSError*类表示的。本章将简要的介绍使用*NSError*对象。以及怎么处理调用框架方法失败返回的错误。

## 使用NSError处理大部分错误

错误是程序生命周期中不可避免的一部分。比如，如果你需要从web service请求数据，那么将有大量潜在的问题可能会出现，比如：

* 没有网络连接
* 无法连接远程的web service
* 远程web service可能无法处理你的请求
* 返回的数据可能不是你期望的

遗憾的事，你不可能为每一种错误构建计划和解决方案。相反，你应该知道怎么处理这些错误，尽自己最大的努力给用户最好的体验。

### 一些代理方法会向你提醒错误

如果你实现了一个delegate对象来执行特定的任务，比如从远程服务器下载信息，你会发现你至少需要实现一个与错误相关的代理方法。比如，*NSURLConnectionDelegate*协议，包含一个*connection:didFailWithError:*方法：

	- (void)connection:(NSURLConnection *)connection didFailWithError:(NSError *)error;
	
如果发生了错误，那么这个方法就会被调用，然后提供给你一个描述问题的*NSError*对象。

*NSError*对象包含数字错误代码，域，以及描述，其他的相关信息包装在一个user info字典里。

比起每一个错误有一个唯一的数字错误代码，Cocoa 和 Cocoa Touch将错误分到了不同的域中。如果在*NSURLConnection*中发生了错误，比如，*connection:didFailWithError:*方法将会提供一个来自*NSURLErrorDomain*域的错误。

这个错误对象也包含一个本地描述，比如"A server with the specified hostname could not be found"。

### 一些方法用过引用传递错误

一些Cocoa 和 Cocoa Touch API通过引用会回传一个错误。比如，你可能想将从服务器获得的数据写入磁盘，你使用*NSData*的*writeToURL:options:error:*方法。这个方法的最后一个参数接收一个*NSError*的指针：

	- (BOOL)writeToURL:(NSURL *)aURL
           options:(NSDataWritingOptions)mask
             error:(NSError **)errorPtr;

在调用这个方法之前，你需要创建一个合适的指针，然后将地址传递给这个方法：

	NSError *anyError;
    BOOL success = [receivedData writeToURL:someLocalFileURL
                                    options:0
                                      error:&anyError];
    if (!success) {
        NSLog(@"Write failed with error: %@", anyError);
        // present error to user
    }
    
如果发生了错误，*writeToURL:...*方法会返回一个*NO*，然后更新*anyError*指针指向描述错误的对象。

当处理传递的引用错误时，重要的是要判断方法的返回值是否出现了错误，如上所示。不要只是测试错误指针是否被设置指向了一个错误。

*提示：*如果你对错误不感兴趣，你可以传递*NULL*给* error:*参数。

### 尽可恢复程序，或者向用户展示出错

最佳的用户体验是将程序从那些显然的错误中恢复。比如，你在请求远程web service，你可能尝试请求不同的服务器。你可能在再次请求之前需要从用户那里获取合法的用户名或者密码。

如果你不能将程序从错误中恢复，你应该提醒用户。如果你使用Cocoa Touch开发iOS应用，你需要创建配置*UIAlertView*来显示错误。如果你在使用Cocoa开发OS X，你应该调用*NSResponder*对象(比如view，window或者应用程序本身)的*presentError:*方法，然后错误会被传递到响应链更深处进行配置或者修复。当到达application对象时，应用程序将通过提醒面板向用户展示错误。

### 生成你自己的错误

为了创建你自己的*NSError*对象，你需要定义你自己的domain，应该有如下形式：

	com.companyName.appOrFrameworkName.ErrorDomain
	
同样你也需要给每一个错误定义一个唯一的错误码，还有合适的描述，存储在user info字典里，比如：

	NSString *domain = @"com.MyCompany.MyApplication.ErrorDomain";
    NSString *desc = NSLocalizedString(@"Unable to…", @"");
    NSDictionary *userInfo = @{ NSLocalizedDescriptionKey : desc };
 
    NSError *error = [NSError errorWithDomain:domain
                                         code:-101
                                     userInfo:userInfo];
                                     
这个示例使用了*NSLocalizedString*函数在*Localizable.strings*文件里查找错误描述的本地版本。

如果你需要回传一个之前描述的错误的引用，你的方法签名应该包括一个指向*NSError*对象的指针。同样也需要使用返回值来指示成功或者失败了，如下：

	- (BOOL)doSomethingThatMayGenerateAnError:(NSError **)errorPtr;
	
如果引起了错误，你需要先检查error参数是否是空指针，如下：

	- (BOOL)doSomethingThatMayGenerateAnError:(NSError **)errorPtr {
	    ...
	    // error occurred
	    if (errorPtr) {
	        *errorPtr = [NSError errorWithDomain:...
	                                        code:...
	                                    userInfo:...];
	    }
	    return NO;
	}

## 异常被用来处理编程错误

Objective-C也支持异常，如同其他编程语言一样，跟Java或者C++有相似的语法。与*NSError*一样，异常在Cocoa 和 Cocoa Touch中也是对象，由*NSException*类表示。

如果你需要编写可能抛出异常的代码，你可以将代码装入try-catch block内：

	@try {
        // do something that might throw an exception
    }
    @catch (NSException *exception) {
        // deal with the exception
    }
    @finally {
        // optional block of clean-up code
        // executed whether or not an exception occurred
    }
    
如果*@try*block里面的代码抛出了异常，*@catch*block将会捕捉异常然后处理。如果你在使用低层级的C++库，使用异常来处理错误，比如，你可能需要捕捉异常然后生成合适的*NSError*对象来展示给用户。

如果异常没有被捕捉，默认的没有捕捉异常的处理句柄将会打印消息到控制台然后终止程序。

你不应该为Objective-C方法使用try-catch block替代标准的检查。比如，*NSArray*，你需要在访问给定的下标时始终检查数组项的数量。*objectAtIndex:*方法在你访问越界的数组项时将会抛出异常，所以你在开发程序的过程中就能发现这个BUG。你应该避免抛出异常当程序发布给用户的时候。

# 约定

当你使用框架类的时候，你可能注意到了Objective-C的代码有很高的可读性。类和方法名比C语言的函数以及类库更具描述性，使用驼峰规则来命名多个词的名字。你也应该遵循Cocoa 和 Cocoa Touch使用的规则来编写你自己的类，使其更具可读性，使你的代码保持简洁。

除此之外，Objective-C和一些框架的功能要求你遵循严格的命名规则使一些机制正常工作。比如，访问器方法的名字必须遵循约定，从而才来使用诸如Key-Value Coding (KVC) 或者 Key-Value Observing (KVO)这些技术。

本章描述了Cocoa 和 Cocoa Touch代码中使用的大部分常见的约定，也解释了哪些情况下命名需要唯一，同时也包括了链接框架。

## 在你的应用程序中一些命名必须是唯一的

每次你创建新的类型，符号，或者标识的时候，你应该首先考虑在作用域范围内哪些名字必须是唯一的。有时这个作用域是整个应用程序，包括链接的框架；有时作用域被限制在了类里面或者在一个block代码块里。

### 类名在整个应用程序中必须唯一

Objective-C类的命名在你编写代码的项目以及链接的框架或者bundles中都必须是唯一的。比如，你应该避免使用那些一般的类名，比如*ViewController*或者*TextParser*，因为你应用程序包含的框架的类已经使用这些名字，根据约定你就不能创建相同名字的类了。

为了保证类名唯一，约定所有的类名都是用前缀。你也许注意到了Cocoa 和 Cocoa Touch类使用了*NS*或者*UI*前缀。这些两个字母的前缀被苹果保留使用在框架类中。当你学会更多的Cocoa 和 Cocoa Touch，你将会遇到大量使用前缀的框架：

![alt text](/img/ProgrammingwithObjectiveC/prefixes.png)

你自己的类应该使用三个字母的前缀。这三个字母可能使你公司名字和程序名字的组合，或者是你程序内特定一个组件。比如，你的公司叫做Whispering Oak，正在开发一个叫做Zebra Surprise的游戏，你可能选择*WZS*或者*WOZ*作为你类的前缀。

你也应该是用名词来命名你的类，清楚的表明这个类代表什么，比如Cocoa 和 Cocoa Touch中得类：


	NSWindow
	CAAnimation
	NSWindowController
	NSManagedObjectContext

如果你需要多个词来命名类名，你需要使用驼峰规则，大写每个词的首字母。

### 方法名必须具有表达性而且在类中是唯一的

一旦你为类选择了一个唯一的名字，那么定义在类中的方法名也必须是唯一的。在其他的类中可以有跟该类方法同名的方法，比如，重写父类的方法，或者利用多态性。在多个类中执行相同任务的类应该有相同的名字，返回值和参数类型。

方法名不需要前缀，应该以小写字母开始；驼峰规则，如*NSString*类：

	length
	characterAtIndex:
	lengthOfBytesUsingEncoding:

如果一个方法需要一个或者多个参数，方法名应该指示每一个参数;

	substringFromIndex:
	writeToURL:atomically:encoding:error:
	enumerateSubstringsInRange:options:usingBlock:

方法名的第一部分应该指示主要的意图和调用方法的结果。如果一个方法返回一个值，比如，第一个词应该指示将会返回什么，比如*length*，*character...*和*substring...*。如果你想指示关于返回值的一些重要的信息，你可以使用多个词，比如*NSString*类里的*mutableCopy*，*capitalizedString* 或者 *lastPathComponent*方法，比如写入磁盘和枚举内容的方法，第一个词应该指示这个方法的操作，比如*write...* 和 *enumerate...*。

如果方法包含当引起错误时设置指向错误的指针参数，它应该是方法的最后一个参数。如果方法有block参数，为了让方法内嵌block时更有可读性应当把block当做最后一个参数。同样的理由，应该避免使用多个block作为参数，不管是否可能。

当然方法名清晰简洁也很重要。太简洁或者太冗长都不是最优选择，所以中等长度可能是最优解了：

	stringAfterFindingAndReplacingAllOccurrencesOfThisString:withThisString:
	Too verbose
	
	strReplacingStr:str:
	Too concise
	
	stringByReplacingOccurrencesOfString:withString:
	Just right
	
你应该避免在方法名种使用缩写，除非你知道那个缩写在不同语言及文化中是被熟知的。缩写列表，详见[Acceptable Abbreviations and Acronyms](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/CodingGuidelines/Articles/APIAbbreviations.html#//apple_ref/doc/uid/20001285)

#### 在给框架类添加类别时，方法名总是使用前缀

当使用类别给已经存在的框架类添加方法时，你应该给方法名使用前缀来避免冲突，如[Avoid Category Method Name Clashes](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/CustomizingExistingClasses/CustomizingExistingClasses.html#//apple_ref/doc/uid/TP40011210-CH6-SW4)所述。

### 在作用域范围内本地变量必须是唯一的

因为Objective-C是C语言的超集，所以C变量作用域的规则同样适用于Objective-C。本地变量的名字必须不能和作用域范围内其他的变量冲突：

	- (void)someMethod {
	    int interestingNumber = 42;
	    ...
	    int interestingNumber = 44; // not allowed
	}
	
虽然C语言允许你使用相同的名字定义一个本地变量，而之前在封闭作用域内已经定义了一个，如下：

	- (void)someMethod {
	    int interestingNumber = 42;
	    ...
	    for (NSNumber *eachNumber in array) {
	        int interestingNumber = [eachNumber intValue]; // not advisable
	        ...
	    }
	}
	
这让代码的可读性很低，而且让人觉得困惑，所以不管是否可能，应当避免这样做。

## 一些方法的命名必须遵循约定

除了应该考虑唯一性之外，一些基本的重要的方法也应该遵循严格的约定。这些约定被用在一些Objective-C基础的机制当中，也用在编译器和运行时当中，除此之外，也被Cocoa 和 Cocoa Touch的一些类的行为要求。

### 访问器方法必须遵循约定

当你在对象里面使用*@property*声明属性时，如[Encapsulating Data](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/EncapsulatingData/EncapsulatingData.html#//apple_ref/doc/uid/TP40011210-CH5-SW1)所述，这个编译器自动合成了相关的getter和setter方法(除非你另外指定了)。出于一些原因，你需要提供自己的访问器方法，确保你使用了正确的方法名，来保证使用点语法的时候会被调用。

除非特别指定，getter方法的名字应该和属性名一样。比如属性叫做*firstName*，那么访问器方法也叫做*firstName*。这种情况的特例就是布尔类型的属性，getter方法以*is*开头。比如一个属性叫做*paused*，getter方法就叫做*isPaused*。

属性的setter方法应该使用*setPropertyName:*的形式，比如属性叫做*firstName*，setter方法应该叫做*setFirstName:*；如果是不二属性，叫做*paused*，setter方法应该叫做*setPaused:*。

尽管属性允许你使用不同的访问器访问名字，但是你应该只在布尔属性的情形下这样做。不然诸如像Key Value Coding(使用*valueForKey:* 和 *setValue:forKey:*来设置或者得到属性的能力)的技术将无法使用。更多KVC信息，详见[Key-Value Coding Programming Guide](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/KeyValueCoding/Articles/KeyValueCoding.html#//apple_ref/doc/uid/10000107i)。

### 创建对象的方法名必须遵循约定

如前面章节所见，提供了几种方式来创建类的实例。你可能使用了内存分配和初始化的组合(两段式构造)，如下：

	NSMutableArray *array = [[NSMutableArray alloc] init];

或者使用*new*方法当做*alloc*和*init*的替代方法：

	NSMutableArray *array = [NSMutableArray new];
	
一些类也提供了工厂方法：

	NSMutableArray *array = [NSMutableArray array];
	
类的工厂方法应该始终以类名(没有前缀)开头来创建，这种情况的例外就是类的子类。比如，*NSArray*类的工厂方法以*array*开头。*NSMutableArray*类没有定义任何自己的工厂方法，所以mutable array的工厂方法仍然以*array*开头。

Objective-C中有各种各样的基础内存管理规则，编译器用来确保对象在内存中保留足够长的时间。虽然你不太需要担心这些规则，编译器将会根据创建的方法名字来判断需要遵循的规则。通过工厂方法创建的对象与使用传统allocation和initialization或者*new*创建的对象的内存管理有一些不一样，工厂方法创建的对象使用了autorelease pool blocks(自动释放池)。更多关于自动释放池和内存管理，详见[Advanced Memory Management Programming Guide](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/MemoryMgmt/Articles/MemoryMgmt.html#//apple_ref/doc/uid/10000011i)。
