---
author: njuxjy
comments: true
date: 2011-07-01 01:19:13+00:00
layout: post
slug: symbian%e5%ad%a6%e4%b9%a0%e7%ac%94%e8%ae%b0%e4%ba%8c%ef%bc%88%e5%ad%97%e7%ac%a6%e4%b8%b2%ef%bc%89
title: symbian学习笔记二（字符串）
wordpress_id: 146
categories:
- symbian
---

12213123




123




12




3












  1. _LIT(KHello, "Hello!");   


  2. _LIT(KWorld, "World!");    


  3. HBufC* heapBuf = HBufC::NewL(KHello().Length());   


  4. *heapBuf = KHello;    //buf holds "Hello!"   


  5.   


  6. heapBuf = heapBuf->ReAllocL(KHello().Length()   KWorld().Length());   


  7. CleanupStack::PushL(heapBuf);   


  8.   


  9. TPtr ptr (heapBuf->Des());  //DON"T use TPtr ptr = heapBuf->Des(); this will set maxlen to 6 but not 12...   


  10. ptr[KHello().Length() - 1] = " ";   


  11. ptr  = KWorld;   


  12. iTopLabel -> SetTextL(ptr);   


  13. CleanupStack::PopAndDestroy();   


  14. DrawNow();   


  15.   





 [casino online](http://italyslotscasinos.it/)
