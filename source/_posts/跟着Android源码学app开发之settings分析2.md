layout: post
title: 跟着Android源码学app开发之settings分析（二）
comment: true
tags: [Android, Setting, 技术]
date: 2016-08-04 21:12:53
updated: 2016-08-04 21:12:53
---

------
### `android:taskAffinity`
[谷歌官网解释](https://developer.android.com/guide/topics/manifest/activity-element.html#aff)
The task that the activity has an affinity for. Activities with the same affinity conceptually belong to the same task (to the same "application" from the user's perspective). The affinity of a task is determined by the affinity of its root activity.
The affinity determines two things — the task that the activity is re-parented to (see the allowTaskReparenting attribute) and the task that will house the activity when it is launched with the FLAG_ACTIVITY_NEW_TASK flag.

By default, all activities in an application have the same affinity. 
You can set this attribute to group them differently, and even place activities defined in different applications within the same task. 
To specify that the activity does not have an affinity for any task, set it to an empty string.

If this attribute is not set, the activity inherits the affinity set for the application (see the `<application> `element's taskAffinity attribute). The name of the default affinity for an application is the package name set by the `<manifest>` element.

看完是否有一头雾水的状况，直接感觉懵逼了；于是参考stackoverflow的链接进一步分析解释：[What is Android Task Affinity used for?](http://stackoverflow.com/questions/17872989/android-task-affinity-explanation)
<!-- more -->
* 每个Activity都有taskAffinity属性，这个属性指出了它希望进入的Task；
* 如果一个Activity没有显式的指明该Activity的taskAffinity，那么它的这个属性就等于Application指明的taskAffinity；
* 如果Application也没有指明，那么该taskAffinity的值就等于包名。而Task也有自己的affinity属性，它的值等于它的根Activity的taskAffinity的值。

一开始，创建的Activity都会在创建它的Task中，并且大部分都在这里度过了它的整个生命。然而有一些情况，创建的Activity会被分配其它的Task中去，有的甚至，本来在一个Task中，之后出现了转移。我们首先分析一下android文档给我们介绍的两种情况。

#### **第一种情况**
如果该Activity的allowTaskReparenting设置为true，它进入后台，当一个和它有相同affinity的Task进入前台时，它会重新宿主，进入到该前台的task中。

我们验证一下这种情况:
* Application Activity taskAffinity allowTaskReparenting
* application1 Activity1 com.winuxxan.affinity true
* application2 Activity2 com.winuxxan.affinity false


我们创建两个工程，application1和application2，分别含有Activity1和Activity2，它们的taskAffinity相同，Activity1的allowTaskReparenting为true。

首先，我们启动application1,加载Activity1，然后按Home键，使该task（假设为task1）进入后台。然后启动application2，默认加载Activity2。

我们看到了什么现象？没错，本来应该是显示Activity2，但是我们却看到了Activity1。实际上Activity2也被加载了，只是Activity1重新宿主，所以看到了Activity1。

第二种情况。如果加载某个Activity的intent，Flag被设置成FLAG_ACTIVITY_NEW_TASK时，它会首先检查是否存在与自己taskAffinity相同的Task，如果存在，那么它会直接宿主到该Task中，如果不存在则重新创建Task。

我们来做一个测试。

我们首先写一个应用，它有两个Activity（Activity1和Activity2），AndroidManifest.xml如下：
```xml
<application android:icon="@drawable/icon" android:label="@string/app_name">
      <activity android:name=".Activity1"
             android:taskAffinity="com.winuxxan.task"
             android:label="@string/app_name">
      </activity>
      <activity android:name=".Activity2">
             <intent-filter>
                   <action android:name="android.intent.action.MAIN" />
                   <category android:name="android.intent.category.LAUNCHER" />
             </intent-filter>
      </activity>
</application>
```
Activity2的代码如下：
```java
public class Activity2 extends Activity {
     private static final String TAG = "Activity2";
     @Override
     protected void onCreate(Bundle savedInstanceState) {
          super.onCreate(savedInstanceState);
          setContentView(R.layout.main2);
     }
 
     @Override
     public boolean onTouchEvent(MotionEvent event) {
          Intent intent = new Intent(this, Activity1.class);
          intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
          startActivity(intent);
          return super.onTouchEvent(event);
     }
}
```
然后，我们再写一个应用MyActivity，它包含一个Activity（MyActivity），AndroidManifest.xml如下：
```xml
<application
       android:icon="@drawable/icon"
       android:label="@string/app_name">
       <activity android:name=".MyActivity"
             android:taskAffinity="com.winuxxan.task"
             android:label="@string/app_name">
             <intent-filter>
                   <action android:name="android.intent.action.MAIN"/>
                   <category android:name="android.intent.category.LAUNCHER"/>
             </intent-filter>
        </activity>
</application>
```
我们首先启动MyActivity，然后按Home键，返回到桌面，然后打开Activity2，点击Activity2，进入Activity1。然后按返回键。

我们发现，我们进入Activity的顺序为Activity2->Activity1，而返回时顺序为Activity1->MyActivity。这就说明了一个问题，Activity1在启动时，重新宿主到了MyActivity所在的Task中去了。 （因为  android:taskAffinity=”com.winuxxan.task” ）

以上是验证了文档中提出的两种TaskAffinity的用法。

下面就是见证奇迹的时刻，同志们，不要眨眼！

我们现在将上一段时间文章中的launchMode和本文讲的taskAffinity结合起来。

首先是singleTask加载模式与taskAffinity的结合。

我们还是用上一段时间文章的singleTask的代码，这里就不在列出来了，请读者自己查阅上一文。唯一不同的就是，我们为MyActivity和Activity1设置成相同的taskAffinity，重新执行上文的测试。

我们发现测试结果令我们惊讶：从同一应用程序启动singleTask和不同应用程序启动的结果完全与上文讲的相反！

我们经过思考，就可以把从同一应用程序执行和从不同应用程序执行另种方式同一起来，得到一个结论：

当一个应用程序加载一个singleTask模式的Activity时，首先该Activity会检查是否存在与它的taskAffinity相同的Task。

1、如果存在，那么检查是否实例化，如果已经实例化，那么销毁在该Activity以上的Activity并调用onNewIntent。如果没有实例化，那么该Activity实例化并入栈。

2、如果不存在，那么就重新创建Task，并入栈。

用一个流程来表示：

然后我们来检测singleInstance模式融入taskAffinity时的情况，我们也是用上文中测试singleInstance的例子，在此不列出，读者翻阅前文查阅。唯一不同的是，我们将MyActivity和Activity2设置成相同的taskAffinity。

我们发现测试结果也有一定的出入，就是，当从singleInstance中启动Activity时，并没用重新创建一个Task，而是进入了和它具有相同affinity的MyActivity所在的Task。

于是，我们也能得到以下结论：

1、当一个应用程序加载一个singleInstance模式的Activity时，如果该Activity没有被实例化，那么就重新创建一个Task，并入栈，如果已经被实例化，那么就调用该Activity的onNewIntent；

2、singleInstance的Activity所在的Task不允许存在其他Activity，任何从该Activity加载的其它Activity（假设为Activity2）都会被放入其它的Task中，如果存在与Activity2相同affinity的Task，则在该Task内创建Activity2。如果不存在，则重新生成新的Task并入栈。