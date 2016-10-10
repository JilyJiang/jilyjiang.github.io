layout: post
title: Android_Studio使用技巧
comment: true
tags: [AS,技术]
date: 2016-10-10 20:48:45
updated: 2016-10-10 20:48:45
---

------
### AS设置耍酷的背景黑色&IDE中文乱码
![图1](http://)
中文乱码记得安装中文字体：`sudo aptitude install ttf-wqy-microhei`
### 复用eclipse的快捷键
![图2](http://)
#### AS导入android 源代码
###### 编译源码idegen模块及生成AS配置文件(*.ipr)

在开始编译idegen模块前，你一定知道需要先全局编译出out目录及相关文件吧，这个不再过多说了，我们通过如下命令编译idegen模块：

`mmm development/tools/idegen/`

这行命令的意思是编译idegen这个模块项目，然后生成idegen.jar文件（不信你自己去查看这个模块的Android.mk的目标吧，不多解释）。运行完以后如果看到如下信息则说明编译OK：
<!--more-->
......
`#### make completed successfully (7 seconds) ####`

接着执行如下脚本：

`development/tools/idegen/idegen`

这行命令的意思是在根目录生成对应的android.ipr、android.iml IEDA工程配置文件。等待片刻得到类似如下信息说明OK：

`excludes: ms
Traversed tree: 194799ms`

通过如上操作我们就完成了基本的源码配置工作。

###### 导入Android Studio前的一些客户化操作

大家都知道使用Eclipse倒入源码很慢，Android Studio导入源码时也慢，所以建议修改android.iml文件（将自己不用的代码去掉），然后再导入Studio。

就像下面摘取的android.iml文件1887行开始的这些一样：

```
<sourceFolder ="file://$MODULE_DIR$/./sdk/testapps/userLibTest/src" isTestSource="true"/>
<sourceFolder ="file://$MODULE_DIR$/./tools/external/fat32lib/src/main/java" isTestSource="false"/>
<excludeFolder ="file://$MODULE_DIR$/out/eclipse"/>
<excludeFolder ="file://$MODULE_DIR$/.repo"/>
<excludeFolder ="file://$MODULE_DIR$/external/bluetooth"/>
<excludeFolder ="file://$MODULE_DIR$/external/chromium"/>
<excludeFolder ="file://$MODULE_DIR$/external/icu4c"/>
<excludeFolder ="file://$MODULE_DIR$/external/webkit"/>
<excludeFolder ="file://$MODULE_DIR$/frameworks/base/docs"/>
<excludeFolder ="file://$MODULE_DIR$/out/host"/>
<excludeFolder ="file://$MODULE_DIR$/out/target/common/docs"/>
<excludeFolder ="file://$MODULE_DIR$/out/target/common/obj/JAVA_LIBRARIES/android_stubs_current_intermediates"/>
<excludeFolder ="file://$MODULE_DIR$/out/target/product"/>
<excludeFolder ="file://$MODULE_DIR$/prebuilt"/>
```

我们可以仿照上面这段代码的<excludeFolder url="file://$MODULE_DIR$/.repo"/>写法一样过滤掉不需要的内容，这样在导入时就会快很多。

也可以通过Android Studio的Project Stucture 打开左侧Modules，然后将右侧Sources中一些目录Excluded掉。
###### 使用技巧

上图我们看见了，可以通过Android Studio搜索整套源码的代码了。但是这时候如果你打开一个Service.Java类，然后把鼠标放在其中任意方法的Intent参数上按住CTRL+鼠标左键跳转到Intent类你会发现跳转过去的是一个Intent.class文件，为啥呢？因为他跳转的是你的默认SDK中的jar内部的class文件。既然要修改查看整套源码，这么跳转得多蛋疼啊，所以我们需要配置让其能跳转到Intent.java文件，具体做法如下：

首先删掉依赖中的所有依赖，只保留下图中没被选中的那两个（当然你可以选择保留一些你用到的其他jar），如下：
![AS1](http://)
接着点击加号的JARs or directories将你源码的frameworks及external和你用到的其他跳转目录添加到依赖中，然后apply即可。

这时候我们在像上面一样打开Service.java跳转Intent，你会发现像下图一样直接跳转到你源码路径下的Intent.java文件了，如下：
![AS2](http://)
到此对于平时只是查看源码的人来说已经够用了。