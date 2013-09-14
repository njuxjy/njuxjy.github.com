---
author: njuxjy
comments: true
date: 2013-05-02 10:57:53+00:00
layout: post
slug: android%e5%bc%80%e5%8f%91%ef%bc%88%e4%ba%8c%ef%bc%89%e5%9c%a8android%e5%ba%94%e7%94%a8%e7%a8%8b%e5%ba%8f%e4%b8%ad%e4%bd%bf%e7%94%a8%e5%8a%a0%e9%80%9f%e5%ba%a6%e4%bc%a0%e6%84%9f%e5%99%a8
title: (旧文迁移) Android开发（二）——在Android应用程序中使用加速度传感器
wordpress_id: 648
categories:
- 技术
---

最近工作需要看了点传感器的东西，顺便熟悉了下Android平台上开发传感器程序的流程，作为借鉴。好久没用Android，竟然发现搭环境也有些生疏了，于是打算从如何搭建环境写起，也算是复习一遍，题目就叫《从零开始构建Android加速度传感器程序》。 

 

由于国内Android官网（developer.android.com）没法访问（当然google下还是能找到访问的办法），图省事就找了公司文件服务器上的老版本sdk（android-sdk-1.5_r2-windows）装上，结果发现只要一用SensorManager.getSystemService(SENSOR_SERVICE)这句，程序就死了，折腾半天发现是sdk版本问题，说是1.5r3版本（不包括r3）以前的版本在传感器这块有bug，在r3开始以后的版本中已经修复，又搜遍了服务器没发现有更新的版本，只好自己去网上（http://androidappdocs.appspot.com/）下一个。 更多的废话请看下面…… 

 

======================================================================= 

 

**配置Android****开发环境**

 

1. 下载eclipse3.5版（当然其他版本也可以，文中用的是此版本，其他版本情况基本类似）。当然jdk之类的是先要安装的。 

 

2. 在这里（http://androidappdocs.appspot.com/sdk/index.html）下载android sdk的安装程序[![clip_image002](http://www.xiaojiayi.com/wp-content/uploads/2013/05/clip_image002_thumb.jpg)](http://www.xiaojiayi.com/wp-content/uploads/2013/05/clip_image002.jpg)

 

并点AndroidSDKSetup.exe进行安装，注意勾选这个： 

 

