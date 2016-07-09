layout: post
title: Markdown基本语法入门
comment: true
tags: [技术, Markdown]
date: 2016-07-9 22:24:50
---

------
目前markdown正火，简单学习和总结一些基本指令，便于后续写blog查阅！

 
## 标题
	语法：
	# 一级标题
	## 二级标题
	### 三级标题
	#### 四级标题
	##### 五级标题
	###### 六级标题
<!-- more -->
## 无序列表
	语法：
	* 列表1
	* 列表2
* 无序列表
- 无序列表
* 无序列表

## 有序列表
	语法：
	1. 列表1
	2. 列表2
	3. 列表3
1. 张三
2. 李四
3. 王五

## 引用
	语法：
	> 引用内容
	> > 多级引用
	> 
	> * 无序列表
	> 1. 有序列表
	> [链接](地址)
> 锄禾日当午，汗滴禾下土，谁知盘中餐，粒粒皆辛苦。
春眠不觉晓，处处闻啼鸟，夜来风雨声，花落知多少。

>> 日照香炉生紫烟
遥看瀑布挂前川

>>> 白日依山尽
黄河入海流

>* 无序列表
>* 无序列表
> [Macdown Website]()

> 1.有序列表
> 2.有序列表


## 链接
	语法：
	[链接标题](链接地址)
[Ubuntu下优秀的Markdown编辑器](https://remarkableapp.github.io/)


## 邮箱
	语法：
	<邮箱地址>
<jily.jiang@hotmail.com>


## 图片链接
	语法：
	![可选Alt Text]()
![耳机](http://yiyacn.com/uploads/tubiaoimg/icon/Mario.png) 


## 字体强调
## 粗体
	语法：
	**粗体内容**
	快捷键 ctrl + B
**这里是本文的重点**

## 斜体
	语法：
	*斜体内容*
	快捷键 ctrl + I
*这是本文的末尾了*

## 删除线
	语法：
	~~删除线内容~~
~~本行需要删除划掉~~

## 嵌入代码区块
	语法：
	嵌入多行代码
	前后各三个`，中间是代码内容
	嵌入单行代码
	前后各一个`， 中间是代码内容
	//单行注释
	/*多行注释*/
***

```c
#include  <UIKit/UIKit.h>
#include  "AppDelegate.h"

int main(int argc, char * argv[]) {    	
    return UIApplicationMain(argc, argv, nil, NSStringFromClass([AppDelegate class]));
 }
```

```java
static ContextImpl getImpl(Context context) {
        Context nextContext;
        while ((context instanceof ContextWrapper) && (nextContext=((ContextWrapper)context).getBaseContext()) != null) {
            context = nextContext;
        }
        return (ContextImpl)context;
}
```

## 分割线
	语法：
	***
***
上面是一条分割线，看到了吗？

更多内容请参见[Markdown语法基本入门指南](http://wowubuntu.com/markdown/basic.html)