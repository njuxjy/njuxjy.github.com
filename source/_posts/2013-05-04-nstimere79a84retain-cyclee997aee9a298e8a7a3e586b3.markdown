---
author: njuxjy
comments: true
date: 2013-05-04 01:46:02+00:00
layout: post
slug: nstimer%e7%9a%84retain-cycle%e9%97%ae%e9%a2%98%e8%a7%a3%e5%86%b3
title: NSTimer的retain cycle问题解决
wordpress_id: 779
categories:
- iOS
---

环境为iOS 5.1，6.1，开启了ARC。

在某个Custom的UIView里开启了一个定时器，原想在其dealloc函数中将timer invalidate并置为nil，但发现退出view controller时view没有被释放，timer也在在不停的fire。原来的写法是这样的：

    
    @property (nonatomic, strong) NSTimer *timer;
    
    -(void)startTimer
    {
        if(self.timer == nil)
        {
            self.timer = [NSTimer scheduledTimerWithTimeInterval:TIMER_DURATION target:self selector:@selector(timerFired) userInfo:nil repeats:YES];
        }
    }
    
    -(void)stopTimer
    {
        [self.timer invalidate];
        self.timer = nil;
    }
    
    -(void)dealloc
    {
        [self stopTimer];
    }


由于NSTimer scheduledTimerWithTimeInterval会对self强引用，而self对timer也是强引用，这里存在了retain cycle。

尝试将strong的timer改为weak，但并不奏效：

    
    @property (nonatomic, weak) NSTimer *timer;
    
    -(void)startTimer
    {
        if(self.timer == nil)
        {
            __weak MainCycleScrollView *weakSelf = self;
            self.timer = [NSTimer scheduledTimerWithTimeInterval:TIMER_DURATION target:weakSelf selector:@selector(timerFired) userInfo:nil repeats:YES];
        }
    }


尝试覆盖父类的removeFromSuperView函数，但也不奏效：

    
    -(void)removeFromSuperview
    {
        [self stopTimer];
        [super removeFromSuperview];
    }


解决办法有两个，一个是将timer的启动和关闭移到controller，让controller来决定何时开启解决，但这样破坏了封装性。

最后用了另一个方法，通过覆盖willMoveToWindow，并将self.timer改为_timer解决了问题：

    
    - (void)willMoveToWindow:(UIWindow *)newWindow
    {
        [super willMoveToWindow:newWindow];
        if (newWindow == (id)[NSNull null] || newWindow == nil)
        {
            [self stopTimer];
        }
    }
    
    -(void)startTimer
    {
        if(_timer == nil)
        {
            _timer = [NSTimer scheduledTimerWithTimeInterval:TIMER_DURATION target:self selector:@selector(timerFired) userInfo:nil repeats:YES];
        }
    }
    
    -(void)stopTimer
    {
        [_timer invalidate];
        _timer = nil;
    }



