---
author: njuxjy
comments: true
date: 2012-10-12 02:57:00+00:00
layout: post
slug: '%e4%bb%a3%e7%a0%81%e6%88%92%e5%be%8b%ef%bc%9aobjective-c%e6%9c%80%e4%bd%b3%e5%ae%9e%e8%b7%b5%ef%bc%88%e8%af%91%e6%96%87%ef%bc%89'
title: 代码戒律：Objective-C最佳实践（译文）
wordpress_id: 489
categories:
- iOS
- 翻译
---

在逛V2EX时候发现了这篇文章，遂决定翻译一下，自己也顺带学习下。

作者的一些观点有些偏激，自己心里有数就好。

原文网址：http://ironwolf.dangerousgames.com/blog/archives/913

=========================


### 2012.4.26更新，加入了ARC




## 前言


我通常在自己的博客中不会写太技术性的东西，但这次例外，因为我希望为Mac和iOS（iPhone&iPad）开发者社区做些贡献。如果你不是社区中的一员，请自行绕道吧。


## 介绍




 这篇文章是在我多年的Objective-C使用经验中所目睹的那些最容易被Objective-C程序员触犯的最佳实践积累下来的一个列表。我称之为“戒律”，我们有太多理由要去遵守它们，几乎没理由不去遵守。然而当我向其他开发者展示这些实践经验时，他们往往非常反对...







## 强烈反对：这不会影响性能吗？


这个反对基本没什么道理。如果你能向我举出一个例子来证明遵循以下任何一条规范写出来的代码**竟然**成了性能瓶颈，那我祝愿你能不按照规范优化出好的代码，只要你能够**在代码中清楚地写明**你是**怎样**打破规范的以及**为什么**。


## 为什么？


这些规范为了让代码更安全，节省内存，提高可读性和可维护性。大多数规范**不能**帮助提高性能。然而，写代码时不应该首先考虑性能。我们应该写出简洁、正确的代码，仅仅当性能分析告诉你有必要优化时才去做优化。在典型的UI驱动应用中，最大的瓶颈在于用户，然后是网络访问，接着是磁盘访问。但是，过早优化在程序员中还是非常普遍，他们认为如果他们不用尽所有手段去优化的话，他们的应用将会慢的跟狗一样。这肯定是不对的。


## 重要建议


