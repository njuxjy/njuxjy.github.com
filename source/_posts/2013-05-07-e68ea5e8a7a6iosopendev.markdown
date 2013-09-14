---
author: njuxjy
comments: true
date: 2013-05-07 15:01:24+00:00
layout: post
slug: '%e6%8e%a5%e8%a7%a6iosopendev'
title: 接触iOSOpenDev
wordpress_id: 781
categories:
- iOS
---

1. 安装时遇到问题：

    
    installd: ./postinstall: You have not agreed to the Xcode license agreements, please run xcodebuild standalone from within a Terminal window to review and agree to the Xcode license agreements.


解决办法在这：https://github.com/kokoabim/iOSOpenDev/wiki/Troubleshoot

2. 参考http://www.cnblogs.com/xiongwj0910/archive/2012/09/03/2668362.html 写个Logo Tweak。出现错误：

    
    Failed to locate Logos Processor. Is Theos installed? If not, see http://iphonedevwiki.net/index.php/Theos/Getting_Started.


解决办法：https://github.com/kokoabim/iOSOpenDev/wiki/Logos-(Theos)-Support

3. 在bash_profile中加上export iOSOpenDevDevice=ip address or host name。host name可以通过iphone上的terminal查看前缀，在后面加上.local即可。如jerrys-iphone.local。

4. Build for profile时出现ssh错误，参考https://github.com/kokoabim/iOSOpenDev/wiki/SSH-Public-Key-Authentication 做完还出现错误（原因可能跟设置了root密码有关，不设置可能就不会有下面的错误）：

    
    ssh_askpass: exec(/usr/libexec/ssh-askpass): No such file or directory


解决办法：http://rritw.com/a/caozuoxitong/OS/20121115/253801.html 中，在mac的/usr/libexec/目录下自己建立ssh-askpass脚本，内容为：

    
    #! /bin/sh
    TITLE=${MACOS_ASKPASS_TITLE:-"SSH"}
    DIALOG="display dialog \"$@\" default answer \"\" with title \"$TITLE\""
    DIALOG="$DIALOG with icon caution with hidden answer"
    result=`osascript -e 'tell application "Finder"' -e "activate"  -e "$DIALOG" -e 'end tell'`
    if [ "$result" = "" ]; then
            exit 1
    else
            echo "$result" | sed -e 's/^text returned://' -e 's/, button returned:.*$//'
            exit 0
    fi


并执行命令：

    
    sudo chmod +x /usr/libexec/ssh-askpass


再次Build for profile，搞定。
