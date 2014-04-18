---
layout: post
title: "给第三方库打patch"
date: 2014-04-18 15:36
comments: true
categories: 
---

标题有个前提，就是在不修改第三方库代码的基础上。

项目中用到的第三方库，有时候不能满足项目的特定需求，比如我们项目中用到的TTTAttributedLabel, OLImageView和AFNetworking。都能在github上找到。

###1. TTTAttributedLabel

TTTAttributedLabel是增强版的UILabel，它有一个属性links如下定义：

```
@property (readonly, nonatomic, strong) NSArray *links;
```

并且提供了接口往这个属性里增加元素：

```
- (void)addLinkToURL:(NSURL *)url
           withRange:(NSRange)range
```

但是没有清空里面元素的接口，没有的话就自己创造一个。首先创建TTTAttributedLabel的一个分类TTTAttributedLabel+RemoveLinks，增加方法brk_removeAllLinks:

```
- (void)brk_removeAllLinks
{
#pragma clang diagnostic push
#pragma clang diagnostic ignored "-Warc-performSelector-leaks"
    SEL setLinksSelector = NSSelectorFromString(@"setLinks:");
    if([self respondsToSelector:setLinksSelector])
    {
        [self performSelector:setLinksSelector withObject:[NSArray array]];
    }
#pragma clang diagnostic pop
}
```
让TTTAttributedLabel自己重新初始化了自己的links属性。

###2. OLImageView