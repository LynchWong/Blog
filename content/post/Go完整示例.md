---
title: "Go完整示例"
date: 2015-06-27 11:17:45
categories: 
- Go
tags: 
- Go语言编程
metaAlignment: center
autoThumbnailImage: no
coverImage: //d1u9biwaxjngwg.cloudfront.net/welcome-to-tranquilpeak/city.jpg

---

<!--more-->

在开始之前还是讲下Go编程环境以及开发工具的配置和使用。

# 安装Go

操作系统是Yosemite 10.10.3。

安装方式有很多，博主是直接使用的一键安装包。安装包这里就不提供了，大家网上自行搜索下载，安装包截图如下：

![alt text](/img/GoFullDemo/1.png)

安装成功之后在终端中输入**go**命令能看到Usage信息，截图如下：

![alt text](/img/GoFullDemo/2.png)

# 配置开发环境

IDE博主使用的是IntelliJ IDEA 14 CE，截图如下，即下载的是右边的这个免费的社区版。

![alt text](/img/GoFullDemo/3.png)

安装完成之后这个IDE还并不能创建Go的Project，还需要插件的支持。

打开IDE后**⌘,**打开偏好设置，如下图所示：

![alt text](/img/GoFullDemo/4.png)

大家看我已经装好了。点击图中高亮的那个按钮**Browse Repositories**，弹出如下界面：

![alt text](/img/GoFullDemo/5.png)

如果大家直接搜索*Go*是搜索不到，点击图中高亮的按钮**Manage Repositories**，界面如下：

![alt text](/img/GoFullDemo/6.png)

点击**+**号添加这个地址**https://plugins.jetbrains.com/plugins/alpha/5047**：

![alt text](/img/GoFullDemo/7.png)

点击OK，然后就能找到插件了，点击安装，安装完成后会要求重启：

![alt text](/img/GoFullDemo/8.png)

重启完成后我们创建新项目，发现就有了**Go**的选项：

![alt text](/img/GoFullDemo/9.png)

## 设置GOPATH

如果你这时候新建项目，如下所示IDE会提醒你没有设置**GOPATH**。

![alt text](/img/GoFullDemo/10.png)

**GOPATH**其实就是一个目录而已，按照约定该目录下默认有三个文件夹：

* src存放源代码
* pkg编译后生成的文件
* bin编译后生成的可执行文件

博主使用IntelliJ IDEA 14 CE的工作空间**IdeaProjects**当做GOPATH：

![alt text](/img/GoFullDemo/11.png)

同时创建了三个子文件夹(MyGo.iml请忽略)：

![alt text](/img/GoFullDemo/12.png)

所以博主的GOPATH就是**/Users/Lynch/IdeaProjects**，如下终端所示：

![alt text](/img/GoFullDemo/13.png)

现在我们就开始设置GOPATH，把**export GOPATH=/Users/Lynch/IdeaProjects**这一行加到.bash_profile或者.zshrc中，加到哪一个里面取决于你使用的是什么Shell。OS X默认使用的是bash，而你没有修改过，那就加到.bash_profile中。博主使用的是zsh，所以加入到了.zshrc。输入**vim .zshrc**命令进行编辑：

![alt text](/img/GoFullDemo/14.png)

如图所示，设置好后保存退出，ESC然后输入“:wq”。然后输入**go env**命令查看：

![alt text](/img/GoFullDemo/15.png)

如图所示GOPATH已经设置好了。

推荐大家使用zsh，你可以使用**cat /etc/shells**来查看系统支持的Shell。OS X默认是支持zsh的，如下所示：

![alt text](/img/GoFullDemo/16.png)

使用**chsh -s /bin/zsh**命令切换到想要使用的Shell，这里我们切换到zsh。然后安装**oh my zsh**，之后进行配置。

你可以安装很多插件，比如**autojump**这个插件，太强悍了，对于像博主这样的手残党来说真是太方便。简单来说就是在目录之间进行快速跳转，比如我经常到Hexo目录下，那么我只需要输入**j h**就能跳转到这个目录，如下所示：

![alt text](/img/GoFullDemo/17.png)

以及提示，自动补全，git支持等，这里值简单提一下，安装使用，大家自行了解。

现在我们来创建一个新项目，Project name叫做“MyGo”，Project location是我们之前设置好的GOPATH，如下所示：

![alt text](/img/GoFullDemo/18.png)

最后的效果图如下：

![alt text](/img/GoFullDemo/19.png)

# 完整示例

现在我们就来做一个简单的示例程序。根据之前设置GOPATH时的叙述，源代码应该都放在src文件夹中。所以我们在src中新建一个文件夹，如下所示：

