layout: post
title: Android5.0新特性有哪些会影响到开发者？
comment: true
tags: [Android, 技术]
date: 2016-07-11 11:21:52
updated: 2016-07-11 11:21:52
---

------
# Android5.0有哪些新特性
因为公司的平台全部切到android5.1.1,除了大名鼎鼎的ART的模式，android5.0以后的新特性一直没有进行汇总，上周出差porting客户的apk发现bind service的大坑后，见我另一篇文章：[Android 5.0 (L) Service Intent must be explicit](http://jily.org/2016/07/10/Android%205.0%20%28L%29%20Service%20Intent%20must%20be%20explicit/)；后来又发现了一个坑，[armeabi-v7a与armeabi的坑]()，这个坑后续补上；

除了[谷歌官方高大上的特性说明](https://developer.android.com/about/versions/android-5.0-changes.html)外，还有如下坑需要注意：
<!-- more -->
1. Service must be explitict，从Lollipop开始，service必须显性声明，解决方案：http://blog.android-develop.com/2014/10/android-l-api-21-javalangillegalargumen.html
源代码参考
`sdk/sources/android-21/android/app/ContextImpl.java`
```java
 private void validateServiceIntent(Intent service) {
        if (service.getComponent() == null && service.getPackage() == null) {
            if (getApplicationInfo().targetSdkVersion >= Build.VERSION_CODES.LOLLIPOP) {
                IllegalArgumentException ex = new IllegalArgumentException(
                        "Service Intent must be explicit: " + service);
                throw ex;
            } else {
                Log.w(TAG, "Implicit intents with startService are not safe: " + service
                        + " " + Debug.getCallers(2, 3));
            }
        }
    }
```
2. [Material Design 按钮文字默认大写](https://code.google.com/p/android-developer-preview/issues/detail?id=487),奇怪吧，看看源代码就知道
```xml
    <style name="TextAppearance.Material.Button">
        <item name="textSize">@dimen/text_size_button_material</item>
        <item name="fontFamily">@string/font_family_button_material</item>
        <item name="textAllCaps">true</item>
        <item name="textColor">?attr/textColorPrimary</item>
    </style>
```

3.  JobScheduler 是可以通过`System serviceContext.getSystemService(Context.JOB_SCHEDULER_SERVICE)`来获得的，它不依赖于implicit service或者是explicit service，但explicit service是可以与JobScheduler交互，[参考sample](https://github.com/googlesamples/android-JobScheduler)。 在app中声明的explicit service，用电是在app头上，所以使用者看到的结果是android os 耗电不会很高，取而代之是各种app在耗电list裹。

4. Material Design Action Bar 默认是有shadow的，是因为新的elevation API, 所以你想去除它，要买在action bar style 的xml中，定义elevation 属性为0，要么
 ```java
getActionBar().setElevation(0)
```
5. FlashLight按照以前的接口是不能用了，注意是不能用了，所以一定要做版本适配，Lollipop上把Camera直接杠掉，从新弄了一套Camera2，详情可以去看Lollipop状态栏闪光灯开关的[源码](https://android.googlesource.com/platform/frameworks/base/+/android-5.0.0_r2/packages/SystemUI/src/com/android/systemui/statusbar/policy/FlashlightController.java)

6. ActivityManger中通过getRunningTask获取当前列表也是禁用了，这个是代码上注释的描述
>@deprecated As of {@link android.os.Build.VERSION_CODES#LOLLIPOP}, this method is no longer available to third party applications: the introduction of document-centric recents means it can leak person information to the caller. For backwards compatibility, it will still return a small subset of its data: at least the caller's own tasks, and possibly some other tasks such as home that are known to not be sensitive.
这就禁止后我就无法获取到当前运行的packageName了，也就没法进行一些涉及到需求的判断了。目前我还没找到很好的解决方案，涉及到系统设计的问题，可能只能从产品上去改进了

7. 终于支持Vector图片了，还是SVG标准的，再也不用担心叠加的相框失真了，请戳
[Working with Drawables](https://developer.android.com/training/material/drawables.html#VectorDrawables)
```xml
<!-- res/drawable/heart.xml -->
<vector xmlns:android="http://schemas.android.com/apk/res/android"
    <!-- intrinsic size of the drawable -->
    android:height="256dp"
    android:width="256dp"
    <!-- size of the virtual canvas -->
    android:viewportWidth="32"
    android:viewportHeight="32">

  <!-- draw a path -->
  <path android:fillColor="#8fff"
      android:pathData="M20.5,9.5
                        c-1.955,0,-3.83,1.268,-4.5,3
                        c-0.67,-1.732,-2.547,-3,-4.5,-3
                        C8.957,9.5,7,11.432,7,14
                        c0,3.53,3.793,6.257,9,11.5
                        c5.207,-5.242,9,-7.97,9,-11.5
                        C25,11.432,23.043,9.5,20.5,9.5z" />
</vector>
```
## 参考链接：
* https://www.zhihu.com/question/26225078
* https://developer.android.com/about/versions/android-5.0-changes.html
