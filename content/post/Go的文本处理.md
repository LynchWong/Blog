---
title: "Go的文本处理"
date: 2015-07-02 12:34:11
categories: 
- Go
tags: 
- Go语言编程
metaAlignment: center
autoThumbnailImage: no
coverImage: //d1u9biwaxjngwg.cloudfront.net/welcome-to-tranquilpeak/city.jpg

---

应用开发中经常要处理文本信息，包括像字符串、数字、JSON、XML等。网络交互主要使用JSON和XML，现在大部分的互联网应用都是使用JSON格式的数据，所以这里我们主要讲一下JSON的处理。
<!--more-->

JSON(JavaScript Object Notation)是一种轻量级的数据交换格式。它基于ECMAScript的一个子集。 JSON采用完全独立于语言的文本格式，但是也使用了类似于C语言家族的习惯（包括C、C++、C#、Java、JavaScript、Perl、Python等）。这些特性使JSON成为理想的数据交换语言。易于人阅读和编写，同时也易于机器解析和生成(网络传输速率)。

Go语言的标准库已经非常好的支持了JSON，可以很容易的对JSON数据进行编、解码的工作。

## 解析JSON到结构体

假设我们有这样的一个JSON串`{"students":[{"name":"Lynch","age":"25"},{"name":"Wong","age":"26"}]}`，那么我们该怎么解析JSON串呢，Go的JSON包中有如下函数：

	func Unmarshal(data []byte, v interface{}) error
	
详细代码例子如下：

	package main
	
	import (
		"encoding/json"
		"fmt"
	)
	
	type Student struct {
		Name string
		Age string
	}
	
	type Studentslice struct {
		Students []Student
	}
	
	func main() {
		var s Studentslice
		str := `{"Students":[{"Name":"Lynch","Age":"25"},
							 {"Name":"Wong","Age":"26"}]}`
		json.Unmarshal([]byte(str), &s)
		fmt.Println(s)
	}

输出结果如下所示：

	{[{Lynch 25} {Wong 26}]}
	
首先我们定义了与JSON数据对应的结构体，Student的字段名对应数组里面对象的KEY，然后Studentslice的字段对应JSON数组的KEY。

KEY和字段的匹配方式如下，如果KEY是`Foo`：

- 首先查找tag含有`Foo`的可导出的struct字段(首字母大写)。
- 其次查找字段名是`Foo`的导出字段。
- 最后查找类似`FOO`或者`FoO`这样的除了首字母之外其他大小写不敏感的导出字段。

而且能够被复制的字段必须是可导出字段(即首字母大写)。同时JSON解析的时候只会解析能找得到的字段，找不到的字段会被忽略，这样的一个好处是：当你接收到了一个很大的JSON数据结构而你却只想获取其中的部分数据的时候，你只需需要将想要的数据对应的字段名大写即可。

## 解析JSON到interface

上面的情况是因为我们知道解析的JSON数据的结构，如果不知道该怎么解析呢？

我们可以使用interface{}来解析未知结构的JSON数据。使用map[string]interface{}和[]interface{}来存储任意的JSON对象和数组。Go类型和JSON类型的对应关系如下：

- bool 代表 JSON booleans,
- float64 代表 JSON numbers,
- string 代表 JSON strings,
- nil 代表 JSON null.

假设如下的JSON字串：

	b := []byte(`{"Name":"Lynch", "Books":["One", "Two"]}`)
	var f interface{}
	json.Unmarshal(b, &f)
	fmt.Println(f)
	
输入结果如下：

	map[Books:[One Two] Name:Lynch]
	
如何来访问这些数据呢？通过断言的方式：

	m := f.(map[string]interface{})
	for k, v := range m {
		switch vv := v.(type) {
			case string:
				fmt.Println(k, "is string", vv)
			case int:
				fmt.Println(k, "is int", vv)
			case float64:
				fmt.Println(k, "is float64", vv)
			case []interface{}:
				fmt.Println(k, "is an array", vv)
				for i, u := range vv {
					fmt.Println(i, u)
				}
			default:
				fmt.Println(k, "is of a type I don't know how to handle")
		}
	}
	
输入结果如下：

	Name is string Lynch
	Books is an array [One Two]
	0 One
	1 Two
	
对于处理未知结构的JSON，[ go-simplejson ](https://github.com/bitly/go-simplejson)很方便，请自行了解。

## 生成JSON

我们可以使用JSON包里的如下函数来生成JSON数据串：

	func Marshal(v interface{}) ([]byte, error)
	
比如我们要生成最开始的JSON串，代码如下;

	var stu Studentslice
	stu.Students = append(stu.Students, Student{Name:"Lynch", Age:"25"})
	stu.Students = append(stu.Students, Student{Name:"Wong", Age:"26"})
	b, err := json.Marshal(stu)
	if err != nil {
		fmt.Println("json err: ", err)
	}
	fmt.Println(string(b))
	
输出如下：

	{"Students":[{"Name":"Lynch","Age":"25"},{"Name":"Wong","Age":"26"}]}
	
我们发现JSON串输出的字段名都是大写，如何变成小写的，如果改成小写的，那么JSON串就不会输出该字段。这时就需要使用struct tag定义来实现；

	type Student struct {
		Name string `json:"name"`
		Age string `json:"age"`
	}
	
	type Studentslice struct {
		Students []Student `json:"students"`
	}	
	
最后输出如下：

	{"students":[{"name":"Lynch","age":"25"},{"name":"Wong","age":"26"}]}
	
针对JSON的输出，我们在定义struct tag的时候需要注意的几点是：

- 字段的tag是`"-"`，那么这个字段不会输出到JSON
- tag中带有自定义名称，那么这个自定义名称会出现在JSON的字段名中，例如上面例子中name
- tag中如果带有`"omitempty"`选项，那么如果该字段值为空，就不会输出到JSON串中
- 如果字段类型是bool, string, int, int64等，而tag中带有`",string"`选项，那么这个字段在输出到JSON的时候会把该字段对应的值转换成JSON字符串

`Marshal`函数只有在转换成功的时候才会返回数据，在转换的过程中我们需要注意几点：

- JSON对象只支持string作为key，所以要编码一个map，那么必须是map[string]T这种类型(T是Go语言中任意的类型)
- Channel, complex和function是不能被编码成JSON的
- 嵌套的数据是不能编码的，不然会让JSON编码进入死循环
- 指针在编码的时候会输出指针指向的内容，而空指针会输出null
