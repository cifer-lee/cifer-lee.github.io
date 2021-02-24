---
title: PEAR 与 PECL 介绍
date: 2015-02-09 09:39:30
modified: 2017-03-15 10:09:31
slug: php-pear-pecl
categories:
    - PHP
tags:
    - PHP
---

## PEAR

PEAR 全称是 PHP Extension and Application Repository, 和水果 "梨" 的英文发音是相同的. PEAR 存在的目的是:

* 提供一个有组织结构的开源代码仓库给 PHP 用户们
* 提供一个代码发布以及包维护的系统
* 制定一份 PHP 代码风格规范 (在这里: http://pear.php.net/manual/en/standards.php)
* 运作 PHP Extension Community Lbrary (PECL) 姐妹组织
* 维护相关的网站, 邮件列表, 源镜像, PEAR/PECL 社区
* PEAR 是一个社区驱动的组织, 由开发者管理.

### PEAR 的使命

PEAR 的使命就是为 PHP 用户提供良好可重用的组件 (避免让用户自造轮子), 以及领导 PHP 革新, 努力为 PHP 开发者提供最佳的开发体验.

### 由 PHP 书写的结构良好的代码库以及应用

PEAR 中的代码以 "包" 为单元. 每一个包都是一个独立维护的项目, 有专门的开发团队, 有自己的版本号, 发布周期, 项目文档, 以及与其他包的依赖关系信息.

PEAR 中的包都是以 gzip tar 档案格式发布的. 在你的系统上, 你可以使用 "PEAR installer" (http://pear.php.net/package/PEAR/) 来安装这些包.

不同的包之间可以显示的指定依赖关系, 不要根据包的名字相似程度想当然的认为他们有依赖关系.

### 代码发布与包维护

所有的要发布的包都需要在 pear.php.net 上注册, 这些包当然也可以被下载. pear.php.net 是 PEAR 的中心服务器. 还有很多第三方的服务器, 也可以在上面发布包以及被 PEAR installer 下载安装它们上面的包. 这些第三方的服务器被称为为 Channel.

上面所说的 pear.php.net 以及其他的 Channel, 所提供的包都是不同的, 就是说, 如果一个包在 pear.php.net 里有了, 就不会发布在别的 Channel 里了, 反之亦然. 而且, 绝大部分的 Channel 都是只提供专门的一个包, 通常他们也不会接受发布其他的包, 比如 pear.dropbox-php.com 这个 channel 就只是 dropbox-php 的几个开发者用来发布 dropbox-php 的.

但是也有例外, 比如像是 pecl.php.net, 这是一个专门提供用 C 语言写的扩展的 Channel, 从这个 Channel 下载的扩展, 需要编译, 安装, 然后在 php.ini 里加上相应的 extension=extname.so 行. 而 pear.php.net 上的扩展都是直接用 PHP 写成的, 下载后不需要编译, 因为就是 PHP 代码, 也不是动态库, 所以也不会在 php.ini 里加 extension=extname.so 这样的行, 直接 include 就行了.

pear.php.net 提供了两种介面来展示它上面的那些包, 一种是对人友好的 HTML, 一种是 PEAR installer 友好的 REST 接口. 这两种介面都使用 HTTP 协议.

前面说了, 每一个包都是以 gzip tar 档案形式发布, 其中包含一个 xml 描述文件, 描述这个包的一些信息, 包括这个包里所包含的文件及其作用, 以及这个包的依赖关系等.

### 关于 PEAR installer

PEAR installer 指的应该就是系统上的 pear 命令, 以及上面链接 (http://pear.php.net/package/PEAR/) 给出的 PEAR 这个包, 从这个链接可以看出, PEAR 这个包是 PEAR 的基础包, PEAR 仓库里相当一部分的包都依赖于 PEAR 包.

另外, 现在又出来了一个 PEAR2, PEAR2 有一个新的 installer, 叫做 pyrus, 是直接用 php 语言写的 (pear 命令, 应该是 C 写的), 压缩成 phar 格式, 可以直接执行 php pyrus.phar <subcommand> 来安装包. PEAR2 应该和 PEAR 是一班人马维护的, PEAR2 的 installer, pyrus, 目的是比 pear 更易用.

## PHP Extension Community Lbrary (PECL)

PECL (pronounced "pickle") is a separate project that distributes PHP extensions (compiled code written in C, such as the PDO extension). PECL extensions are also distributed as packages and can be installed using the PEAR installer with the pecl command.

> (以上部分译自: http://pear.php.net/manual/en/about.pear.php)

### PECL 和 PEAR 的关系

> » PECL is a repository of PHP extensions that are made available to you via the» PEAR packaging system.

这句话简单明了, PEAR 是 PHP 所使用的包管理系统, 而 PECL 是集中存放 PHP 扩展的一个仓库, 这个仓库的扩展是可以使用 PEAR 来安装的. 所以, PEAR 和 PECL 的关系, 就好比 Protage 和 Gentoo 源的关系.

### PECL 扩展的安装

通过以上说明, 想必我们都清楚了 PEAR 和 PECL 的关系.

PECL, pecl.php.net, 实际上也是 PEAR 系统的一个 Channel, pecl.php.net 里的包也是可以通过 PEAR installer (pear 命令) 来安装的. 但是, pecl.php.net 这里面的扩展都是 C 语言写的, 这些扩展下载下来之后还需要编译什么的, 编译的时候还需要准备扩展编译的环境, pear 命令当初设计的时候主要是适合于安装 PHP 语言写的扩展的.

因此, pecl 这个工具也出现了, 它是为 pecl.php.net 定制的, 能够自动下载 pecl.php.net 里的扩展包, 准备编译环境, 编译扩展, 然后安装它. 最后要使用的话我们只需在 php.ini 里添加 extension=extname.so 行. 
对于 pecl 这个工具, 一般发行版们会将它直接包含到 pear 工具所在的包里. 所以你安装完了 pear, 也就有了 pecl 了.

#### phpize

有些时候你可能处于防火墙内, 无法使用 pecl (原因我不知, 难道 pecl 还用了 80 以外的端口?), 这时候, 你可以使用 phpize, phpize 是包含在 php 源码包里的一个脚本 (有待验证, 我猜是脚本. 已经过验证, 确实是脚本), 只要你安装过 php, phpize 也就会被安装到你的系统中.

然后当你想要安装新的 PECL 扩展, 而又不能联网使用 pecl 命令时. 就可以到 php 源代码根目录 (显然, 你需要有一个份 php 源代码) 的 extname 目录下, 执行 phpize, 就会给你生成编译这个扩展所需要的编译环境 (主要是 Zend extension framework 或者 PHP extension framework 这两个扩展框架接口).

然后, 只要运行经典的三部曲: ./configure && make && make install 就可以了.

## 参考

1. 很好的解释了 PEAR 和 PECL 的关系: http://stackoverflow.com/questions/1385346/what-are-differences-between-pecl-and-pear
2. 推荐: http://php.net/manual/en/install.pecl.php
