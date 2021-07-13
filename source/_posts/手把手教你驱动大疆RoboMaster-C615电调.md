---
title: 手把手教你驱动大疆RoboMaster C615电调
date: 2021-05-02 20:59:35
categories:
- 硬件
tags:
- RoboMaster
---
通过查阅各种资料掌握了通过C615电调让大疆Snail电机转起来的方法，在此尽可能详细地记录过程备查。

<!-- more -->

# STM32 Cube MX部分

由于使用的是A型板，在Cube中选择STM32F427IIHx这款MCU，双击打开新工程。

## Pinout & Configuration页

在`System Core -> RCC`中，将`High Speed Clock`与`Low Speed Clock`设置为`Crystal`开头的那一项；

{% asset_img img 1.jpg RCC %}

在`Timers -> Tim1`中，将`Clock Source`设置为`Internal Clock`，将`Channel 1`设置为`PWM Generation CH1`，将下方设置中的`Prescaler`设置为`167`，`Counter Period`设置为`1999`，数值的原理请参阅文末的参考链接。

{% asset_img img 2.jpg Tim1 %}

## Clock Configuration页

将`HSE`左侧的`Input Frequency`设为`12`，右侧的单选按钮组选中`HSE`，再右侧的`/M`下拉框选择`/6`，再右侧的`System Clock Mux`单选按钮组选中`PLLCLK`，再右侧的`HCLK`设为`168`并按回车，Cube会自动调整其他值。

{% asset_img img 3.jpg 时钟设置 %}

# 代码部分

生成工程后，在main.c的User Code Begin 2后面加上：

```C++
HAL_TIM_PWM_Start(&htim1, TIM_CHANNEL_1);
__HAL_TIM_SET_COMPARE(&htim1, TIM_CHANNEL_1, 1000);
HAL_Delay(3000);
```

在while块中加入：

```C++
__HAL_TIM_SET_COMPARE(&htim1, TIM_CHANNEL_1, 1200);
```

{% asset_img img 4.jpg 代码 %}

# 硬件连接

由于前面选择的是Tim1，所以将电调连接在如图所示的位置：

{% asset_img img 5.jpg 连接 %}

重启设备，电机发出与报错声不同的声音表示开机成功。三秒之后开始转动。要停下只需拔掉电机电源（关闭A型板会导致电机急停，若固定不牢会乱飞）。

# 参考链接

[如何在stm32cubeide上使用pwm驱动snail电机_Suk1111的博客-CSDN博客](https://blog.csdn.net/Suk1111/article/details/104739316)

[【RoboMaster电控教程】课程3 PWM与舵机 - 哔哩哔哩专栏 (bilibili.com)](https://www.bilibili.com/read/cv6345910/)