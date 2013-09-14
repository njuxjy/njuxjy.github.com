---
author: njuxjy
comments: true
date: 2013-04-24 15:55:04+00:00
layout: post
slug: '%e5%88%9d%e5%b0%9d%e8%b6%8a%e7%8b%b1%e5%ba%94%e7%94%a8%e5%bc%80%e5%8f%91%ef%bc%9a%e7%94%a8theos%e5%86%99%e4%b8%aahello-world'
title: 初尝越狱应用开发：用Theos写个Hello World
wordpress_id: 574
categories:
- iOS
---

参考文章如下：

http://brandontreb.com/ 的 Beginning Jailbroken iOS Development系列

http://iphonesdkdev.blogspot.com/2012/06/how-to-install-thoes-under-xcode-44.html

=============

环境是Xcode4.6，SDK iOS5.1和6.1。要写个程序让系统在刚启动时弹出hello world对话框，like this:

[![](http://www.xiaojiayi.com/wp-content/uploads/2013/04/11111.jpg)](http://www.xiaojiayi.com/wp-content/uploads/2013/04/11111.jpg)

首先是安装开发工具：

    
    # clone theos.git
    cd ~
    git clone git://github.com/DHowett/theos.git
    
    # clone iphoneheaders.git
    cd ~/theos/
    mv include include.bak
    git clone git://github.com/rpetrich/iphoneheaders.git include
    for FILE in include.bak/*.h; do mv $FILE include/; done
    rmdir include.bak/
    
    # get IOSurfaceAPI.h
    cd ~/theos/include/IOSurface/
    curl -O -k https://raw.github.com/javacom/toolchain4/master/Projects/IOSurfaceAPI.h
    
    # clone theos-nic-templates.git
    cd ~/theos/templates/
    git clone git://github.com/orikad/theos-nic-templates.git
    
    # get dpkg-deb for Mac OS X
    cd ~/theos
    curl -O http://test.saurik.com/francis/dpkg-deb-fat
    chmod a+x dpkg-deb-fat
    sudo mkdir -p /usr/local/bin
    sudo mv dpkg-deb-fat /usr/local/bin/dpkg-deb
    
    # get ldid for Mac OS X
    cd ~/theos/bin
    curl -O http://dl.dropbox.com/u/3157793/ldid
    chmod a+x ldid
    
    # get libsubstrate.dylib substrate.h
    cd ~/theos
    curl -OL http://apt.saurik.com/debs/mobilesubstrate_0.9.3366-1_iphoneos-arm.deb
    dpkg-deb -x mobilesubstrate_0.9.3366-1_iphoneos-arm.deb mobilesubstrate
    cp mobilesubstrate/Library/Frameworks/CydiaSubstrate.framework/CydiaSubstrate  ~/theos/lib/libsubstrate.dylib
    cp mobilesubstrate/Library/Frameworks/CydiaSubstrate.framework/Headers/CydiaSubstrate.h include/substrate.h


一开始由于疏忽或者操作错误漏装了ldid，导致后续编译通不过，如果ldid有问题的话，还可以尝试以下：

    
    curl -s http://dl.dropbox.com/u/3157793/ldid > ~/Desktop/ldid
    chmod +x ~/Desktop/ldid
    mv ~/Desktop/ldid $THEOS/bin/ldid


接着要用Macports安装dpkg用于打debian包，装完Macports得重启terminal生效，装dpkg:

    
    sudo port install dpkg


接着配置环境变量:




    
    export THEOS=/opt/theos/
    export SDKVERSION=6.1
    export THEOS_DEVICE_IP=192.168.1.122


device_ip可以通过SBSettings(不过在我的i5里不管用)查看，或者通过设置里面直接可以查看。

接着用命令创建工程：

    
    $THEOS/bin/nic.pl






在10个模版中选择一个：

    
     [1.] iphone/application
      [2.] iphone/cydget
      [3.] iphone/dashboardx_widget
      [4.] iphone/framework
      [5.] iphone/library
      [6.] iphone/notification_center_widget
      [7.] iphone/preference_bundle
      [8.] iphone/sbsettingstoggle
      [9.] iphone/tool
      [10.] iphone/tweak






这里选择10，可以覆盖系统的一些实现。接着修改Makefile文件，将第一行的include语句改为：

    
    export ARCHS = armv7
    export TARGET=iphone:latest:5.1
    include $(THEOS)/makefiles/common.mk






关于这里我浪费了很多时间，由于之前看的教程用的是Xcode4.5和iOS6.0，而我是4.6和6.1，会出现编译错误：




    
    collect2: ld terminated with signal 6 [Abort trap: 6]
        ld(8724,0x7fff78fd2960) malloc: *** error for object 0x7f89b35003f0: pointer being freed was not allocated
        *** set a breakpoint in mallocerror_break to debug


这边target写6.1不行，得装低版本的SDK才行。接着改写Tweak.xm文件，在下面加上（其实上面都是注释）：

    
    #import <SpringBoard/SpringBoard.h>
    
    %hook SpringBoard
    
    -(void)applicationDidFinishLaunching:(id)application {
        %orig;
    
        UIAlertView *alert = [[UIAlertView alloc] initWithTitle:@"Welcome" 
            message:@"Hello world，你好世界" 
            delegate:nil 
            cancelButtonTitle:@"确定" 
            otherButtonTitles:nil];
        [alert show];
        [alert release];
    }
    
    %end






这里hook了SpringBoard类里的applicationDidFinishlaunching函数，第一行的%orig是调用了原实现，如果不调用SpringBoard就起不起来，你的手机就没发正常开机显示桌面了，得用ssh去删程序（这个没试过，不敢试...）。就这样是通不过编译的，因为用到了UIKit库中的UIAlertView，得在MakeFile里加上引入框架：

    
    HelloWorldTweak_FRAMEWORKS = UIKit




 接着可以编译打包装到手机了，在这步之前，手机得做些准备工作。插一句题外话，iOS的root默认密码是alpine，要修改的话去cydia里下了MobileTerminal，用手机上的命令行改：






    
    iPhone:~ mobile$ su root
    Password:
    iPhone:/var/mobile root# cd
    iPhone:~ root#
    iPhone:~ root# passwd
    Changing password for root.
    New password:
    Retype new password:
    iPhone:~ root#












手机要安装iSSH，或者通过其他手段允许ssh接入。最后开始最重要的工作，就一句话，包括编译打包和装机：

    
    make package install












期间会要求输几次密码，不出意外的话程序安装成功。手机会resping，好了以后就出现hello world了~

==================

感觉这样做越狱会比较麻烦，肯定有更简单的方法，搜了下，有个叫iOSOpenDev的，下次尝试下。

































