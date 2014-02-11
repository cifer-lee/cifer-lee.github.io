---
layout: post
title: linux 打印系统
date: 2014-02-08 23:50
---

## PostScript

不得不先说说 PostScript, PostScript 是一种用于创建矢量图形的计算机语言. 它最有名的用途是作为"页面描述语言"用于在打印时$描述页面的排版布局.

解释 PostScript 语言的解释器也叫做 RIP (Raster Image Processor), PostScript 在 80-90 年代盛行, 直到后来图形工作站的普及, PostScript 的弊端逐渐显现出来 --- 实现 PostScript 的成本太高. 计算机输出原生的(raw) PS 代码给打印机, 然后打印机需要把这 PS 代码解释为栅格图像(raster image), 并且按照打印机的分辨率. 这个过程需要打印机配备高频处理器和充足的内存, 80-90 年代, 硬件的生产研发成本相较于在打印机上实现一个 PS 代码解释器而言较高, 但随着硬件成本约来越低, 在打印机上实现 PS 代码解释器的成本逐渐占了几乎全部的比例(打印机也算是嵌入式设备, 图形处理能力低, 想必实现解释器着实麻烦), 再后来, 桌面操作系统/图形工作站越来越普及, 也就没有必要将 PS 代码到栅格图像的转换交给打印机来做了, 在接在计算机这边做好就可以了.

到 2001 年, 一些低端打印机仍在在硬件层面上支持 PostScript, 但那是为了和那些新起的造价更便宜的喷墨打印机竞争.

而后, 一种在软件层面上(就是在计算机层面而不是打印机层面)的方法被发明出来, 使得 PS 代码在计算机上便能够被解释并显示解释出的图像来(而不用非得把 PS 代码交给打印机, 经打印机解释打印才能看到图像), 这样这个图像便能够在任何打印机都能够打印(而不必非得能够解释 PS 代码). PDF --- PostScript 的后代, 就是这样一种方法, PDF 基本上已经取代 PostScript 作为电子文档发布的格式了(因为 PDF 文档能被几乎所有的打印机打印出来, 而 PostScript 格式的文档只能被能够解释 PostScript 的文档打印出来).

高端打印机上, PostScript 解释功能还是很常见的, 这能显著的降低计算机 CPU 的工作量.

### 在打印行业的应用

#### PostScript 出现之前

在 PostScript 出现之前, 打印机们被设计为能够打印字符 --- ASCII 字符, 有很多技术实现这一任务, 但这些技术都有一个共同的不足, 就是打印出的字体单一, 难以改变, 就好像被固定印到键盘上的字母一样不能改变.

后来点阵打印机问世, 这种情况有所改观, 因为在点阵打印机中字符是一个点一个点的被"画"出来的, 这些点在打印机中定义为字体 --- 规定了每个字符应该怎么画, 随着点阵打印机越来越复杂, 它们本身也都包含了几种字体供用户选择, 甚至它们还能让用户上传自己的字体.

由于点阵打印机的特性, 使得它还能够打印栅格图形 (Raster Image, aks bitmap), 图形被计算机解释为一系列的点传给点阵打印机打印.

矢量图的打印有专门的仪器 --- 绘图仪, 昂贵, 用的人少.

#### PostScript 出来之后

以下拆自中文维基百科:

>   PostScript 将打印机和绘图仪的优点组合在一起从而打破了传统。同绘图仪一样，PostScript具有高质量的曲线处理能力并且控制语言简单能够用于不同品牌的打印机；同点阵打印机一样，PostScript提供了一个生成文本和光栅图形的简单方法。与它们二者不同的是，PostScript能够将所有这些不同的内容放在同一页上，这样就比以前的打印机或者绘图仪提供了更具灵活性。

PostScript 超越了传统打印控制语言而且已经自成体系成为一门完全的编程语言. 很多软件程序都能够将一篇文档转化为一段 PostScript 程序, 这段 PostScript 程序的执行结果就是原始文档. 这段 PostScript 程序被发送到打印机的话, 打印机中的 PostScript 程序解释解就能够解释这段程序, 并将文档打印出来. 这段 PostScript 程序也可以被发送到另一个软件, 然后将文档在屏幕上显示出来. 由于这段 PostScript 既可以发送给打印机, 又可以发送给电脑软件或其他设备, 这被称作设备独立.

### PostScript 中的字体处理

PostScript 的字体和 PostScript 本身几乎一样复杂, 得益于 PostScript 语言对图形绘制的支持, 这些字体是用 PostScript 语言画出来的 (因而 PostScript 字体都是矢量字体), 打印机中的解释器自然能很好的解释这些字体, 因为是矢量字体, 这些字体能够以在任何分辨率下都展现的很好.

这听起来很简单, 实际上并不那么容易, 比如说当字体缩小时就不能简单的对字体进行线性缩放了 --- 矢量字在小尺寸时效果是不如点阵字的. PostScript 使用 _Hints_ 解决这个问题 (关于 Hints 可自行 Google), 后来 Hints 技术被 Adobe 公司申请了专利, 并在自己的 Type 1 字体 (用 PostScript 画的, 也叫做 PostScript Type 1) 中采用. Adobe 公司向那些使用 Type 1 字体的厂商收取高额的授权费, 不交费的厂商只能使用 Type 3, 而 Type 3 字体没有使用 Hints 技术, 没有 Type 1 字体美观. 

