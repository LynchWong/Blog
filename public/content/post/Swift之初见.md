---
title: "Swift之初见"
date: 2015-01-24 18:08:22
categories: 
- iOS
tags: 
- Swift
metaAlignment: center
autoThumbnailImage: no
coverImage: //d1u9biwaxjngwg.cloudfront.net/welcome-to-tranquilpeak/city.jpg

---

上篇博客写完到现在已经很久了，最近看起来闲了点，所以就有空写博客了。同学提了建议说我博客里面的博文比较散，东搞搞，西搞搞的。确实是这样啊，没有形成专题。蓝牙和自定义控件都是那时做项目时正好遇到的东西，所以就写了博文。我觉得写博客不一定就是讲解一些专题什么的，也可以是记录BUG产生的原因，修复的过程；记录生活方面的文章等等。以后的文章我回尽量紧凑、全面一些。
<!--more-->

14年苹果在WWDC大会上发布了Swift语言，刚出没多久就上手把完了下，一点陌生感都没有。你能在Swift语言关键字和语法里面看到其他语言的身影，诸如JS、C#、Ruby、OC等等。用Swift开发了一两个项目的感觉就是在项目结构、语法上比OC要简洁很多，其他的方面到是没有多大差别，感觉跟OC是一样的。由于Swift刚发布没多久，所以一直都在完善、修改，所以现在就完全使用Swift是比较冒险的事情，需要时常修改代码和新版本同步。不过我的建议还是尽早学会Swift，说实话用不了多少时间，Swift是上手极快的语言。我就花了几小时了解了下大概，用了一周不到的时间就把自己的一个项目用Swift做了一遍。但是要深入了解Swift，估计还是需要花不少的时间。这个专题就是来介绍Swift的，内容主要就是从苹果的[The Swift Programming Language](https://developer.apple.com/library/prerelease/ios/documentation/Swift/Conceptual/Swift_Programming_Language/index.html#//apple_ref/doc/uid/TP40014097)里面来的。因为苹果的这个文档内容还是比较多的，翻阅起来比较麻烦。所以在这里做一些记录，也让没有接触Swift的同学看到这些博文后就会使用Swfit。

## Swift初见

#### 简单值

使用 let 来声明常量,使用 var 来声明变量。一个常量的值,在编译的时候,并不需要有明确的值,但是你只能 为它赋值一次。也就是说你可以用常量来表示这样一个值:你只需要决定一次,但是需要使用很多次。

    var myVariable = 42
    myVariable = 50
    let myConstant = 42

这里需要多说的就是Swift里面是没有宏的，所以原来在OC里面用define定义的常量在Swift里面请使用let；同样的那些复杂功能的函数宏也请改为方法实现。

常量或者变量的类型必须和你赋给它们的值一样。然而,你不用明确地声明类型,声明的同时赋值的话,编译器会自动推断类型。在上面的例子中,编译器推断出 myVariable 是一个整数(integer)因为它的初始值是整数。

如果初始值没有提供足够的信息(或者没有初始值),那你需要在变量后面声明类型,用冒 号分割。

	let implicitInteger = 70
	let implicitDouble = 70.0
	let explicitDouble: Double = 70

值永远不会被隐式转换为其他类型。如果你需要把一个值转换成其他类型,请显式转换。
    
    let label = "The width is"
    let width = 94
    let widthLabel = label + String(width)

有一种更简单的把值转换成字符串的方法:把值写到括号中,并且在括号之前写一个反斜杠。例如:

    let apples = 3
    let oranges = 5
    let appleSummary = "I have \(apples) apples."
    let fruitSummary = "I have \(apples + oranges) pieces of fruit."

使用方括号 [] 来创建数组和字典,并使用下标或者键(key)来访问元素。

	var shoppingList = ["catfish", "water", "tulips", "blue paint"] 
	shoppingList[1] = "bottle of water"

    var occupations = [
						"Malcolm": "Captain",
    					"Kaylee": "Mechanic", 7. ] 
    occupations["Jayne"] = "Public Relations"

要创建一个空数组或者字典,使用初始化语法。

	let emptyArray = [String]()
	let emptyDictionary = [String: Float]()

如果类型信息可以被推断出来,你可以用[]和[:]来创建空数组和空字典——就像你声明变量或者给函数传参数的时候一样。

	shoppingList = []
	occupations = [:]

#### 控制流

