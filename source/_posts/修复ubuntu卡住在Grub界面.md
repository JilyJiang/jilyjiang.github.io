layout: post
title: 修复ubuntu14.04启动卡住在Grub界面
comment: true
tags: [技术, Ubuntu,Grub,Linux]
date: 2016-07-17 16:07:27
updated: 2016-07-17 16:07:27
---

------
   今天本来是起来本来是准备写一篇Android的指令及调试技巧的文章；Ubuntu系统后，弹出一个提示框“是否需要report一个error给ubutu……”，平时遇到这种情况一般直接close；心血来潮，要不修正一下试试看！于是乎迅速的敲完如下指令：

```bash

$ sudo apt-get install -f
$ sudo apt-get update
$ sudo apt-get upgrate
```
上述三个指令，大家肯定都耳熟能详的了，但各自的区别是什么呢？
>Q：
What is the difference between `apt-get update and upgrade`?
Which should I run first?
<!-- more -->
很多人肯定有相关的疑问，下面简单解释一下：
>A:
You should first run update, then upgrade. Neither of them automatically runs the other.

>`apt-get update` **updates the list **of available packages and their versions, but it d**oes not install or upgrade any packages.**
`apt-get upgrade` **actually installs **newer versions of the **packages** you have. After updating the lists, the package manager knows about available updates for the software you have installed. This is why you first want to update.

### 好了，扯了一点别的，现在进入正题：
当进入upgrade运行到一半的时候，<font color=red size=5>Oh，My God!系统引导区坏掉了 </font>系统直接条Grub界面里面去了，给我一个Grub的界面

```bash 
     grub>
     grub>
```
心理暗暗祈祷不要救不回来，重装系统就麻烦了，里面的开发工具都是折腾好久的！
于是乎敲入如下命令：
```bash
	grub> ls
	grub> Secure boot forbids loading module from	(path)/boot/grub/gettext.mod
```
oh,既然是安全启动禁止了，那我就在bios里面把Secure Boot禁止了，然后再启动系统. 此时界面再次回到了
```bash
	grub>
	grub>
```
1.  先使用ls命令，找到Ubuntu的安装在哪个分区：
```bash
     grub> ls
```
     会罗列所有的磁盘分区信息，比方说：

     (hd0,1),(hd0,5),(hd0,3),(hd0,2)

2. 然后依次调用如下命令： X表示各个分区号码
```bash
grub > ls (hd0,X)/boot/grub
```
    
 如果都找不到的话，需要查一下是否因为Linux版本差异，造成grub的路径不对，例如直接ls(hd0,X)/grub等等。若执行正确会显示出<font color=red>grub.cfg</font>的配置文件

3. 假设找到（hd0,5）时，显示了文件夹中的文件，则表示Linux安装在这个分区。

4. 调用如下命令：
```bash
    grub >set root=(hd0,5)

    grub >set prefix=(hd0,5)/boot/grub

    grub >insmod /boot/grub/normal.mod
```
5. 然后调用如下命令，就可以显示出丢失的grub菜单了。
```bash
    grub >normal
```
6. 不过不要高兴，如果这时重启，问题依旧存在，我们需要进入Linux中，对grub进行修复。

    进入Linux之后，在命令行执行：
```bash
$ sudo update-grub

$ sudo grub-install /dev/sda
```
    （sda是你的硬盘号码，千万不要指定分区号码，例如sda1，sda5等都不对）

7. 重启测试是否已经恢复了grub的启动菜单？ 恭喜你恢复成功！


