layout: post
title: 跟着Android源码学app开发之settings分析(一)
comment: true
tags: [技术, Android, Settings]
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
项目如期完成，于是决定通过setting的源码把app开发的知识点全部回顾和整理一遍；
<!-- more -->
![setting](http://oa1wnpe3m.bkt.clouddn.com/settings.png) 
## 解析AndroidManifest.xml的一些不太常见的知识点：
```xml
	<manifest xmlns:android="http://schemas.android.com/apk/res/android"
        package="com.android.settings"
        coreApp="true"
        android:sharedUserId="android.uid.system">
    <original-package android:name="com.android.settings" />
	……
	……
	……
</manifest>
```
##  `coreApp="true"`：
关于coreApp=true的说明，在manifest中增加该属性，其实并不是代表该APP具有系统权限，**而是把该类app归类为核心APP，核心app其实也是最小android framework系统。**那么核心APP的作用是什么呢？在Android3.0之后，Android就增加了加密机制，当系统开机时检测到系统加密，他就把核心APP全部启动，并显示UI提供用户输入密码，密码正确后才会启动完整系统。

## `<original-package>`
   <manifest>标签中package属性用于设置应 用程序的进程名，即在运行时使用ddms查看到的进程名。
   <original-package>标签用以设置应用 源码包名，<font color=red>即Java文件所在的源码程序包层次，android工程中真实的源代码层次结构</font>。
   <manifest>中package属性若与<original-package>的android:name值相同，配置组建时android:name属性值 可以使用".ClassName"形式。
   使用<original-package>标签后，在<activity><service><receiver><provider>中的android:name属性需要写完整的包名，".ClassName"形式无效。
   
   <font color=red size=5>注意:</font>**<manifest>标签中package属性只是告诉系统应用的进程名；因此进程名（Manifest中package属性的值）与<original-package>的值可以不一样**。
  
需要注意下
```xml
<manifest
        xmlns:android="http://schemas.android.com/apk/res/android"
        package="com.android.settings"
        android:sharedUserId="@string/sharedUserId"
        >
```
这里`package="com.android.settings"`，产生的R.java就会在com.android.settings

<original-package android:name="com.android.settings2" /> 这个地方表示，源码包是com.android.settings2。所以在代码中引用的R.java必须是import com.android.settings.R;
##  `requiredForAllUsers`&`supportsRtl`
***requiredForAllUsers***
Flag to specify if this application needs to be present for all users.(此apk是否需要默认面对所有用户，因为android支持多用户系统切换)
***supportsRtl***
Declare that your application will be able to deal with RTL (right to left) layouts.（此应用支持从右向左布局,此属性为解决入阿拉伯文，希伯来语等从右向左的阅读习惯）[The project references RTL attributes, but does not explicitly enable or disable RTL support](http://stackoverflow.com/questions/27378921/the-project-references-rtl-attributes-but-does-not-explicitly-enable-or-disable)
[中文文档](http://www.apkbus.com/blog-327085-57866.html)

```xml
   <application android:label="@string/settings_label"
            android:icon="@mipmap/ic_launcher_settings"
            android:taskAffinity=""
            android:theme="@style/Theme.Settings"
            android:hardwareAccelerated="true"
            android:requiredForAllUsers="true"
            android:supportsRtl="true">
        <!-- Settings -->

        <activity android:name="Settings"
                android:label="@string/settings_label_launcher"
                android:taskAffinity="com.android.settings"
                android:launchMode="singleTask">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <action android:name="android.settings.SETTINGS" />
                <category android:name="android.intent.category.DEFAULT" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
```