使用 if 和 switch 来进行条件操作,使用 for-in 、 for 、 while 和 repeat-while 来进行循环。包裹条件和循环 变量括号可以省略,但是语句体的大括号是必须的。

	let individualScores = [75, 43, 103, 87, 12]
	var teamScore = 0
	for score in individualScores {
 
		if score > 50 {
			teamScore += 3
		} else {
			teamScore += 1
		}
		
	}
	println(teamScore)

在 if 语句中,条件必须是一个布尔表达式——这意味着像 if score { ... } 这样的代码将报错,而不会隐形地与 0 做对比。

你可以一起使用 if 和 let 来处理值缺失的情况。这些值可由可选值来代表。一个可选的值是一个具体的值或者是 nil 以表示值缺失。在类型后面加一个问号来标记这个变量的值是可选的。

	var optionalString: String? = "Hello"
	optionalString == nil
	
	var optionalName: String? = "John Appleseed"
	var greeting = "Hello!"
	if let name = optionalName { 
		greeting = "Hello, \(name)"
	}

如果变量的可选值是 nil ,条件会判断为 false ,大括号中的代码会被跳过。如果不是 nil ,会将值赋给 let 后面 的常量,这样代码块中就可以使用这个值了。

switch 支持任意类型的数据以及各种比较操作——不仅仅是整数以及测试相等。

	let vegetable = "red pepper"
	switch vegetable { 
		case "celery":
			let vegetableComment = "Add some raisins and make ants on a log." 
		case "cucumber", "watercress":
			let vegetableComment = "That would make a good tea sandwich." 
		case let x where x.hasSuffix("pepper"):
			let vegetableComment = "Is it a spicy \(x)?"
		default:
			let vegetableComment = "Everything tastes good in soup." 
	}

声明 let 可用于匹配某部分固定值的模式。

运行 switch 中匹配到的子句之后,程序会退出 switch 语句,并不会继续向下运行,所以不 需要在每个子句结尾写 break。

你可以使用 for-in 来遍历字典,需要两个变量来表示每个键值对。字典是一个无序的集合,所以他们的键和值以 任意顺序迭代结束。

	let interestingNumbers = [
		"Prime": [2, 3, 5, 7, 11, 13],
		"Fibonacci": [1, 1, 2, 3, 5, 8],
		"Square": [1, 4, 9, 16, 25],
	]
	var largest = 0
	for (kind, numbers) in interestingNumbers { 
		for number in numbers {
			if number > largest { 
				largest = number
			}
		}
	}
	largest

使用 while 来重复运行一段代码直到不满足条件。循环条件可以在开头也可以在结尾。
	var n = 2
	while n < 100 {
		n = n * 2 
	}
	print(n)
	
	var m = 2
	repeat {
		m = m * 2
	} while m < 100
	print(m)

你可以在循环中使用..<来表示范围,也可以使用传统的写法,两者是等价的:

	var firstForLoop = 0
	for i in 0..<4 {
	    firstForLoop += i
	}
	println(firstForLoop)
	 
	var secondForLoop = 0
	for var i = 0; i < 4; ++i {
	    secondForLoop += i
	}
	println(secondForLoop)
	
使用..<创建的范围不包含上界,如果想包含的话需要使用...。

#### 函数和闭包

使用 func 来声明一个函数,使用名字和参数来调用函数。使用 -> 来指定函数返回值的类型。

	func greet(name: String, day: String) -> String {
	    return "Hello \(name), today is \(day)."
	}
	greet("Bob", "Tuesday")

使用元组来让一个函数返回多个值。该元组的元素可以用名称或数字来表示。

	func calculateStatistics(scores: [Int]) -> (min: Int, max: Int, sum: Int) {
	    var min = scores[0]
	    var max = scores[0]
	    var sum = 0
	    
	    for score in scores {
	        if score > max {
	            max = score
	        } else if score < min {
	            min = score
	        }
	        sum += score
	    }
	    
	    return (min, max, sum)
	}
	let statistics = calculateStatistics([5, 3, 100, 3, 9])
	print(statistics.sum)
	print(statistics.2)

函数可以带有可变个数的参数,这些参数在函数内表现为数组的形式:

	func sumOf(numbers: Int...) -> Int {
	    var sum = 0
	    for number in numbers {
	        sum += number
	    }
	    return sum
	}
	sumOf()
	sumOf(42, 597, 12)
	
函数可以嵌套。被嵌套的函数可以访问外侧函数的变量,你可以使用嵌套函数来重构一个太长或者太复杂的函数。

	func returnFifteen() -> Int {
	    var y = 10
	    func add() {
	        y += 5
	    }
	    add()
	    return y
	}
	returnFifteen()
	
