---
author: njuxjy
comments: true
date: 2013-03-06 02:10:01+00:00
layout: post
slug: ios%e6%80%a7%e8%83%bd%e4%bc%98%e5%8c%96%ef%bc%88wwdc%e7%ac%94%e8%ae%b0%ef%bc%89
title: iOS性能优化（WWDC笔记）
wordpress_id: 551
categories:
- iOS
---

程序启动的几个阶段：链接和加载， UIKit初始化，应用回调，第一次的Core Animation transaction




### 1. 启动速度优化：




方法一：测量启动时间，在main里面第一行记录开始时间，在didfinishLaunching第一行记录结束时间。




方法二：使用Time Profiling工具




     1）链接和加载阶段。减少不必要的框架引用，可以减少程序启动时链接和加载的时间；不要将必要的框架设置为optional，因为optional会让链接器做额外工作；避免使用静态初始化，包括静态c++对象，加载时会运行的代码，如+(void) load{} ，会造成在Main函数之前运行额外的代码，




     2）UIKit初始化阶段。减少main nib文件的大小； 在preferences中不要存入太多的数据，比如在某个Key中存入一个很大的NSData对象如图片，因为preferences是用plist来保存的，所有的plist都是一次性反序列化出来的；




     3) 应用回调阶段。UIKit会首先调用willfinishLaunchingWithOptions，然后恢复应用状态，然后调用didfinishLaunchingWithOptions。




     4) 最后阶段。在_reportAppLaunchFinished中会调用CA::Transaction::commit()。 










一些戒律：




用工具度量，不要去猜。




 不要做比不必要的工作。不必要的shadows和masks，同一数据的多次查询，启动时无数的logging。




重用，而不是重复。重用那些创建代价昂贵的类：tablecell，date/number formatters，正则表达式，sqlite语句。[NSCalendar currentCalendar]看起来像是个单例，其实不是，每次调用都会创建一个新对象。调用nslog的时候会创建一个calendar对象，不要过度nslog。 每次使用sqlite_prepare会将SQL查询编译成字节码，要使用bind，重用那些已经prepared的语句。




提高代码效率。要选择合适的数据结构和算法。 1）选择正确的数据类型。少量数据用plist，大量数据用core data或者sqlite。由于要访问plist中某个数据需要反序列化整个表，所以只能存少量数据。一些API底层使用plist实现的：preferences,通过NSCoding序列化的数据。2）优化数据库查询。用sqlite3_trace和sqlite3_profile来查找性能差的语句。




预计算结果存入内存或者磁盘。1）注意内存增长。如screenSizedImage就不适合用static来一直保存在内存里。  




异步加载。1）GCD等API




保持在大数据量下伸缩性良好。如iPhone上的通讯录程序在通讯录里的人变多时启动时间控制的不错。1）保持关键方法快速。比如在Table加载sections时，numberofsectionsintableview,titleforheaderinsecion,numberofrowsinsection等方法就要性能好。 







### 2. 用户事件处理优化：




     1）保持主线程用来响应用户事件。




          a) 在主线程中加快CPU处理速度，比如UILabel的cache处理；




          b) 将事情移到主线程外面来做。隐式并发和显式并发。隐式：view和layer的动画，layer的组合，PNG解码；显式：GCD，NSOperationQueue，NSThread。




          c)不要block掉主线程。







### 3. 内存优化




     1）系统如何回收内存？不是把page放到disk，而是直接扔掉不保存。 




     2）干净内存和脏内存？干净：磁盘上存在的一些东西的拷贝，如代码、框架、内存映射文件。 脏：其他所有，如堆分配、解压的图片、数据库缓存。 NSString * a = [NSString stringWithFormat:@"hello"];是dirdy；NSString * a = @"hello"是clean。




     3）在你重复一个操作的时候，内存不应该持续增长。比如：push然后pop一个view controller，滚动table，执行数据库查询。
