layout: post
title: NDK开发相关技巧汇总
comment: true
tags: [技术, Android, NDK]
date: 2016-08-03 12:26:09
updated: 2016-08-03 12:26:09
---

------
### 简介
Android NDK(Native Development Kit)是基于Java JNI的使用C/C++和Java来混合开发应用的一种方式

### 函数签名的写法

Java代码中调用Native的代码还是比较简单的，把方法声明成为native，然后可以当作普通的Java方法一样来使用，只不过这个native的方法的实现是在Native中。

JNI是一个桥梁，让不同的语言能够在一起使用，不但Java能调用Native层代码，在Native层也是能够调用Java的代码。但是JNI的最初目的是能够让Java使用C/C++语言来解决Java做不到的事情，所以在Native中调用Java的方法要稍微费一点劲。要获取JNI的执行环境(JNIEnv)，要找到类和对象，更重要的是要写对函数签名，否则是找不到的。

### 函数签名的写法
<!-- more -->
“([type1];[type2]…)[return types]”

type包括：

B = byte
C = char
D = double
F = float
I = int
J = long
S = short
V = void
Z = boolean
V = void
Lfully-qualified-class = fully qualified class. For instance String – Ljava/lang/String;
[type = array of type
Samples: * “(Ljava/lang/String;I)V” // void foo(String str, int limit);

### 线程解惑

Native的代码是执行在其直接调入的Java的方法所在的调用栈里的，比较绕，简单来说吧，JNI的方法也是一个方法，只不过它是在Native层实现的，所以都是一系列的方法的调用，因此调用栈从Java层开始，一直到Native，JNI不会改变调用栈，因此也不会改变线程环境，除非你让它改变。

当你改变线程时，就要注意了，如果你在Native用pthread开启了一个新的线程，且这个线程又需要与Java通信，要调用Java层的方法，那么常规的方式是不行的，要先把线程attach到JNI环境，findClass也不会找到相应的类，因为这个线程是pthread_create出来的，不具备JNI的环境，甚至常规的类，方法和对象的引用在新衍生出的线程中统统都是无效的。

那么该如何做呢？首先，要先调用AttachCurrentThread来把线程attach到JVM；然后，把要在此线程里访问的Java类，方法和对象生成JVM的Global引用，也就是NewGlobalRef来保存引用；最后，当完成与Java的通信后要调用DetachCurrentThread来做detach。

### 注意内存问题

到了Native的环境，就要注意内存问题，因为Native的代码都是要手动的申请内存，手动的释放。当然，业务逻辑里面的申请和释放用标准的new/delete或者malloc/free，或者用智能指针之类的。JNI部分是有封装好的方法的，比如NewGlobalRef，NewLocalRef, DeleteGlobalRef, DeleteLocalRef等。

需要注意的是用这些方法创建出来的引用要及时的删除。因为这些引用都是在JVM中一个表中存放的，而这个表是有容量限制，当到达一定数量后就不能再存放了，就会报出异常。所以要及时删除创建出来的引用。

### 版本的兼容性

使用SDK开发应用时可以用minsdk和targetsdk来解决版本的兼容性问题，minsdk指定最低SDK版本要求，targetsdk指定目标的版本。<font color=red>但在NDK，只能用一个android-target来指定最低的版本要求，其实这就是限定了在NDK你能使用的API的范围。为了保证最好的兼容性，要保证NDK中的android-target与minsdk保持一致。</font>

SDK中的做法是指定了minsdk后，选择尽可能高的targetsdk，这样可以获取最好和最新的编译toolchains的支持。**但是NDK中不建议这样做，尽管你没有使用高版本的API，但是使用高版本来编译会链到高版本的库，有可能会导致问题，因为高版本的某些API实现方式会变。**比如signal.h中的signal函数，如果使用android-21编译，那么在低于5.0 版本的手机上是跑不起来，错误是无法找到signal函数，原因就是5.0以后signal.h中的signal函数的实现方式变了。

### 支持64位

5.0开始，Android有了64位处理器了。对于以Java作为平台语言的Android来说，特别是广大的应用开发者来说，这并不需要做什么特殊的处理。但是对于涉及到Native的代码时就要注意了，在编译的时候要为arm64准备东西了。在编译的时候要为arm64编译出一个target。

但是问题来了，arm64只有当android-target设置为21时才能编译出来，而我们的应用不可能只target到5.0，前面讲到了我们要对齐到最低版本。**解决方案就是构建二次：**

第一次正常target到最低版本构建出arm和armeabi-v7a的库
第二次target到21，编译出arm64的库
再把这些so打包起来就可以了。

### 使用第三方工具来简化开发

最好的开发方式不是自己写，而是去用别人现有的东西，子曰：不能重复造轮子。NDK的开发，也是有一些第三方的工具来帮助我们减少开发量的。SWIG就是一个优秀的工具，它能免去写丑陋的JNI方法的痛苦，而且SWIG是编译工具链的一个组件，不是运行时，所以不会带来性能上的损失。

### 多多参考NDK文档以及官方教程和指导

使用任何的别人提供的东西，最好获取帮助的方式就是看人家给你的文档和指导。现在的文档都写的很详细了。Android开发者官网上面也有很多关于NDK开发的教程，都值得仔细读一读的。

### C++的兼容性调试
1. __int64找不到符号
采用int64_t来代替：
```c
#if defined(__ANDROID__)
typedef int64_t __int64;
#endif
```
2. <sys/io.h>找不到android下不需要直接引用该文件，用下面的宏去掉即可
```c
#if !defined(__APPLE__) && !defined(__ANDROID__)
#include <sys/io.h>
#endif
```
3 .SO_NOSIGPIPE找不到
SO_NOSIGPIPE在mac中存在，可惜在android中不存在。请使用MSG_NOSIGNAL来代替
```
#if defined(__ANDROID__)
#define SO_NOSIGPIPE MSG_NOSIGNAL
#endif
```
4. uint64_t, int64_t, uint32_t, int32_t等类似类型找不到
请检查你的头文件包含，将系统的头文件放在自已的头文件之前。因为你自己的头文件有可以定义了重复的类型，导致系统头文件出错。

5. S_IREAD、S_IWRITE或者__S_IREAD、__S_IWRITE找不到
请用S_IRUSR、S_IWUSR代替

6. pthread_cancel找不到
这个android并未实现，有一些替代方法，[具体见](http://bbs.rosoo.net/thread-10289-1-1.html)

7. getifaddrs, <ifaddr.h> 找不到
android并没有实现。不过谢天谢地，有人已经帮我们[实现了](https://github.com/kmackay/android-ifaddrs)。


8. <sys/statvfs.h>找不到
请用此来代替：
```c
#if defined(__ANDROID__)
#  include <sys/vfs.h>
#  define statvfs statfs
#else
#  include <sys/statvfs.h>
#endif
```
### 参考书籍：
>《Pro Android C++ with NDK》是一本相当不错的书
