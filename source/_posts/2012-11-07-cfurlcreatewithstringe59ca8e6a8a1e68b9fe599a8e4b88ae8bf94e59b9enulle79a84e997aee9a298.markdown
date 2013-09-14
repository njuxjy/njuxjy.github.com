---
author: njuxjy
comments: true
date: 2012-11-07 04:50:45+00:00
layout: post
slug: cfurlcreatewithstring%e5%9c%a8%e6%a8%a1%e6%8b%9f%e5%99%a8%e4%b8%8a%e8%bf%94%e5%9b%9enull%e7%9a%84%e9%97%ae%e9%a2%98
title: CFURLCreateWithString在模拟器上返回null的问题
wordpress_id: 524
categories:
- iOS
---

在程序中用到了

CFURLRef  url = CFURLCreateWithString(kCFAllocatorDefault, (CFStringRef)fileName, NULL);

其中fileName的值是应用目录下的某个文件，在真机上一切正常，但在模拟器上url始终为Null。

最后查出是因为fileName的值在模拟器上是“~/Library/Application Support/iPhone Simulator/.../”，路径中间有空格，而真机上没有，所以要对fileName中的空格进行处理：

CFStringRef fileNameEscaped = CFURLCreateStringByAddingPercentEscapes(NULL, (CFStringRef)fileName, NULL, NULL, kCFStringEncodingUTF8);

这下就对了。
