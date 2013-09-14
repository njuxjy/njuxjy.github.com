---
author: njuxjy
comments: true
date: 2013-05-28 05:57:08+00:00
layout: post
slug: '%e3%80%8aios-5-programming-pushing-the-limits%e3%80%8b%e8%af%bb%e4%b9%a6%e7%ac%94%e8%ae%b01-%e7%94%a8%e5%85%b3%e8%81%94%e5%bc%95%e7%94%a8%e4%b8%ba%e5%88%86%e7%b1%bb%e5%a2%9e%e5%8a%a0'
title: 《iOS 5 Programming Pushing the Limits》读书笔记1——用关联引用为分类增加数据
wordpress_id: 790
categories:
- iOS
---

无法在分类中增加成员变量，但是可以用关联引用来实现这一点。

假设有一个Person类：

    
    @interface Person : NSObject
       @property (readwrite, copy) NSString *name;
       @end
       @implementation Person
       @synthesize name=name_;
       @end


可以创建Person类的一个分类来为Person新增一个emailAddress变量：

    
    #import <objc/runtime.h>
       @interface Person (EmailAddress)
       @property (readwrite, copy) NSString *emailAddress;
       @end
       @implementation Person (EmailAddress)
       static char emailAddressKey;
       - (NSString *)emailAddress {
         return objc_getAssociatedObject(self, &emailAddressKey);
    }
       - (void)setEmailAddress:(NSString *)emailAddress {
         objc_setAssociatedObject(self, &emailAddressKey,emailAddress,OBJC_ASSOCIATION_COPY);
    } @end
    emailAddress,
    OBJC_ASSOCIATION_COPY);


之后Person类就动态“增加”了属性emailAddress。

关联引用有个比较实用的地方在于可以动态为一些控件增加数据对象：

    
    ViewController.m (AssocRef)
         id interestingObject = ...;
         UIAlertView *alert = [[UIAlertView alloc]
                               initWithTitle:@”Alert” message:nil
                               delegate:self
                               cancelButtonTitle:@”OK”
                               otherButtonTitles:nil];
         objc_setAssociatedObject(alert, &kRepresentedObject,
                                  interestingObject,
                               OBJC_ASSOCIATION_RETAIN_NONATOMIC);
         [alert show];


以上代码将interestingObject对象附在了UIAlertView控件对象上，当警告框消失时候，可以取出该对象，并做一些操作：

    
    - (void)alertView:(UIAlertView *)alertView
       clickedButtonAtIndex:(NSInteger)buttonIndex {
    UIButton *sender = objc_getAssociatedObject(alertView, &kRepresentedObject);
         self.buttonLabel.text = [[sender titleLabel] text];
       }


不用关联引用的话就得在类里新建一个成员变量来保存interestingObject对象的引用了。

类似的代码在开源库SDWebImage中的UIImageView分类等文件也能看到：

    
    - (void)setImageWithURL:(NSURL *)url placeholderImage:(UIImage *)placeholder options:(SDWebImageOptions)options progress:(SDWebImageDownloaderProgressBlock)progressBlock completed:(SDWebImageCompletedBlock)completedBlock;
    {
        [self cancelCurrentImageLoad];
    
        self.image = placeholder;
    
        if (url)
        {
            __weak UIImageView *wself = self;
            id<SDWebImageOperation> operation = [SDWebImageManager.sharedManager downloadWithURL:url options:options progress:progressBlock completed:^(UIImage *image, NSError *error, SDImageCacheType cacheType, BOOL finished)
            {
                __strong UIImageView *sself = wself;
                if (!sself) return;
                if (image)
                {
                    sself.image = image;
                    [sself setNeedsLayout];
                }
                if (completedBlock && finished)
                {
                    completedBlock(image, error, cacheType);
                }
            }];
            objc_setAssociatedObject(self, &operationKey, operation, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
        }
    }
    
    - (void)cancelCurrentImageLoad
    {
        // Cancel in progress downloader from queue
        id<SDWebImageOperation> operation = objc_getAssociatedObject(self, &operationKey);
        if (operation)
        {
            [operation cancel];
            objc_setAssociatedObject(self, &operationKey, nil, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
        }
    }
