---
title: 挂载 (mount) 深入理解
layout: post
date: 2014-05-06 20:30
---

首先引用一句 wiki 上的定义来开篇:

> Mounting takes place before a computer can use any kind of storage device (such as a hard drive, CD-ROM, or network share). The user or their operating system must make it accessible through the computer's file system. A user can only access files on mounted media.

意思是说, "挂载" 发生在计算机想要使用任何类型的存储设备 (如硬盘, CD-ROM, 网络设备) 之前. 操作系统必须将这个设备纳入自己的文件系统中去.

要注意的是, 这里的存储设备不一定必须是外部的存储设备, 也可以是你安装系统的硬盘上的分区.

# 例子先

光看上面说的还不够, 先看个例子吧, 这个例子摘自 man mount, 在 man 手册中这个例子下的一句话非常好的解释了 mount 到底是什么.

    mount -t _type_ _device_ _dir_
    
在这个例子下面有这么一句话: 

> This tells the kernel to attch the filesystem fount on _device_ (which is of type _type_) at the directory _dir_.
    
这句话非常重要, 我们一定要明白, 挂载操作, 实际上是把设备 _device_ 中的**文件系统**附加到 _dir_ 上, 然后我们就可以通过访问 _dir_ 来访问这个设备.

明白了这一点, 我们就能明白 "挂载" 的本质了, 挂载的本质就是针对某一设备, 分析出其文件系统结构, 并根据其文件系统类型调用 linux 中相应的驱动, 处理其的元数据, 将这些信息附加到 linux 的目录树上呈现出来.

明白这一点之后, 后面的 bind mount, loop mount 以及 remount 的区别就能够很清楚了.

# 挂载点

什么是挂载点呢? 还是先借用 Wiki 上的一句话:

> A mount point is a physical location in the partition used as a root filesystem.

不幸的是, Wiki 上的这句话并不准确, 这句话的意思也就是说 "挂载点就是 root 分区中的一个位置", 这句话错在 "root 分区" 上.

我们知道在安装 Linux 系统时可能会为磁盘分多个区, 最普遍的情况就是很多用户会给 /home 目录单独分一个区. 而且有一部分用户还会在 /home/username 目录下建立一个专门用来挂载各种设备的目录 (如 /home/username/mnt-point) 而不使用系统的 /mnt 目录. 那么这时候, 难道说 /home/username/mnt-point 这个目录就不是挂载点了吗? 显然它也是挂载点, 但它确并不是位于 root 分区 (即 / 分区).

