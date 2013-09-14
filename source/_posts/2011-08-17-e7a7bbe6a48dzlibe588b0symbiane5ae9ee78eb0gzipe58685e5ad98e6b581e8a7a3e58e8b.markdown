---
author: njuxjy
comments: true
date: 2011-08-17 14:11:00+00:00
layout: post
slug: '%e7%a7%bb%e6%a4%8dzlib%e5%88%b0symbian%e5%ae%9e%e7%8e%b0gzip%e5%86%85%e5%ad%98%e6%b5%81%e8%a7%a3%e5%8e%8b'
title: 移植zlib到Symbian实现gzip内存流解压
wordpress_id: 193
categories:
- symbian
---

客户端（v3版本）原先在处理服务器端传回gzip数据时采用的策略是先将gzip流保存成.gz文件，再用解压文件的方式将数据解压出来。这种方式显然不如直接从内存中实现gzip解压来得高效，但由于Symbian SDK中zlib的版本过低（1.1.3）等原因，网上介绍的很多内存流解压方法并不适用于此：



	
  * [http://apps.hi.baidu.com/share/detail/8355062](http://apps.hi.baidu.com/share/detail/8355062)

	
  * [http://blog.sina.com.cn/s/blog_4d6f62190100md6k.html](http://blog.sina.com.cn/s/blog_4d6f62190100md6k.html)

	
  * [http://blog.sina.com.cn/s/blog_65db99840100kwh9.html](http://blog.sina.com.cn/s/blog_65db99840100kwh9.html)

	
  * [http://www.devdiv.com/thread-8625-1-1.html](http://www.devdiv.com/thread-8625-1-1.html)

	
  * [http://www.developer.nokia.com/Community/Discussion/showthread.php?155614-GZip-and-RReadStream-problem](http://www.developer.nokia.com/Community/Discussion/showthread.php?155614-GZip-and-RReadStream-problem)




在解决过程中遇到了一些困难，开始使用[http://apps.hi.baidu.com/share/detail/8355062](http://apps.hi.baidu.com/share/detail/8355062) 中的方法，并且将服务器返回的gzip数据去掉开头的10个gzip header，但程序始终卡在第13行：








	
  1. CBufFlat* CETNetOperator::DeCompressMemL(const TDesC8& aData)   

	
  2.      {   

	
  3.      TInt nBufferSize = 128;   

	
  4.      HBufC8* nSrc = NULL;   

	
  5.      HBufC8* nTemp = aData.Mid(10).Alloc();   //去掉开头10个字节   

	
  6.      nSrc = nTemp;   

	
  7.      CleanupStack::PushL(nSrc);   

	
  8.      CBufFlat* nBufFlat = CBufFlat::NewL(nBufferSize);   

	
  9.      CleanupStack::PushL(nBufFlat);   

	
  10.      CBufferManager* nBufferManager = CBufferManager::NewLC(*nSrc, *nBufFlat,   

	
  11.                  nBufferSize);   

	
  12.      CEZDecompressor* decompressor = CEZDecompressor::NewLC(*nBufferManager);   

	
  13.      while (decompressor->InflateL())   

	
  14.            {// loop here until the file is compressed   

	
  15.            }   

	
  16.      //    nBufFlat->Ptr(0);   

	
  17.      CleanupStack::PopAndDestroy(3);   

	
  18.      return nBufFlat;   

	
  19.      }   





然后尝试使用Symbian SDK自带的zlib库，include <ezlib.h>，代码如下：






	
  1. int ungzip(char* source, int len, char* des)   

	
  2.     {   

	
  3.     int ret, have;   

	
  4.     int offset = 0;   

	
  5.     z_stream d_stream;   

	
  6.     Byte compr[KETNET_BUFFER_SIZE] ={0}, uncompr[KETNET_BUFFER_SIZE * 4] ={0};   

	
  7.     memcpy(compr, (Byte*) source, len);   

	
  8.     uLong comprLen, uncomprLen;   

	
  9.     comprLen = len;   

	
  10.     uncomprLen = KETNET_BUFFER_SIZE * 4;   

	
  11.     strcpy((char*) uncompr, "garbage");   

	
  12.     d_stream.zalloc = Z_NULL;   

	
  13.     d_stream.zfree = Z_NULL;   

	
  14.     d_stream.opaque = Z_NULL;   

	
  15.     d_stream.next_in = compr;   

	
  16.     d_stream.avail_in = comprLen;   

	
  17.     ret = inflateInit2(&d_stream,47);   

	
  18.     if (ret != Z_OK)   

	
  19.         {   

	
  20.         return ret;   

	
  21.         }   

	
  22.     do  

	
  23.         {   

	
  24.         d_stream.next_out = uncompr;   

	
  25.         d_stream.avail_out = uncomprLen;   

	
  26.         ret = inflate(&d_stream, Z_NO_FLUSH);   

	
  27.         switch (ret)   

	
  28.             {   

	
  29.             case Z_NEED_DICT:   

	
  30.                 ret = Z_DATA_ERROR;   

	
  31.             case Z_DATA_ERROR:   

	
  32.             case Z_MEM_ERROR:   

	
  33.                 (void) inflateEnd(&d_stream);   

	
  34.                 return ret;   

	
  35.             }   

	
  36.         have = uncomprLen - d_stream.avail_out;   

	
  37.         memcpy(des + offset, uncompr, have);   

	
  38.         offset += have;   

	
  39.         }   

	
  40.     while (d_stream.avail_out == 0);   

	
  41.     inflateEnd(&d_stream);   

	
  42.     memcpy(des + offset, "\0", 1);   

	
  43.     return ret;   

	
  44.     }  





发现程序始终在第17行返回-2，即流初始化Z_STREAM_ERROR错误，zlib1.1.3的zlib.h中说原因是参数设置不正确，stream为空或者windowBits为负。而inflateInit2（）要求windowBits在8~15之间，在其他范围内的值会造成初始化错误。随后改成了15，但是在第26行inflate进行解压时返回值为-3，即Z_DATA_ERROR错误，原因是"the input data is corrupted (input stream not conforming to the zlib format or incorrect adler32 checksum)"。

于是怀疑是Symbian自带的zlib的版本问题，刚哥说得自己移植zlib库进去。由于没移植过程序，一开始犯了个错误，原地绕了个圈子：我从网上下了个人家已经编译好的1.2.3版本的zlib.lib文件和几个头文件放到项目工程中去，以为光更换了zlib版本就行了，结果还是那两个错误。最后才走移植的路子，并编译运行成功，现将步骤记录如下：

从网上下载1.2.3的源代码，将下面这些.h .c文件放入工程下面（全部放进去会使工程太大，由于只需要用zlib的解压功能，所以删除了部分没用的文件和函数，还可以继续减减肥，这个以后再做吧）：

[![](http://www.xiaojiayi.com/wp-content/uploads/2011/08/2.jpg)](http://www.xiaojiayi.com/wp-content/uploads/2011/08/2.jpg)

使用zlib提供的uncompress函数进行解压缩：






	
  1. ZEXTERN int ZEXPORT uncompress OF((Bytef *dest, uLongf *destLen, const Bytef *source, uLong sourceLen));  




这个函数的实现其实就是上面的ungzip函数。在inflateInit2（）的时候使用了47，在1.2.3版本的zlib.h中有这么一段话，是1.1.3版本中没有的：




[![](http://www.xiaojiayi.com/wp-content/uploads/2011/08/21.jpg)](http://www.xiaojiayi.com/wp-content/uploads/2011/08/21.jpg)





难怪1.1.3版本的不行，因为在1.1.3版本中的inflateInit2()中对gzip只字未提，只有gzopen()等函数能对.gz的文件进行解压。因此1.1.3版本不支持内存流的gzip解压。

这样以后，一般的项目就算是可以大功告成了。下面的步骤仅针对于自己参与的这个项目，没兴趣的可以跳过：） 由于它在其他地方用到了Symbian SDK自带的系统文件ezlib.h，而这一部分代码没法更改为使用新移植进去的zlib库，所以只能在系统中保持两个库的并存，这样就会造成很多函数名和类、结构体声明重复，无法通过编译。唯一想到的办法就是将新加入的zlib源文件中与老版本的zlib冲突的部分全部重命名，但这是一个巨大的工作量，因为宏定义特别多。那就先给zlib减减肥吧，仅保留需要的解压缩的那部分。过程就不详说了，总之多尝试吧，也不是个容易的活。

在这过程中产生了些副产品：



	
  1. Carbide的断点调试，比打日志好多了，f6单步, f8执行

	
  2. 明白了移植是怎么回事

	
  3. 最好服务端过来的数据头部有gzip数据包的大小信息，可以减少客户端动态分配的内存







