layout: post
title: Android 5.0 (L) Service Intent must be explicit
comment: true
tags: [android， service，intent，技术]
date: 2016-07-10 11:42:08
updated: 2016-07-10 11:42:08
---

------
# android 5.0 service bind 必须要显式声明
上周在南京出差，在把江苏中国移动的apk从android4.4.4移植到android5.1.1；所有的内容和结构都porting完后，满怀信心的登陆账号+密码，从界面上的第一个icon开始点击测试，一路下来一切正常，正得意的想，工作终于快搞完了O(∩_∩)O哈哈~；结果给我来了一个**surprise，What？**在点击到最后的应用商城时crash了，于是引出这篇blog
## 问题现象：
<!-- more -->
```java
Process: com.xxx.appstore, PID: 5719
java.lang.RuntimeException: Unable to start activity ComponentInfo{com.xxx.appstore/com.xxx.appstore.authorClient}: java.lang.IllegalArgumentException: Service Intent must be explicit: Intent { act=com.xxx.authorClient.BIND }
        at android.app.ActivityThread.performLaunchActivity(ActivityThread.java:2255)
        at android.app.ActivityThread.handleLaunchActivity(ActivityThread.java:2317)
        at android.app.ActivityThread.access$800(ActivityThread.java:143)
        at android.app.ActivityThread$H.handleMessage(ActivityThread.java:1258)
        at android.os.Handler.dispatchMessage(Handler.java:102)
        at android.os.Looper.loop(Looper.java:135)
        at android.app.ActivityThread.main(ActivityThread.java:5070)
        at java.lang.reflect.Method.invoke(Native Method)
        at java.lang.reflect.Method.invoke(Method.java:372)
        at com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:836)
        at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:631)
 Caused by: java.lang.IllegalArgumentException: Service Intent must be explicit: Intent { act=com.xxx.authorClient.BIND}
        at android.app.ContextImpl.validateServiceIntent(ContextImpl.java:1603)
        at android.app.ContextImpl.bindServiceCommon(ContextImpl.java:1702)
        at android.app.ContextImpl.bindService(ContextImpl.java:1680)
        at android.content.ContextWrapper.bindService(ContextWrapper.java:528)
        at com.tbse.wnswfree.util.IabHelper.startSetup(IabHelper.java:262)
        at com.tbse.wnswfree.InfoPanel.onStart(InfoPanel.java:709)
        at android.app.Instrumentation.callActivityOnStart(Instrumentation.java:1217)
        at android.app.Activity.performStart( Activity.java:5736)
        at android.app.ActivityThread.performLaunchActivity( ActivityThread.java:2218)
        at android.app.ActivityThread.handleLaunchActivity( ActivityThread.java:2317)
        at android.app.ActivityThread.access$800( ActivityThread.java:143)
        at android.app.ActivityThread$H.handleMessage( ActivityThread.java:1258)
        ...
```
因为上帝提供的apk是不提供代码，并且原则上他们也不可能修改的代码的；于是乎反编译上帝的apk，然后进行查看源代码：
```java
Intent intent = new Intent("com.xxx.authorClient.BIND");
getContext().bindService(intent, serviceConnection, Context.BIND_AUTO_CREATE);
```
看完代码我一脸的懵逼了o(╯□╰)o，这代码看起来没有任何问题啊，于是乎反过来继续排查上述log：<font color=red>`java.lang.IllegalArgumentException: Service Intent must be explicit:`</font>
**What？非法声明异常，service的intent必须显式声明？**
印象中，android以前没有这个要求的啊，于是查阅android的官网+Google 之；终于找到如下结论：

