---
layout: post
title:  "初学PX4之mixer"
categories: "drons_lifes"
author: nephen
tags: 工作生活
donate: true
comments: true
update: 2017-06-16 07:43:37 Utk
---
>`通知`：**如果你对本站无人机文章不熟悉，建议查看[无人机学习概览](/arrange/drones)！！！**

# PX4 mixer定义
## 术语

Mixer是一组独立的标量/映射器，它从控制输入读入，写入执行器输出。一个模块，结合一组根据预先定义的规则和参数的输入，以产生一组输出。    

Scaler是一个数学模块，能根据参数调整一个单独的输入从而参数一个对于的输出。scale工作流如下：

```c
if (input < 0)
    output = (input * NEGATIVE_SCALE) + OFFSET
else
    output = (input * POSITIVE_SCALE) + OFFSET

if (output < LOWER_LIMIT)
    output = LOWER_LIMIT
if (output > UPPER_LIMIT)
    output = UPPER_LIMIT
```

Input对于roll,yaw和pitch的输入范围在-1.0(-10000)到1.0(10000)，这样可用于mixer的模块。

<!--more-->
<img src="/images/mixer.png" style="max-width:100%;"/>

## [语法](https://github.com/PX4/Firmware/blob/master/ROMFS/px4fmu_common/mixers/README.md)

Mixer为文本文件；每行一个大写字母和冒号开头作为标准，其余行将忽略。这意味着解释性文字可以自由的定义。    

每个文件可以定义多个mixer；mixer的执行器分配是特定于阅读mixer定义的设备，由一个mixer产生的执行器输出的数量是特定的。   

mixer行开始格式：

	<tag>: <mixer arguments>

标签选择了mixer类型;“M”一个简单的mixer,“R”为multirotor mixer,等等。

## 空 Mixer

空mixer不需要控制，生成一个单独的执行器输出，它的值总是0。通常，为了达到一个特定的驱动器输出模式，空mixer在mixers集合中被用来做一个占位符。

格式如下：

	Z:

## 简单mixer

一个简单的混合器结合零个或多个控制输入到一个驱动器输出。按比例输入，应用到输出定标器之前计算结果。   

<!--more-->
例如：

	M: <control count>
	O: <-ve scale> <+ve scale> <offset> <lower limit> <upper limit>

	 input      input     input
	   |          |         |
	   v          v         v
	 scale      scale     scale
	   |          |         |
	   |          v         |
	   +-------> mix <------+
	              |
	            scale
	              |
	              v
	            output

简单mixer参数为 

- 输入集
- 输入标量参数
- 输出标量参数   

输入输出都是线性运算。

如果是0，实际输出和也是0，mixer将输出一个受限于and的固定值。   

第二行定义了输出标量与如上所述标量参数。同时计算以浮点数形式进行，定义文件中存储的值是扩大了10000倍； i.e. an offset of -0.5 is encoded as -5000.   

接下来是描述控制输入和它们比例的条目。

	S: <group> <index> <-ve scale> <+ve scale> <offset> <lower limit> <upper limit>

这个值定义了哪个定标器将读取的控制组，和一个偏移量。这些值是特定于阅读mixer定义的设备。   

当用于混合车辆时，mixer group 0是车辆姿态控制组，index 0~3依次为roll, pitch, yaw 和 thrust。      

Control Group #0 (Flight Control)

- 0: roll (-1..1)
- 1: pitch (-1..1)
- 2: yaw (-1..1)
- 3: throttle (0..1 normal range, -1..1 for variable pitch / thrust reversers)
- 4: flaps (-1..1)
- 5: spoilers (-1..1)
- 6: airbrakes (-1..1)
- 7: landing gear (-1..1)

Control Group #3 (Manual Passthrough)

- 0: RC roll
- 1: RC pitch
- 2: RC yaw
- 3: RC throttle
- 4: RC mode switch
- 5: RC aux1
- 6: RC aux2
- 7: RC aux3

其余字段的配置如上所述控制标量参数。同时计算以浮点数形式进行，定义文件中存储的值是扩大了10000倍； i.e. an offset of -0.5 is encoded as -5000.      

## Multirotor Mixer

multirotor混合器结合四个控制输入(横滚、俯仰、偏航、推力)为一组驱动器输出旨在驱动马达速度控制器。   

定义为简单的一行：

	R: <geometry> <roll scale> <pitch scale> <yaw scale> <deadband>

参数解释：

- 机械结构，包括common quad (4X,4+), hex (6X,6+) and octo (8X,8+) and tri (3y)configurations
- 单独的roll, pitch 和 yaw 控制测量因子
- 电机输出死区

支持以下配置：

- 4x - quadrotor in X configuration
- 4+ - quadrotor in + configuration
- 6x - hexcopter in X configuration
- 6+ - hexcopter in + configuration
- 8x - octocopter in X configuration
- 8+ - octocopter in + configuration

每个roll, pitch, yaw测量值决定了与油门控制相关的roll, pitch, yaw的范围。同时计算以浮点数形式进行，定义文件中存储的值是扩大了10000倍； i.e. an offset of -0.5 is encoded as -5000。    

Roll, pitch和yaw输入的范围为-1.0到1.0，而油门输入的范围为0.0到1.0。对每个执行器的输出范围为-1.0到1.0。   

在执行器饱和的情况下，所有的执行器的值重新调整并限制在1.0。

## 例子

<img src="/images/mixereg.png" style="max-width:100%;"/>

<br>
参考文章：https://pixhawk.org/dev/mixing
