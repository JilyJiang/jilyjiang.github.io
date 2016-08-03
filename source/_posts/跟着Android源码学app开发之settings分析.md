layout: post
title: 跟着Android源码学app开发之settings分析
comment: true
tags: [技术, Android, settings]
date: 2016-07-26 11:56:50
updated: 2016-07-26 11:56:50
---

------
## 问题背景：
前段时间，因为某个项目的特殊要求，要求在三天之内提供一个BlueTooth的apk可以使用，界面风格尽量符合某司要求即可；操作方式不限；于是和老大协商后，决定只修改Android的setting的背景和UI风格，尽快出一个单独的bluetooth给客户！
## 现状：
* Android 的setting 源码
* 木有蓝牙的相关开发经验，查看setting的src，发现高达40几个java类
* 时间紧迫，只有三天时间
## 正题：
所有的apk项目分析一样，先结构后细节；

