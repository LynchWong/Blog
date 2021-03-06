---
title: "Svn"
date: 2014-12-13 17:04:27
categories: 
- 杂项
tags: 
- Svn
metaAlignment: center
autoThumbnailImage: no
coverImage: //d1u9biwaxjngwg.cloudfront.net/welcome-to-tranquilpeak/city.jpg

---

从开始做iOS开发后就一直用的Git，代码托管在Github上。而之前是做.Net开发，用的VS2010。团队协作，代码托管都是用的TFS，不知道是不是叫这个，已经是很久之前的事了。
<!--more-->

TFS用起来还是挺方便的，但是后来换了公司。新公司架构师是做Ruby的，大家都知道最开始将代码托管在Github的那群人就是Ruby社区的。所以公司所有的代码都托管在Github上。

刚转到Git时，各种不适应，但公司其他人都是命令行操作，所以也只能硬着头皮学了。熟悉了之后还是觉得挺好用的，用了没多久又换公司了，所以你懂的，这次的公司是用Svn的。

### Svn常用命令

##### 1、将文件checkout到本地目录

svn checkout path（path是服务器上的目录）

简写：svn co


##### 2、往版本库中添加新的文件

svn add file

##### 3、将改动的文件提交到版本库

svn commit -m “LogMessage” [-N] [--no-unlock] PATH(如果选择了保持锁，就使用–no-unlock开关)

简写：svn ci

##### 4、加锁/解锁

svn lock -m “LockMessage” [--force] PATH

svn unlock PATH

##### 5、更新到某个版本

svn update -r m path

简写：svn up

##### 6、查看文件或者目录状态

1）svn status path（目录下的文件和子目录的状态，正常状态不显示）

2）svn status -v path(显示文件和子目录状态)

简写：svn st

##### 7、删除文件

svn delete path -m “delete test fle”

简写：svn (del, remove, rm)

##### 8、查看日志

svn log path

##### 9、查看文件详细信息

svn info path

##### 10、比较差异

svn diff path(将修改的文件与基础版本比较)

svn diff -r m:n path(对版本m和版本n比较差异)

简写：svn di

##### 11、将两个版本之间的差异合并到当前文件

svn merge -r m:n path

##### 12、SVN 帮助

svn help

svn help ci

##### 13、svn cleanup

当Subversion修改你的工作副本时（或者任何在.svn中的信息），它尝试尽可能做到安全。在改变一个工作副本前，Subversion把它的意 图写到一个日志文件中。接下来它执行日志文件中的命令来应用要求的修改。最后，Subversion删除日志文件。从架构上来说，这与一个日志文件系统 （journaled filesystem）类似。如果一个 Subversion操作被打断（例如，进程被杀掉了，或机器当掉了）了，日志文件仍在硬盘上。重新执行日志文件，Subversion可以完成先前开始 的操作，这样你的工作副本能回到一个可靠的状态。 

以下是svn cleanup所做的：它搜索你的工作副本并执行所有遗留的日志，在这过程中删除锁。如果Subversion曾告诉你你的工作副本的一部分被“锁定”了，那么你应该执行这个命令。另外， svn status会在锁定的项前显示L。 

$ svn status

L    somedir

M   somedir/foo.c 


$ svn cleanup

$ svn status

M      somedir/foo.c

##### 14、svn import

使用svn import是把未版本化的文件树复制到资料库的快速办法，它需要创建一个临时目录。 

$ svnadmin create /usr/local/svn/newrepos

$ svn import mytree file:///usr/local/svn/newrepos/some/project

Adding         mytree/foo.c

Adding         mytree/bar.c

Adding         mytree/subdir

Adding         mytree/subdir/quux.h

Committed revision 1.

上面的例子把在some/project目录下mytree目录的内容复制到资料库中。 

$ svn list file:///usr/local/svn/newrepos/some/project
bar.c
foo.c
subdir/

注意在导入完成后，原来的树没有被转化成一个工作副本。为了开始工作，你仍然需要svn checkout这个树的一个新的工作副本。
