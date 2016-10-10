layout: post
title: 屏蔽系统默认错误弹框--xxx，停止运行/ANR
comment: true
tags: [Android，Framework, 技术]
date: 2016-08-27 19:21:28
updated: 2016-08-27 19:21:28
---

------

因为运营商某些apk的兼容性存在一定程度问题，但apk因为特殊体制的问题，不会对其apk进行更改；这时就需要底层协助厂商处理一些兼容性的apk，比如某些时候apk退出会弹出一个‘xxx，已停止运行’，但实际上apk所有功能均正常，不影响用户的实际体验，于是乎就去想办法去掉这个ErrorDialog；
依旧根据常规分析思路，现在找到其对应的source code：
  <!--more-->
  1. 根据“已停止运行”找到关键字;
 插入图1
   2. 根据aerr_application 查找对应的dialog
   插入图2
   3. 根据dialog找到调用dialog的地方，
   插入图3
   4. 然后对此位置进行一个判断进行兼容性处理即可，这样即可控制对应的界面情况
   插入图4
   