`**Since Android 5.0 (Lollipop) bindService() must always be called with an explicit intent. This was previously a recommendation, <font color=red>but since Lollipop it is enforced:</font>** java.lang.IllegalArgumentException: Service Intent must be explicit is thrown every time a call to bindService() is made using implicit intents. The difference between implicit and explicit intents is that the latter specifies the component to start by name (the fully-qualified class name). [See the documentation about intent types here](https://developer.android.com/guide/components/intents-filters.html).

The issue you are having is due to not having upgraded to a newer version of the google libraries, which comply with Android's restrictions on implicit intents when binding a service on Android 5 Lollipop. To fix the issue, you can upgrade the library to a newer version if available, or update the library code yourself and build your project with the modified version.

If there's no suitable library upgrade in the general case, you need to modify the source code (in your case  com.google.analytics.tracking.android.AnalyticsGmsCoreClient.connect()) to call intent.setPackage(packageName) before calling bindService() where intent is the first argument of the bundService() call and packageName is the name of the package containing the service the code is attempting to start (in your case "com.google.android.gms.analytics").

You can use this code as an example how to do this: [Unity's updated version of the Google Licensing Library (LVL) which calls bindService with an explicit intent and does not result in IllegalArgumentException](https://github.com/Unity-Technologies/UnityOBBDownloader/blob/master/Plugins/Android/UnityOBBDownloader/src/com/google/android/vending/licensing/LicenseChecker.java).

Yet another way to get around the problem is to rebuild your project and the google libraries using targetSDK no later than 19. This will make it run without crashing on Lollipop but is the less secure option and prevents you from using SDK functionality introduced in later versions (for Android 5).
`
哈哈，我知道你们是没有耐心把上述的rootcause全部看完的，肯定急切的想知道，我们只要结论，How to solve this problem？
## 解决方案：
### apk的解决办法：
So Easy， add one line  code；
```java
Intent intent = new Intent("com.xxx.authorClient.BIND");
```

