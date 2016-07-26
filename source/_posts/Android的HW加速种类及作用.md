layout: post
title: Android的HW加速种类及作用？
comment: true
tags: [Android, HW加速, 技术]
date: 2016-07-18 09:11:19
updated: 2016-07-18 09:11:19
---

------
HW加速的这个内容，其实[谷歌的官方网站](https://developer.android.com/guide/topics/graphics/hardware-accel.html)上已经描述的很清楚了，可能由于天朝的网络原因及英文的缘故，导致一些人对此理解存在一定的偏差；
下面根据google的官方解释+实际开发经验简单做一个介绍：

### 控制硬件加速的方式有如下几种：
**1. Application**
**2. Activity**
**3. Window**
**4. View**
<!-- more -->
### Application level
```xml
<application android:hardwareAccelerated="true" ...>
```
###  Activity level
```xml
<application android:hardwareAccelerated="true">
    <activity ... />
    <activity android:hardwareAccelerated="false" />
</application>
```
### Window level
从window层面可以控制就意味着从framework也可以控制哦，WindowManager、WindowManagerService里面针对性控制（有些IPTV的客户apk不允许修改，又要打开HW加速提升性能，这时就要考虑这个方向处理）
```java
getWindow().setFlags(
    WindowManager.LayoutParams.FLAG_HARDWARE_ACCELERATED,
    WindowManager.LayoutParams.FLAG_HARDWARE_ACCELERATED);
```
> Note: You currently<font color=red> cannot disable </font>hardware acceleration at the window level.

### View level
You can disable hardware acceleration for an individual view at runtime with the following code:

```java
myView.setLayerType(View.LAYER_TYPE_SOFTWARE, null);
```
>Note: <font color=red> You currently cannot enable hardware acceleration at the view level.</font> View layers have other functions besides disabling hardware acceleration. See  [View layers](https://developer.android.com/guide/topics/graphics/hardware-accel.html#layers)for more information about their uses.




