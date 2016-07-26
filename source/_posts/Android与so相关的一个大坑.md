layout: post
title: Android与so相关的一个大坑
comment: true
tags: [Android, 技术, NDK]
date: 2016-07-11 14:29:02
updated: 2016-07-11 14:29:02
---

------
# 与armeabi和armeabi-v7a相关的一个坑
上周验证253个游戏apk的兼容性问题时，发现有一部分apk在android4.4上跑的极为欢快，但在android5.1时候，一运行就FC；
突然想起以前在知乎上看到一篇文章[与 .so 有关的一个长年大坑](https://zhuanlan.zhihu.com/p/21359984),决定搬运过来做个记录，感谢Caspar的分享！

Android 应用开发中不可避免的会引入第三方的代码。如果是开源项目风险相对可控，如果引入商用的 SDK 那就要谨慎了，难免会有这样或那样的问题。比如我们今天要说的这一个。
![结构](http://oa1wnpe3m.bkt.clouddn.com/armeabi.png  "结构")
对集成过第三方 SDK 的同学，上图中的目录结构应该不陌生。正常情况下我们只需要将不同版本的 .so 文件分别放置。***但如果我们要集成的这个第三方 SDK 偏偏没有 arm-v7a 的版本呢？是删除 armeabi-v7a 目录只保留 armeabi ？还是说两个目录下 .so 文件数不同也没有关系？系统会加载哪个 .so 呢？***

<font  color=red size=5>** 如果只对结论感兴趣可以直接跳到最后**</font>
<!-- more -->
为了方便说明我们先引入 FAT Binary 的概念。我们知道不同的 CPU 支持的指令集也不一样，那么如果我们需要让 App 尽可能不同的 CPU 上都可以正常运行该怎么做呢？简单，只需要将不同版本的 Binary 放在一个文件里，运行时按需取用就可以了。这就是 FAT Binary 的典型实现。Android 实现 FAT 的方式有些不同，就是上边提到的将 .so 文件放置在相应文件夹中。在 Android 系统中 ndk 默认会生成如下 7 种 .so。
![libs](http://oa1wnpe3m.bkt.clouddn.com/libs.png  "libs")
在 apk 文件中带这么多版本的 .so 是一种很不经济的做法：

mips / mips64: 极少用于手机可以忽略
x86 / x86_64: x86 架构的手机都会包含由 Intel 提供的称为 Houdini 的指令集动态转码工具，实现 对 arm .so 的兼容，再考虑 x86 1% 以下的市场占有率，x86 相关的两个 .so 也是可以忽略的
armeabi: ARM v5 这是相当老旧的一个版本，缺少对浮点数计算的硬件支持，在需要大量计算时有性能瓶颈
armeabi-v7a: ARM v7 目前主流版本
arm64-v8a: 64位支持
**这样我们就可以明确 mips, mips64, x86, x86_64 这 4 个 .so 我们是不需要的。**

我们回到开头提到的问题：

假定我们现在的情况是这样的（b.so 就是那个只有 armeabi 版本的第三方 .so）：
![b](http://oa1wnpe3m.bkt.clouddn.com/b.so.png  "b")

如果这样放置的话，在 ARM / ARM v7 两种设备上运行 apk 时会分别执行哪个 .so 呢？

<font color=red>答案是：不确定……</font>

这么坑爹的答案是怎么来的呢？

由于 Android 上 FAT binrary 的设计如此阳春，在 apk 安装时就需要根据 CPU 情况执行对应版本 .so 的拷贝。对上边的情况最合理的一种做法应该是使用 armeabi-v7a/a.so 和 armeabi/b.so 这两个文件。Google 最初也是这么想的，然后就引入了 Bug…

[Native library copy issue when install apk with different abi native libraries on device](https://android-review.googlesource.com/#/c/80810/)
![1](http://oa1wnpe3m.bkt.clouddn.com/4.4.png  "1")

上图是到 Android 4.4 还在使用的 .so 文件拷贝逻辑，看起来没有问题？

坑爹是 Android 在安装 apk 文件时没有保证 zip entry 的扫描顺序，所以同样的文件放置会带来两种不同的安装结果：
![2](http://oa1wnpe3m.bkt.clouddn.com/4.4.2.png  "2")
![3](http://oa1wnpe3m.bkt.clouddn.com/4.43.png  "3")


看的有点头晕？简而言之，如果按我们上面的放置方式，安装后系统可能只拷贝了 armeabi-v7a/a.so。如果执行到 b.so 的逻辑，程序显然会 crash。

这边还有个小插曲，这个 bug 的发现者在提交时其实已经给出了完善的解决方案，但在经历了快有小一年的 code review 后 Android 官方表示：我们自己另起炉灶修好了=_=。
![p](http://oa1wnpe3m.bkt.clouddn.com/patch.png  "p")

这个问题确实在 [Android 5.0 已经 “修复”](https://android-review.googlesource.com/#/c/90261/) 了。“修复” 方式简单粗暴，不再以文件为粒度匹配 abi，直接拷贝整个文件夹=_=。所以如果按我们之前的放置方法，在 Android 5.0+ 如果执行到 b.so 也是一定会 crash 的。

上面提到，只保留 armeabi 文件夹从性能角度是不明智的。<font  color=red size=5>**正确的做法是将 armeabi/b.so 复制一份到 armeabi-v7a/b.so. </font>这是由于 ARM v7 是前向兼容 ARM v5 的。**

撇开上面曲折离奇的故事，放置 .so 文件的正确姿势其实就两句话：

<font  color=red size=4>1.为了减小 apk 体积，只保留 armeabi 和 armeabi-v7a 两个文件夹，并保证这两个文件夹中 .so 数量一致
2.对只提供 armeabi 版本的第三方 .so，原样复制一份到 armeabi-v7a 文件夹</font>

	还有一个简单粗暴的办法，就是直接把系统的abilist改成只有armeabi，拿掉armeabi-v7a等其他配置选项就OK，
改变<font color=red>ro.product.cpu.abilist</font>即可

