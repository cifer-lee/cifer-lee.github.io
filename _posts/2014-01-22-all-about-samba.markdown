---
layout: post
title: 关于 Samba 的一切
date: 2014-01-22 16:22
---

Samba 实现了 CIFS 网络协议.

Samba 套件中还包括客户端工具, 这是给 Unix 用户使用来和 Windows 与 Samba server 通信的.

samba 包括三部分:

*   smbd
    
    这个守护进程处理文件和打印机的分享, 以及提供身份验证与授权的工作.

*   nmbd

    这个守护进程负责名字注册, 它实现了 Microsoft-compitable 的 NetBIOS 名字服务(NBNS), 这个名字服务又称为 WINS, 这个进程还负责浏览选择(Browse elections).

*   winbindd

    这个守护进程负责与 domain controller 通信, 来提供用户与组信息. 它同时提供了一个供 Windows 的 LanManager 验证机制(NTLM)的接口.

### CIFS(Common Internet File System) 协议

CIFS 协议是微软公司对 SMB 协议的翻版, CIFS 协议是一种面向连接, 有状态的的协议, 它依赖于如下三种网络服务:

*   名字服务
*   可以发送数据报给单个或多个主机
*   能够在客户端与服务器之间建立长期连接

samba3 以及 Windows 2000/XP/2003 都支持用 TCP/IP 协议来实现这三种服务. 而在 Windows 2000 之前, 微软的客户端要依赖 NetBIOS 这一层来实现上述三项. 较新的 CIFS 客户端和服务器, 比如 Samba 已经不必使用 NetBIOS 了, 不过, 它们往往提供一个 legacy 模式. 它们之间的关系像下面这样: 

                SMB/CIFS
            ----------------
                NetBIOS
            ----------------
         NetBEUI  TCP/IP  IPX
        ------------------------

#### 理解 NetBIOS 协议

早期 BIOS 包含一些操纵文件系统的底层代码, 但只用与本地. NetBIOS 被设计为在令牌环网上实现在不同主机之间进行文件系统的操作, 这需要一个更底层的传输层协议, 于是 IBM 又提出了 NetBEUI(NetBIOS Extended User Interface) 协议, 作为 NetBIOS 的运输层. NetBEUI 是为小型区域网设计的, 它提出一种"名字"的概念, 用来标识每一个加入区域网的主机. NetBEUI 协议越来越火, 以至于有人心生嫉妒, 于是后来, NetBIOS 协议被在 Novell 的 IPX 运输层协议上实现了出来, 两家竞争激烈.

谁料, 最终最火的运输层协议是 TCP/IP, 于是, NetBIOS 协议又不得不再次实现基于 TCP/IP 的版本.

但是 TCP/IP 是以数字(IP 地址)来标识区域网上的主机的, 而 NetBIOS 协议只认识名字, 于是就需要在名字和 IP 地址之间做个转换, 有很多 RFC 文档规定了具体的转换机制, 这些文档目前仍然是众多 NetBIOS 名字解析实现的参考. 而那之后, 基于 TCP/IP 的 NetBIOS 又被称为 NetBIOS over TCP/IP 或者 NBT.

NetBIOS 名字服务(上面的 nmbd 守护进程所提供的服务)解决了 NetBIOS 的名字与 TCP/IP 的数字地址间的转换问题, 然而随着 TCP/IP 与 DNS 变得随处可见, NetBIOS 的名字解析服务已经基本被 DNS 取代了.

在各大 Linux 发行版中, 安装 samba 的时候, 其依赖的都是 bind9 这个 dns 软件.

#### 名字

在 NetBIOS 协议中, 每当一台主机上线时, 它都要声明自己的名字, 这个过程叫做名字注册. 但是不同主机不能有相同的名字.

#### 节点类型

NBT 网络中的主机可以根据其使用的名字注册与解析的方式来分类, 有 b-node, p-node, m-node, h-node 这几类. 

#### 浏览

"浏览"是查找其它主机以及共享资源的过程(比如在 Windows 下打开网络邻居时会自动查找别的主机, 这个过程就是浏览, 但是浏览并不仅限于此). 在 SMB 网络中有两种类型的浏览:

*   浏览主机与共享资源列表
*   浏览特定主机的共享资源

对于第一种, 每个包含工作组或者是域的子网中, 都有一台主机负责维护当前网络上的可访问的主机和资源, 这台主机被称作 Local Master Browser. 它维护的列表叫做 Browse List. 对于第二种, 要想浏览某一主机上的共享资源, 就要首先连接到这台主机.

每当一台主机注册到了名字, 或者它关机离开时, 它都要向 Local Master Browser 宣告.

Local Master Browser 可以有很多备份.


### 参考资料

1.  man smb.conf (强烈推荐)
2.  鸟哥私房菜
