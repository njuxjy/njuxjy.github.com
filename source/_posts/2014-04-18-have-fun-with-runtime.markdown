---
layout: post
title: "给第三方库打patch"
date: 2014-04-18 15:36
comments: true
categories: [iOS]
---

标题有一个前提，即在不修改第三方库代码的基础上。

项目中用到的第三方库，有时候不能满足项目的特定需求，这时需要对它们打一些patch。比如我们项目中用到的TTTAttributedLabel, OLImageView和AFNetworking。都能在github上找到。

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

OLImageView是个支持GIF的UIImageView，它和AFNetworking的UIImageView+AFNetworking分类配合起来可以下载并显示一张GIF图，就跟显示普通的jpg没什么两样，使用方法如下：

```
OLImageView *imageView = [[OLImageView alloc] init];
[imageView setImageWithURL:[NSURL URLWithString:@"http://***.com/***.gif"]];
```

原因在于这个库的作者利用runtime对AFNetworking做了一些手脚。它创建了AFImageRequestOperation的一个分类AFImageRequestOperation+OLImage，并在`+load`时将AFImageRequestOperation类中的`setResponseImage:`方法和该分类中的`setResponseOLImage:`方法交换：

```
+ (void)load
{
    method_exchangeImplementations(class_getInstanceMethod(self, @selector(setResponseImage:)), class_getInstanceMethod(self, @selector(setResponseOLImage:)));
}

- (void)setResponseOLImage:(UIImage *)responseImage
{
    if ([self.responseData length] > 0 && [self isFinished])
    {
        [self setResponseOLImage:[OLImage imageWithData:self.responseData scale:self.imageScale]];
    }
}
```

这样，当AFImageRequestOperation下载完图片数据，准备调用`setResponseImage:`时，它就会被hook进这里定义的`setResponseOLImage`，做一些定制化工作。

如果想加下载进度提示怎么办？由于项目中使用了UIImageView+AFNetworking来实现图片下载，没有直接操作AFImageRequestOperation，没办法调用AFImageRequestOperation的`setDownloadProgressBlock:`来实现定制，于是想到了模仿之前的做法，hook AFImageRequestOperation的`initWithRequest`方法，在初始化方法里调用`setDownloadProgressBlock:`，做法如下：

```
+ (void)load
{
    method_exchangeImplementations(class_getInstanceMethod(self, @selector(initWithRequest:)), class_getInstanceMethod(self, @selector(initWithOLRequest:)));
}

- (id)initWithOLRequest:(NSURLRequest *)request
{
    self = [super initWithRequest:request];
    if (!self)
    {
        return nil;
    }
    
#if defined(__IPHONE_OS_VERSION_MIN_REQUIRED)
    self.imageScale = [[UIScreen mainScreen] scale];
    self.automaticallyInflatesResponseImage = YES;
    [self setDownloadProgressBlock:^(NSUInteger bytesRead, long long totalBytesRead, long long totalBytesExpectedToRead)
    {
        dispatch_async(dispatch_get_main_queue(), ^
        {
            if([[self.request.URL absoluteString] isKindOfClass:[NSString class]])
            {
                [[NSNotificationCenter defaultCenter] postNotificationName:BKPImageDownloadingProgressNotification object:[self.request.URL absoluteString] userInfo:@{BKPImageTotalBytesReadKey : @(totalBytesRead), BKPImageTotalBytesExpectedToReadKey : @(totalBytesExpectedToRead)}];
            }
        });
    }];
    
#endif
    
    return self;
}

```

###3. AFNetworking

Matt大神的AFNetworking应该都比较熟悉了，项目里图片下载的底层都是用了之前提到的UIImageView+AFNetworking分类，这个分类很简洁易用，不过它把图片缓存的逻辑也完全封装了，有时候需要了解某个给定的图片URL是否已经被缓存了，或者是否正在被请求，于是自己动手，丰衣足食：

```
- (BOOL)isRequestingForUrl:(NSURL *)url
{
#pragma clang diagnostic push
#pragma clang diagnostic ignored "-Wundeclared-selector"
    UIImageView *helper = [[UIImageView alloc] init];
    if ([[helper class] respondsToSelector:@selector(af_sharedImageRequestOperationQueue)])
    {
        NSOperationQueue *operationQueue = [[helper class] performSelector:@selector(af_sharedImageRequestOperationQueue)];
        for(AFImageRequestOperation *op in [operationQueue operations])
        {
            if([op.request.URL.absoluteString isEqualToString:url.absoluteString])
            {
                return YES;
            }
        }
        return NO;
    }
    else
    {
        return NO;
    }
#pragma clang diagnostic pop
}

- (BOOL)isCachedForUrl:(NSURL *)url
{
#pragma clang diagnostic push
#pragma clang diagnostic ignored "-Wundeclared-selector"
    UIImageView *helper = [[UIImageView alloc] init];
    if ([[helper class] respondsToSelector:@selector(af_sharedImageCache)])
    {
        id afImageCache = [[helper class] performSelector:@selector(af_sharedImageCache)];
        if([afImageCache respondsToSelector:@selector(cachedImageForRequest:)])
        {
            UIImage *cachedImage = [afImageCache performSelector:@selector(cachedImageForRequest:) withObject:[NSURLRequest requestWithURL:url]];
            return cachedImage != nil;
        }
        else
        {
            return NO;
        }
    }
    else
    {
        return NO;
    }
#pragma clang diagnostic pop
}
```

###注意点

利用runtime来给第三方库打patch，不是没有代价的，需要小心使用：

1. 当第三方库升级了，需要检查是否patch的方法已过期，甚至更糟糕的，是否会造成crash，因为之前的做法屏蔽了编译器警告，好看的同时也有一些隐患。
2. method-swizzling本身会有一些副作用，可以参考这篇：http://herkuang.info/blog/2014/02/25/慎用method-swizzling/

