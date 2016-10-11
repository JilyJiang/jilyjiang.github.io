layout: post
title: Android系统指令查缺补漏
comment: true
tags: [Android,技术]
date: 2016-10-11 11:57:45
updated: 2016-10-11 11:57:45
---

------
Android命令查缺补漏
#### 查询
* croot : 回到android的根目录
* cgrep：查询*.c/*.cpp的文件
* jgrep：
* resgrep：查找res/*.xml中的内容
* godir：是croot的逆命令，快速进入到包含某个文件的目录
>比如我们要进到包含init.rc目录
$godir init.rc
第一次运行会提示建立索引，会在你根目录建立filelist文件

#### pm 指令
pm list package -f 列出已安装的apk的包名
<!--more-->
#### dumpsys 指令
dumpsys activitys/services
dumpsys input
dumpsys surfaceFlinger

#### 截图/抓图/录像 指令
screencap
screenshot
screenrecord

#### wm 指令
wm size
wm density




