---
author: njuxjy
comments: true
date: 2011-07-27 03:16:17+00:00
layout: post
slug: symbian%e5%ad%a6%e4%b9%a0%e7%ac%94%e8%ae%b0%e4%b8%80%ef%bc%88%e5%9f%ba%e6%9c%ac%e6%95%b0%e6%8d%ae%e7%b1%bb%e5%9e%8b%e5%8f%8a%e5%91%bd%e5%90%8d%e8%a7%84%e8%8c%83%ef%bc%89
title: symbian学习笔记一（基本数据类型及命名规范）
wordpress_id: 126
categories:
- symbian
---

本文内容非原创，属于网上资源的整理。

========================================


**基本数据类型**




 在Symbian中，很多C++基本类型都被重新定义了，最好使用Symbian的，理由如下：





	
  * 所有Symbian API都是用的Symbianc重定义的

	
  * 将来Symbian OS由32位转为64位时，支持性更好 

	
  * 这本身就是Symbian C++ Coding Standards所要求的




**1.  Integers**




    typedef signed int TInt;  C++中的signed int，32位，基本用法类似。




    typedef unsigned int TUint;  一般用于计数器(Counter)或者标记(Flags)。




    其他Int类型：TInt64， TInt32， TInt16，TInt8； 同时有一份TUint的版本。




**2. Text**




 text类型在Symbian编程中基本不用，而一般采用描述符（descriptor）。TText默认是16位的。




**3. Boolean **




    typedef int TBool;有两个枚举值：ETrue和EFalse。TBool变量最好不要直接和ETure和EFalse比较。如下：




TBool flag = ETrue;
if (flag)  // if (!flag)
{
flag = EFalse;
}




**4.  Floating Point**




    对浮点数的支持视处理器而定，如果没有FPU，效率非常低，所以最好是不要用浮点数。如果一定要用，尽量转化为整数操作。
typedef float TReal32;  typedef double TReal64; typedef double TReal;




**5. TAny**




    typedef void TAny;




    TAny一般只用作指针，其他情况下用void比较好。




    TAny* MyFunction();     void MyOtherFn();




    TAny* 在很多Symbian API中都用到了，如：




    static TUint8* Copy( TAny* aTrg, const TAny* aSrc, TInt aLength);




**5. Enumerations**




enum TState {EOff, Eon, EInit};




Enumeration类型应该以T开头，而枚举值应该以E开头。




TState  state = GetState();
if (state == EOn)
{
//Do something here
}




**命名规范**




    _T类_：只包含值，而不包含指针以及外部的资源，在栈上分配空间。




                TVersion osVersion = User::Version();




    _C类_：所有需要分配内存的类都必须从CBase继承并且以C开头。




class CExample : public CBase
{
private:
CDesCArrayFlat* iArray;
}




CExample* example = new (ELeave) CExample;




    _R类_：包含指向某个资源的handler。




                RTimer timer;
timer.CreateLocal();




    _M类_：定义一个接口，一般只包含纯虚函数，不包含成员数据，减少类之间的依赖，用来接受回调消息。




class MEikStatusPaneObserver
{
public:
virtual void HandleStatusPaneSizeChange() = 0;
}




任何实现MEikStatusPaneObserver接口的类都必须实现HandleStatusPaneSizeChange()函数。




**1. 变量命名规范**





	
  *     成员变量以“i”开头

	
  *     参数以“a”开头

	
  *     动态变量随便，以小写字母开头

	
  *     常量以“K”开头

	
  *     尽量不要使用全局变量，不能使用全局静态变量。




**2. 函数命名规范**





	
  *     函数以大写字母开头，如AddFileNameL();

	
  *     以D结尾表示deletion of an object

	
  *     以L结尾表示函数可能leave

	
  *     以C结尾表示一个item被放到cleanup stack




**Casting**




    Casting用于在类（classes）和类型（types）之间作转化，Symbian中仍然可以使用C中语法。




    dynamic_cast：不支持，Symbian中没有RTTI。




    static_cast：把一个基类转化为一个继承类。




                   TInt intValue = 0xff;
TUint8 byteValue = static_cast<TUint8>(intValue);




    reinterpret_cast：把一个指针类型转化为另外一个指针类型，如integer转化为point类型或者相反。




                   TUint32 fourBytes = 0;
TUint8* bytePtr = reinterpret_cast<TUint8*> (&fourBytes);
bytePtr++;
*bytePtr = 0xFF;




    const_cast：移除一个类的const属性。



