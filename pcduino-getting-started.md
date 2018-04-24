title: 初入 pcDuino
slug: pcduino-getting-started
date: 2014-01-02 11:16
tags: pcduino, ubuntu

网上关于 pcDuino 的教程相对其他板子来说还较少, 尤其是 Getting Started 这一块, 官方的网站上关于这一话题的惟一一篇文章只是介绍如何通过 Ethernet over USB OTG 连接到板子, 而且还是只针对 windows 用户, 要不是当时正好我旁边闲着一台 windows 的机器, 加上我现在已经"厌倦了折腾", 我才不会去尝试用 windows 去连接它.

当我尝试按照官网的教程做的时候, 我被卡在了第一步, 每当我反复的把一端接着 pcDuino OTG 口的 USB 线插入 win PC 时, windows 右下角就弹出个带着红叉的气泡告诉我这个硬件设备不能被安装/不能被识别, 也可以理解, Ethernet over USB 这东西我之前也没听过, 大概比较新, win 肯定没有内置驱动, 于是我开着驱动精灵检测 PC 上的硬件设备, 我一直觉得这是能够进入 windows 下十佳好用软件的软件, 可令人失望的是驱动精灵也不能检测出电脑 USB 口上插的是个 Ethernet 设备, 之所以检测不出我觉得可能是驱动精灵的驱动库里也没有这个驱动. 那看来只能我手动在网上搜驱动了, 我尝试着 google 了一下 "usb ethernet driver" 检索结果不太令人满意, pcDuino 的入门教程这块做的太不好了, 仅有的这篇教程里, 就不能给个驱动的下载地址么?

最后, 我很不屑的放弃了在 windows 下继续折腾.

我开始寻找在 linux 下连接到 pcDuino 的方案, 我找到了四个很有价值的页面:  
[http://ceworkbench.wordpress.com/2013/04/08/accessing-the-pcduino-desktop-without-an-hdmi-monitor-part-1/](http://ceworkbench.wordpress.com/2013/04/08/accessing-the-pcduino-desktop-without-an-hdmi-monitor-part-1/)  
[http://www.element14.com/community/thread/26532/l/quick-start-of-pcduino-without-a-hdmi-monitor-and-serial-debug-cable](http://www.element14.com/community/thread/26532/l/quick-start-of-pcduino-without-a-hdmi-monitor-and-serial-debug-cable)  
[https://learn.sparkfun.com/tutorials/getting-started-with-pcduino/serial-debugging](https://learn.sparkfun.com/tutorials/getting-started-with-pcduino/serial-debugging)  
[https://github.com/sparkfun/pcDuino/wiki/Getting-started](https://github.com/sparkfun/pcDuino/wiki/Getting-started)

其中第三个链接要自己接线, 所幸我不用用那个方法就能成功连接到 pcDuino 了.

参考 1, 2 两个链接, 我将我的 Debian PC, pcDuino 放到一个子网下(pcDuino 通过网线连接路由器), 路由器是之前搞到的 OpenWRT/TL-WR703n, 然后本机运行 `nmap --top-ports 10 192.168.1.*`, 惊喜的发现我的这块 pcDuino 默认开启了 sshd 服务(我这块 pcDuino 是 2013/09/17 出土的), 这下可以直接 ssh 连接了. 

但是... root 密码多少还不知道, 尼玛官网上对这个完全是只字未提啊, google "pcDuino ssh default password", 找到了第 4 个链接, 尼玛藏这么隐蔽. 第 4 个链接告诉我们, 买回来的 pcDuino 默认有两个用户的, root 和 ubuntu, root 用户默认没密码, ubuntu 用户密码是"ubuntu". 

于是 `ssh ubuntu@192.168.1.101`, 输入密码后成功进入, 发现已经装了 build-essential(ubuntu 下好像是叫这个名吧...), 于是 gcc, make 都有了, 省的我下载了, 然后我想开 tmux 时发现没有安装, 算了, 将就下 screen 吧, 靠! 竟然也没安装!

算了, 反正总算是进入系统了.