Type 1 费用太高 Adobe 不降价, 于是 Apple 公司自己开发了一套字体 TrueType (这个好像不是用 PostScript 画的), 然后 Adobe 就公开了 Type 1, 然后 Type 1 的使用也广泛了起来.

### 作为显示系统的应用

PostScript 作为打印输出的事实标准, 很自然人们也希望用它来描述屏幕输出, PostScript 作为屏幕输出标准也是有很多有点的, 这里不多说. 目前使用 PostScript 作为显示技术的两个主要例子是 Display PostScript (DPS) 和 NeWS.

### PostScript 编程语言

PostScript 是一门图灵完备的编程语言 (图灵完备的意思 google 之), 也就是说, 你可以手动用 PostScript 编写电脑程序, 就像使用 c, c++, java 一样, 然而不过, PostScript 程序一般都是由其它的程序软件生成的, 比如 LibreOffice 之类, 而不是人手写的.

PostScript 是一种堆栈似的语言, 不再详述, 可参考中英文维基的 PostScript 词条.

## PostScript Printer Description (PPD)

引用维基的解释:

>   PostScript Printer Description (PPD) files are created by vendors to describe the entire set of features and capabilities available for their PostScript printers.

一个 PPD 也包含一些 PostScript 代码, 用来唤醒打印工作, 事实上, PPD 扮演了 PostScript 打印机的驱动程序的角色, 将打印机的功能, 特征抽象为一系列同一的接口. 

在打印机服务器上添加 PostScript 打印机给 CUPS 管理的时候便是需要此打印机的 PPD 文件, 客户端在请求服务器打印文件的时候, 也需要读取这个 PPD 文件来创建一个 job.

Windows 也是要用 PPD 文件的, 不管是客户端还是服务器, 只不过在 windows 中将这个 PPD 文件给二进制化了.

## Ghostscript

Ghostscript 有两大功能, 一个是将 PostScript 转换为 PDF, 另一个是将 PostScript 或者 PDF 这样的页面描述语言 (page description languages) 栅格化 (以得到打印机或显示器能够理解的形式).

Ghostscript 可以被用做 raster image processor (RIP) 来为栅格打印机/设备提供栅格图形 --- 比如说, 作为 lpd (在打印机打印出来) 的input filter, 或者作为 PostScript 和 PDF 查看软件 (在屏幕显示) 的后端引擎.

Ghostscript 也可以被用作一种文件格式转换器, 比如说将 PostScript 转换为 PDF, 将 PDF 转换成栅格图(就是位图, png, tiff, jpeg, 等). 这个用途常常和 "virtual printer" 一起用.

Ghostscript 既然是个言语解释器, 自然也能被用于平常的编程开发环境中.

### 前端程序

有很多图形界面的工具被写出来, 使得用户可以在屏幕上看 PDF, PostScript 文件. 比如 Ghostview, gv (Ghostview 的升级), 这两个都是基于 Unix/X11, 没有使用 gtk 之类的图形库.

## Virtual Printer (虚拟打印机)

这个没啥好说

## PDF

引用 OpenPrinting 上的一句话:

>   All important desktop applications (GTK/GNOME, QT/KDE, LibreOffice/OpenOffice.org, Firefox, Thunderbird, ...) send print jobs in PDF and not in PostScript any more by default. In addition, a complete CUPS filter chain to process print jobs in PDF is available and used.

## 打印协议

### Line Printer Deamon Protocol / Line Printer Remote Protocol


## Berkeley Printing System

这个打印系统是 BSD 发明的, 它包含几个典型的命令:

*   lpr --- 供用户打印文档的命令
*   lpq --- 查看当前的打印队列
*   lprm --- 从打印队列中删除一个工作

以及一个 daemon:

*   lpd --- 与上述三个命令交互

Berkeley 的打印系统使用的打印协议是 Line Printer Daemon Protocol(也叫做 Line Printer Remote Protocol, 简写为 LPD 或 LPR), 这个协议支持从一台机器上发送打印作业给另一台运行着 lpd daemon 的机器.

## System V Printing System

这个打印系统是商业化 Unix 系统 --- System V 开发的, 它包含几个典型的命令:

*   lp -- the user command to print
*   lpstat -- shows the current print queue
*   cancel -- deletes a job from the print queue
*   lpadmin -- a sysadmin command that configures the print system
*   lpmove -- a sysadmin command that moves jobs between queues

以及一个 daemon (也叫做 lpd):

*   lpd --- 与上述三个命令交互

## LPRng

这是个开源的打印系统, 由个人开发者开发, 曾被作者抛弃, 又被新作者拾起, 目前用的似乎不是很多.

## CUPS

这个打印系统在功能上尽量模仿 Berkeley 和 System V 的打印系统.

## 参考资料

1.  [http://en.wikipedia.org/wiki/PostScript](http://en.wikipedia.org/wiki/PostScript)
2.  [http://zh.wikipedia.org/wiki/PostScript](http://zh.wikipedia.org/wiki/PostScript)
