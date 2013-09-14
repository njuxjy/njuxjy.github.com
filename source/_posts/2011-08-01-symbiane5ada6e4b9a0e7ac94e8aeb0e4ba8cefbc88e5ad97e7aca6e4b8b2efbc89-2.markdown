---
author: njuxjy
comments: true
date: 2011-08-01 08:18:15+00:00
layout: post
slug: symbian%e5%ad%a6%e4%b9%a0%e7%ac%94%e8%ae%b0%e4%ba%8c%ef%bc%88%e5%ad%97%e7%ac%a6%e4%b8%b2%ef%bc%89-2
title: symbian学习笔记二（字符串）
wordpress_id: 128
categories:
- symbian
---

本文内容非原创，属于网上资源的整理。

========================================

[![](http://www.xiaojiayi.com/wp-content/uploads/2011/08/cfdb8e3b00aa7e6fb9998fd9.jpg)](http://www.xiaojiayi.com/wp-content/uploads/2011/08/cfdb8e3b00aa7e6fb9998fd9.jpg)

[![](http://www.xiaojiayi.com/wp-content/uploads/2011/08/a3b3aa30c8e67cb31b4cffa13.jpg)](http://www.xiaojiayi.com/wp-content/uploads/2011/08/a3b3aa30c8e67cb31b4cffa13.jpg)



	
  * 8位：（TDesC8），用于二进制数据或者ASCII字符串

	
  * 16位：（TDesC16），默认，Unicode




[![](http://www.xiaojiayi.com/wp-content/uploads/2011/08/32.jpg)](http://www.xiaojiayi.com/wp-content/uploads/2011/08/32.jpg)




描述符可以分为五类：






	
  1. 抽象类（Abstract）：（TDes、TDesC、Tdes8、TdesC8），其他描述符的基类，仅提供接口和基本功能，不能被实例化，一般只用作函数的参数。

	
  2.  文字常量（Literal）：（TlitC、_LIT()），用于存储文字字符串（literal string），即C中字符串常量，通常使用_LIT()这种方式（当然还有_L()和_L8()的描述方式，但都不提倡用）。

	
  3.  栈类（Buffer）：（Tbuf、TbufC、 Tbuf8、TbufC8），数据存储于栈上，最基本的描述符变量类型，大小在编译时确定，包含描述符本身数据，使用最为普遍

	
  4. 堆类（Heap）：（HbufC、HbufC8），数据存储于堆上，大小在运行时确定，也就是是用来处理动态申请的描述符类。

	
  5. 指针类（Pointer）：（TPtr、TPtrC、TPtr8、TPtrC8），本身不包含描述符数据，但是包含长度数据，而且还包含一个指向位于描述符之外数据的指针。




1、  文字描述符常量









	
  1. LIT(KMyFile, "c:\System\Apps\MyApp\MyFile.jpg");  







_L()可以生成一个指向字符值的地址（TPtrC），它经常被用来传递字符串到函数中（包括描述符的构造函数和格式化函数）；同理_L8()则可以生成一个指向二进制数据的地址（TPtrC8）举例如下：









	
  1. NEikonEnvironment::MessageBox(_L("Error: init file not found!"));   

	
  2. //数字转字符串   

	
  3. TBuf16<20> buf;   

	
  4. TInt iNum = 20;   

	
  5. buf.Format( _L( "%d" ) , iNum  );  







2、  栈描述符









	
  1. LIT(Ktext , "Test Text");   

	
  2. _LIT(Ktext1 , "Test1 Text");   

	
  3. _LIT(KXtraText , "New:");   

	
  4. _LIT(NewText , "New1");   

	
  5. _LIT(NewText1 , "New2");   

	
  6. TBufC<10> Buf1 ( Ktext );//Buf1长度为9 内容 “Test Text”   

	
  7. TBufC<10> Buf2 ( Ktext1 );//Buf2长度为10 内容 “Test1 Text”   

	
  8. // 通过赋值的方式改变数据   

	
  9. Buf2 = Buf1; //Buf2长度变为9 内容 “Test Text”   

	
  10. //通过使用Des()生成指针改变TBufC的数据   

	
  11. TPtr Pointer = Buf1.Des();   

	
  12. // 删除后四个字符   

	
  13. Pointer.Delete(Pointer.Length()-4, 4 ); //Buf1长度变为5 内容“Test ”//但是内存应该没变   

	
  14. // 增加新的数据   

	
  15. Pointer.Append(KXtraText);//Buf1长度为9 内容为“Test New：”   

	
  16. // 也可以使用下列方式改变数据   

	
  17. TBufC<10> Buf3(NewText);   

	
  18. Pointer.Copy(Buf3);//Buf1长度为4，内容为New1   

	
  19. // 或直接从字符串里获得数据   

	
  20. Pointer.Copy(NewText1);//Buf1长度为4，内容为New2  







    以上介绍的是不可修改的栈描述符，而可修改的描述符就不用通过那么复杂的方法来实现修改，它直接可以用Copy、Delete等方法，但是无论可修改的还是不可修改的，一旦指定最大的数据长度后，最大长度就不能进行修改了。




在内存中如下所示:









	
  1. TBuf<16> helloWorld = KHelloWorld; TInt len = KHelloWorld().Length(); helloWorld[len-1]='?';  







TBufC的用法如下：









	
  1. _LIT(KHelloWorld, "Hello World"); const TInt maxBuf = 32; TBufCbuf; TInt currentLen = buf.Length(); // == 0 buf = KHelloWorld; currentLen = buf.Length(); // == 11 TText ch = buf[2]; // == 'l'  







 TBuf的用法如下：









	
  1. const TInt bufLen = 6; TUInt8 objType = 1; TUInt8 objId = 1; TUInt8 xCoord = 128; TUInt8 yCoord = 192; .... TBuf8<bufLen> buf; buf.Append(objType); buf.Append(objId); ... //we can now do something with the buffer such as writting it to a binary file or send via socket.  







3、  堆描述符




堆描述符虽然都是不可修改类型的，但是它仍然具有构造和修改，与栈描述符不同的是：首先对内存需要显示释放，其次是堆描述符没有最大长度的限制，任何时候都可以用ReAlloc（）函数重新申请分配。具体见示例：









	
  1. //例1、构造   

	
  2. //有两种方式来生成一个Heap Descriptor   

	
  3. //第一种方式用New(),NewL(),或NewLC()   

	
  4. //如下操作便可以构建一个存放数据的空间，空间为15，不过目前大小为0   

	
  5. HBufC * Buf = HBufC::NewL(15);   

	
  6. //第二种方式是采用Alloc()，AllocL()或AllcLC()来处理，   

	
  7. //不过这是已经存在的数据的管理方式。新的Heap Descriptor   

	
  8. //可以自动的根据这个内容来构造。   

	
  9. _LIT (KText , "Test Text");   

	
  10. TBufC<10>  CBuf = KText;   

	
  11. HBufC * Buf1 = CBuf.AllocL();   

	
  12. CleanupStack::PushL(Buf1);   

	
  13. //例2、修改   

	
  14. //下面是通过赋值方式改变其数据的方法   

	
  15. _LIT ( KText1 , "Text1");   

	
  16. *Buf1 = KText1;   

	
  17. // 通过可修改指针来改变数据的方式   

	
  18. TPtr Pointer = Buf1->Des();   

	
  19. //添加数据   

	
  20. Pointer.Delete(Pointer.Length() - 2, 2);   

	
  21. //删除数据   

	
  22. _LIT ( KNew, "New:");   

	
  23. Pointer.Append(KNew);   

	
  24. //例3、重新申请内存   

	
  25. Buf1 = Buf1->ReAllocL(KText().Length() + KNew().Length());   

	
  26. CleanupStack::PushL(Buf1);   

	
  27. //例4、释放内存   

	
  28. //直接用delete   

	
  29. delete Buf;   

	
  30. Buf = NULL;   

	
  31. //如果在使用NewL、ReAllocL等异常函数后我们使用清除栈压入的话   

	
  32. //那么我们也可以用清除栈来释放内存   

	
  33. CleanupStack::PopAndDestroy();   

	
  34. Buf1 = NULL;  







注：关于以上用清除栈的方式，个人只是猜测，因为对Symbian的异常处理三部曲，至今仍没有很好的掌握，所以如果有什么误用还望指点。









	
  1. HBufC* heapBuf = HBufC::NewL(KHelloWorld().Length()); *heapBuf = KHelloWorld(); delete heapBuf;  







     在内存中的情况如下图所示：




    HBufC通常在以下几种情况下使用：






	
  *  在运行时从资源文件中加载字符串

	
  *  从用户界面中接收用户输入的字符串

	
  * 从应用程序引擎中接收字符串，如contacts database中的名字




     对HBufC中的内容进行修改：









	
  1. HBufC* heapBuf = HBufC::NewL(KHelloWorld().Length()); *heapBuf = KHelloWorld(); delete heapBuf;  







4、  指针描述符









	
  1. //例1、用TBuf和TBufC构造出TPtrC对象   

	
  2. _LIT(KText , "Test Code");   

	
  3. TBufC<10> Buf ( KText );   

	
  4. //或者为 TBuf<10> Buf ( KText );   

	
  5. // Creation of TPtrC using Constructor   

	
  6. TPtrC  Ptr (Buf);   

	
  7. // Creation of TPtrC using Member Function   

	
  8. TPtrC     Ptr1;   

	
  9. Ptr1.Set(Buf);   

	
  10. //例2、用TText*构造TPtrC   

	
  11. const TText* text = _S("Hello World\n");   

	
  12. TPtrC ptr(text);   

	
  13. // 或者   

	
  14. TPtrC Ptr2;   

	
  15. Ptr2.Set(text);   

	
  16. //如果要存储TText的一部分数据，我们使用下列方法   

	
  17. TPtrC   ptr4(text, 5);   

	
  18. //例3、从另一个TPtrC中构造TPtrC   

	
  19. const TText * text1 = _S("Hello World\n");   

	
  20. TPtrC Ptr3(text1);   

	
  21. // 从一个TPtrC中获得另一个TPtrC   

	
  22. TPtrC p1(Ptr3);   

	
  23. // 或   

	
  24. TPtrC p2;   

	
  25. p2.Set(Ptr3);  







以上是不可修改的TPtrC的构造，相对应的也有可修改的TPtr的构造，不过我们下面省略了用Set()函数的构造方法









	
  1.  //例1、通过TBufC,HBufC的Des()方法获取   

	
  2. _LIT(KText, "Test Data");   

	
  3. TBufC<10> NBuf ( KText );   

	
  4. TPtr Pointer = NBuf.Des();   

	
  5. //例2、通过指定内存区域和大小来生成   

	
  6. const TText * Text = _S("Test Second");   

	
  7. TPtr Pointer1((TText*)Text, 11, 12);   

	
  8. //例3、 通过另一个TPtr对象来生成   

	
  9. TPtr Pointer2 ( Pointer );  







对于可修改的TPtr虽然前面用过，但是我们在这里在简单的添加两个例子加深下印象，并且说明指针修改的始终是它指向的描述符：









	
  1. //例1、改变已有TPtr数据的方式：赋值和Copy()方法   

	
  2. _LIT(KText, "Test Data");   

	
  3. _LIT(K1, "Text1");   

	
  4. _LIT(K2, "Text2");   

	
  5. TBufC<10> NBuf ( KText );//NBuf内容为“Test Data”   

	
  6. TPtr Pointer = NBuf.Des(); //Pointer指向NBuf的内容   

	
  7. Pointer = K1; // NBuf内容为“Text1”   

	
  8. Pointer.Copy(K2); // NBuf内容为“Text2”   

	
  9. //例2、直接通过修改长度改变数据内容   

	
  10. Pointer.SetLength(2); // NBuf内容为"Te" 注：实际内存的内容应该没变   

	
  11. const unsigned char KBuffer[ ] = {0x00, 0x33, 0x66, 0x99, 0xbb, 0xff}; TPtrC8 bufferPtr( KBuffer, sizeof(KBuffer)); iSocket.Write(bufferPtr, iStatus);  







在内存中如下所示：




[![](http://www.xiaojiayi.com/wp-content/uploads/2011/08/4.jpg)](http://www.xiaojiayi.com/wp-content/uploads/2011/08/4.jpg)




TPtr的用法：









	
  1. _LIT(KHelloWorld, "Hello World"); const TInt maxBuf = 32; TBufC<maxBuf> buf; buf = KHelloWorld; TPtr ptr = buf.Des(); ptr[7] = 'a';  ptr[8] = 'l';  ptr[9] = 'e';  ptr[10] = 's'; CEikonEnv::Static()->InfoMsg(ptr); // "Hello Wales"  







5、  抽象描述符




抽象描述符，没有什么好说的，正如前面所说，只用在函数的形参中，通常要强调参数是不可修改的，就用const TDesC&表示，可修改的参数用TDesC&表示。






	
  * 在函数参数中尽量使用基类

	
  * 使用中性的描述符，一般情况下使用TDesC而不是TDesC8或者TDesC16

	
  * 当描述符内容不应该改变时，使用const修饰符

	
  * 经典用法：void SetText(const TDesC& aText);    TPtrC Text() const;




描述符之间的转换




**不可修改向可修改描述符的转换**




原则1：通过不可修改描述符类内的Des()函数，将不可修改的描述符转换成可修改的指针描述符




示例1：TBufC转换成TPtr









	
  1. _LIT(KText, "Test Data");   

	
  2. TBufC<10> NBuf ( KText );   

	
  3. TPtr Pointer = NBuf.Des();  







示例2：HBufC转换成TPtr









	
  1. HBufC * Buf = HBufC::NewL(15);   

	
  2. _LIT (KText , "Test Text");   

	
  3. *Buf = KText;   

	
  4. TPtr Pointer = Buf->Des();  







原则2：通过TPtr的构造函数或Set()函数可以将TPtrC描述转换为可修改的指针描述符




示例3：TPtrC到TPtr









	
  1. const TText * text1 = _S("Hello World\n");   

	
  2. TPtrC Ptr1(text1);   

	
  3. TPtrC Ptr2(Ptr1);   

	
  4. //可以通过构造函数   

	
  5. TPtr Ptr3((TUint16 *)(Ptr1.Ptr()), Ptr1.Length());   

	
  6. //也可以通过Set()函数   

	
  7. Ptr3.Set((TUint16 *)(Ptr1.Ptr()),Ptr1.Length(), Ptr1.Length());  






