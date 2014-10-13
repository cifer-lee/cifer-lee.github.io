---
layout: post
title: AVR 单片机的编程方式
date: 2014-08-16 13:07
---

AVR 单片机支持很多编程方式, 如下:

# ISP

大名鼎鼎的 In-system programming, 这种方法是通过 SPI 总线, 是 AVR 单片机最常用的编程方式.

AVR 的 ISP 编程方式提供的是 6 针或者 10 针的编程接口, 大多数的 AVR 单片机都支持这种编程方式, 因此都提供 6 针或 10 针的接口.

AVRISP, AVRISPv2, USBtiny, USBasp, AVRISP mkII 等编程器都是使用这种方式来给 AVR 单片机编程的.

# PDI

Program and Debug Interface, PDI 是 Atmel 在其 XMEGA 系列单片机上特有的编程接口. PDI 最牛逼的地方在于只使用 2 针的引脚, 一个是 RESET/CLOCK 引脚 (PDI\_CLK), 一个是数据引脚 (PDI\_DATA), 就能完成对片内 flash, eeprom, fuses, lock-bits 等所有组件的编程. 这是因为 XMEGA 系列单片机内部都有一个强大的 PDI 接口控制器.

# High voltage serial

High-voltage serial programming (HVSP).

# High voltage parallel

# Bootloader

大多数的 AVR 芯片都能够保留一个 bootloader 的空间, 一般在 256 B ~ 4 KB, 这里也可以存放 re-programming 代码. 在单片机复位后 (包括上电和重启), bootloader 会首先运行, 并根据其他的因素决定是执行 main 程序还是执行烧写动作.

一般来说, 单片机上所有的编程接口, bootloader 都支持, 也就是数, 不管你通过什么接口对单片机编程, 可能都会经过 bootloader 的接手. 

# 最后

要给单片机编程的话, 免不了要有个上位机软件, 这个上位机软件结合驱动程序会识别出它所支持的编程器 (比如有些编程器会通过串口和上位机通信, 创建 /dev/ttyUSB0 或 /dev/ttyACM0, 上位机软件应该要知道这些细节), 要负责和编程器通信, 给编程器下达指令, 提供镜像等.

avrdude 这个软件, 支持上面所介绍的四种比编程方式下的所有编程器.

# 参考

1.  非常有价值: http://en.wikipedia.org/wiki/Atmel_AVR#Programming_interfaces