即使在你写的时候知道你的代码是如何工作的，那也不意味着其他人在尝试修改你的代码前会真正理解它，也不意味着你在几个月后再次看这段代码时能理解它。使用详细的符号命名（译者注：即变量和方法名等用详细的词来命名），在仍然不够易于理解的代码段中加入详细的注释，这样就能**总是**确保你的代码能够自说明（译者注：self-documenting，即代码易于理解，不需要写文档也能让人读懂它）。每当你解bug或者API误用的时候，将你认为可能会出问题的地方定位在某个方法里，解释这个方法，然后在注释中加入类似[KLUDGE](http://en.wikipedia.org/wiki/KLUDGE)的关键字，之后你就能轻易的发现并且解决这些问题了。


## 文章（尚）未涉及的内容





	
  * 键值编码(KVC)和基于键值的观察者(KVO)。你有可能会用到它们。




### 2012.4.26新增




### 戒律...


**总是**使用自动引用技术(ARC)。这篇文章已经移除了非ARC代码风格。所有的新代码应当用ARC来写，所有的遗留代码应当更新为ARC(或者封装完备并给出精妙的接口)。


### 戒律...


**总是**在任何实例变量的声明前加上@private指令，**永远不要**在类外面直接访问实例变量。


### 为什么？





	
  * 将信息隐藏(封装)

	
  * 只在类实现的方法中才去直接访问实例变量

	
  * 实例变量的默认访问属性是@protected，意味着子类可以自由访问这些实例变量。但没有足够的理由不要允许子类这么做——父类暴露给外界的所有东西是它的一种契约，改变类成员的内部表示而不改变它的接口或者契约是面向对象封装的重要好处。将你的实例变量设为私有，你就告诉外界它们是实现的细节而非类的接口




### 坏的做法











	
  1. 


@interface Foo : [NSObject](http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSObject_Class/) {


	
  2. 


        int a;


	
  3. 


        [NSObject](http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSObject_Class/)* b;


	
  4. 


        // …


	
  5. 


}


	
  6. 


// method & property declarations…


	
  7. 


@end











### 好的做法








	
  1. 


@interface Foo : [NSObject](http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSObject_Class/) {


	
  2. 


        @private


	
  3. 


        int a;


	
  4. 


        [NSObject](http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSObject_Class/)* b;


	
  5. 


        // …


	
  6. 


}


	
  7. 


// method & property declarations…


	
  8. @end







### 戒律…


总是为每个数据成员创建@property，然后在类实现中用"self.name"去访问它。永远不要直接访问你的实例变量。


### 为什么?





	
  * 属性强制加上了访问限制(例如readonly)

	
  * 属性强制加上了内存管理策略(strong,weak)

	
  * 属性帮你实现了setter和getter

	
  * 使用属性可以强制执行线程安全策略

	
  * 单一的访问实例变量的方法可以增加代码可读性




### 坏的做法











	
  1. 


@interface Foo : [NSObject](http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSObject_Class/) {


	
  2. 


        @private


	
  3. 


        [NSObject](http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSObject_Class/)* myObj;


	
  4. 


}


	
  5. 


@end


	
  6. 



	
  7. 


@implementation Foo


	
  8. 


- (void)bar {


	
  9. 


        myObj = nil;


	
  10. 


}


	
  11. 


@end











### 好的做法








	
  1. 


@interface Foo : [NSObject](http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSObject_Class/) {


	
  2. 


        @private


	
  3. 


        [NSObject](http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSObject_Class/)* myObj;


	
  4. 


}


	
  5. 



	
  6. 


@property(strong, nonatomic) [NSObject](http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSObject_Class/)* myObj;


	
  7. 



	
  8. 


@end


	
  9. 



	
  10. 


@implementation Foo


	
  11. 


- (void)bar {


	
  12. 


        self.myObj = nil;


	
  13. 


}


	
  14. @end









### 戒律…


**总是**在你的属性上加上"nonatomic"，除非你在写线程安全的类，确实需要原子访问，那就**在代码中写注释表明你是故意这么做的**。


### 为什么?


那些带有未声明为"nonatomic"属性的类会让人们觉得这是个设计成线程安全的类，但实际上并不是。只有在你确实将类设计成线程安全时，才不要在属性前加上"nonatomic"。


### 坏的做法











	
  1. 


@interface Foo : [NSObject](http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSObject_Class/)


	
  2. 



	
  3. 


@property(strong) [NSObject](http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSObject_Class/)* myObj;


	
  4. 



	
  5. 


@end











### 好的做法











	
  1. 


@interface Foo : [NSObject](http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSObject_Class/)


	
  2. 



	
  3. 


@property(strong, nonatomic) [NSObject](http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSObject_Class/)* myObj;


	
  4. 



	
  5. 


@end











### 好的做法











	
  1. 


// This class and all it’s properties are thread-safe.


	
  2. 


@interface Foo : [NSObject](http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSObject_Class/)


	
  3. 



	
  4. 


@property(strong) [NSObject](http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSObject_Class/)* myObj;


	
  5. 



	
  6. 


@end

















### 戒律…


**永远不要**让你的实例变量名字和属性名或者数据成员名有所混淆。**总是**在实例变量名结尾加上下划线。 除非你继承了第三方的类，这个类已经有一个数据成员有同样的名字，这样的话就选择一个不同的名字，或者再加一条下划线，然后**加入注视解释你为什么要这么做**。

在实现中使用"@synthesize name = name_;"，而不要只写"@synthesize name;"


### 为什么?





	
  * 即使是在类实现中你也最好属性访问器来访问数据成员，而不要直接访问，除非特殊情况。

	
  * 苹果对于他们的私有实例变量用了"_name"的方式，那么你自己的就用"name_"的方式好了，可以避免命名冲突。

	
  * 苹果已经开始在那些应用级的代码示例及模板中使用"name_"的命名习惯。




### 坏的做法








	
  1. 


@interface Foo : [NSObject](http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSObject_Class/)


	
  2. 



	
  3. 


@property(strong, nonatomic) [NSObject](http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSObject_Class/)* myObj;


	
  4. 



	
  5. 


@end


	
  6. 



	
  7. 


// …


	
  8. 



	
  9. 


@implementation Foo


	
  10. 



	
  11. 


@synthesize myObj;


	
  12. 



	
  13. @end







### 好的做法








	
  1. 


@interface Foo : [NSObject](http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSObject_Class/)


	
  2. 



	
  3. 


@property(strong, nonatomic) [NSObject](http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSObject_Class/)* myObj;


	
  4. 



	
  5. 


@end


	
  6. 



	
  7. 


// …


	
  8. 



	
  9. 


@implementation Foo


	
  10. 



	
  11. 


@synthesize myObj = myObj_;


	
  12. 


 @end








### 戒律…


**NEVER** redundantly add the data member to the class @interface yourself. Allow the @synthesize directive to implicitly add the data member.

Following this practice will yield many classes that explicitly declare no instance variables in the @interface section. When a class has no instance variables, omit the @private declaration, and even omit the opening and closing braces of the data member section.

**永远不要**自己在类接口中重复增加数据成员。让@synthesize指令来为你隐式地增加数据成员。

遵从这条经验，许多类可以不用在类接口中显式地声明实例变量。当一个类没有实例变量时，可以省略@private声明，甚至连数据成员段的开闭括号都可以去掉。


### 为什么?





	
  * 减少重复。

	
  * 简化类的头文件。

	
  * 不需要把类的声明都写在公共头文件中，你可以在实现文件中声明那些真正会直接访问到的类成员。




### 坏的做法





	
  1. 


@interface Foo : [NSObject](http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSObject_Class/) {


	
  2. 


        @private


	
  3. 


        [NSObject](http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSObject_Class/)* myObj_;


	
  4. 


}


	
  5. 



	
  6. 


@property(strong, nonatomic) [NSObject](http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSObject_Class/)* myObj;


	
  7. 



	
  8. 


@end


	
  9. 



	
  10. 


// …


	
  11. 



	
  12. 


@implementation Foo


	
  13. 



	
  14. 


@synthesize myObj = myObj_;


	
  15. 



	
  16. @end







### 好的做法











	
  1. 


@interface Foo : [NSObject](http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSObject_Class/)


	
  2. 



	
  3. 


@property(strong, nonatomic) [NSObject](http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSObject_Class/)* myObj;


	
  4. 



	
  5. 


@end


	
  6. 



	
  7. 


// …


	
  8. 



	
  9. 


@implementation Foo


	
  10. 



	
  11. 


@synthesize myObj = myObj_;


	
  12. 



	
  13. 


@end









You may still need to declare the underlying name of the variable if you need to access it directly, as when writing custom getters and setters:

当你自己在为实例变量写setter和getter的时候，如果你在其中需要直接访问变量的话，你得声明名字带下划线的变量：


### 错误的写法








	
  1. 


@interface Foo : [NSObject](http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSObject_Class/)


	
  2. 



	
  3. 


@property(strong, nonatomic) [NSObject](http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSObject_Class/)* myObj;


	
  4. 



	
  5. 


@end


	
  6. 



	
  7. 


// …


	
  8. 



	
  9. 


@implementation Foo


	
  10. 



	
  11. 


@synthesize myObj;


	
  12. 



	
  13. 


- ([NSObject](http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSObject_Class/)*)myObj


	
  14. 


{


	
  15. 


        return self.myObj; // 会递归调用getter!


	
  16. 


}


	
  17. 



	
  18. 


- (void)setMyObj:([NSObject](http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSObject_Class/)*)myObj


	
  19. 


{


	
  20. 


        self.myObj = myObj; // 会递归调用setter!


	
  21. 


}


	
  22. 



	
  23. @end







### 正确的写法








	
  1. 


@interface Foo : [NSObject](http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSObject_Class/)


	
  2. 



	
  3. 


@property(strong, nonatomic) [NSObject](http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSObject_Class/)* myObj;


	
  4. 



	
  5. 


@end


	
  6. 



	
  7. 


// …


	
  8. 



	
  9. 


@implementation Foo


	
  10. 



	
  11. 


@synthesize myObj = myObj_;


	
  12. 



	
  13. 


- ([NSObject](http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSObject_Class/)*)myObj


	
  14. 


{


	
  15. 


        return myObj_; // 没问题.


	
  16. 


}


	
  17. 



	
  18. 


- (void)setMyObj:([NSObject](http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSObject_Class/)*)myObj


	
  19. 


{


	
  20. 


        // 没问题


	
  21. 


        myObj_ = myObj; // 进行赋值(ARC会处理必要的retain和release)


	
  22. 


}


	
  23. 



	
  24. @end







### 戒律…


**NEVER** access a data member through an underscore-suffixed symbol UNLESS you are writing a setter or getter.

**永远不要**访问名字以下划线开头的变量，除非你在写setter或getter。


### 坏的做法








	
  1. 


@interface Foo : [NSObject](http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSObject_Class/)


	
  2. 



	
  3. 


@property(strong, nonatomic) Bar* myObj;


	
  4. 



	
  5. 


@end


	
  6. 



	
  7. 


// …


	
  8. 



	
  9. 


@implementation Foo


	
  10. 



	
  11. 


@synthesize myObj = myObj_;


	
  12. 



	
  13. 


- (void)someMethod


	
  14. 


{


	
  15. 


        myObj_ = [[Bar alloc] init];


	
  16. 


}


	
  17. 



	
  18. @end







### 好的做法








	
  1. 


@interface Foo : [NSObject](http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSObject_Class/)


	
  2. 



	
  3. 


@property(strong, nonatomic) Bar* myObj;


	
  4. 



	
  5. 


@end


	
  6. 



	
  7. 


// …


	
  8. 



	
  9. 


@implementation Foo


	
  10. 



	
  11. 


@synthesize myObj = myObj_;


	
  12. 



	
  13. 


- (void)someMethod


	
  14. 


{


	
  15. 


        self.myObj = [[Bar alloc] init];


	
  16. 


}


	
  17. 



	
  18. @end







### 戒律…


**永远不要**在类的头文件中声明内部(私有)方法或属性。**总是**将所有的内部方法和属性声明放入实现文件中的“类扩展”中。


### 坏的做法








	
  1. 


//


	
  2. 


// Foo.h


	
  3. 


//


	
  4. 



	
  5. 


@interface Foo : [NSObject](http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSObject_Class/)


	
  6. 



	
  7. 


@property(nonatomic) int myPublicProperty;


	
  8. 


@property(strong, nonatomic) Bar* myPrivateProperty; // This can be accessed by anyone who includes the header


	
  9. 



	
  10. 


- (int)myPublicMethod;


	
  11. 


- (int)myPrivateMethod; // So can this.


	
  12. 



	
  13. @end







### 好的做法








	
  1. 


//


	
  2. 


// Foo.h


	
  3. 


//


	
  4. 



	
  5. 


@interface Foo : [NSObject](http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSObject_Class/)


	
  6. 



	
  7. 


// Only the public API can be accessed by including the header


	
  8. 



	
  9. 


@property(nonatomic) int myPublicProperty;


	
  10. 



	
  11. 


- (int)myPublicMethod;


	
  12. 



	
  13. @end











	
  1. 


//


	
  2. 


// Foo.m


	
  3. 


//


	
  4. 



	
  5. 


@interface Foo () // This is a "class extension" and everything declared in it is private, because it’s in the implementation file


	
  6. 



	
  7. 


@property(strong, nonatomic) Bar* myPrivateProperty;


	
  8. 



	
  9. 


- (int)myPrivateMethod;


	
  10. 



	
  11. 


@end


	
  12. 



	
  13. 


@implementation Foo


	
  14. 


// …


	
  15. @end







### 戒律…


**永远不要**在一个方法里有超过一句返回语句，让方法中的最后一个语句带有一个非空值的返回类型。

在返回非空值的方法中，第一句语句就声明一个变量保存返回值，给它一个有意义的默认值。必要时在代码不同路径中给它赋值。在最后一个语句中返回它的值。**永远不要**过早地用返回语句返回它的值。


### 为什么?


过早的返回语句会增加有一些必要资源释放没有被执行的可能性。


### 坏的做法








	
  1. 


@implementation Foo


	
  2. 



	
  3. 


- (Bar*)barWithInt:(int)n


	
  4. 


{


	
  5. 


        // Allocate some resource here…


	
  6. 



	
  7. 


        if(n == 0) {


	
  8. 


                // …and you have to deallocate the resource here…


	
  9. 


                return [[Bar alloc] init];


	
  10. 


        } else if(n == 1) {


	
  11. 


                // …and here…


	
  12. 


                return self.myBar;


	
  13. 


        }


	
  14. 



	
  15. 


        // …and here.


	
  16. 


        return nil;


	
  17. 


}


	
  18. 



	
  19. @end







### 好的做法








	
  1. 


@implementation Foo


	
  2. 



	
  3. 


- (Bar*)barWithInt:(int)n


	
  4. 


{


	
  5. 


        Bar* result = nil;


	
  6. 



	
  7. 


        // Allocate some resource here…


	
  8. 



	
  9. 


        if(n == 0) {


	
  10. 


                result = [[Bar alloc] init];


	
  11. 


        } else if(n == 1) {


	
  12. 


                result = self.myBar;


	
  13. 


        }


	
  14. 



	
  15. 


        // …and deallocate the resource here, you’re done!


	
  16. 



	
  17. 


        return result;


	
  18. 


}


	
  19. 



	
  20. @end







### 戒律…


理解自动释放池用来做什么，什么时候为你创建与释放，以及什么时候你需要自己来创建和释放它。



	
  * 由NSRunLoop在每一个循环自动创建及释放

	
  * 由NSOperation自动创建及释放

	
  * 在线程的开始及结尾处手动创建及释放

	
  * 每当你需要在某个循环中创建及释放大量对象的时候，在循环外手动创建及释放


在ARC模式下，你通过@autoreleasepool{...}指令来创建自动释放池。



* * *





### 戒律…


**总是**倾向于使用类层面的便利构造函数，而非init构造函数。所有基础框架里的容器类都提供这些。


### 坏的做法











	
  1. 


[NSMutableDictionary](http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSMutableDictionary_Class/)* dict = [[[NSMutableDictionary](http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSMutableDictionary_Class/) alloc]init];











### 好的做法








	
  1. 


[NSMutableDictionary](http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSMutableDictionary_Class/)* dict = [[NSMutableDictionary](http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSMutableDictionary_Class/)dictionary];









* * *





### 戒律…


**总是**在你所写类的接口中提供类层面的便利构造函数。


### 为什么?


这样你所写类的使用者就能够遵循上面那条戒律了。


### 坏的做法











	
  1. 


@interface Foo : [NSObject](http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSObject_Class/)


	
  2. 



	
  3. 


- (id)initWithBar:(int)bar baz:(int)baz;


	
  4. 



	
  5. 


@end











### 好的做法








	
  1. 


@interface Foo : [NSObject](http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSObject_Class/)


	
  2. 



	
  3. 


- (id)initWithBar:(int)bar baz:(int)baz;


	
  4. 



	
  5. 


+ (Foo*)fooWithBar:(int)bar baz:(int)baz;


	
  6. 



	
  7. 


@end









* * *





### 戒律…


**一定**要理解对象所有权的转移什么时候会发生。ARC会帮你处理很多，但你还是得知道在背后到底发生了什么。

常见的获得对象所有权的方法：



	
  * 当你对一个类调用+alloc的时候你就拥有了新对象。

	
  * 当你对一个实例调用-copy或者-mutableCopy的时候你就拥有了新对象。

	
  * 当你将对象赋值给retain或者strong属性时，你就成为了该对象的拥有者。


常见的释放对象所有权的方法：

	
  * Assign another object (or nil) to a property with the (strong) attribute.

	
  * Let an owning local variable go out of scope.

	
  * Another object holding a reference to the object is destroyed.

	
  * 将另一个对象(或者nil)赋值给strong属性。

	
  * 本来拥有其所有权的一个局部变量超出了其生命周期范围。

	
  * 拥有该对象引用的另一个对象被释放了。





* * *





### 戒律…


**一定要**理解你属性和实例变量的内存管理策略，尤其当你在写自己的getter和setter时。确保你的实例变量上定义的存储策略是正确的，这样ARC才会正确工作。最简单的办法是单用一个@synthesize，然后覆盖setter及/或getter，在其中访问名字带下划线的实例变量，其行为跟属性中所声明的存储策略是一致的。






	
  1. 


@interface bar


	
  2. 



	
  3. 


@property (strong, nonatomic) id foo;


	
  4. 



	
  5. 


@end


	
  6. 



	
  7. 



	
  8. 


@implementation bar


	
  9. 



	
  10. 


@synthesize foo = foo_;


	
  11. 



	
  12. 


- (id)foo


	
  13. 


{


	
  14. 


        return foo_;


	
  15. 


}


	
  16. 



	
  17. 


- (void)setFoo:(id)foo;


	
  18. 


{


	
  19. 


        foo_ = foo;  // Retained/released automatically by ARC because of (strong) attribute on @property above


	
  20. 


        [self syncToNewFoo];  // The reason for our custom setter


	
  21. 


}


	
  22. 


 @end









* * *





### 戒律…


**永远不要**在你类中出现-dealloc方法，除非需要释放一些特殊的资源(关闭文件，释放由malloc()申请的内存，使定时器失效等)。其他的事情ARC都会帮你做的。



* * *





### 戒律…


只要你为属性加上了自己写的setter，那么**总是**也要加上自己写的getter，反之亦然。


### 为什么?









Getter和setter在内存管理、线程安全以及其他一些它们可能会造成副作用的地方需要有对等的行为。你不能指望一个合成的getter或setter能跟一个自己写的getter或setter有对等的行为。因此，如果你打算自己写一个getter或者setter的话，请你两个都写上。



* * *





### 戒律…


如果你**总是**需要在释放一个对象的时候做一些事情(比如使定时器失效)，那么你**一定要**为它的属性自己写一个setter，然后在你的-dealloc方法中将属性置为nil。


### 坏的做法











	
  1. 


@implementation Foo


	
  2. 



	
  3. 


@synthesize myTimer;


	
  4. 



	
  5. 


- (void)dealloc


	
  6. 


{


	
  7. 


        self.myTimer = nil; // 定时器没有置为失效，在对象回收后我们可能还会收到回调！


	
  8. 


}


	
  9. 



	
  10. 


@end











### 好的做法








	
  1. 


@implementation Foo


	
  2. 



	
  3. 


@synthesize myTimer = myTimer_;


	
  4. 



	
  5. 


- ([NSTimer](http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSTimer_Class/)*)myTimer


	
  6. 


{


	
  7. 


        return myTimer_;


	
  8. 


}


	
  9. 



	
  10. 


- (void)setMyTimer:([NSTimer](http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSTimer_Class/)*)myTimer


	
  11. 


{


	
  12. 


        [myTimer_ invalidate];


	
  13. 


        myTimer_ = myTimer;


	
  14. 


}


	
  15. 



	
  16. 


- (void)dealloc


	
  17. 


{


	
  18. 


        self.myTimer = nil; // 定时器在对象消失后肯定不会再回调了！在ARC模式下也需要这样。


	
  19. 


}


	
  20. 



	
  21. 


@end








### 为什么?









ARC负责在-dealloc的时候释放你的strong或retain实例变量，但它是直接释放变量，而不会调用你自己写的setter。因此如果你自己写的setter有其他的一些副作用(比如使定时器失效)，你还是得自己去调用它。



* * *





### 戒律…


在写构造函数的时候，**一定不要**在[super init]调用前写很多代码，写得越少越好。


### 为什么?


一般情况下，对父类构造函数的调用有肯能会失败，导致调用返回nil。如果发生这样的事，在调用父类构造函数之前你所做的一切初始化工作将变得毫无价值，甚至是有害的，必须要撤销。


### 坏的做法








	
  1. 


@implementation Foo


	
  2. 



	
  3. 


- (id)initWithBar:(Bar*)bar


	
  4. 


{


	
  5. 


        [bar someMethod];


	
  6. 


        // other pre-initialization here


	
  7. 



	
  8. 


        if(self = [super init]) {


	
  9. 


                // other initialization here


	
  10. 


        } else {


	
  11. 


        // oops! failed to initialize super class


	
  12. 


        // undo anything we did above


	
  13. 


}


	
  14. 



	
  15. 


        return self;


	
  16. 


}


	
  17. 



	
  18. 


@end








### 好的做法








	
  1. 


@implementation Foo


	
  2. 



	
  3. 


- (id)init


	
  4. 


{


	
  5. 


        if(self = [super init]) {


	
  6. 


                // minimal initialization here


	
  7. 


        }


	
  8. 



	
  9. 


        return self;


	
  10. 


}


	
  11. 



	
  12. 


// Other methods that put a Foo into a usable state


	
  13. 



	
  14. 


@end









* * *





### 戒律…


当你没有使用nib文件在写UIViewController的时候，**一定要**在-loadView中创建你的视图层次结构，永远不要在-init中做。你**只能够**在-loadView的实现中对view属性进行赋值。



* * *





### 戒律…


**永远不要**自己调用-loadView！UIViewController的view属性在它被访问时会懒加载。它也会在低内存情况下被自动清除。因此**永远不要**假设UIViewController的view会跟controller活的一样长。



* * *





### 戒律…


**总是**只创建你需要的视图一次，然后在必要时展示、隐藏或移动它们。**永远不要**在每次一有变化就重复回收和再创建你的视图层次结构。



* * *





### 戒律…


永远不要自己在UIView上调用-drawRect。可以调用-setNeedsDisplay。



* * *





### 戒律…


**一定**要避免在代码中做很长很复杂的操作。局部变量是你的朋友。


### 坏的做法








	
  1. 


[NSMutableDictionary](http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSMutableDictionary_Class/)* listDict = [[[NSMutableDictionary](http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSMutableDictionary_Class/)alloc] initWithDictionary:[[[NSUserDefaults](http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSUserDefaults_Class/)standardUserDefaults] objectForKey:@"foo"]];








### 好的做法











	
  1. 


[NSUserDefaults](http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSUserDefaults_Class/)* defaults = [[NSUserDefaults](http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSUserDefaults_Class/)standardUserDefaults];


	
  2. 


[NSDictionary](http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSDictionary_Class/)* dataModelDict = [defaults objectForKey:@"foo"];


	
  3. 


[NSMutableDictionary](http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSMutableDictionary_Class/)* listDict = [[NSMutableDictionary](http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSMutableDictionary_Class/)dictionaryWithDictionary:dataModelDict];











### 为什么?





	
  * 自说明

	
  * 易于理解

	
  * 调试器易于追踪代码





* * *





### 戒律…


当你没有在画什么东西的时候，方法名永远不要类似于-drawUI这样。总是用类似于-setupUI或者-resetUI这样的名字，尤其是当方法可能被调用不止一次的时候。



* * *





### 戒律…


**永远不要**重复。针对每个信息或者功能，都有个地方单独存放，程序中哪里能够用到就部署进去，甚至意味着有时候要触及到他人的代码。**永远不要**通过复制粘贴来编程。



* * *



2010.10.15更新


### 戒律…


**永远不要**调用-stringWithString，除非你：



	
  * 正将NSString转为NSMutableString。

	
  * 正将NSMutableString转为NSString。

	
  * 需要保证你正在处理的NSString*确实是不可变的。

	
  * 真的需要一份NSMutableString的拷贝，因为你打算分别修改它们。




### 为什么?


NSString是不可变的。一定不要去拷贝一个NSString，除非你想要一份可变的拷贝，或者确保一个你想保存的NSString的指针并没有变为NSMutableString的指针(那样的话，之后其他代码可能会改变了它的值)。实际上，试图拷贝NSString的行为仅仅是增加了该字符串的引用计数，然后再返回该字符串本身。
[/code]



* * *



2012.04.26新增


### 戒律...


对于那些可以接收有可变子类的对象(像NSString或NSArray)的属性，存储类型**一定要**使用(copy)。


### 为什么?


属性的(copy)存储类型(还有-copy方法)总是产生这些对象的不可变拷贝。因此你可以依靠这些拷贝的不可变性（译者注：不用担心会改变原来对象的值），而你不能依靠原来对象的不可变性。这也是对调用者的一个承诺：你将不会改变通过这种方式传给你的对象的值，即使你不小心改变了该对象拷贝的值（译者注：也不会影响原对象的值）。



* * *





### 戒律...


When overriding a method in a superclass, **ALWAYS** call the superclass' implementation,**even if** you know that super's implementation does nothing, **unless** you have a very good reason to not call super's implementation, in which case you **must** document your reason in the comments.

当覆盖父类中的某个方法时，**总是**要调用父类的实现，**即使**你知道父类的实现什么事情也没做，**除非**你有不调用父类实现的很好的理由，这种情况下你必须在注释中写下你的理由。


### 为什么?





	
  * 父类的实现在将来可能不会一直什么都不做。

	
  * 可能有其他类最终会介入到你的类和你的父类之间，并且对父类的这个方法有一个非空的实现。

	
  * 使你的代码更加能自说明，看你的代码就能知道这是个覆盖父类的方法，因为调用了父类的实现。




### 坏的做法











	
  1. 


@implementation Foo


	
  2. 



	
  3. 


- (void)awakeFromNib


	
  4. 


{


	
  5. 


        // do my setup


	
  6. 


}


	
  7. 



	
  8. 


@end











### 好的做法








	
  1. 


@implementation Foo


	
  2. 



	
  3. 


- (void)awakeFromNib


	
  4. 


{


	
  5. 


        [super awakeFromNib];


	
  6. 



	
  7. 


        // do my setup


	
  8. 


}


	
  9. 



	
  10. 


@end









* * *



2010.10.15新增


### 戒律...


在条件语句中，**永远不要**把指针或数值当成布尔值。


### 为什么?


布尔值有两个值：true和false(按照Objective-C的习惯我们使用YES和NO)。而指针的值是一个对象或者nil。数值的值是某个数或0。在条件语句上下文中，o和nil和布尔值的false是等同的，这是一种形式的隐式类型转换，会让你的代码难以理解。因此要显式的测试这些值，对于指针看它是否等于nil，对于整型看它是否等于0，对于浮点看它是否等于0.0，等等。


### 坏的做法











	
  1. 


- (void)fooWithBar:(Bar*)bar baz:(BOOL)baz quux:(float)quux


	
  2. 


{


	
  3. 


        if(bar && baz && quux) {


	
  4. 


                // do something interesting


	
  5. 


        }


	
  6. 


}











### 好的做法








	
  1. 


- (void)fooWithBar:(Bar*)bar baz:(BOOL)baz quux:(float)quux


	
  2. 


{


	
  3. 


        if(bar != nil && baz && quux != 0.0) {


	
  4. 


                // do something interesting


	
  5. 


        }


	
  6. 


}









* * *



2010.10.15新增


### 戒律...


**NEVER** use conditional statements to check for nil when you don't have to. In particular, understand that in Objective-C, the act of calling any method on a nil object reference is a no-op that returns zero (or NO, or nil), and write your code accordingly.

当没必要的时候，**永远不要**用条件语句去测试是否为nil。特别的是，在Objective-C里，在nil对象上调用任何方法相当于做了一步返回值为0(或者NO，或者nil)的空操作，你应该相应去写你的代码。


### 坏的做法








	
  1. 


- (void)setObj:([NSObject](http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSObject_Class/)*)obj


	
  2. 


{


	
  3. 


        if(obj_ != nil) {


	
  4. 


                [obj_ doCleanup];


	
  5. 


        }


	
  6. 


        if(obj != nil) {


	
  7. 


                [obj doSetup];


	
  8. 


        }


	
  9. 


        obj_ = obj;


	
  10. 


}








### 好的做法











	
  1. 


- (void)setObj:([NSObject](http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSObject_Class/)*)obj


	
  2. 


{


	
  3. 


        [obj_ doCleanup]; // Does nothing if obj_ == nil


	
  4. 


        [obj doSetup]; // Does nothing if obj == nil


	
  5. 


        obj_ = obj;


	
  6. 


}
























































































