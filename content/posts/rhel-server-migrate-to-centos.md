---
title: 从 Redhat Enterprise Linux 迁移到 CentOS (让 RHEL 使用 CentOS 的源)
date: 2014-01-17
categories:
  - Linux
tags:
  - redhat
  - centos
  - linux
---

这篇文章里我们讨论从 RHEL 到 CentOS 的迁移, 文章标题之所以加了个括号注明"让 RHEL 使用 CentOS 的源", 是因为这两个过程所作的工作基本是一样的. 关于迁移过程 CentOS 官方的 [MigrationGuide](http://wiki.centos.org/HowTos/MigrationGuide) 说的很清楚, 但是这篇文章有点老, 而且这篇文章针对的是 RHEL 普通版, 而不是针对 RHEL Server 版, 所以文章里的几个地方我需要纠正一下.

在这之前, 先解释几个问题: 

1.  为什么我们要从 RHEL 迁移到 CentOS

    这是因为 RHEL 是收费的, 是的, 虽然你能够免费下载和安装 RHEL, 但是如果你想在上面安装其他的软件以及获取软件升级, 系统补丁等, 那就要交钱了, 你通过购买 subscriptions 来获取这些服务. CentOS 是 Community redhat Enterprise linux OS 的简称, 可以看出它是"社区版的 RHEL", 不收钱.

2.  为什么不直接安装 CentOS

    我们使用的 HP Proliant MicroServer Gen8, 使用了较新的磁盘阵列卡 - Smart Array Controller B120i, 这种磁盘阵列 HP 只提供了 OpenSUSE 和 RHEL 的驱动程序, 我们在安装系统的时候, 并不知道 CentOS 是 100% 的 RHEL, 即使我们知道, 我们也不敢认为 HP 提供的 RHEL 版本的驱动程序能良好的为 CentOS 使用.

3.  就没有其他解决方式了吗

    实际上我们不限于将系统迁移到 CentOS, 有很多免费的第三方源可以使用, 但是 CentOS 名气更大更可靠.

Ok, 下面我们就来看看 CentOS 官方那篇 Guide 有哪些地方需要修改以适用于我们. 

[MigrationGuide](http://wiki.centos.org/HowTos/MigrationGuide) 这篇文档前面说了很多注意事项, 这些对我们适用不需要修改, 我们应该好好阅读一下这些注意事项, 真正的迁移步骤在 _Migrate an existing system from RHEL6 or SL6 to CentOS 6_ 这一节, 我把我们需要做的截取在下面, 在这一节中除了下面我截取的部分, 其它的全是针对 SL6(Sentific Linux 6) 来说的, 我们不需要理会.

>       # mkdir TMP
>       # yum remove rhnlib abrt-plugin-bugzilla redhat-release-notes*
>       # rpm -e --nodeps redhat-release redhat-indexhtml
>       # cd TMP
>       # wget http://mirror.centos.org/centos/6/os/x86_64/Packages/centos-release-6-2.el6.centos.7.x86_64.rpm
>       # wget http://mirror.centos.org/centos/6/os/x86_64/Packages/centos-indexhtml-6-1.el6.centos.noarch.rpm
>       # wget http://mirror.centos.org/centos/6/os/x86_64/Packages/yum-3.2.29-22.el6.centos.noarch.rpm
>       # wget http://mirror.centos.org/centos/6/os/x86_64/Packages/yum-plugin-fastestmirror-1.1.30-10.el6.noarch.rpm
>       # rpm -Uvh \*.rpm
>       # cd ..
>       # rm -rf TMP
>       # yum clean all
>       # yum upgrade

首先是 `rpm -e --nodeps redhat-release redhat-indexhtml` 这一行, 由于我们使用的是 RHEL Server 版, 于是这一行需要改成 `rpm -e --nodeps redhat-release-server redhat-indexhtml`.

其次是以 wget 开头的那几行, 这几行下载了一些包, 有些包随着时间推移已经不存在了, 我们需要访问 http://mirror.centos.org/centos/6/os/x86_64/Packages/ 然后根据包名搜索它们的升级版, 比如说我在迁移的时候, centos-release-6-2.el6.centos.7.x86\_64.rpm 这个包已经升级成了 centos-release-6-5.el6.centos.7.x86\_64.rpm, 因此 wget 原地址就返回了 404.

## 后话

CentOS 的 Guide 中所述的上面的那些工作实际上就是将 RHEL 自带的 RHN 等删了, 然后安装上了自己一些东西. 迁移完之后我们发现 /etc/yum.repos.d/ 下面多了几个 CentOS 的源. 而原来 RHEL 自带的那个 rhel-source.repo 则没有了. 这个过程和我们手动在 /etc/yum.repos.d/ 添加 CentOS 的源是差不多的.