函数是一等公民,这意味着函数可以作为另一个函数的返回值。

	func makeIncrementer() -> (Int -> Int) {
	    func addOne(number: Int) -> Int {
	        return 1 + number
	    }
	    return addOne
	}
	var increment = makeIncrementer()
	increment(7)
	
函数也可以当做参数传入另一个函数。

	func hasAnyMatches(list: [Int], condition: Int -> Bool) -> Bool {
	    for item in list {
	        if condition(item) {
	            return true
	        }
	    }
	    return false
	}
	func lessThanTen(number: Int) -> Bool {
	    return number < 10
	}
	var numbers = [20, 19, 7, 12]
	hasAnyMatches(numbers, lessThanTen)
	
函数实际上是一种特殊的闭包:它是一段能之后被调取的代码。闭包中的代码能访问闭包所建作用域中能得到的变 量和函数,即使闭包是在一个不同的作用域被执行的 - 你已经在嵌套函数例子中所看到。你可以使用 {} 来创建一 个匿名闭包。使用 in 将参数和返回值类型声明与闭包函数体进行分离。

	numbers.map({
	    (number: Int) -> Int in
	    let result = 3 * number
	    return result
	})
	
有很多种创建更简洁的闭包的方法。如果一个闭包的类型已知,比如作为一个回调函数,你可以忽略参数的类型
和返回值。单个语句闭包会把它语句的值当做结果返回。

	let mappedNumbers = numbers.map({ number in 3 * number })
	print(mappedNumbers)

你可以通过参数位置而不是参数名字来引用参数——这个方法在非常短的闭包中非常有用。当一个闭包作为最后
一个参数传给一个函数的时候,它可以直接跟在括号后面。当一个闭包是传给函数的唯一参数,你可以完全忽略
括号。

	let sortedNumbers = sorted(numbers) { $0 > $1 }
	sortedNumbers

#### 对象和类

使用 class 和类名来创建一个类。类中属性的声明和常量、变量声明一样,唯一的区别就是它们的上下文是 类。同样,方法和函数声明也一样。

	class Shape {
	    var numberOfSides = 0
	    func simpleDescription() -> String {
	        return "A shape with \(numberOfSides) sides."
	    }
	}

要创建一个类的实例,在类名后面加上括号。使用点语法来访问实例的属性和方法。

	var shape = Shape()
	shape.numberOfSides = 7
	var shapeDescription = shape.simpleDescription()

这个版本的 Shape 类缺少了一些重要的东西:一个构造函数来初始化类实例。使用 init 来 创建一个构造器。

	class NamedShape {
	    var numberOfSides: Int = 0
	    var name: String
	    
	    init(name: String) {
	        self.name = name
	    }
	    
	    func simpleDescription() -> String {
	        return "A shape with \(numberOfSides) sides."
	    }
	}

注意 self 被用来区别实例变量。当你创建实例的时候,像传入函数参数一样给类传入构造器的参数。每个属性都 需要赋值——无论是通过声明(就像 numberOfSides )还是通过构造器(就像 name )。

如果你需要在删除对象之前进行一些清理工作,使用 deinit 创建一个析构函数。

子类的定义方法是在它们的类名后面加上父类的名字,用冒号分割。创建类的时候并不需要一个标准的根类,所
以你可以忽略父类。

子类如果要重写父类的方法的话,需要用 override 标记——如果没有添加 override 就重写父类方法的话编译器 会报错。编译器同样会检测 override 标记的方法是否确实在父类中。

	class Square: NamedShape {
	    var sideLength: Double
	    
	    init(sideLength: Double, name: String) {
	        self.sideLength = sideLength
	        super.init(name: name)
	        numberOfSides = 4
	    }
	    
	    func area() ->  Double {
	        return sideLength * sideLength
	    }
	    
	    override func simpleDescription() -> String {
	        return "A square with sides of length \(sideLength)."
	    }
	}
	let test = Square(sideLength: 5.2, name: "my test square")
	test.area()
	test.simpleDescription()

