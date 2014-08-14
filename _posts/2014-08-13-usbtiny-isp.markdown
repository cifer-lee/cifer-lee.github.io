---
layout: post
title: USBtinyISP 编程器
date: 2014-08-13
---

@(嵌入式笔记)

翻译自: https://learn.adafruit.com/usbtinyisp/overview

# 介绍

USBtinyISP 是一个开源的 USB AVR 编程器, 它引出单片机 SPI 总线给上位机. USBtinyISP 被 avrdude 和 AVRStudio 所支持.

# FAQ

原文的 FAQ 非常有价值: https://learn.adafruit.com/usbtinyisp/faq

捡几条翻译:

*   USBtinyISP 编程器能工作在 linux 系统吗?

    是的
    
*   为什么我找不到串口设备呢? 怎么没有 /dev/ttyXX 设备文件呢?

    是的, USBtiny 本来就不是个 USB-Serial 设备, 它有自己的 USB 协议, avrdude 能够理解它. 你自然是找不到相应的串行设备文件的.
    
*   使用 USBtinyISP 我能不能既用它给单片机烧写程序, 又用它来当串口通信设备给单片机发送串行信息? 就像 Arduino 那样.

    就如上个回答一样, USBtiny 不是 USB-Serial 设备, 不能创建串行设备文件, 也就不能提供那样的服务. USBtiny 是纯粹的编程器, 它连接的是单片机上的 ISP 总线, 而不是单片机的串行口, 所以 USBtiny 和单片机的通信本就不是串行通信, 怎能给上位机提供和单片机串行通信的服务呢?
    
    Arduino 能做到是因为, 实际上它不是严格意义上的编程器, Arduino 实际上是包含了已经烧写好 bootloader 程序的 AVR 单片机的一块 target board. Arduino 为 AVR 定制的 bootloader 带有和 AVR 单片机的串口引脚通信的功能, 所以在使用 Arduino 时上位机上有串行通讯设备 (/dev/ttyUSB0, /dev/ttyACM0) 也可以通过串口与 Arduino/AVR 单片机 通信. 而在烧程序的时候, 实际上是 avrdude 在和 Arduino 事先写进 avr 单片机 (比如 ATtiny2313) 的 bootloader 程序在通信, 由 bootloader 程序负责将要烧写的程序写进对应的地址上, 这一过程开发者是不能自由控制程序要烧写的地址的, 完全由 bootloader 来控制.
    
*   支持 AVR 的哪些芯片呢?

    使用 ISP 总线来编程的 [1], 并且 flash 空间不超过 64K 的任何 AVR 芯片都能够使用 USBtiny 来编程.
    
    Chips such as the Atmega1280/1281 and Atmega2560/2561 have more than 64K and cannot be programmed.
    
    Chips that use TPI interface, such as Attiny4/5/9/10 cannot be programmed.
    Some very old chips such as the AT90S1200 and similar cannot be programmed
    
*   我可以用 USBtiny 来给单片机烧 bootloader 吗 (就像 Arduino 干的那样)?

    完全可以, 这是一个编程器理所应当的功能. 使用 Arduino 时实际上你被隐藏了这些细节.
    
# 如何使用

![Alt text](./tools_done_t.jpg)

## 指示灯

这段请看原文, 没有翻译的价值

## 烧写线

USBtiny 引出了两根烧写线, 一根是 6 针 ISP 线, 一根是 10 针的 ISP 线. 这两种线是 In-system 编程的标准接口线. USBtiny 没有采用 JTAG 标准编程.

## Jumper JP3 (USB power to target)

这是 USBtiny 相较于其前辈 AVRISP/AVRISPv2 的杀手级优势, USBtiny 可以给单片机供电, 这样烧程序的时候就不必给单片机接另外的电源了.

## Using it as an SPI interface

因为 USBtiny 连接/引出了 ISP 总线, 所以你可以把 USBtiny 用作一个 SPI 总线的接口设备. 你的上位机连接了 USBtiny, 也就连接了单片机的 SPI 总线.

# 驱动程序

没有翻译的价值, 看原文.

简要提一点, 对于 xp/vista/win7/win8, 都有提供驱动, 安装后 USBtiny 被识别为 "USBtinyISP AVR Programmer" 设备.

Linux 不需要为 USBtiny 额外安装驱动.

# AVRDUDE

翻译价值很大, 但是暂时没时间了, 以后再翻译..

暂时可以先看原文.

# AVRStudio

看原文, 没有翻译价值

# 帮助

最后意义篇帮助页面, 可以做参考.

https://learn.adafruit.com/usbtinyisp/help
    
# 脚注

1.  使用 ISP 总线来编程的单片机, 指的是 flash 与 CPU 之间是通过 ISP 总线相连的单片机, 绝大多数情况是这样的.

# 参考

1.  原文: https://learn.adafruit.com/usbtinyisp/overview
