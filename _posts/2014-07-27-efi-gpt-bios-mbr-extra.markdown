---
layout: post
title: "EFI, BIOS, GPT, MBR 的故事"
date: 2014-07-27 15:13
---

BIOS 是早期主板上的固件接口协议, EFI 是一种新的主板上的固件接口协议, GPT/MBR 是磁盘的分区格式, 同样, GPT 是比较先进的, MBR 以前和 BIOS 一起使用

# BIOS

## 简史

BIOS 出现的目的就像我们今天看到的: 初始化以及测试主机硬件, 从永久存储器 (磁盘) 上加载 bootloader 到内存里. BIOS 还抽象除了基本的输入输出接口, 让应用程序能够方便的和鼠标键盘通信而不用考虑底层的细节, 虽然, 现代操作系统 (驱动程序) 一般都喜欢自己考虑底层的细节, 而不去使用 BIOS 提供的接口.

以前, BIOS 固件程序是烧到主板上的非易失性 ROM 里的, 出厂一次就不能改了, 不同的主板厂家, 不同的主板型号, 主板上的芯片组, 设备都不一定相同, 所以固件程序自然也是不同的. 后来随着 flash memory 技术成熟, BIOS 都慢慢的存到 flash memory 芯片上了, 这就使得用户以后可以升级 BIOS 固件.

提前说一句, BIOS 现在已经慢慢被 UEFI 取代了, UEFI 是一种更先进的固件接口标准.

## BIOS 固件程序的启动

但是我们还是要了解 BIOS 的, 很有必要.

BIOS 和 x86 是同一时代的产物, 所以我们以 x86 处理器来说一下这个过程.

当 x86 CPU 重置后, 程序计数器会指向一个固定的地址, 那就是 ROM 中的 BIOS 固件程序的起始地址, BIOS 固件所在的 ROM 本来就是可被寻址的, 所以固件程序也就没有加载到内存这一说, 因为它本来就处在内存中.

注意, 我们上面说的是 CPU "重置" 后, 要知道除了重新给 CPU 上电, 还有另一种办法让 CPU 重置的. 这就是我们常听过的冷启动与热启动. 

如果计算机是被电源或者 reset 按钮上电的, 那叫做冷启动, BIOS 固件程序中的 "加电自检" (Power-On Self Test, POST) 过程会被执行.

而如果是按下了 Ctrl + Alt + Del, 那叫做热启动, "加电自检" 过程就不会被执行.

加电自检过程, 会检查所有主板上的硬件状态是否良好, 包括 CPU, RAM, interrupt, DMA 控制器, 芯片组, 声卡, 显卡, 光驱, 键盘鼠标等等.

## Boot device, Bootloader

BIOS 程序完成了加电自检过程, 就会去找 boot device 上的 bootloader, 将 CPU 交给 bootloader 程序. 至于如何找到 boot device 呢, 默认情况下 BIOS 会按照一个顺序依次检查主板上连接的存储设备, 看能否从这个设备启动. 具体就是将这个设备的第一扇区 (512 字节) 加载到内存 (加载的地址为 0x7C00), 执行一下看看, 如果加载失败或者执行失败, 那这个设备就不能启动, 再继续检查下一个设备. 有些 BIOS 还会检查这第一扇区的最后两个字节是不是 0x55 0xAA (也就是传说中的 boot flag), 不是的话, 也会认为这个设备不能启动.

现在, 几乎所有的 BIOS 都可以让用户设定 boot device 检查启动设备的顺序, 还能让用户在启动时临时选择这一次的 boot device, 比如是硬盘, 软盘, usb, cd, dvd. 

而上面说到的第一扇区, 也叫做 boot sector. 在 BIOS 时代, MBR 是最流行的一种 boot sector, MBR 中那小小的 512 字节记录了整块硬盘的分区结构, 有兴趣的可以查一下 MBR 的结构.

# MBR

正如上面所说, MBR 是 BIOS 时代最流行的一种 boot sector, MBR 中那小小的 512 字节记录了整块硬盘的分区结构, 有兴趣的可以查一下 MBR 的结构.