除了储存简单的属性之外,属性可以有 getter 和 setter 。

	class EquilateralTriangle: NamedShape {
	    var sideLength: Double = 0.0
	    
	    init(sideLength: Double, name: String) {
	        self.sideLength = sideLength
	        super.init(name: name)
	        numberOfSides = 3
	    }
	    
	    var perimeter: Double {
	        get {
	            return 3.0 * sideLength
	        }
	        set {
	            sideLength = newValue / 3.0
	        }
	    }
	    
	    override func simpleDescription() -> String {
	        return "An equilateral triangle with sides of length \(sideLength)."
	    }
	}
	var triangle = EquilateralTriangle(sideLength: 3.1, name: "a triangle")
	println(triangle.perimeter)
	triangle.perimeter = 9.9
	println(triangle.sideLength)
	
在 perimeter 的 setter 中,新值的名字是 newValue。你可以在 set 之后显示的设置一个名 字。

注意 EquilateralTriangle 类的构造器执行了三步:

1. 设置子类声明的属性值.
2. 调用父类的构造器.
3. 改变父类定义的属性值。其他的工作比如调用方法、getters 和 setters 也可以在这个阶 段完成。

如果你不需要计算属性但是需要在设置一个新值之前运行一些代码,使用willSet和didSet。

比如,下面的类确保三角形的边长总是和正方形的边长相同。

	class TriangleAndSquare {
	    var triangle: EquilateralTriangle {
	        willSet {
	            square.sideLength = newValue.sideLength
	        }
	    }
	    var square: Square {
	        willSet {
	            triangle.sideLength = newValue.sideLength
	        }
	    }
	    init(size: Double, name: String) {
	        square = Square(sideLength: size, name: name)
	        triangle = EquilateralTriangle(sideLength: size, name: name)
	    }
	}
	var triangleAndSquare = TriangleAndSquare(size: 10, name: "another test shape")
	println(triangleAndSquare.square.sideLength)
	println(triangleAndSquare.triangle.sideLength)
	triangleAndSquare.square = Square(sideLength: 50, name: "larger square")
	println(triangleAndSquare.triangle.sideLength)
	
处理变量的可选值时,你可以在操作(比如方法、属性和子脚本)之前加 ? 。如果 ? 之前的值是 nil , ? 后面 的东西都会被忽略,并且整个表达式返回 nil 。否则, ? 之后的东西都会被运行。在这两种情况下,整个表达式 的值也是一个可选值。

	let optionalSquare: Square? = Square(sideLength: 2.5, name: "optional square")
	let sideLength = optionalSquare?.sideLength

#### 枚举和结构体

使用 enum 来创建一个枚举。就像类和其他所有命名类型一样,枚举可以包含方法。

	enum Rank: Int {
	    case Ace = 1
	    case Two, Three, Four, Five, Six, Seven, Eight, Nine, Ten
	    case Jack, Queen, King
	    func simpleDescription() -> String {
	        switch self {
	        case .Ace:
	            return "ace"
	        case .Jack:
	            return "jack"
	        case .Queen:
	            return "queen"
	        case .King:
	            return "king"
	        default:
	            return String(self.rawValue)
	        }
	    }
	}
	let ace = Rank.Ace
	let aceRawValue = ace.rawValue


在上面的例子中,枚举原始值的类型是 Int ,所以你只需要设置第一个原始值。剩下的原始值会按照顺序赋 值。你也可以使用字符串或者浮点数作为枚举的原始值。使用 rawValue 来访问一个枚举成员的原始值。

使用 init?(rawValue:) 构造器来从原始值中枚举一个例子。

	if let convertedRank = Rank(rawValue: 3) {
	    let threeDescription = convertedRank.simpleDescription()
	}
	
枚举的成员值是实际值,并不是原始值的另一种表达方法。实际上,以防原始值没有意义,你不需要设置。

	enum Suit {
	    case Spades, Hearts, Diamonds, Clubs
	    func simpleDescription() -> String {
	        switch self {
	        case .Spades:
	            return "spades"
	        case .Hearts:
	            return "hearts"
	        case .Diamonds:
	            return "diamonds"
	        case .Clubs:
	            return "clubs"
	        }
	    }
	}
	let hearts = Suit.Hearts
	let heartsDescription = hearts.simpleDescription()

注意,有两种方式可以引用 Hearts 成员:给 hearts 常量赋值时,枚举成员 Suit.Hearts 需要用全名来引用,因 为常量没有显式指定类型。在 switch 里,枚举成员使用缩写 .Hearts 来引用,因为 self 的值已经知道是一个 s uit 。已知变量类型的情况下你可以使用缩写。

