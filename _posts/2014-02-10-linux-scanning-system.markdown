---
layout: post
title: linux scanner system
date: 2014-02-10 11:00
---

以下内容大部分翻译自 `man sane`

# sane

说到 Linux 的扫描仪, 就不能不说 sane

很多程序都使用 sane, 这些程序都可算是 sane 的前端, 包括 gimp, 包括 OpenOffice.org, 还有 saned, 它是提供 SANE 网络服务的一个 daemon, 不过在软件层面上看它应该属于 backend, 但是由于 SANE 也是一种 API 接口规范, saned 也遵循这种规范, 所以在协议层面上也算是一种前端, 还有其它的就不说了, 这里有一份列表: [http://www.sane-project.org/sane-frontends.html](http://www.sane-project.org/sane-frontends.html)

注意, 当我们说 SANE 的时候, 我们指的是 SANE 这种协议本身.

SANE 协议规定了一套标准的编程接口, 所以 SANE 实际上也扮演了规定 API 的角色, 它提供了一个通用的编程接口来访问所有的栅格图扫描仪. 这带给我们的好处是, 如果我们要写两个软件, 这两个软件能够兼容两台不同的扫描仪, 没有 SANE 的话, 那我们可能得写四份程序: 

*   软件 A
*   扫描仪 A 的驱动
*   软件 B
*   扫描仪 B 的驱动

如果使用 SANE 的话, 我们只需要写三份程序: 

*   软件 A
*   软件 B
*   基于 SANE 的扫描仪的驱动

这个好处是随着扫描仪数量和软件数量的增加而增加的.

## 术语

使用 SANE 接口的程序我们叫它前端, 实现 SANE 接口的驱动程序我们叫他后端. 用来管理其它后端的后端程序, 我们叫它元后端 (meta backend).

## 软件包

sane-backends 包包含了许多的后端程序, 文档, 网络支持, 以及一个命令行前端程序 --- scanimage --- 这是唯一一个例外 --- backends 包里包含一个前端程序. sane-frontends 包里则确实包含很多前端程序了. 这两个包都可以从 SANE 项目的主页上下载到, 也可以从你发行版的源里得到.

## 概览

### SANE 设备列表

SANE 设备列表包含了 SANE 所支持的设备们. 

### SCSI 配置

man sane-scsi

### USB 配置

man sane-usb

## 各种相关的程序

### scanimage

command-line 前端程序. 见 man 1 scanimage.

### saned

SANE 协议的一个网络守护进程, 使得远程主机能够访问服务器上的扫描仪. 间 man 8 saned.

### sane-find-scanner

一个用来查找 SCSI 或者 USB 扫描仪的命令行工具, 还能确定他们的 Unix device files. See man 1 sane-find-scanner.

## Backends for scanners

man 手册里有很多, 我只象征性的翻译几个:

*   apple

    支持苹果的扫描仪.

*   epson

    支持 epson 的扫描仪.

*   hp

    支持 HP 的 ScanJet 系列的扫描仪.

*   hp4200

    支持 HP ScanJet 4200 系列的扫描仪.

*   hpjlm1005

    支持 HP LaserJet M1005 系列的扫描仪.

## Backends for digital cameras

*   dc240

    支持 Kodak DC240 数码相机.

## 其它后端

*   dll

    sane-dll 库是一个用来动态加载其它后端的后端. `man 5 sane-dll`

*   net

    SANE 网络守护进程 saned 提供了访问其它计算机上的打印机的能力. `man 5 sane-net`, `man 8 saned`.

*   pnm

    PNM 图像阅读器是一个 pseduo-backend. 这个后端的目的是帮助调试其它的前端程序. `man 5 sane-pnm`.

*   v4l

    Video for Linux, 可以用这个后端来使用你电脑上的摄像头. 在我的电脑上, 有一个设备:

    `scanimage -L` 得到 v4l:/dev/video0 这个设备, 运行 scanimage -d v4l:/dev/video0 --mode Color 就可以用本机的摄像头拍一张照片. 

*   还有一些不想翻译了

## CHANGING THE TOP-LEVEL BACKEND

默认情况下, 所有的 SANE 后端程序 (驱动) 都由 sane-dll 这个元后端 (meta backend) 来加载. 如果你对动态加载有任何问题的话, `man 5 sane-dll`.

## 后端开发者

这段跟我没关系了, 不翻译了

## 涉及的文件们

*   /etc/sane.d/\*.conf

    后端们的配置文件

*   /usr/lib/sane/libsane-\*.(a|so)

    实现后端的静态库/动态库们

## Problems

If your device isn't found but you know that it is supported, make sure
that  it  is  detected by your operating system. For SCSI and USB scan‐
ners, use the  sane-find-scanner  tool  (see  sane-find-scanner(1)  for
details).  It prints one line for each scanner it has detected and some
comments (#). If sane-find-scanner finds your scanner only as root  but
not  as  normal  user,  the  permissions  for  the device files are not
adjusted correctly. If the scanner isn't found at  all,  the  operating
system hasn't detected it and may need some help. Depending on the type
of your scanner, read sane-usb(5) or sane-scsi(5).  If your scanner (or
other device) is not connected over the SCSI bus or USB, read the back‐
end's manual page for details on how to set it up.

Now your scanner is detected by the operating system but not  by  SANE?
Try  scanimage  -L.   If the scanner is not found, check that the back‐
end's name is mentioned in  /etc/sane.d/dll.conf.   Some  backends  are
commented  out  by default. Remove the comment sign for your backend in
this case. Also some backends aren't compiled at all if  one  of  their
prerequisites  are  missing.  Examples  include dc210, dc240, canon\_pp,
hpsj5s, gphoto2, pint, qcam, v4l, net, sm3600, snapscan,  pnm.  If  you
need  one  of  these backends and they aren't available, read the build
instructions in the README file and the individual manual pages of  the
backends.

Another  reason for not being detected by scanimage -L may be a missing
or wrong configuration in the backend's configuration file. While  SANE
tries  to  automatically  find  most scanners, some can't be setup cor‐
rectly without the intervention of  the  administrator.  Also  on  some
operating systems auto-detection may not work. Check the backend's man‐
ual page for details.

If your scanner is still not found, try setting the various environment
variables  that  are available to assist in debugging.  The environment
variables are documented in the relevant manual pages.  For example, to
get  the maximum amount of debug information when testing a Mustek SCSI
scanner, set environment variables  SANE\_DEBUG\_DLL,  SANE\_DEBUG\_MUSTEK,
and  SANE\_DEBUG\_SANEI\_SCSI  to  128 and then invoke scanimage -L .  The
debug messages for the dll backend tell if the mustek backend was found
and  loaded at all. The mustek messages explain what the mustek backend
is doing while the SCSI debugging shows the low level handling. If  you
can't find out what's going on by checking the messages carefully, con‐
tact the sane-devel mailing list for help (see REPORTING BUGS below).

Now that your scanner is found by scanimage -L, try to do a scan: 
`scanimage >image.pnm`. This command starts a scan for the default scanner
with default settings. All the available options are listed by  running
scanimage  --help. If scanning aborts with an error message, turn on
debugging as mentioned above. Maybe the configuration file  needs  some
tuning,  e.g.  to  setup  the path to a firmware that is needed by some
scanners. See the backend's manual page for details. If you can't  find
out what's wrong, contact sane-devel.

eck  that  the SANE libraries are installed correctly you can use
the test backend, even if you  don't  have  a  scanner  or  other  SANE
device:

    scanimage -d test -T

You  should  get  a list of PASSed tests. You can do the same with your
backend by changing "test" to your backend's name.

So now scanning with scanimage works and you want to  use  one  of  the
graphical  frontends  like  xsane, xscanimage, or quiteinsane but those
frontends don't detect  your  scanner?  One  reason  may  be  that  you
installed two versions of SANE.  E.g. the version that was installed by
your distribution  in  /usr  and  one  you  installed  from  source  in
/usr/local/.   Make  sure  that  only one version is installed. Another
possible reason is, that your system's dynamic loader  can't  find  the
SANE  libraries.  For  Linux,  make  sure that /etc/ld.so.conf contains
/usr/local/lib and does not contain /usr/local/lib/sane.  See also  the
documentation of the frontends.