![alt text](/img/GoFullDemo/20.png)

然后在新建的文件夹中新建一个Go的源文件：

![alt text](/img/GoFullDemo/21.png)

代码很简单，之前就讲解过，如果有没有使用的包，编译器会报错。右键**main.go**：

![alt text](/img/GoFullDemo/22.png)

这是可以看到IDE输出了“Hello world”：

![alt text](/img/GoFullDemo/23.png)

你也可以使用go命令来执行这些操作，输入**go build sampledemo**命令会在当前目录生成一个可执行文件，如下所示：

![alt text](/img/GoFullDemo/24.png)

![alt text](/img/GoFullDemo/25.png)

双击执行：

![alt text](/img/GoFullDemo/26.png)

你可以在任意的目录执行**go build sampledemo**；也可以进入应用包目录，执行**go build**。

因为这里是main包所以生成了可执行文件。如果是其他的包，不会生成任何东西。所以对于普通包，你需要在pkg下生成相应的文件，就得执行**go install**文件。对于main包，你需要在bin下生成相应的文件你需要执行**go install**。

然后我们在sampledemo下创建新的文件夹**utils**，然后创建plus.go源文件，如下所示：

![alt text](/img/GoFullDemo/27.png)

这里需要注意的就是函数名要大写，因为我们要在包外调用这个函数。这个函数简单实现了加法运算。

我们现在先来编译utils包，直接两条命令，貌似**go install sampledemo/utils**就够了：

![alt text](/img/GoFullDemo/28.png)

编译完成之后你能看见pkg下生成了utils.a的函数包：

![alt text](/img/GoFullDemo/29.png)

如图所示，修改main.go文件：

![alt text](/img/GoFullDemo/30.png)

**go build sampledemo**重新编译，双击执行：

![alt text](/img/GoFullDemo/31.png)

# main函数和init函数

Go里面有两个保留的函数：init函数（能够应用于所有的package）和main函数（只能应用于package main）。这两个函数在定义时不能有任何的参数和返回值。虽然一个package里面可以写任意多个init函数，但这无论是对于可读性还是以后的可维护性来说，我们都强烈建议用户在一个package中每个文件只写一个init函数。

Go程序会自动调用init()和main()，所以你不需要在任何地方调用这两个函数。每个package中的init函数都是可选的，但package main就必须包含一个main函数。

程序的初始化和执行都起始于main包。如果main包还导入了其它的包，那么就会在编译时将它们依次导入。有时一个包会被多个包同时导入，那么它只会被导入一次（例如很多包可能都会用到fmt包，但它只会被导入一次，因为没有必要导入多次）。当一个包被导入时，如果该包还导入了其它的包，那么会先将其它包导入进来，然后再对这些包中的包级常量和变量进行初始化，接着执行init函数（如果有的话），依次类推。等所有被导入的包都加载完毕了，就会开始对main包中的包级常量和变量进行初始化，然后执行main包中的init函数 (如果存在的话) ，最后执行main函数。
下图详细地解释了整个执行过程：

![alt text](/img/GoFullDemo/32.png)

# import

我们在写Go代码的时候经常用到import这个命令来导入包文件，而我们经常看到的方式参考如下：

	import(
		"fmt"
	)
	
然后我们代码里可以通过如下的方式调用

	fmt.Println("hello world")
	
上面这个fmt是Go语言的标准库，其实是去GOROOT环境变量指定目录下去加载该模块，当然Go的import还支持如下两种方式来加载自己写的模块：

## 相对路径

	import "./model"
	
当前文件同一目录的model目录，但是不建议这种方式来import。

## 绝对路径

	import "shorturl/model"
	
加载GOPATH/src/shorurl/model模块。

上面展示了一些import常用的几种方式，但是还有一些特殊的import，让很多新手很费解，下面我们来--讲解。

## 点操作

我们有时候会看到如下的方式导入包：

	import(
		."fmt"
	)
	
这个点操作的含义就是这个包导入之后在你调用这个包的函数时，你可以省略前缀的包名，也就是前面你调用的fmt.Println(“hello world”)可以省略的写成Println(“hello world”)。

## 别名操作

别名操作顾名思义我们可以把包命名成另一个我们用起来容易记忆的名字：

	import(
		f"fmt"
	)
	
别名操作的话调用函数时前缀变成了我们的前缀，即f.Println("hello world")。

## _操作

这个操作经常是让很多人费解的一个操作符，请看下面这个import：

	import(
		"database/sql"
		_"github.com/ziutek/mymysql/godrv"
	)
	
_操作其实是引入该包，而不直接使用包里面的函数，而是调用了该包里面的init函数。