[![clip_image004](http://www.xiaojiayi.com/wp-content/uploads/2013/05/clip_image004_thumb.jpg)](http://www.xiaojiayi.com/wp-content/uploads/2013/05/clip_image004.jpg)

 

这玩意的安装过程就是从网上下载你选择的sdk放到它自己的目录里，所以装完后你将这整个文件android-sdk_r5-windows随便放到你喜欢的路径下，并在环境变量Path后加上“……\android-sdk-windows\tools” 

 

3. 接下来要安装的是ADT（Android Development Tool），是个eclipse插件，所以可以依次在eclipse里点击Help->Install new software->Add，按下图所示输入Name和Location： 

 

[![clip_image006](http://www.xiaojiayi.com/wp-content/uploads/2013/05/clip_image006_thumb.jpg)](http://www.xiaojiayi.com/wp-content/uploads/2013/05/clip_image006.jpg)

 

OK，在下拉列表里选择刚加入的站点，并勾选要下载的文件： 

 

[![clip_image008](http://www.xiaojiayi.com/wp-content/uploads/2013/05/clip_image008_thumb.jpg)](http://www.xiaojiayi.com/wp-content/uploads/2013/05/clip_image008.jpg)

 

一路next。要是有问题把站点路径的https改成http。要是还有问题的话，就先下载ADT的zip包（Android2.1要求ADT-0.9.6，在http://androidappdocs.appspot.com/sdk/eclipse-adt.html上有下载），然后在刚刚输入Location的界面点击Archive选择zip包安装： 

 

[![clip_image010](http://www.xiaojiayi.com/wp-content/uploads/2013/05/clip_image010_thumb.jpg)](http://www.xiaojiayi.com/wp-content/uploads/2013/05/clip_image010.jpg)

 

4. 装完ADT，eclipse依次点击Window->Preferences->Android，选择SDK根目录。这样便装完了，可以新建个Android Project跑个helloworld看看效果。 

 

ps:      
1. 要是启动simulator有问题，在命令行中输入android create avd –name mySimulatorName –target 2先创建AVD。 

 

2. 要是控制台出现错误信息Android requires .class compatibility set to 5.0. Please fix project properties，选择 project -> Android Tools ->Fix Project Properties。 

 

3. Android界面可视化编辑工具——>DroidDraw，可以拖出界面，生成的XML布局文件拷贝进自己的工程里。 

 

4. 模拟器要记得定时清理文件，不然有可能会占很多空间：c:\documents and settings\Administrator\Local Settings\Temp\AndroidEmulator中的.tmp文件都可以删除。 

 

5. 每个SDK版本都有其相应的ADT版本。 

 

**简单的helloworld****程序**

 

1.[![clip_image012](http://www.xiaojiayi.com/wp-content/uploads/2013/05/clip_image012_thumb.jpg)](http://www.xiaojiayi.com/wp-content/uploads/2013/05/clip_image012.jpg)

 

[![clip_image014](http://www.xiaojiayi.com/wp-content/uploads/2013/05/clip_image014_thumb.jpg)](http://www.xiaojiayi.com/wp-content/uploads/2013/05/clip_image014.jpg)

 

2. 包的目录结构如下所示： 

 

[![clip_image016](http://www.xiaojiayi.com/wp-content/uploads/2013/05/clip_image016_thumb.jpg)](http://www.xiaojiayi.com/wp-content/uploads/2013/05/clip_image016.jpg)

 

3. 对项目右键Run as-> Android Application，等模拟器启动，第一次启动要很久（google上有解决办法），启动完毕运行应用程序，Hello World ! 

 

[![clip_image018](http://www.xiaojiayi.com/wp-content/uploads/2013/05/clip_image018_thumb.jpg)](http://www.xiaojiayi.com/wp-content/uploads/2013/05/clip_image018.jpg)

 

ps:     
1. 如何看到System.out.println打印的信息？ 

 

不像普通的java程序可以在控制台中看到，我们需要在logcat里查看调试信息。具体方法如下： 

 

1) 点击Window->Show View->Other->Android->Logcat 

 

[![clip_image020](http://www.xiaojiayi.com/wp-content/uploads/2013/05/clip_image020_thumb.jpg)](http://www.xiaojiayi.com/wp-content/uploads/2013/05/clip_image020.jpg)

 

2) 于是在应用程序运行过程中所有的调试信息都可以在这里看到： 

 

[![clip_image022](http://www.xiaojiayi.com/wp-content/uploads/2013/05/clip_image022_thumb.jpg)](http://www.xiaojiayi.com/wp-content/uploads/2013/05/clip_image022.jpg)

 

2. 在这里（http://androidappdocs.appspot.com/reference/packages.html）可以查看到Android API。 

 

**用SensorSimulator****向Android****模拟器发送模拟信号**

 

Android中传感器的类型有方向、加速度、光线、磁场、临近性、温度等，本文以较有代表性的加速度传感器为例，旨在说清传感器应用开发的流程。 

 

我们要想办法向Android模拟器传送加速度改变的信号，以便让应用程序监控到事件并作出响应。Android SDK中貌似没有和模拟器搭配使用的类似工具，幸运的是国外有个OpenIntents团队开发的Sensor simulator工具（可以在http://code.google.com/p/openintents/downloads/ list上找到sensorsimulator-1.0.0-beta1.zip下载，在http://openintents.org/en/node/6上可以看到在线的Applet演示程序）可以解决这个问题。 

 

关于加速度传感器是怎么回事，这里就不多解释了，有兴趣可以参考下这个网页（说的是iphone的，但Android原理类似，http://www.riameeting.com/node/538）。 

 

首先介绍下加速度传感器Demo的应用场景：把手机当做一个平面，平面上有个小球在滚动，Demo中模拟出现实世界球在重力作用下的滚动轨迹。由于没有真机，要在模拟器上实现，所以必须有工具能制造出使模拟器“上下左右前后旋转”的错觉，Sensor simulator便是通过调整虚拟手机的摆放位置向Android模拟器传达信息。 

 

下面介绍如何使用Sensor Simulator： 

 

1. 将文件解压，打开bin下的sensorsimulator.jar 

 

2. 将bin下的SensorSimulatorSettings.apk安装到模拟器中，具体方法是先打开模拟器，然后在命令行中输入adb install SensorSimulatorSettings.apk，如图安装成功： 

 

[![clip_image024](http://www.xiaojiayi.com/wp-content/uploads/2013/05/clip_image024_thumb.jpg)](http://www.xiaojiayi.com/wp-content/uploads/2013/05/clip_image024.jpg)

 

3. 在模拟器中打开Sensor Simulator Settings，在模拟器中将IP配置为和PC上Sensor Simulator中的IP相同，如图所示： 

 

[![clip_image026](http://www.xiaojiayi.com/wp-content/uploads/2013/05/clip_image026_thumb.jpg)](http://www.xiaojiayi.com/wp-content/uploads/2013/05/clip_image026.jpg)

 

4. 在Sensor Simulator Settings中点击Testing，点击Connect，勾选几种传感器类型使之enable，如下图所示： 

 

[![clip_image028](http://www.xiaojiayi.com/wp-content/uploads/2013/05/clip_image028_thumb.jpg)](http://www.xiaojiayi.com/wp-content/uploads/2013/05/clip_image028.jpg)

 

**加速度传感器演示程序**

 

Sensor Simulator配置完毕了，我们可以建个小项目来验证下： 

 

1. 新建个SensorBall的项目，通过Add jars的方法将sensorsimulator-1.0.0-beta1\lib下的sensorsimulator-lib.jar添加到工程中来 

 

2. 在AndroidManifest.xml中添加 

 

<uses-permission android:name="android.permission.INTERNET"/>

 

3. 跟传统的mSensorManager = (SensorManager) getSystemService(SENSOR_SERVICE);方式获取传感器服务不同，这里要用 

 

mSensorManager = SensorManagerSimulator.getSystemService(this, SENSOR_SERVICE);这种方式 

 

4. 使用mSensorManager.connectSimulator();连接到Sensor simulator 

 

由于代码不是自己写的，网上看到的，就不在这里贴代码了，有兴趣的可以参考这里： 

 

http://ophonesdn.com/article/show/183 

 

最后的效果如下图所示： 

 

[![clip_image030](http://www.xiaojiayi.com/wp-content/uploads/2013/05/clip_image030_thumb.jpg)](http://www.xiaojiayi.com/wp-content/uploads/2013/05/clip_image030.jpg)

 

当调整Sensor simulator中虚拟手机位置时，Android模拟器中的小球位置也会发生变化。 

 

关于Sensor simulator的更详细的介绍和使用方法，可以参考下面这两个网页：http://openintents.org/en/node/23     
http://code.google.com/p/openintents/wiki/SensorSimulator 

 

（完） 