// This is the key line that fixed this problem for me
<font color=red>intent.setPackage(getContext().getPackageName());</font>
```java
getContext().bindService(intent, serviceConnection, Context.BIND_AUTO_CREATE);
```
But,人生总是充满了But，每一个but后面都有一个心酸的故事o(╯□╰)o；上帝的apk可是不会修改的，要改只能我们底层进行修改进行适配：于是从context进行入手了，在里面进行添加校验intent的工作；
### framework的修改方法：
[有google大神android提交一个patch](https://android.googlesource.com/platform/frameworks/base/+/fd6c7b1%5E!/)

	Fix issue #10378741: configupdater needs to be explicit when it calls startService()
	Not enough time to fix everything, so instead we'll make it a warning
	in this release and finish up turning it into a target-SDK based
	exception in the next release.
	
	Change-Id: I5aae64a1225a145f03ba4162238b53d5e401aba2
```java
diff --git a/core/java/android/app/ContextImpl.java b/core/java/android/app/ContextImpl.java
index 300424c..0ad0a99 100644
--- a/core/java/android/app/ContextImpl.java
+++ b/core/java/android/app/ContextImpl.java
@@ -1453,29 +1453,39 @@
         }
     }
 
+    private void validateServiceIntent(Intent service) {
+        if (service.getComponent() == null && service.getPackage() == null) {
+            if (true || getApplicationInfo().targetSdkVersion >= Build.VERSION_CODES.KITKAT) {
+                Log.w(TAG, "Implicit intents with startService are not safe: " + service
+                        + " " + Debug.getCallers(2, 3));
+                //IllegalArgumentException ex = new IllegalArgumentException(
+                //        "Service Intent must be explicit: " + service);
+                //Log.e(TAG, "This will become an error", ex);
+                //throw ex;
+            }
+        }
+    }
+
     @Override
     public ComponentName startService(Intent service) {
         warnIfCallingFromSystemProcess();
-        return startServiceAsUser(service, mUser);
+        return startServiceCommon(service, mUser);
     }
 
     @Override
     public boolean stopService(Intent service) {
         warnIfCallingFromSystemProcess();
-        return stopServiceAsUser(service, mUser);
+        return stopServiceCommon(service, mUser);
     }
 
     @Override
     public ComponentName startServiceAsUser(Intent service, UserHandle user) {
+        return startServiceCommon(service, user);
+    }
+
+    private ComponentName startServiceCommon(Intent service, UserHandle user) {
         try {
-            if (service.getComponent() == null && service.getPackage() == null) {
-                if (getApplicationInfo().targetSdkVersion >= Build.VERSION_CODES.KITKAT) {
-                    IllegalArgumentException ex = new IllegalArgumentException(
-                            "Service Intent must be explicit: " + service);
-                    Log.e(TAG, "This will become an error", ex);
-                    //throw ex;
-                }
-            }
+            validateServiceIntent(service);
             service.prepareToLeaveProcess();
             ComponentName cn = ActivityManagerNative.getDefault().startService(
                 mMainThread.getApplicationThread(), service,
@@ -1499,15 +1509,12 @@
 
     @Override
     public boolean stopServiceAsUser(Intent service, UserHandle user) {
+        return stopServiceCommon(service, user);
+    }
+
+    private boolean stopServiceCommon(Intent service, UserHandle user) {
         try {
-            if (service.getComponent() == null && service.getPackage() == null) {
-                if (getApplicationInfo().targetSdkVersion >= Build.VERSION_CODES.KITKAT) {
-                    IllegalArgumentException ex = new IllegalArgumentException(
-                            "Service Intent must be explicit: " + service);
-                    Log.e(TAG, "This will become an error", ex);
-                    //throw ex;
-                }
-            }
+            validateServiceIntent(service);
             service.prepareToLeaveProcess();
             int res = ActivityManagerNative.getDefault().stopService(
                 mMainThread.getApplicationThread(), service,
@@ -1526,12 +1533,17 @@
     public boolean bindService(Intent service, ServiceConnection conn,
             int flags) {
         warnIfCallingFromSystemProcess();
-        return bindServiceAsUser(service, conn, flags, Process.myUserHandle());
+        return bindServiceCommon(service, conn, flags, Process.myUserHandle());
     }
 
     /** @hide */
     @Override
     public boolean bindServiceAsUser(Intent service, ServiceConnection conn, int flags,
+            UserHandle user) {
+        return bindServiceCommon(service, conn, flags, user);
+    }
+
+    private boolean bindServiceCommon(Intent service, ServiceConnection conn, int flags,
             UserHandle user) {
         IServiceConnection sd;
         if (conn == null) {
@@ -1543,14 +1555,7 @@
         } else {
             throw new RuntimeException("Not supported in system context");
         }
-        if (service.getComponent() == null && service.getPackage() == null) {
-            if (getApplicationInfo().targetSdkVersion >= Build.VERSION_CODES.KITKAT) {
-                IllegalArgumentException ex = new IllegalArgumentException(
-                        "Service Intent must be explicit: " + service);
-                Log.e(TAG, "This will become an error", ex);
-                //throw ex;
-            }
-        }
+        validateServiceIntent(service);
         try {
             IBinder token = getActivityToken();
             if (token == null && (flags&BIND_AUTO_CREATE) == 0 && mPackageInfo != null
diff --git a/core/java/android/os/Debug.java b/core/java/android/os/Debug.java
index ea9fd06..2e3b81d 100644
--- a/core/java/android/os/Debug.java
+++ b/core/java/android/os/Debug.java
@@ -1582,6 +1582,22 @@
     }
 
     /**
+     * Return a string consisting of methods and locations at multiple call stack levels.
+     * @param depth the number of levels to return, starting with the immediate caller.
+     * @return a string describing the call stack.
+     * {@hide}
+     */
+    public static String getCallers(final int start, int depth) {
+        final StackTraceElement[] callStack = Thread.currentThread().getStackTrace();
+        StringBuffer sb = new StringBuffer();
+        depth += start;
+        for (int i = start; i < depth; i++) {
+            sb.append(getCaller(callStack, i)).append(" ");
+        }
+        return sb.toString();
+    }
+
+    /**
      * Like {@link #getCallers(int)}, but each location is append to the string
      * as a new line with <var>linePrefix</var> in front of it.
      * @param depth the number of levels to return, starting with the immediate caller.
```
## 相关链接
* [google-in-app-billing-illegalargumentexception-service-intent-must-be-explicit/](http://stackoverflow.com/questions/24480069/google-in-app-billing-illegalargumentexception-service-intent-must-be-explicit/26318757#26318757)
* [Android 5.0 (L) Service Intent must be explicit in Google analytics](http://stackoverflow.com/questions/26530565/android-5-0-l-service-intent-must-be-explicit-in-google-analytics)

