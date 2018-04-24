title: Books
slug: books
date: 2015-08-03 00:14
modified: 2016-05-07

## Algo & Data Structure

+ *Data Structures and Algorithm Analysis in C, 2nd edition*

此书作者 Weiss, 导师是 Sedgewick, Sedgewick 所著的 *Algorithms* 一书与其导师 Donald Knuth 的神作 TAOCP 可谓一脉相承. 只可惜不是 C 语言描述的, 所以 Weiss 这本可谓是 C 语言患者的福音.

## C

+ *K&R C* 无需解释, 记载的是 C 语言历史上第一个标准 ("事实标准")
+ *K&R C, 2nd* , 第二版是根据 ANSI C89 标准修订的版本, 目前在网上能够找到的也是这个版本. 我们也不需要看第一版了
+ [http://port70.net/~nsz/c/c89/c89-draft.html](http://port70.net/~nsz/c/c89/c89-draft.html), 正宗 C89 标准, 我在读 K&R 时发现有个别 case K&R 没有 cover 到, 所以加上这个作为补充
+ *Expert C Programming, Deep Secrets*

## MIPS

+ *See MIPS Run, 2nd* 这本书是我学习处理器架构时看的书, 少有的将处理器架构和操作系统结合起来讲的书, 而且作者的写作思想和我的信仰完全一致, 这是一本绝世好书!

## FreeRTOS

+ *Using the FreeRTOS Real Time Kernel - Standard Edition* FreeRTOS 官方的书, 针对不同的处理器这本书都有不同的版本, 对于只是了解 FreeRTOS 而不必学习 FreeRTOS 针对不同平台的差异时, 我们看 "Standard Edition" 就可以了

## lwIP

+ *lwIP - A Minimal TCP/IP implementation* lwIP 作者的论文, 学习了解 lwIP 最好的教材也是它, 胜过市面一切书籍

## Linux

+ *Understanding the Linux Kernel, Third Edition* 讲述 Linux 内核, 但是没有涵盖网络部分
+ *Understanding Linux Network Internals* 讲述 Linux 内核网络相关的实现, 不仅仅是 IP 网络. 但是似乎没有涵盖防火墙
+ iptables
  - *[iptables tutorial by frozentux.net](https://www.frozentux.net/documents/iptables-tutorial/)* 比较详尽的介绍了 Linux 内核中网络防火墙部分
  - *Linux iptables Pocket Reference, Oreilly* iptables 小书, 上面那本较详尽, 这本则可以作为快速查阅用
+ *Linux Device Drivers, 3rd Edition* 介绍 Linux 驱动开发的经典书

## Bash

+ *Advanced Bash-Scripting Guide* 这就是那本所谓的 ABS 了, 标准的好, 大, 全, 此书在手, 无需再看任何其他 bash 书籍
+ *sed and awk Pocket Reference, 2nd edition* bash 里的两个神器 sed, awk, 看这书就够了, ABS 里也涉及到一些

## GNU Make

+ *Managing Proejct with GNU Make* 比官方文档说的更像人话. 值得一提的是, 此书的中文版翻译的不错.
+ *https://www.gnu.org/software/make/manual/make.html* 官方文档, 读起来不如上面那本书, 官方文档更全, 可做查阅之用

## X Window System & X Input Method

+ [*A Brief intro to X11 Programming*](http://math.msu.su/~vvb/2course/Borisenko/CppProjects/GWindow/xintro.html)
+ *X Power Tools, O'Reilly*
+ [*Xlib - C Language X Interface*](http://www.x.org/releases/X11R7.7/doc/libX11/libX11/libX11.html)
+ [*XIM*](http://www.x.org/releases/X11R7.7/doc/libX11/XIM/xim.html), X Input Method 协议
+ [*IMdkit*](http://xorg.freedesktop.org/archive/unsupported/lib/IMdkit/), 一个方便人们从头写 IM server 的库 (不借助 toolkit, 不借助输入法框架, 这个库遵循标准的 XIM 协议, 是比 ibus, fcitx 更 raw 的实现)

## ARM Cortex

+ *The Definitive Guide to the ARM Cortex-M3, Second Edition*, 对 ARM Cortex 架构做了简明扼要的介绍, 包括异常/中断机制, 内存系统, 也拿出了不少篇幅介绍了常用的指令. 这本书的缺点就是重复性内容比较多, 但是不影响我对它的认同

## eCos

曾经接触过一段时间 eCos, 我不喜欢这个嵌入式内核, c++ 写的, 一点也不嵌入式. 但如果想了解并精通它, 下面三本按顺序看, 就完全够了.

+ eCos 3.0 User Guide, official document
+ eCos 3.0 Reference Manual, official document
+ The eCos Component Writer’s Guide
