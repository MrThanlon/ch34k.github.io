---
title: 在PX4Flow上使用HC-SR04
tags: 光流
categories:
  - 航模
  - 飞控
  - 光流
abbrlink: 3102
date: 2019-07-14 22:05:58
---

参照https://blog.csdn.net/hxudhdjuf/article/details/79671892

因为上面那个文章写的很乱，代码也不怎么规范，这里记录一下，顺便给后人排坑。

在校队的时候申了一块PX4Flow，260多不带超声波，真特么贵，超声波要260多，原版的超声波型号是MB1240（这里给出[Datasheet](https://www.maxbotix.com/documents/XL-MaxSonar-EZ_Datasheet.pdf)），但不得不说效果真的很好，精度和测距范围都很棒。当然当时我是不知道的是PX4Flow如果不带超声波基本没法工作，代码的话我也看得不是很懂。淘宝上可以搜到300不到的PX4Flow，顺便校队里有一个用Pixhawk的无人机就是用的那个光流模块，当时真TM后悔。

好吧闲话不多说，先给出修改原理。HC-SR04的驱动方法相当简单，就是在Trig引脚给一个脉冲，然后等待Echo引脚电平两次变化读取时间，就是发送和接收到超声波的间隔，根据空气音速就能算出距离。所以这里的思路就是用PX4Flow上的STM32F7单片机GPIO引脚来做驱动。

连接方法如下，Trig接4脚（RX），Echo接2脚（PW），GND和VCC供电，分别可以接7脚和6脚，后面应该有GND和VDD的丝印。

需要修改官方的代码，这里提供[我修改之后的代码](https://github.com/MrThanlon/Flow)，具体改了哪可以看commit log，这里就不在描述了，修改的部分是超声波的驱动`src/modules/flow/sonar.c`，以及在`src/modules/flow/module.mk`和`CMakeLists.txt`中加入exti和syscfg。