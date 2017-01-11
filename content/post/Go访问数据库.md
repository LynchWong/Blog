---
title: "Go访问数据库"
date: 2015-07-02 09:16:56
categories: 
- Go
tags: 
- Go语言编程
metaAlignment: center
autoThumbnailImage: no
coverImage: //d1u9biwaxjngwg.cloudfront.net/welcome-to-tranquilpeak/city.jpg

---

大部分应用程序都需要数据库的支持，用来存储和查询信息，比如用户数据，产品目录，新闻列表等。
<!--more-->

其实访问数据库没什么难的，都有相关的数据库驱动，参照文档完全没有问题。本篇也就简单讲讲Go访问MongoDB，MongoDB是非关系型数据库。像MySQL、SQL Server等一些关系型数据库怎么访问就不涉及了，毕竟基础的增删改查这些操作很简单。

我们先从安装MongoDB开始。

# MongoDB

MongoDB是目前在IT行业非常流行的一种非关系型数据库(NoSql)，其灵活的数据存储方式备受当前IT从业人员的青睐。MongoDB很好的实现了面向对象的思想(OO思想)，在MongoDB中每一条记录都是一个Document对象。Mongo DB最大的优势在于所有的数据持久操作都无需开发人员手动编写SQL语句，直接调用方法就可以轻松的实现CRUD操作。

## 安装MongoDB

我是参照官方文档[ Install MongoDB on OS X ](http://docs.mongodb.org/manual/tutorial/install-mongodb-on-os-x/)安装的，是手动安装的。

通过shell下载，敲入如下命令：

	curl -O https://fastdl.mongodb.org/osx/mongodb-osx-x86_64-3.0.4.tgz
	tar -zxvf mongodb-osx-x86_64-3.0.4.tgz
	mkdir -p mongodb
	cp -R -n mongodb-osx-x86_64-3.0.4/ mongodb
	
执行完这些命令后在当前目录下生成了一个mongodb的文件夹，如下所示：
	
![alt text](/img/GoDatabaseMongoDB/1.png)

然后我们设置PATH变量，和之前设置GOPATH类似的，我用的shell是zsh，所以我在.zshrc中设置,如下所示：

![alt text](/img/GoDatabaseMongoDB/2.png)

如果你不设置这个PAHT，那么你每次执行命令都要在前面加上PATH。比如我就需要每次在执行命令前加上**/Users/Lynch/mongodb/bin**。

## 运行MongoDB

在第一次运行之前，你应该创建一个mongod进程写数据的目录，默认使用/data/db目录。如果你设置了其他的，那么你在启动mongod进程的时候在**dbpath**选项指定你设置的目录。

	mkdir -p /data/db
	
如果你使用上面的命令创建了/data/db的目录，那么这个就是默认的目录。你在启动mongod进程的时候不需要在指定**dbpath**选项，直接输入下面的命令就可以启动：

	mongod
	
注意：我这里是设置过mongod的PATH的，如之前安装的时候说明的。如果你没设置需要在前面加上PATH，请参看安装的最后一步。

博主使用默认的/data/db的时候有些权限问题，MongoDB官方文档也说了要确保你的账户有这个目录的读写权限。

博主将/data/db目录放在了我们安装好的mongodb文件夹里面了，如下所示：

![alt text](/img/GoDatabaseMongoDB/3.png)

所以我使用如下命令启动：

	mongod --dbpath /Users/Lynch/mongodb/data/db
	
如下所示：
	
![alt text](/img/GoDatabaseMongoDB/4.png)

![alt text](/img/GoDatabaseMongoDB/5.png)

可以看见提示说在端口27017等待连接了，你可以在这里使用Control + C来退出。

## 使用MongoDB

如果你退出了，先启动起来。首先我们应该连接到数据库，启动之后输入如下命令：

	mongo
	
如下所示，已经连接上了：

![alt text](/img/GoDatabaseMongoDB/6.png)

默认连接使用test。这里有个警告，官方说是操作系统的原因，和MongoDB无关，所以这里就不管了。

你可以输入`show dbs`来查看数据库，如下所示：

![alt text](/img/GoDatabaseMongoDB/7.png)

这里有些概念需要悉知：数据库，集合，文档。这里集合就相当于我们关系型数据库里面的表，而每一条数据都是一个文档，而文档是JSON的拓展(BSON)形式。

这里我们创建一个"student"的集合：

### insert操作

输入如下命令：

	db.student.insert({"name":"Wong", "age":25})
	
如下所示，返回了写入结果：

![alt text](/img/GoDatabaseMongoDB/8.png)

### find操作
	
	db.student.find({"name":"Wong"})
	
返回结果如下所示：

![alt text](/img/GoDatabaseMongoDB/9.png)

### update操作

	db.student.update({"name":"Wong"},{"name":"Wong","age":26})
	
![alt text](/img/GoDatabaseMongoDB/10.png)

然后查看：

	db.student.find({"name":"Wong"})	
	
![alt text](/img/GoDatabaseMongoDB/11.png)

### remove操作

	db.student.remove({"name":"Wong"})
	
![alt text](/img/GoDatabaseMongoDB/12.png)

你可以使用`db.student.count()`来查看数据条数。

### 关闭MongoDB

除了之前我们使用Control + C退出之外，你还可以使用如下命令来关闭MongoDB：

	use admin
	db.shutdownServer()
	
如下所示：
	
![alt text](/img/GoDatabaseMongoDB/13.png)

这些只是一些简单的操作，冰山一角。详细复杂高端的操作还请自行了解、学习。

# Go访问MongoDB

Go的MongoDB最好的驱动就是[ mgo ](http://labix.org/mgo)。

使用如下命令安装mgo：

	go get gopkg.in/mgo.v2
	
安装完成后在我们GOPATH的pkg目录下就有了相应的包。

	package main
	
	import (
	        "fmt"
		"log"
	        "gopkg.in/mgo.v2"
	        "gopkg.in/mgo.v2/bson"
	)
	
	type Person struct {
	        Name string
	        Phone string
	}
	
	func main() {
	        session, err := mgo.Dial("server1.example.com,server2.example.com")
	        if err != nil {
	                panic(err)
	        }
	        defer session.Close()
	
	        // Optional. Switch the session to a monotonic behavior.
	        session.SetMode(mgo.Monotonic, true)
	
	        c := session.DB("test").C("people")
	        err = c.Insert(&Person{"Ale", "+55 53 8116 9639"},
		               &Person{"Cla", "+55 53 8402 8510"})
	        if err != nil {
	                log.Fatal(err)
	        }
	
	        result := Person{}
	        err = c.Find(bson.M{"name": "Ale"}).One(&result)
	        if err != nil {
	                log.Fatal(err)
	        }
	
	        fmt.Println("Phone:", result.Phone)
	}
	
如上代码是mgo给的示例，下面我们新建一个项目，将上面的代码添加进去，如下所示：

![alt text](/img/GoDatabaseMongoDB/14.png)

如果你现在就Run，一段时间后就得到如下错误：

![alt text](/img/GoDatabaseMongoDB/15.png)

我们应该把`mgo.Dial("server1.example.com,server2.example.com")`修改成`mgo.Dial("127.0.0.1:27017")`，然后还要打开MongoDB，最后结果如下所示：

![alt text](/img/GoDatabaseMongoDB/16.png)