过去的产物随着时间流逝, 不足之处逐渐露了出来, 比如 MBR 中标识分区的字段只有 40 bit, 这导致 MBR 所能支持的硬盘大小不超过 2 TB; 还有 MBR 中能够记录的分区数量有限, 最多只能记录 4 个主分区, 要想多要分区, 就要拿出一个主分区当扩展分区用, 然后再在扩展分区的分区表 (也叫 VBR) 中记录逻辑分区. 这是很麻烦很笨拙的方法.

总之, 分区多了, 硬盘大了 MBR 就不适用了. 所以出现了 GPT 分区表.

# UEFI

为了保证向前兼容, 主板商在实现 UEFI 时会保留一个 legacy 模式, 那就是使用 BIOS 的模式.

UEFI 在启动的时候需要磁盘上有一个 ESP 分区.

EFI 自己就能够读取文件系统, 比如说读取 ESP 分区, 调用 boot manager, 让用户选择调用哪个 bootloader, 这是 BIOS 所不能的.

> EFI implementations are larger than BIOS implementations, so an EFI can read partition tables and filesystems, which a typical BIOS can't do. This fact enables multiple boot loaders to coexist on the hard disk, and to be accessed entirely using normal file access mechanisms. 

UEFI 规范, 在启动时, 不会去读 MBR.

## 关于 Secure Boot



# GPT

GPT 是一种教 MBR 更先进的磁盘分区格式, 它解决了 MBR 的诸多问题, 是跟随 UEFI 一起出世的. 大多数的系统已经都支持了 GPT. 

Linux 当然也是支持了, 支持 GPT 意味着, bootloader 要支持, 比如 grub 0.96 + patch, grub 2; 磁盘管理工具要支持, 比如 GNU Parted, gparted;

## 从 GPT 启动

GPT 和 UEFI 是标准的一部分, 当然能很容易的从 UEFI 启动, 这可以被称作 UEFI-GPT. GPT 刚出来之期, 从 BIOS 启动还是有困难的, 但随着时间流逝, 这种情况慢慢改善了, 这种启动可以叫做 BIOS-GPT.

另外, 再加上 UEFI 渐渐普及, 越来越多的人使用 GPT 作为他们的硬盘的分区格式, Apple 电脑一直都是 UEFI 的先行者, Microsoft 的 windows 8 系统也要求主板商必须使用 UEFI (还有一个附带的 Secure boot 机制) 才能启动 windows 8.

UEFI 标准中, 要求有一个 ESP 分区, 来存放 EFI boot loaders (不同的系统的 boot loader 都放在这里, 比如有 debian 的, ubuntu 的, windows 8 的等等), 相关的驱动程序, 以及其他文件. 如果你想要你的硬盘能从 UEFI 下启动, 就要创建这个 ESP 分区, ESP 分区的大小在 100 ~ 500 MB 之内都差不多.

BIOS-based 的电脑, 不管启动硬盘使用的是 MBR 分区格式还是 GPT 分区格式, BIOS 需要的只是硬盘上第一个扇区的 bootloader, 实际上 MBR 的前 440 字节就是这个 bootloader. DOS 和 windows 会放一个很简单的 boot loader 在 MBR 里, 其它系统, 或者第三方工具如 grub, 会在 MBR 中放复杂的 boot loader, 这种 bootloader 会继续加载第二阶段的 bootloader --- stage 2.

我们的建议是使用 GPT 分区的时候, 创建一个 2 MB 的 BIOS Boot Partition, 以及一个 OS boot partition (指的是存放系统的 bootloader, 以及内核等的分区). 

## 从 UEFI 启动

从 UEFI 启动 GPT 磁盘, 很简单, 因为 GPT 是 UEFI 标准的一部分. 但是针对一些特定操作系统和 bootloader 有一些特殊情况, 这里说下.

### EFI boot loader 和 boot managers

boot manager, 是一个程序, 使得你可以选择启动哪个系统. 而 boot loader, 是一个程序, 使得你的系统能够被加载并获得 CPU. 很多软件如 grub 同时包含了这两个功能, 但是有一些软件没有. UEFI 标准将 boot manager 的功能囊括了其中, 目的是让主板自身提供 boot manager 的功能, 操作系统只需要提供自己的 bootloader 就可以了. UEFI 规定, boot manager 与操作系统的 bootloader 放在 ESP 的 EFI/ 目录下, 其中 bootloader 是每个操作系统单独放在自己的子文件夹里, 比如 EFI/debian/grubx64.efi.

