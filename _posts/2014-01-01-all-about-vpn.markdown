---
layout: post
title: 关于 vpn 的一切
date: 2014-01-01 19:37
---

我们知道 VPN 全称是虚拟专用网(Virtual Private Network), 就像很多计算机名词一样, 这个词也已经被过分使用, 而导致很多人知道它的全称却不知道它具体代表什么意思了. 在这里我们先好好复习一下 VPN 的定义.

我不喜欢肢解名字来解释一件东西, 但这里还是挺适用的, 首先从名字可以看出来, VPN 有两个特点: 

### 专用 (Private)
之所以称为专用网, 是因为这种网络只是用于本机构的主机和本机构的主机之间进行通信. 而不是用于本机构内的主机和外界网络通信. 

### 虚拟 (Virtual)
而之所以称之为虚拟, 则是因为实际上并没有使用专用的物理的线路来连接分散于各地的本机构的网络. 而是使用了现有的互联网的线路.

我们发现, 实际上虚拟专用网和我们接触最多的局域网很像:

*   首先, 它们使用的都是专用地址  
    10.0.0.0 --- 10.255.255.255  
    172.16.0.0 --- 172.31.255.255  
    192.168.0.0 --- 192.168.255.255     
*   其次, 它们都能划分不同的子网

实际上, 我们平常所说的局域网也可以说成是专用网, 因为严格来讲, 局域网中的主机是不能直接和外界通信的, 只能直接和自己网络的主机进行通信. 要和外界通信, 首先要有一个公网的地址, 这要借助 NAT, 那就是间接和外界通信了.

我们来看一下不同的地方, 不同的是, 在局域网中, 不管划分多少个子网, 子网中的主机物理位置都不会里的太远, 而在虚拟专用网中就不同了, 主机的距离离的非但远, 还不是一般的远, 可能跨国跨海. 

在局域网中, 不同的主机通过物理专线连接路由器然后主机之间进行交流, 这在虚拟专用网中就不太实际了, 总不能我在太平洋东岸放个路由器连几台主机, 然后拉个海底电缆到西岸连几台主机吧, 虽然理论上是可行的. 于是, 借助现有的因特网线路资源, 就造就了虚拟专用网的"虚拟"特性.

## 数据安全
但是, 使用了因特网的线路, 数据也就暴露给了整个因特网, 一般机构内部的通信是不希望泄漏给外界的, 所以一般 VPN 网络都有加密的要求.

## 搭建 VPN (PPTP)

1.  安装 PPTPD

    *   RHEL/CentOS

            rpm -i http://poptop.sourceforge.net/yum/stable/rhel6/pptp-release-current.noarch.rpm
            yum -y install pptpd

        实际上, 我也不知道为什么 CentOS 自己的源里没有 pptp 这个软件包.

    *   Debian/Ubuntu

            # apt-get install pptpd

2.  稍加配置

    *   /etc/pptpd.conf

        添加如下两行: 

            localip 192.168.80.1
            remoteip 192.168.80.224-238,192.168.80.245

        这里 localip 是 pptp server 的 ip 地址, remoteip 是每一个连接到 pptp server 的客户端能够获得的地址. 这里我故意使用了两种语法来指定 remoteip, 目的是向你展示一下.

    *   /etc/ppp/chap-secrets

        添加几个用户吧

            # Secrets fro authentication using CHAP
            # client    server  secret      IP address
              username1 pptpd   passwd1      *

        这里 client 是用户名, server 是服务类型 --- 我们这里是 pptpd, secret 是密码, IP address 允许连接到此 pptp server 的 IP, * 在这里表示任何.

    *   /etc/ppp/options.pptpd

        添加如下两行:

            ms-dns 8.8.8.8
            ms-dns 8.8.4.4

        顺便解释一下, /etc/ppp/options.pptpd 这个文件是会被 /etc/pptpd.conf 包含的, /etc/ppp/options.pptpd 里有两个比较重要的两个参数: 
            
            # Require the peer to authenticate itself using MS-CHAPv2 [Microsoft
            # Challenge Handshake Authentication Protocol, Version 2] authentication.
            require-mschap-v2
            # Require MPPE 128-bit encryption
            # (note that MPPE requires the use of MSCHAP-V2 during authentication)
            require-mppe-128
            # }}}

        启用了这两个参数后, 客户端也要启用这两个参数. 其中 mppe 需要响应的内核模块的支持. 如果你发现连接失败的话, 你可能需要安装并载入响应的模块.


Okay, 现在可以这样了:  `/etc/init.d/pptpd restart`

3.  还没完呢

    还要让系统允许 IP forward, 编辑 /etc/sysctl.conf 这个文件, 确保有这一行:

        net.ipv4.ip_forward = 1
    
    然后, 运行 `sysctl -p`.

4.  路由配置

    这是最重要的一步, 因为这一步把我绊住过. 路由配置其实也很简单, 运行下面的命令就可以:

        # iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE 
        # iptables-save

    但不幸的是, 很多 openvz 因为缺少内核模块, 导致第一条命令无法使用, MASQUERADE 这个 taget 不被支持. 运行第一条命令就返回 no target/match 之类的错误. 因此我们采取的是另一种效果基本一样的写法:

        # iptables -t nat -A POSTROUTING -s 192.168.80.0/24 -j SNAT --to-source _PUBLIC IP_

    这里, 192.168.80.0/24 是你为 vpn 分配的内网段, PUBLIC IP 就是你的 vpn server 的公网地址.

    另外, 你很可能还要加上这一句:

        # iptables -A FORWARD -p tcp --syn -s 192.168.80.0/24 -j TCPMSS --set-mss 1356

    这一句用来调整 TCP 报文的大小, 当你发现 vpn 无法正常工作时不妨加上这句试试.

### 客户端配置

略

## 参考

1.  [DigitalOcean 的 VPN 搭建教程: https://www.digitalocean.com/community/articles/how-to-setup-your-own-vpn-with-pptp](https://www.digitalocean.com/community/articles/how-to-setup-your-own-vpn-with-pptp)
2.  [很经典的教程: http://www.putdispenserhere.com/pptp-vpn-setup-guide-for-a-debian-openvz-vps/](http://www.putdispenserhere.com/pptp-vpn-setup-guide-for-a-debian-openvz-vps/)
