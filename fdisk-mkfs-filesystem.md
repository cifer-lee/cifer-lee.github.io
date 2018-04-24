title: 记一次关于 fdisk, mkfs, filesystem 的心得
slug: fdisk-mkfs-filesystem
date: 2015-02-13 15:26:51
tags: fdisk, mkfs, filesystem

今天碰到一个奇怪的问题, 当然, 解决了之后也就不觉得奇怪了, 在这里记录下来.

我的 U 盘上有一个分区, /dev/sdd1, 这个分区本来上面的文件系统是 FAT32, 在 fdisk -l 的输出中可以看到, 最后的 System 字段是 HPFS/NTFS/exFAT. 然后我使用 mkfs 将其分区格式化为 ext4 格式: mkfs -v -t ext4 /dev/sdd1. 然而再次运行 fdisk -l, 发现最后的 System 字段仍然是 HPFS/NTFS/exFAT, 并不是预期的 Linux. 为什么呢?

经过我的调查, 底部的三个链接阐述了答案, 这里概括一下:

首先, 我这块 U 盘是使用的 msdos 也就是 MBR 分区表, MBR 分区表中每一个表项都有一个 partition type 字段, fdisk -l 输出的最后的 System 字段就是对应着分区表项里的 partition type 字段. 这个字段表示的是这个分区的类型, 但是不一定是这个分区里装着的文件系统的类型. 有些应用程序会根据这个字段来判断分区里的文件系统的类型, 但是 Linux 下的大多数程序都不使用这个字段来判断分区里的文件系统类型. 在使用 mkfs 创建文件系统时, mkfs 更是不会去碰 MBR 分区表表项里的 partition type 字段, 所以使用 mkfs 创建了分区之后, fdisk -l 输出的依然是以前的类型.

这样的一个分区 (partition type 字段与文件系统不一致), 在 Linux 系统下工作一般是没什么问题的, 但不排除有一些应用程序, 会检查 partition type 字段来判断里面的文件系统, 这样的话, 最好还是手动改一下 partition type 字段比较好.

fdisk 工具的 t 命令就是可以修改 partition type 的, 具体有哪些 type 呢, 可以用 l 命令看一下. ext2, ext3, ext4, ReiserFS, XFS, 等等都对应着 Linux 分区类型, 类型码是 83.

== 参考 ==

http://superuser.com/questions/643765/creating-ext4-partition-from-console
http://unix.stackexchange.com/questions/18510/why-do-we-need-to-specify-partition-type-in-fdisk-and-later-again-in-mkfs
http://unix.stackexchange.com/questions/114485/fdisk-l-shows-ext3-file-system-as-hpfs-ntfs