## 从 BIOS-based 启动

### Linux, GRUB

现在的 Linux 很多都使用 GRUB 作为 bootloader, grub 2, grub 0.97 patch 版都支持 GPT. 有些人系统上安装的是 grub 2, 但是他们的硬盘分区表是 MBR 格式的, 于是他们就将使用 GNU parted 等类似的工具将分区表转换成了 GPT 格式, 却发现启动不了了. 这就是典型的 no zuo no die.

那么原因是什么呢, 是因为 MBR 里的 grub stage 1 代码和 GPT 里的 grub stage 1 代码是不一样的, 只改动了分区表格式, 而没有改变 grub 的 stage 1 代码是不行的. 再说得详细一些, 是这样的:

在 MBR 分区的硬盘中, 相对于 512 字节的 MBR, grub/grub 2 比较大, 放不进 MBR 中, 所以只会将自己的一小部分放进 MBR, 这一小部分会找到 grub 的另一部分, 这就形成了二阶引导, MBR 中的代码在 stage 1 阶段执行, 另一部分代码在 stage 2 阶段执行, 操作系统内核进入内存时, 实际上 stage 3 了.

在 MBR 分区的硬盘中, 上述的 "另一部分代码" 是紧挨着 MBR 后面放的.

在 GPT 中, 虽然有保留 protective MBR, 使得 BIOS 能够将 CPU 管理权顺利交给 grub 的 stage 1 阶段. 但是, protective MBR 后面的扇区被 GPT 做其它事情用了, 存放的不是 grub stage 2 的代码. 

所以, 虽然, 你的 grub 已经达到能够支持 GPT 的版本, 但是在安装系统的时候, 你是在 BIOS-based 下 用的 MBR 分区表安装, 所以 grub 安装进 MBR 的 stage 1 代码, 是会往紧跟在 MBR 后面的扇区内找 stage 2 代码的. 但是在 GPT 里, protective MBR 后面放的已经不是 grub stage 2 的代码了, 所以启动就回失败.

要想修复, 也不难, 首先你要准备一张你安装系统时的系统盘, 姑且叫做 System Rescue CD, SRCD, 然后:

1.  Mount your /boot partition over the SRCD's /boot directory, or copy the contents of your /boot/grub directory over the SRCD's directory.
2.  If your drives' device filenames are different under SRCD than under a normal boot of your distribution, edit /boot/grub/devices.map appropriately.
3.  Type grub-install /dev/sda or grub-install /dev/hda to re-install GRUB.
4.  Reboot. Assuming GRUB is properly configured, your system should now boot.

那么, 在 GPT 里, grub 的 stage 2 代码是放在哪里的呢? 答案是 BIOS boot partition.

BIOS boot partition 的概念是随着 GPT 的出现而出现的, 他在 GPT 中的 GUID 是 21686148-6449-6E6F-744E-656564454649, 这个号码以及 BIOS boot partition 都不是 GPT 标准里面规定的 (因为 UEFI 是希望所有的操作系统都把自己的 bootloader 放在 ESP 分区里的), 但是这是为了兼容以前的情况出现的大众接受的事实上的解决方案.

因为不是 GPT 标准里规定的, 所以很多实现了 GPT 的软件也不会在创建 GPT 分区格式的时候自动创建这个分区, 但这些软件还是提供了相关的支持. 拿 grub 来说, 在安装系统时, 我们只需要自己创建这个 BIOS boot partition (建议是 2 MB 左右), 然后打上 bios\_grub 标识, 这样 grub 将会将自己的 stage 2 代码放进这个分区, 以后启动的时候也能够自己找到.

关于 GPT 的结构, 维基上有一张图, 可以看看, 很不错: https://en.wikipedia.org/wiki/GUID\_Partition\_Table.

关于这一块内容, 可以参考: http://www.rodsbooks.com/gdisk/booting.html

# 参考

1. 讲了从 GPT 结构的磁盘启动, 强烈推荐: http://www.rodsbooks.com/gdisk/booting.html
2. 维基上关于 GPT 的条目, 那张图不错: https://en.wikipedia.org/wiki/GUID_Partition_Table
3. 维基上关于 BIOS boot partition 的条目: https://en.wikipedia.org/wiki/BIOS_Boot_partition
