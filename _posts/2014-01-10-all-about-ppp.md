---
layout: post
title: 关于 ppp 的一切
date: 2014-01-10 15:56
published: false
---

### 参考

1.  `man pppd`

###  基本概念

在 PPP 协议中, 无所谓客户端, 无所谓服务器, 两端是平等的, 如果硬要区分的话, 无非就是将发起连接(拨号)那端称作客户端, 被连的那端称作服务器端.

PPP 是通过一个特殊的串行线标准实现的. 要想将某一串行线作为 PPP 连接, 你首先要使用 modem 建立这个连接(这一般是 chat 所作的工作, pon 程序会读 peer 的配置文件, 而这个配置文件中)然后将这个连接

### 拨号的常见方式

*   调用 `$ sudo pon provider`, pon 这个脚本会执行 `/usr/sbin/pppd call provider`, pppd 的 call 选项会使得 pppd 去 /etc/ppp/peers/provider 文件中读取额外的参数, 而在 /etc/ppp/peers/provider 文件中则有这么一行 

        > connect "/usr/sbin/chat -v -f /etc/chatscripts/pap -T ********"

    pppd 的 connect 选项会使得在 ppp 连接建立之前执行一些必要的操作, 在这里我们执行了 chat 程序, 在使用拨号 modem 的时候, 一般会用到 chat 程序以便于在建立 ppp 连接之前完成拨号操作.

    注: 当使用 wvdial 的时候, 就不必用 chat 程序, 因为 wvdial 会完成 chat 的工作.


### 核心

*   pppd

### 拨号软件

*   chat
*   wvdial

### PPP startup

*   pon
*   wvdial

### 配置文件

*   /etc/ppp/options - 这是 pppd 的默认配置文件, 可以把这个文件当作模板文件来添加你自己的配置文件, 比如 options.pptp
*   /etc/ppp/peers - 这里面的文件用户指定某一确定的 peer, 比如提供 pptpd 服务的服务器, 通常我们为其建立一个配置文件 /etc/ppp/peers/pptpd, 然后再其中指定选项 file /etc/ppp/options.pptp, 当然还有其他的选项, 如此一来, 我们就可以直接 pon pptpd 来连接 pptpd 服务器了
