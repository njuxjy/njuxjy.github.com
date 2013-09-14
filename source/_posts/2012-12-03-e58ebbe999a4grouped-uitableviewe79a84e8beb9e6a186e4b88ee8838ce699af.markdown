---
author: njuxjy
comments: true
date: 2012-12-03 09:19:46+00:00
layout: post
slug: '%e5%8e%bb%e9%99%a4grouped-uitableview%e7%9a%84%e8%be%b9%e6%a1%86%e4%b8%8e%e8%83%8c%e6%99%af'
title: 去除Grouped UITableView的边框与背景
wordpress_id: 527
categories:
- iOS
---

viewDidLoad中，

self.tableView.backgroundView = nil; //去除table背景

[self.tableView setSeparatorColor:[UIColor clearColor]]; //去除边框

cell创建过程中，

[cell setBackgroundColor:[UIColor clearColor]];  //去除背景

然后就可以根据cell的位置是top、middle、bottom和single来贴不同的自定义背景图了。

ps: 如果只要去掉某个cell的背景和边框，可以对某个cell调用：

    
    <code>cell.backgroundView = [[[UIView alloc] initWithFrame:CGRectZero] autorelease];</code>