国外有[一篇文章](http://www.linuxnix.com/2013/09/what-is-a-mount-point-in-linuxunix.html), 用毫不装逼的方式说出了"挂载点"的本质:

> In simple words a mount point is a directory to access your data (files and folders) which is stored in your disks.
    
所以说白了, 挂载点就是一个目录. 所以下文中当我应该说"挂载到某一挂载点"的时候我都直接说"挂载到某一目录".

# 假设备挂载 (loop mount)

## loop device

明白 loop mount 之前, 最好先清除什么是 loop device, 有耐心的话可以参见[维基百科](http://en.wikipedia.org/wiki/Loop_device)中的条目, 比较长, 没耐心的话可以直接看我下面的描述, 简洁些.

简单来说, loop device 能够提供将一个**档案**挂载到某一目录的功能. 这和 bind mount (下文会介绍) 有些类似, 但并不相同. 原始的 mount 只是为了将正常的设备挂载, bind mount 使得可以挂载目录, 而 loop device 使得可以挂载**档案**.

在 linux 中, loop device 就是指 /dev/loop0, /dev/loop1, /dev/loop2 ... 这些设备, 它们是虚假的设备(pseudo device), 不像 /dev/sda 在你的主机里物理存在. loop device 需要你在编译内核的时候将其静态编译或者编译为动态模块, 然后需要使用 `modprobe` 加载其模块(这个模块包含了 loop device 的驱动程序以及 losetup 这种提供给用户来操作 loop device 的程序), 这时其驱动程序就回创建 /dev/loop0, /dev/loop1 ... 这几个设备文件. 

## 档案

注意, 我在说档案的时候, 指的是英文中的 archive, 它和文件 file 是不同的东西, 档案 archive 是一个打包的文件集, 里面一般包含许多文件, 比如 tar, jar, iso 就是常见的档案格式.

用过 dd 的人应该知道, 这个强大的命令可以将整个磁盘或者磁盘分区克隆下来, 放到一个文件里, 一般, 这样的文件我们都以 .img 后缀为其命名并称这样的文件为镜像文件. 我所说的**档案**也包含这类情况.

## loop mount

ok, 明白了什么是 loop device, 也明白了**档案**是什么, 那么到底如何把一个档案挂载到某个目录下呢?

实际上 loop mount 采取了一个瞒天过海的方式, 它先将这个档案映射到某个 loop device 上, 像这样:

    # losetup /dev/loop0 xxxx.iso
    
通过这种方式来欺骗 `mount` 命令, 让 `mount` 命令以为 /dev/loop0 上面真的有设备. 这时运行 `mount` 就行了:

    # mount -t iso9660 /dev/loop0 /path/to/mount/point
    
这么看起来, 当你想挂载某一个档案的时候(比如某个 iso), 你首先得把这个档案和某一个 loop device 关联起来, 使用 losetup 命令. 然后使用 mount 命令将这个 loop device 设备挂载到某个目录上. 实际上不必这样, `mount` 命令自身其实就有一个能把这两步合并的功能, 那就是这样:

    # mount -t iso9660 -o loop /dev/loop0 /path/to/mount/point

最后我们再来想一想, 是不是所有的档案都可以用这种方式挂载? 显然不是的, 根据 `mount` 命令有个 -t 参数来看, 在挂载的时候是需要指定文件系统的类型的(不指定的话 `mount` 命令会自动识别), 还记得上面说的挂载的本质吗? 

    "挂载操作, 实际上是把设备 _device_ 中的**文件系统**附加到 _dir_ 上,".
    
不被识别的文件系统是不能被挂载的, 如果你没有加载 ReiserFS 模块, 那么挂载具有 ReiserFS 文件系统的设备时就会报 "unknown file system" 错误. 像上面说的 tar, jar, zip 这样的档案, 它们只是一种打包/压缩格式, 本身就不是一种文件系统格式, 当然是不能被 linux 识别的. 它们虽然可以映射到某一个 loop device, 但并不能被挂载. 

但是像 .iso 文件, 它一般包含 iso 9660 文件系统, 都知道这是一种 CD 上采用的文件系统. 还有就是你可以使用 dd, mkfs 命令来创建一个 ext2, ext3 等文件系统的档案. 这样的档案才是可以被挂载的.

loop mount 一直以来是 Unix-like 系统下很有用的特性, 能帮助你当你拿到一个 iso 文件后, 不必将其刻录到 CD/DVD 里就能查看里面的内容. windows 下直到 windows 7 才支持这一特性, 在此之前都需要借助第三方软件如 Daemon Tools 来实现虚拟光驱的功能.

# 绑定式挂载 (bind mount)

上面所说的 "挂载" 都是指让你将某个设备挂载到某一目录, 不管这个设备是真实的物理设备, 还是假的 loop 设备, 它都是设备. 而 "绑定式挂载" 能够允许你将已经的存在目录挂载到另一目录. 比如:
    
     #  mount --bind / /home/username/mnt-point

这样, 你的 mnt-point 目录下也会有 etc, opt, usr 等目录, 这一过程我们称作 "将根目录绑定到 /home/username/mnt-point 上", 所以, 你在一处改变目录下的内容的话, 在另一处也能够看到改变. 

需要注意的一点是如果根目录树下有某个目录是挂载到另一个磁盘分区的话, 那么它可能不会被绑定到新的目录下. 比如说如果 /usr 和 / 处于不同的磁盘分区(/ 在 sda1, /usr 在 sda2), 那么你可能会发现 /home/username/mnt-point/usr 是空的, 那么这时可以额外挂载一次来使得 /usr 也出现在 /home/username/mnt-point/usr:
 
     #  mount --bind /usr /home/username/mnt-point/usr

不过你也可以在一开始就执行:
     
     #  mount --rbind / /home/username/mnt-point

关于绑定式挂载, `man 2 mount` 中的描述是 "使一个文件, 或者一个目录树在另一个目录上可见". 这地方不太理解, 就我所知, 只能将目录绑定到目录, 不能将文件绑定到目录的. 我尝试过将一个普通的文件绑定到目录, 但报错了. 不知道 man 手册里这个说法是什么意思. 我只能这么理解: 目录也是文件, 所以这种说法没错吧....

# 重新挂载 (remount)

借助于**绑定式挂载**, 可以实现有趣的效果, 比如说, 你可以将 / 绑定到 /, 将 /tmp/test/ 绑定到 /tmp/test/ (运行 mount 命令就能看到效果). 不过... 这么干有个鸟用啊!! 谁这么无聊会去这么干啊!!

这就是 remount 存在的原因, 我们虽然可以通过绑定式挂载耍点小聪明, 将自己绑定到自己上, 但这与没绑定没有任何区别啊; 然而借助 remount, 我们就可以在重新挂载的时候修改挂载的参数. 

remount 最常用的情况就是将一个文件系统由只读重新挂载为读写, 或者相反. 比如:

    # mount -o remount,rw /
    
关于 remount 的详情, 可以看一下 man 手册, 这里就不多介绍了.

# supermount

"超级挂载", 这个项目的目的是让你能够免去手动 mount/umount 的过程, 达到 "插上 U 盘就开始拷文件" 以及 "拷完文件就拔掉 U 盘" 的效果.
     
# 非常好的一篇文章

这篇文章也是 参考1 中提到的文章, 由于写的太好, 为了防止其丢失, 我把它转到这里了, 下面那篇英文就是. 我之所以敢转这篇文章是因为这篇文章的版权声明停留在 2006 年, 它的主人大概是忘了更新. 我从这篇文章里摘取了几个比较重要的点作为笔记.

*   在 "挂载" 的概念中, "设备" 可以是一个分区 (如 /dev/sda9), 可以是另一块磁盘, 可以是 CDROM, 软盘, USB, 磁带等等.
*   "挂载点" 是一个目录, 而且往往是一个空目录, 但这不是必须的. 如果这个目录不是空的, 那么挂载之后, 这个目录中以前的内容会被 "隐藏" 起来变得不可访问.

> # Mounting Definition

> Mounting is the attaching of an additional filesystem to the currently accessible filesystem of a computer.

> A filesystem is a hierarchy of directories (also referred to as a directory tree) that is used to organize files on a computer or storage media (e.g., a CDROM or floppy disk). On computers running Linux or other Unix-like operating systems, the directories start with the root directory, which is the directory that contains all other directories and files on the system and which is designated by a forward slash ( / ). The currently accessible filesystem is the filesystem that can be accessed on a computer at a given time.

> In order to gain access to files on a storage device, the user must first inform the operating system where in the directory tree to mount the device. A device in a mounting context can be a partition (i.e., a logically independent section) on a hard disk drive (HDD), a CDROM, a floppy disk, a USB (universal serial bus) key drive, a tape drive, or any other external media. For example, to access the files on a CDROM, the user must inform the system to make the filesystem on the CDROM appear in some directory, typically /mnt/cdrom (which exists for this very purpose).

> The mount point is the directory (usually an empty one) in the currently accessible filesystem to which a additional filesystem is mounted. It becomes the root directory of the added directory tree, and that tree becomes accessible from the directory to which it is mounted (i.e., its mount point). Any original contents of a directory that is used as a mount point become invisible and inaccessible while the filesystem is still mounted.

> The /mnt directory exists by default on all Unix-like systems. It, or usually its subdirectories (such as /mnt/floppy and /mnt/usb), are intended specifically for use as mount points for removable media such as CDROMs, USB key drives and floppy disks.

> On some operating systems, everything is mounted automatically by default so that users are never even aware that there is any such thing as mounting. Linux and other Unix-like systems can likewise be configured so that everything is mounted by default, as a major feature of such systems is that they are highly configurable. However, they are not usually set up this way, for both safety and security reasons. Moreover, only the root user (i.e., administrative user) is generally permitted by default to mount devices and filesystems on such systems, likewise as safety and security measures.

> In the simplest case, such as on some personal computers, the entire filesystem on a computer running a Unix-like operating system resides on just a single partition, as is typical for Microsoft Windows systems. More commonly, it is spread across several partitions, possibly on different physical disks or even across a network. Thus, for example, the system may have one partition for the root directory, a second for the /usr directory, a third for the /home directory and a fourth for use as swap space. (Swap space is a part of HDD that is used for virtual memory, which is the simulation of additional main memory).

> The only partition that can be accessed immediately after a computer boots (i.e., starts up) is the root partition, which contains the root directory, and usually at least a few other directories as well. The other partitions must be attached to this root filesystem in order for an entire, multiple-partition filesystem to be accessible. Thus, about midway through the boot process, the operating system makes these non-root partitions accessible by mounting them on to specified directories in the root partition.

> Systems can be set up so that external storage devices can be mounted automatically upon insertion. This is convenient and is usually satisfactory for home computers. However, it can cause security problems, and thus it is usually not (or, at least, should not be) permitted for networked computers in businesses and other organizations. Rather, such devices must be mounted manually after insertion, and such manual mounting can only be performed by the root account.

> Mounting can often be performed manually by the root user by merely using the mount command followed by the name of the device to be mounted and its mounting destination (but in some cases it is also necessary to specify the type of filesystem). For example, to mount the eighth partition on the first HDD, which is designated by /dev/hda8, using a directory named /dir8 as the mount point, the following could be used:

>       mount /dev/hda8 /dir8

> Removing the connection between the mounted device and the rest of the filesystem is referred to as unmounting. It is performed by running the umount (with no letter n after the first u) command, likewise followed by the name of the device to be unmounted and its mount point. For example, to unmount the eighth partition from the root filesystem, the following would be used:

>       umount /dev/hda8 /dir8

> A list of the devices that are currently mounted can be seen by viewing the /etc/fstab file. This plain text configuration file also shows the mount points and other information about the devices, and it is employed during the boot process to tell the system which partitions to automatically mount. It can be safely viewed by using the cat command, i.e.,

>       cat /etc/fstab

# 参考

1.  非常好的解释了 "挂载" 的概念, 强烈推荐这篇文章: http://www.linfo.org/mounting.html

2.  维基, 也很好的解释了 "挂载" 的概念, 唯有一点不足的是对 "挂载点" 解释的不够好: http://en.wikipedia.org/wiki/Mount_(computing)

3.  其中有一句话不装逼的说出了 "挂载点" 是什么: http://www.linuxnix.com/2013/09/what-is-a-mount-point-in-linuxunix.html

4.  解释了 bind mount 的概念: http://www.funtoo.org/Funtoo_Filesystem_Guide,_Part_3#Bind_Mounts

