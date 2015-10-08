---
layout:     post
title:      build protobuf on Mac
date:       2015-10-08 9:32:18
summary:    build Google Protocol Buffer
categories: protobuf
---

十一之前编译了protobuf给iOS用，但是网上的教程良莠不齐，这次做一个总结，也给自己一个记录。前面部分是可以方便的编译完然后立刻使用，后面部分是详细的叙述。

###前提准备

1 一个针对sh脚本 地址在这里*[https://gist.github.com/BennettSmith/9487468ae3375d0db0cc](https://gist.github.com/BennettSmith/9487468ae3375d0db0cc)*
*推荐使用这个脚本，因为如果你赶时间，这个会让你很方便。*

2.一个过墙的命令行代理（最好有）
   


###编译
	$ cd build-protobuf
	$ sudo ./build-protobuf.sh
可以看到这个脚本中还有从github/google/protobuf/release中下载源码，所以整个过程需要一些时间。

成功之后会.a文件会在 当前目录的生成一个目录 protobuf-2.x.x
现在的版本号为2.6.1。一共有3个文件夹,目录树如下。

	.
	├── bin
	├── include
	│   └── google
	└── lib
	    ├── libprotobuf-lite.a
	    ├── libprotobuf-lite.la
	    ├── libprotobuf.a
	    ├── libprotobuf.la
	    ├── libprotoc.a
	    ├── libprotoc.la
	    └── pkgconfig

###使用
1. 在Header Search Paths 处添加 /usr/local/include
2. libprotobuf.a拖入Xcode 

然后没问题就可以运行了。

感谢脚本的作者。

###可能碰到的问题

之后补上。
  

