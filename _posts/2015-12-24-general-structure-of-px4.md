---
layout: post
title:  "初学PX4之大体构架"
categories: "drons_lifes"
author: Lever
tags: 工作生活
donate: true
comments: true
editpage: true
update: 2017-06-16 07:43:37 Utk
---
# PX4源代码
PX4项目建立在这些主要软件模块:

- PX4 Flight Stack (estimation and control, cross-platform)
- PX4 Middleware (IPC / ORB, *nix (NuttX, Linux, MacOS, etc))
- PX4 ESC Firmware (for motor controllers)
- PX4 Bootloader (for STM32 boards)
- Operating System (NuttX or Linux/Mac OS)

项目地址：

- [PX4 Firmware source](http://github.com/PX4/Firmware)

**PX4飞行栈**

PX4飞行栈能控制多轴飞行器，航模，直升机，实验飞机和地面车辆的飞行。它由一组单独的应用程序/节点组成。

- [flight control modules](http://github.com/PX4/Firmware/tree/master/src/modules)

**PX4中间件**

PX4中间件提供了硬件接口和进程间通信。

- [drivers](http://github.com/PX4/Firmware/tree/master/src/drivers)
- [uORB](http://github.com/PX4/Firmware/tree/master/src/modules/uORB)

**PX4电调固件**

PX4电调固件控制无刷电机，可以通过UAV CAN进行交互。

<!--more-->
- [PX4 ESC source](http://github.com/PX4/px4esc)

**PX4 Bootloader**

PX4引导装载程序是用于STM32微控制器新飞行软件加载到闪存。用于Pixhawk。

- [PX4 Bootloader source](http://github.com/PX4/Bootloader)

**操作系统**

PX4飞行栈和中间件可以在微控制器Nuttx小型操作系统上执行，或者在全面POSIX系统，如Linux，Mac OS 或 BSD。

- [NuttX OS](http://github.com/PX4/NuttX)

>`Tip`:px4编译系统使用的是[cmake](http://www.cnblogs.com/coderfenghc/archive/2013/01/20/2846621.html)，关于cmake与make的区别见[CMake与Make ](http://blog.csdn.net/shaoxiaohu1/article/details/9179715)     
![cmake](http://static.oschina.net/uploads/space/2012/1102/210924_Gx9w_191887.jpg)

<br>
# 代码运行分析
>关于系统的启动过程请查看我之前文章里的[系统启动](/2015/12/RTOS-of-px4#1-3)。   

知道了系统的启动过程，那么就知道代码运行的思路基本上就是脚本[rcS](https://github.com/PX4/Firmware/blob/master/ROMFS/px4fmu_common/init.d/rcS)的写法了，如果你想深入了解NSH启动脚本的自定义，可以参考[定制NSH初始化](/2015/12/RTOS-of-NuttX#1-5-4)，所以下面就从脚本开始看起。

首先通过sercon开启串口驱动CDC/ACM，这样才能打印出下面echo出的串口信息。

设置模式为自动启动。

再设置文件路径如下

```sh
set FRC /fs/microsd/etc/rc.txt
set FCONFIG /fs/microsd/etc/config.txt
set LOG_FILE /fs/microsd/bootlog.txt
#=> 运行出错时会发出下面的声音
set TUNE_ERR ML<<CP4CP4CP4CP4CP4
```

挂载microSD卡`"mount -t vfat /dev/mmcsd0 /fs/microsd"`，如果/dev/mmcsd0不存在，还要创建文件系统`"mkfatfs /dev/mmcsd0"`，这里边调用了tone_alarm应用来根据声音判断，这个应用可以在源代码里找到，如果你使用了[Qt creator](http://www.nephen.com/2015/12/初学PX4之环境搭建/#qtcreator-ide建立工程)，你可以这样搜到它。
<img src="/images/findapp.png">
<hr>
可以看出函数的原定义是这样的，借助于IDE可以很快的查出应用程序的来龙去脉，后面碰到的应用都可以这样去查看。
<img src="/images/define.png">
<hr>
接着分析，接下来很关键的一步，检查是否存在`/fs/microsd/etc/rc.txt`，如果存在，执行它，设置模式被设置为自定义，否则为默认的自动启动。一般情况是没有的，因为etc/rc.txt文件的创建将完全禁用内置启动进程，高级用户可以这样做而已。    

然后就是自动启动的一些东西。   

>- 由`nshterm /dev/ttyACM0 &`启用/dev/ttyACM0串口；   
- 启动uorb；    
- 载入`/fs/microsd/params`里的参数（一般是由mtd将其载入ram中，然后路径变为/fs/mtd_params），param应用功能比较多，如select select/select load/select compare；   
- 对比参数RC_MAP_THROTTLE、RC_MAP_ROLL、RC_MAP_PITCH、RC_MAP_YAW并保存默认值；   
- 启动系统状态指示灯；   
- 设置这些[参数](https://pixhawk.org/start?id=dev/system_startup#configuration_variables)的默认值，一般为none；   
- 判断SYS_AUTOCONFIG，设置AUTOCNF为yes；   
- 根据板子型号设置USE_IO；    
- SYS_AUTOSTART设置，启动`/etc/init.d/rc.autostart`脚本，从nsh里可以看到，在这个脚本里调用机身配置文件；    
- 打开用户设置文件`etc/config.txt`设置参数，如果存在的话；    
- 保存参数；   
- crc校验`/etc/extras/px4io-v2.bin`是否需要更新及启动px4io打开电机是否安全；    
- 设置输出模式OUTPUT_MODE；     
- 启动航点存储；    
- 启动传感器`/etc/init.d/rc.sensors`、GPS；     
- 开始主输出TTYS1_BUSY；
- 启动commander；    
- 检查UAVCAN是否可用；
- 根据OUTPUT_MODE，启动px4io，运行`/etc/init.d/rc.io`、fmu、mkblctrl、pwm_out_sim；    
- 开启mavlink；
- 运行`/etc/init.d/rc.uavcan`；
- 运行`/etc/init.d/rc.logging`；
- 根据VEHICLE_TYPE启动设置（如四轴：`rc.interface/rc.mc_apps`），姿态估计、控制算法在`/etc/init.d/rc.interface`里，mixer导入在`/etc/init.d/rc.mc_apps`里；   
- 启动导航navigator；
>- 运行etc/extras.txt； 

自启动完后告诉MAVLink app启动完成；启动光流；     

下面就可以单独的研究应用软件了。

总个程序运行的框架如下：
<center><a href="http://dev.px4.io/concept-flight-stack.html#estimation-and-control-architecture"><img src="/images/app_run.png"></a></center>

下面给出一个大神的图：   
<img src="/images/dshen.png">

<hr>
参考文件：[PX4 Source Code](https://pixhawk.org/firmware/source_code)/[Nuttx系统启动](http://blog.chinaunix.net/uid-29786319-id-4393303.html)/[NuttX 编译系统](http://blog.csdn.net/zhumaill/article/details/24400441)
