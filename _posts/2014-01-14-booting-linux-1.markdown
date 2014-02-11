---
layout: post
title: 系统引导系列(一)
date: 2014-01-14 16:09
---

MBR 相关的知识我以前看过多次, 但是对它的认识仍然一直都处于一种朦胧的状态, 这次一定要把 MBR 彻底搞清楚, 还有 GPT.

我在学习计算机操作系统的知识时经常会发现这么一种情况, 就是你本来想学 A, 却发现你应该先学 B, 要学 B, 最好是先把 C 搞明白. 

虽然我只是想搞清楚 MBR, 但是 MBR 周边的东西却也不得不学习, 首先看几个概念.

#Boot Sector (引导扇区)

## 定义

> A **boot sector** or **boot block** is a region of a hard disk, floppy disk, optical disc, or other data storage device that contains machine code to be loaded into random-access memory (RAM) by a computer system's built-in firmware.

上面的定义摘自维基百科, 光看定义未必能很好的理解, 所以给个例子是有必要的, 下面这段话就是维基百科给的例子: 

> On an IBM PC compatible machine, the BIOS selects a boot device, then copies the first sector from the device (which may be a MBR, VBR or any executable code), into physical memory at memory address 0x7C00. On other systems, the process may be quite different.

就是说, 在 IBM 兼容 PC 上, BIOS 会选择一个启动设备, 然后将这个设备的第一个扇区里的内容 (MBR, VBR or any executable code) 拷贝到物理内存中.

## boot sector 的种类

硬盘, 光盘等存储设备上所能见到的引导扇区一般有下面两种: 

* MBR
* VBR

其中, MBR 是已分区的存储设备上的第一个扇区, MBR 扇区里可能包含着活动分区的地址, 以及唤起活动分区的 VBR 的代码. 

而 VBR 则代表未分区存储设备上的第一个扇区, 或者是已分区存储设备上的某个分区的第一个扇区. VBR 里可能包含着加载以及唤起安装到此分区的操作系统的代码.

另外, 截取一段维基上的个人觉得很有用的:

> On IBM PC compatible machines, the BIOS is ignorant of the distinction between VBRs and MBRs, and of partitioning. The firmware simply loads and runs the first sector of the storage device. If the device is a floppy or USB flash drive, that will be a VBR. If the device is a hard disk, that will be an MBR. It is the code in the MBR which generally understands disk partitioning, and in turn, is responsible for loading and running the VBR of whichever primary partition is set to boot (the active partition). The VBR then loads a second-stage bootloader from another location on the disk.

Furthermore, whatever is stored in the first sector of a floppy diskette, USB device, hard disk or any other bootable storage device, is not required to immediately load any bootstrap code for an OS, if ever. The BIOS merely passes control to whatever exists there, as long as the sector meets the very simple qualification of having the boot record signature of 0x55, 0xAA in its last two bytes. This is why it's easy to replace the usual bootstrap code found in an MBR with more complex loaders, even large multi-functional boot managers (programs stored elsewhere on the device which can run without an operating system), allowing users a number of choices in what occurs next. With this kind of freedom, abuse often occurs in the form of boot sector viruses.

# Booting (引导)

简单来说, 引导的过程就是将 MBR 中的的内容加载到 RAM, 执行 MBR 中的引导代码, 然后进一步将操作系统加载到 RAM, 从而完成引导. 

## Second-stage boot loader (二阶段引导)

那么什么是二阶引导呢? 

比较常见的引导程序如 gnu grub, bootmgr, syslinux, ntldr, 他们本身是一种程序, 并不是操作系统, 如果在引导过程中 MBR 中的引导代码并不是直接把操作系统装载到 RAM, 而是把这种引导程序装载到 RAM, 然后由这些引导程序负责完成操作系统的加载过程, 那么这就形成了二阶引导.

那么为什么要多加一层像是 gnu grub, bootmgr 这样的引导程序呢? 直接由 MBR 中的引导代码把操作系统加载到 RAM 不就得了? 是这样的, gnu grub, bootmgr 这类引导程序是有自己的亮点的, 那就是这类程序可以配置来引导多个操作系统. 而且有些引导程序还能引导其它的引导程序, 比如 gnu grub 可以引导 bootmgr.


