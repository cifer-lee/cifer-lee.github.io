---
layout: post
title:
date: 2014-01-07 16:18
published: false
---

### 预备知识

*   我们需要手动做一些网络配置工作, 不过为了我们可以少操点心, 我们可以把这一工作放到 /etc/init.d 中.
*   2.4 内核中, 内核提供 /proc 文件系统供用户空间与其交流, 在 2.6 内核中, 内核提供了 sysfs.
*   运行 dhcpd, 从 dhcp 服务器那里获取 ip 地址, 你的网络接口, 如 eth0 会被重新配置. 很多 dhcp 服务器还会提供默认路由和 dns 信息(重写你的 /etc/resolv.conf).
*   若是想架设一个 dhcp 服务器, 要确保网络接口开启了 multicast 支持.
*   route 不知从哪个版本起, 添加路由必须这样: `route add -net 127.0.0.0/8 lo`, `-net` 和 `/8` 都不能少

### IP 别名

IP别名使得可以把你的主机配置得好像是多台主机, 每台有它自己的IP地址. 这种配置有时候被称作: 虚拟主机---尽管这个词也被用在其它地方(主机托管服务等).
但是与主机托管不同的是, 主机托管服务里的虚拟主机, 是在只有一个IP地址的情况下, 虚拟出很多虚拟主机来.

### 关于 ifconfig 命令

ifconfig 的调用格式是这样的:   

    ifconfig *interface* [*address* [*parameters or option*]]   

如果 __ifconfig__ 被调用时只传了 _interface_ 参数, 它会显示出这个接口的配置信息. 如果没有传任何参数, 它会显示出所有的 active 接口的配置信息; 如果指定 -a 选项, 它会显示所有接口的信息, 包括 inactive 的接口.   

在 ifconfig 的输出中, RX 与 TX 行表示有多少包被无误的接收和发送, 有多少错误发生, 有多少包因为内存不足而丢失(dropped), 有多少包因为来得太快/发送得太快内核来不及处理而溢出(overrun).

下面介绍一些参数(_parameters or options_)的用途

* up   

这个参数会打开接口, 使接口可以被 IP 层访问到. 当指定了 _address_ 的时候, 这个参数是会隐含指定的. 这个参数对应于输出结果中的 UP 和 RUNNING 标识. 

* down

这个参数关闭接口, 使得 IP 层不能访问它. 这个参数会禁止所有通过接口的 IP 报文. 这个参数还会删除路由表中所有使用此接口的条目.

* netmask _mask_

这个没什么好解释了

* pointopint _address_

这个参数是用于 point-to-point IP 连接的. 当你配置一个 SLIP 或者 PLIP 接口的时候会需要这个参数. 接口被配置了一个这样的地址时, ifconfig 的输出里会显示 POINTOPOINT 标识.

* broadcast address

当一个接口的广播地址被设置时, ifconfig 的输出会显示 BROADCAST 标识.

* mtu _bytes_

对以太网来说, 默认的 MTU 的值是 1500 (也是以太网允许的最大值); 对 SLIP 网来说, 这个默认值是 296. (其实对 SLIP 而言这个值没有限制, 只不过 296 是多年来人们总结出的一个比较好的值)

* arp

这个参数只适用于广播类型的网络, 如以太网. 它启用 ARP 协议. 对于广播型网络来说, 这个参数是隐含设置的.

* -arp 

禁用 ARP

可以做一个有趣的实验, 运行 `sudo ifconfig wlan0 -arp`, 然后 `sudo arp -d 192.168.1.1` (视自己的电脑的接口配置更改这两个命令), 这样的结果是你不能上网了.

* promisc

这个选项值得说, 它使得接口工作在混杂模式下. 在广播型网络中, 这会使得接口接收所有的数据包, 而不论这个数据包的 MAC 地址是不是本接口. 这个参数对应 PROMISC 标识.

### netstat 命令

* -r -n

这个选项会显示出内核的路由表, 就像我们使用 route 命令一样, -n 使得禁止解析 ip.

* -i

这个选项会显示出每个处于 active 状态的接口的统计信息. 

* -t, -u, -w, -x

这些选项会分别显示 tcp, udp, raw, unix 连接

# **arp 命令**

* -s _hostname_ _hostaddr_

用来永久的设置主机名和以太地址的关系
