layout: post
title: Android 的view焦点控制
comment: true
tags: [技术, Android]
date: 2016-08-1 15:20:49
updated: 2016-08-2 15:20:49
---

-----
### Understanding Control Focus Order

因为做TV或STB的应用，不像手机或pad一样，焦点的控制主要是通过触摸事件来控制焦点；有时候需要自己通过程序来控制焦点，才能使遥控器的操作符合我们期望的要求；


### Step 1: Define a Layout with Controls
Android默认焦点控制顺序与人的读书习惯一样；<font color=red>从左到右，从上到下</font>
<!-- more -->
```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="fill_parent"
    android:layout_height="fill_parent">
    <Button
        style="@style/clockFaceNum"
        android:text="12"
        android:id="@+id/button12"
        android:layout_alignParentTop="true"
        android:layout_centerHorizontal="true">
    </Button>
    <Button
        style="@style/clockFaceNum"
        android:text="11"
        android:id="@+id/button11"
        android:layout_below="@+id/button12"
        android:layout_toLeftOf="@+id/button12">
    </Button>
    <Button
        style="@style/clockFaceNum"
        android:text="1"
        android:id="@+id/button1"
        android:layout_below="@+id/button12"
        android:layout_toRightOf="@+id/button12">
    </Button>
    <Button
        style="@style/clockFaceNum"
        android:text="10"
        android:id="@+id/button10"
        android:layout_below="@+id/button11"
        android:layout_toLeftOf="@+id/button11">
    </Button>
    <Button
        style="@style/clockFaceNum"
        android:text="2"
        android:id="@+id/button2"
        android:layout_below="@+id/button1"
        android:layout_toRightOf="@+id/button1">
    </Button>
    <Button
        style="@style/clockFaceNum"
        android:text="9"
        android:id="@+id/button9"
        android:layout_below="@+id/button10"
        android:layout_toLeftOf="@+id/button10">
    </Button>
 
    <Button
        style="@style/clockFaceNum"
        android:text="3"
        android:id="@+id/button3"
        android:layout_below="@+id/button2"
        android:layout_toRightOf="@+id/button2">
    </Button>
    <Button
        style="@style/clockFaceNum"
        android:text="8"
        android:id="@+id/button8"
        android:layout_below="@+id/button9"
        android:layout_toRightOf="@+id/button9">
    </Button>
    <Button
        style="@style/clockFaceNum"
        android:text="4"
        android:id="@+id/button4"
        android:layout_below="@+id/button3"
        android:layout_toLeftOf="@+id/button3">
    </Button>
    <Button
        style="@style/clockFaceNum"
        android:text="7"
        android:id="@+id/button7"
        android:layout_below="@+id/button8"
        android:layout_toRightOf="@+id/button8">
    </Button>
    <Button
        style="@style/clockFaceNum"
        android:text="5"
        android:id="@+id/button5"
        android:layout_below="@+id/button4"
        android:layout_toLeftOf="@+id/button4">
    </Button>
    <Button
        style="@style/clockFaceNum"
        android:text="6"
        android:id="@+id/button6"
        android:layout_below="@+id/button5"
        android:layout_centerHorizontal="true">
    </Button>
</RelativeLayout>
```
The style called clockFaceNum is defined in the /res/values/styles.xml file as follows:
```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <style
        name="clockFaceNum">
        <item
            name="android:layout_width">38dp</item>
        <item
            name="android:layout_height">38dp</item>
        <item
            name="android:onClick">numClicked</item>
        <item
            name="android:textSize">9sp</item>
    </style>
</resources>
```
The resulting screen looks like this:
![1](http://oa1wnpe3m.bkt.clouddn.com/Figure1.png) 
Screen showing 12 buttons
### Step 2: Review the Default Focus Order

The default “Down” path is shown here:
![2](http://oa1wnpe3m.bkt.clouddn.com/Figure2.png) 

Screen showing the default, and unintuitive d-pad focus change
### Step 3: Provide a Custom Control Focus Order
若我想让焦点从顺时针走，那该如何处理呢:
![3](http://oa1wnpe3m.bkt.clouddn.com/Figure3.png) 

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="fill_parent"
    android:layout_height="fill_parent">
    <Button
        style="@style/clockFaceNum"
        android:text="12"
        android:id="@+id/button12"
        android:layout_alignParentTop="true"
        android:layout_centerHorizontal="true"
        android:nextFocusUp="@+id/button11"
        android:nextFocusLeft="@+id/button11"
        android:nextFocusRight="@+id/button1"
        android:nextFocusDown="@+id/button1">
    </Button>
    <Button
        style="@style/clockFaceNum"
        android:text="11"
        android:id="@+id/button11"
        android:layout_below="@+id/button12"
        android:layout_toLeftOf="@+id/button12"
        android:nextFocusUp="@+id/button10"
        android:nextFocusLeft="@+id/button10"
        android:nextFocusRight="@+id/button12"
        android:nextFocusDown="@+id/button12">
    </Button>
    <Button
        style="@style/clockFaceNum"
        android:text="1"
        android:id="@+id/button1"
        android:layout_below="@+id/button12"
        android:layout_toRightOf="@+id/button12"
        android:nextFocusUp="@+id/button12"
        android:nextFocusLeft="@+id/button12"
        android:nextFocusRight="@+id/button2"
        android:nextFocusDown="@+id/button2">
    </Button>
    <Button
        style="@style/clockFaceNum"
        android:text="10"
        android:id="@+id/button10"
        android:layout_below="@+id/button11"
        android:layout_toLeftOf="@+id/button11"
        android:nextFocusUp="@+id/button9"
        android:nextFocusLeft="@+id/button9"
        android:nextFocusRight="@+id/button11"
        android:nextFocusDown="@+id/button11">
    </Button>
    <Button
        style="@style/clockFaceNum"
        android:text="2"
        android:id="@+id/button2"
        android:layout_below="@+id/button1"
        android:layout_toRightOf="@+id/button1"
        android:nextFocusUp="@+id/button1"
        android:nextFocusLeft="@+id/button1"
        android:nextFocusRight="@+id/button3"
        android:nextFocusDown="@+id/button3">
    </Button>
    <Button
        style="@style/clockFaceNum"
        android:text="9"
        android:id="@+id/button9"
        android:layout_below="@+id/button10"
        android:layout_toLeftOf="@+id/button10"
        android:nextFocusUp="@+id/button8"
        android:nextFocusLeft="@+id/button8"
        android:nextFocusRight="@+id/button10"
        android:nextFocusDown="@+id/button10">
    </Button>
 
    <Button
        style="@style/clockFaceNum"
        android:text="3"
        android:id="@+id/button3"
        android:layout_below="@+id/button2"
        android:layout_toRightOf="@+id/button2"
        android:nextFocusUp="@+id/button2"
        android:nextFocusLeft="@+id/button2"
        android:nextFocusRight="@+id/button4"
        android:nextFocusDown="@+id/button4">
    </Button>
    <Button
        style="@style/clockFaceNum"
        android:text="8"
        android:id="@+id/button8"
        android:layout_below="@+id/button9"
        android:layout_toRightOf="@+id/button9"
        android:nextFocusUp="@+id/button7"
        android:nextFocusLeft="@+id/button7"
        android:nextFocusRight="@+id/button9"
        android:nextFocusDown="@+id/button9">
    </Button>
    <Button
        style="@style/clockFaceNum"
        android:text="4"
        android:id="@+id/button4"
        android:layout_below="@+id/button3"
        android:layout_toLeftOf="@+id/button3"
        android:nextFocusUp="@+id/button3"
        android:nextFocusLeft="@+id/button3"
        android:nextFocusRight="@+id/button5"
        android:nextFocusDown="@+id/button5">
    </Button>
    <Button
        style="@style/clockFaceNum"
        android:text="7"
        android:id="@+id/button7"
        android:layout_below="@+id/button8"
        android:layout_toRightOf="@+id/button8"
        android:nextFocusUp="@+id/button6"
        android:nextFocusLeft="@+id/button6"
        android:nextFocusRight="@+id/button8"
        android:nextFocusDown="@+id/button8">
    </Button>
    <Button
        style="@style/clockFaceNum"
        android:text="5"
        android:id="@+id/button5"
        android:layout_below="@+id/button4"
        android:layout_toLeftOf="@+id/button4"
        android:nextFocusUp="@+id/button4"
        android:nextFocusLeft="@+id/button4"
        android:nextFocusRight="@+id/button6"
        android:nextFocusDown="@+id/button6">
    </Button>
    <Button
        style="@style/clockFaceNum"
        android:text="6"
        android:id="@+id/button6"
        android:layout_below="@+id/button5"
        android:layout_centerHorizontal="true"
        android:nextFocusUp="@+id/button5"
        android:nextFocusLeft="@+id/button5"
        android:nextFocusRight="@+id/button7"
        android:nextFocusDown="@+id/button7">
    </Button>
</RelativeLayout>
```
Therefore, our new “Down” path, starting at the 12 Button, looks like this now:

![4](http://oa1wnpe3m.bkt.clouddn.com/Figure4.png) 

### Step 4: Setting Initial Control Focus

若一开始就想设置某个view指定焦点, like this:
```xml
<Button
    style="@style/clockFaceNum"
    android:text="12"
    android:id="@+id/button12"
    android:layout_alignParentTop="true"
    android:layout_centerHorizontal="true"
    android:nextFocusUp="@+id/button11"
    android:nextFocusLeft="@+id/button11"
    android:nextFocusRight="@+id/button1"
    android:nextFocusDown="@+id/button1">
    <requestFocus />
</Button>
```
用java  code  method called **requestFocus().**
