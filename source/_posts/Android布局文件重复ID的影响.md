layout: post
title: layout布局文件有重复ID，会有什么影响
comment: true
tags: [Android,技术]
date: 2016-08-27 19:21:28
updated: 2016-08-27 19:21:28
---

前一段时间，一个同事来找我处理一个他觉得很奇怪的一个问题；
>同一个运营商的APK的在Android4.4上，电影的节目详细列表上，电影的名称显示正常；但在Android5.1上，电影的节目详细列表上，节目名称始终不显示；他们初步看现象以为是Android5.1上有什么特殊的图层渲染机制导致的;

反编译apk发现对应的布局文件中，有重复的ID（一般情况下，eclipse和AS都会提示重复id报错，但若apk的开发人员是直接在android 源代码编译的时候，编译则不会报错）
### 正文
##### **情况一：同个一个Xml文件中的同名**
<!--more-->

在同个一个Xml文件的中若同名了，<font color=red>则前一个有效，而后一个无效</font>，即后一个Null掉。如：


```xml
<Button  
    android:id="@+id/button"  
    android:layout_width="wrap_content"  
    android:layout_height="wrap_content"  
    android:layout_above="@+id/textView1"  
    android:layout_alignRight="@+id/textView1"  
    android:text="Button1" />  
  
<Button  
    android:id="@+id/button"  
    android:layout_width="wrap_content"  
    android:layout_height="wrap_content"  
    android:layout_alignRight="@+id/button"  
    android:layout_centerVertical="true"  
    android:text="Button2" />
```


前一个Button有效（即 android:text="Button1" ），后一个无效。
>客户的apk就是在4.4上正常就是因为此原因，因为客户原本第二个布局文件重复ID是无意义的；所以在Android4.4上没有发现任何异常;但Android5.0后解析是根据最近的id解析的，从而使第一个也无效了

#####  **情况二：在不同的Xml 文件中的同名**

在同个一个Xml文件的中若同名了，两者都有效的。

当android的工程越来越大。xml文件越来越多时，避免不了两个xml文件中同样的组件使用同样的id名字，gen目录下的R.java文件中，有关id的声明都在id的class中，即public static final class id{}；
当两个xml文件中同样的组件，比如Button，有可能很多个文件中，都有id=”@+id/Button”，开始以为在Java类中引用会重复的id造成程序的不识别。

后来偶然一次错误发现，只需你setContentView(R.layout.test);中的xml文件以及这个xml文件相关的xml文件中的id不重复，在类中使用findViewById(R.id.Button);时，程序是可以正确识别的。

##### 结论：
因为在Android的框架设计中，每一个控件都隶属于一棵控件树，每个控件都被其父控件所管理与调配，而根控件是一个容器控件，所有的子控件都是构造在这个根控件之上，这样并形成了一个控件树的控件域，在这个控件域中是不允许重名的，超出了这个控件域则这些控件的ID是无效的，也就是说<font color=red>**在容器控件中的子控件是不允许重名的，而不在同一容器控件中的两个控件重名也无所谓**</font>。