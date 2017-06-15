---
layout: post
title:  "初学PX4之光流智能相机"
categories: "drons_lifes"
author: nephen
tags: 工作生活
donate: true
comments: true
update: 2017-06-16 07:43:37 Utk
---
>`通知`：**如果你对本站无人机文章不熟悉，建议查看[无人机学习概览](/arrange/drones)！！！**

PX4Flow 是一款智能光学流动传感器。传感器拥有原生 752×480 像素分辨率，计算光学流的过程中采用了4倍分级和剪裁算法，计算速度达到250Hz（白天，室外），具备非常高的感光度。与其他滑鼠传感器不同，它可以以120Hz（黑暗，室内）的计算速度在室内或者室外暗光环境下工作，而无需照明LED。你也可以对它重新编程，用于执行其他基础的，高效率的低等级机器视觉任务。 

<!--more-->
# 编译
从`https://github.com/PX4/Flow`克隆源代码。

```sh
~ $ git clone git@github.com:PX4/Flow
~ $ cd Flow
# => 第一次必须使用make archives
~ $ make archives
~ $ make
# => 通过PX4 bootloader下载，默认target 为px4flow-v1_default，先运行命令，再连接电路板
~ $ make <target> upload-usb
```
然后连接光流传感器，如果出现一下信息就表示成功了。

```sh
Found board 6,0 bootloader rev 3 on /dev/ttyACM1
erase...
program...
verify...
done, rebooting.
```
用户必须在plugdev组。

```sh
~ $ sudo usermod -a -G plugdev $USER
```

`注意`：你也可以通过make help查看px4flow tatgets。

<br>
# 期刊翻译
[px4Flow](http://ieeexplore.ieee.org/xpl/login.jsp?tp=&arnumber=6630805&url=http%3A%2F%2Fieeexplore.ieee.org%2Fxpls%2Fabs_all.jsp%3Farnumber%3D6630805)/[flow.doc](/assets/flow.doc)

<hr>
相关链接：[官网](https://pixhawk.org/modules/px4flow)/[中文官网](http://www.pixhawk.com/zh/modules/px4flow)/[开发手册](https://pixhawk.org/dev/px4flow)/[PX4Flow源码](https://pixhawk.org/dev/px4flow)/[README](https://github.com/PX4/Flow)