使用 struct 来创建一个结构体。结构体和类有很多相同的地方,比如方法和构造器。它们之间最大的一个区别就 是结构体是传值,类是传引用。

	struct Card {
	    var rank: Rank
	    var suit: Suit
	    func simpleDescription() -> String {
	        return "The \(rank.simpleDescription()) of \(suit.simpleDescription())"
	    }
	}
	let threeOfSpades = Card(rank: .Three, suit: .Spades)
	let threeOfSpadesDescription = threeOfSpades.simpleDescription()

一个枚举成员的实例可以有实例值。相同枚举成员的实例可以有不同的值。创建实例的时候传入值即可。实例值
和原始值是不同的:枚举成员的原始值对于所有实例都是相同的,而且你是在定义枚举的时候设置原始值。

例如,考虑从服务器获取日出和日落的时间。服务器会返回正常结果或者错误信息。

	enum ServerResponse {
	    case Result(String, String)
	    case Error(String)
	}
 
	let success = ServerResponse.Result("6:00 am", "8:09 pm")
	let failure = ServerResponse.Error("Out of cheese.")
	 
	switch success {
	case let .Result(sunrise, sunset):
	    let serverResponse = "Sunrise is at \(sunrise) and sunset is at \(sunset)."
	case let .Error(error):
	    let serverResponse = "Failure...  \(error)"
	}

注意:如何从 ServerResponse 中提取日升和日落时间。

#### 协议和扩展

使用 protocol 来声明一个协议。

	protocol ExampleProtocol {
	    var simpleDescription: String { get }
	    mutating func adjust()
	}

类、枚举和结构体都可以实现接口。

	class SimpleClass: ExampleProtocol {
	    var simpleDescription: String = "A very simple class."
	    var anotherProperty: Int = 69105
	    func adjust() {
	        simpleDescription += "  Now 100% adjusted."
	    }
	}
	var a = SimpleClass()
	a.adjust()
	let aDescription = a.simpleDescription
	 
	struct SimpleStructure: ExampleProtocol {
	    var simpleDescription: String = "A simple structure"
	    mutating func adjust() {
	        simpleDescription += " (adjusted)"
	    }
	}
	var b = SimpleStructure()
	b.adjust()
	let bDescription = b.simpleDescription

注意声明 SimpleStructure 时候 mutating 关键字用来标记一个会修改结构体的方法。 SimpleClass 的声明不需 要标记任何方法因为类中的方法经常会修改类。

使用 extension 来为现有的类型添加功能,比如新的方法和参数。你可以使用扩展在别处修改定义,甚至是从外 部库或者框架引入的一个类型,使得这个类型遵循某个协议。

	extension Int: ExampleProtocol {
	    var simpleDescription: String {
	        return "The number \(self)"
	    }
	    mutating func adjust() {
	        self += 42
	    }
	}
	7.simpleDescription

你可以像使用其他命名类型一样使用接口名——例如,创建一个有不同类型但是都实现一个接口的对象集合。当你处理类型是接口的值时,接口外定义的方法不可用。

	let protocolValue: ExampleProtocol = a
	protocolValue.simpleDescription
	// protocolValue.anotherProperty  // Uncomment to see the error

即使 protocolValue 变量运行时的类型是 simpleClass ,编译器会把它的类型当做 ExampleProtocol 。这表示 你不能调用类在它实现的协议之外实现的方法或者属性。

#### 泛型

在尖括号里写一个名字来创建一个泛型函数或者类型。

	func repeat<Item>(item: Item, times: Int) -> [Item] {
	    var result = [Item]()
	    for i in 0..<times {
	        result.append(item)
	    }
	    return result
	}
	repeat("knock", 4)

你也可以创建泛型类、枚举和结构体。

	// Reimplement the Swift standard library's optional type
	enum OptionalValue<T> {
	    case None
	    case Some(T)
	}
	var possibleInteger: OptionalValue<Int> = .None
	possibleInteger = .Some(100)
	
在类型名后面使用 where 来指定一个需求列表——例如,要限定实现一个协议的类型,需要限定两个类型要相同,或者限定一个类必须有一个特定的父类。

	func anyCommonElements <T, U where T: SequenceType, U: SequenceType, 	T.Generator.Element: Equatable, T.Generator.Element == U.Generator.Element> 	(lhs: T, rhs: U) -> Bool {
	    for lhsItem in lhs {
	        for rhsItem in rhs {
	            if lhsItem == rhsItem {
	                return true
	            }
	        }
	    }
	    return false
	}
	anyCommonElements([1, 2, 3], [3])
	
简单起见,你可以忽略 where,只在冒号后面写接口或者类名。<T: Equatable>和<T where T:Equatable>是等价的。
