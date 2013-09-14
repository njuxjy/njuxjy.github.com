---
author: njuxjy
comments: true
date: 2013-05-27 08:38:00+00:00
layout: post
slug: ipad-uimodalpresentationpagesheet%e8%a7%86%e5%9b%be%e7%9a%84resize%e5%92%8cdismiss
title: iPad UIModalpresentationPageSheet视图的resize和dismiss
wordpress_id: 787
categories:
- iOS
---

自定义弹出视图的frame:

    
    - (void)viewDidLoad
    {
        CGRect newFrame = CGRectMake(0,
                                  0,
                                  POP_WINDOW_WIDTH,
                                  POP_WINDOW_HEIGHT);
    
        [self.view setFrame:newFrame];
    
        //Now the bounds have changed so we save them to be used later on
        self.realBounds = self.view.bounds;
    
        [super viewDidLoad];
    
        //视图周围一圈加阴影
        UIImage *image = [[UIImage imageNamed:@"popup_view_shadow_bg"] stretchableImageWithLeftCapWidth:15 topCapHeight:15];
        self.shadowView = [[UIImageView alloc] initWithFrame:CGRectMake(-7, -7, self.view.bounds.size.width + 14, self.view.bounds.size.height + 14)] ;
        self.shadowView.image = image;
        [self.navigationController.view.superview addSubview:self.shadowView];
    
    }
    
    -(void)viewWillAppear:(BOOL)animated
    {
        [super viewWillAppear:animated];
    //可以对view的边界做些定制，去除圆角等
        self.navigationController.view.superview.layer.cornerRadius = 0;
        self.navigationController.view.layer.cornerRadius = 0;
        self.navigationController.view.superview.layer.borderColor = [UIColor darkGrayColor].CGColor;
        self.navigationController.view.superview.layer.borderWidth = 1;
    //设置bounds
        self.navigationController.view.superview.bounds = self.realBounds;
    }


点击灰色区域dismiss掉view:

    
    -(void)showPopupView
    {
        PopupViewController *vc = [[PopupViewController alloc] init];
        vc.delegate = self;
        self.navController = [[UINavigationController alloc] initWithRootViewController:vc];
        self.navController.modalPresentationStyle = UIModalPresentationFormSheet;
        self.navController.modalTransitionStyle = UIModalTransitionStyleCrossDissolve;
        [self presentViewController:self.navController animated:YES completion:nil];
    
        if(self.recongnizer)
        {
            [self.view.window removeGestureRecognizer:self.recongnizer];
            self.recognizer = nil;
        }
        self.recongnizer = [[UITapGestureRecognizer alloc] initWithTarget:self action:@selector(handleTapBehind:)];
        [self.recongnizer setNumberOfTapsRequired:1];
        self.recongnizer.cancelsTouchesInView = NO;
        [self.view.window addGestureRecognizer:self.recongnizer];
    }
    
    - (void)handleTapBehind:(UITapGestureRecognizer *)sender
    {
        if (sender.state == UIGestureRecognizerStateEnded)
        {
            CGPoint location = [sender locationInView:nil];
            BOOL inSelfView;
            BOOL inSelfNavigationView;
            BOOL inPopupView;
            inSelfView = [self.view pointInside:[self.view convertPoint:location fromView:self.view.window] withEvent:nil];
            inSelfNavigationView = [self.navigationController.view pointInside:[self.navigationController.view convertPoint:location fromView:self.view.window] withEvent:nil];
            inPopupView = [self.navController.view pointInside:[self.navController.view convertPoint:location fromView:self.view.window] withEvent:nil];
            if ((inSelfView || inSelfNavigationView) && !inPopupView)
            {
                [self.view.window removeGestureRecognizer:sender];
                [self dismissModalViewControllerAnimated:YES];
            }
        }
    }
