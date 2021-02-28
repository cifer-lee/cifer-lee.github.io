---
title: 记一次 Indigo 与 Ryu 的连接建立问题
slug: a-problem-ryu-indigo-conn-setup
date: 2016-12-25 16:03:13
categories:
  - SDN
tags:
  - OpenFlow
  - SDN
  - Indigo
  - Ryu
---

之前在研究 Indigo 和 Ryu 的衔接时碰到一个 OpenFlow 连接建立失败的问题, 特此记录下, 希望能够供别人参考.

在了解这个问题之前, 我们先回顾一下 OpenFlow 连接建立过程.

## OpenFlow 协议的连接建立过程

根据 OpenFlow 协议标准的陈述我们能够知道, 交换机和控制器之间使用 TCP (或者 SSL) 传输协议, 交换机必须能够主动发起连接 (实际应用中, 连接一般都是都由交换机主动发起), 另外就是所有的 OpenFlow 消息, 都要用网络序 (大端序) 发送 (参见 OpenFlow Spec v1.3 以及 v1.4 的第 7 章)

TCP 连接的建立我们很熟悉了, 就是典型的三此握手过程. 在 TCP 连接建立以后, 交换机和控制器双方在 TCP 连接建立后需要立即发送给 `OF_HELLO` 消息给对方, 并且 `OF_HELLO` 必须是双发发送给对方的第一个消息, `OF_HELLO` 消息同时起到协商 OpenFlow 版本的功能.

当双方都收到了对方的 `OF_HELLO` 消息并且两边都共同支持一个最小版本, OpenFlow 连接就成功建立了, 接下来控制器就可以向交换机发送其它的消息, 比如一般第一次要发送的就是 `OFPT_FEATURES_REQUEST` 消息.

## 遇到的问题

在这部分工作中基于 Ryu 框架我写了个简单的 Ryu 小程序令它与 Indigo 通信, 但是发现似乎连接建立都不成功, 好在 indigo 项目的错误日志部分做的很好, 我打开了 verbose 级别的日志, 发现连接建立过程中 indigo 在收取 `OF_HELLO` 这个消息失败了, 结合代码发现是在读 socket 时发生了 `EAGAIN (Resource temporary unvailable)` 错误. 熟悉 Linux 开发的人都知道这个错误还有另一个名字叫做 `EWOULDBLOCK`, 其含义是指我去读一个非阻塞的 socket, 这个 socket 本来是被期望可读的, 但是实际读它的时候发现并没有数据.

这就有点奇怪了, 控制器发送 `OF_HELLO` 消息给交换机, 然而交换机读这个消息时却发生了这样的系统错误. 原本我怀疑是我的交换机系统环境有问题, 于是我写了一个最基本的 tcp socket server 和一个 tcp socket client, 让 tcp socket server 运行在控制器主机并监听控制器的标准端口 6653, tcp socket client 则跑在我的交换机系统上, 发现这两者是能够正常收发包的, 而且, 我按照 OpenFlow 协议组了一个 `OF_HELLO` 消息, 从 tcp socket server 发送给 tcp socket client, 我的 socket client 也能够正常的接收这个消息, 并没有出现 `EAGAIN` 错误.

所以说交换机系统环境是没有问题的, 可是为什么 indigo 的代码里收不到这个 `OF_HELLO` 消息呢? 看来必须得窥探一下 Ryu 发过来的这个 `OF_HELLO` 消息是什么样的.

## 问题的分析

为了确定 Ryu 发过来的 `OF_HELLO` 消息没问题, 用 `tcpdump` 抓包看一下两者通信的细节, 下面就是交换机和控制器连接建立过程中抓取的数据包:

```
09:59:26.578677 ethertype IPv4 (0x0800), length 74: 10.0.0.1.33430 > 10.0.0.2.6653: Flags [S], seq 992465816, win 43690, options [mss 65495,sackOK,TS val 15346082 ecr 0,nop,wscale 7], length 0
        0x0000:  4510 003c fa3a 4000 4006 426f 7f00 0001  E..<.:@.@.Bo....
        0x0010:  7f00 0001 8296 19fd 3b27 d398 0000 0000  ........;'......
        0x0020:  a002 aaaa fe30 0000 0204 ffd7 0402 080a  .....0..........
        0x0030:  00ea 29a2 0000 0000 0103 0307            ..).........
09:59:26.578703 ethertype IPv4 (0x0800), length 74: 10.0.0.2.6653 > 10.0.0.1.33430: Flags [S.], seq 710500324, ack 992465817, win 43690, options [mss 65495,sackOK,TS val 15346082 ecr 15346082,nop,wscale 7], length 0
        0x0000:  4500 003c 0000 4000 4006 3cba 7f00 0001  E..<..@.@.<.....
        0x0010:  7f00 0001 19fd 8296 2a59 5fe4 3b27 d399  ........*Y_.;'..
        0x0020:  a012 aaaa fe30 0000 0204 ffd7 0402 080a  .....0..........
        0x0030:  00ea 29a2 00ea 29a2 0103 0307            ..)...).....
09:59:26.578723 ethertype IPv4 (0x0800), length 66: 10.0.0.1.33430 > 10.0.0.2.6653: Flags [.], ack 710500325, win 342, options [nop,nop,TS val 15346082 ecr 15346082], length 0
        0x0000:  4510 0034 fa3b 4000 4006 4276 7f00 0001  E..4.;@.@.Bv....
        0x0010:  7f00 0001 8296 19fd 3b27 d399 2a59 5fe5  ........;'..*Y_.
        0x0020:  8010 0156 fe28 0000 0101 080a 00ea 29a2  ...V.(........).
        0x0030:  00ea 29a2                                ..).
10:10:56.193800 ethertype IPv4 (0x0800), length 76: 10.0.0.1.33430 > 10.0.0.2.6653: Flags [P.], seq 992465817:992465827, ack 710500325, win 342, options [nop,nop,TS val 15518486 ecr 15346082]
        0x0000:  4510 003e fa3c 4000 4006 426b 7f00 0001  E..>.<@.@.Bk....
        0x0010:  7f00 0001 8296 19fd 3b27 d399 2a59 5fe5  ........;'..*Y_.
        0x0020:  8018 0156 fe32 0000 0101 080a 00ec cb16  ...V.2..........
        0x0030:  00ea 29a2 0400 0800 0000 0000            ..).........                  (注: 交换机发给控制器的 OF_HELLO 消息, 即 0400 0800 0000 0000)
10:10:56.193822 ethertype IPv4 (0x0800), length 66: 10.0.0.2.6653 > 10.0.0.1.33430: Flags [.], ack 992465827, win 342, options [nop,nop,TS val 15518486 ecr 15518486], length 0
        0x0000:  4500 0034 7647 4000 4006 c67a 7f00 0001  E..4vG@.@..z....
        0x0010:  7f00 0001 19fd 8296 2a59 5fe5 3b27 d3a3  ........*Y_.;'..
        0x0020:  8010 0156 fe28 0000 0101 080a 00ec cb16  ...V.(..........
        0x0030:  00ec cb16                                ....
10:11:01.246773 ethertype IPv4 (0x0800), length 75: 10.0.0.2.6653 > 10.0.0.1.33430: Flags [P.], seq 710500325:710500334, ack 992465827, win 342, options [nop,nop,TS val 15519749 ecr 15518486]
        0x0000:  4500 003d 7648 4000 4006 c670 7f00 0001  E..=vH@.@..p....
        0x0010:  7f00 0001 19fd 8296 2a59 5fe5 3b27 d3a3  ........*Y_.;'..
        0x0020:  8018 0156 fe31 0000 0101 080a 00ec d005  ...V.1..........
        0x0030:  00ec cb16 0400 0008 eed3 0ebf            ............                  (注: 控制器发给交换机的 OF_HELLO 消息, 即 0400 0008 eed3 0ebf)
10:11:01.246800 ethertype IPv4 (0x0800), length 66: 10.0.0.1.33430 > 10.0.0.2.6653: Flags [.], ack 710500334, win 342, options [nop,nop,TS val 15519749 ecr 15519749], length 0
        0x0000:  4510 0034 fa3d 4000 4006 4274 7f00 0001  E..4.=@.@.Bt....
        0x0010:  7f00 0001 8296 19fd 3b27 d3a3 2a59 5fee  ........;'..*Y_.
        0x0020:  8010 0156 fe28 0000 0101 080a 00ec d005  ...V.(..........
        0x0030:  00ec d005                                ....
```

在这里, 10.0.0.1 是交换机地址, 10.0.0.2 是控制器地址. 其中前三条报文显然就是 tcp 三次握手, 后四条就是两者相互发送的 `OF_HELLO` 消息, 上面两个括号里的注释是我加的. 在 OpenFlow 协议中, `OF_HELLO` 消息是一条只有 header 没有 payload 的消息, 所以 `OF_HELLO` 消息的长度就是 OpenFlow 消息头的长度: 8 字节, 其中第一个字节是 Ryu 与 Indigo 协商好的 OpenFlow 版本号, 这里是 `0x04`, 代表 OpenFlow v1.3 (注意不是 v1.4), 接下来的一个字节 `0x00` 表示消息类型是 `OF_HELLO`, 后四个字节是消息的唯一标识码我们可以不用理会, 中间的两个字节表示整个 OpenFlow 消息的长度, 有意思的地方就在这里.

Ryu 和 Indigo 相互发给对方的 `OF_HELLO` 消息中, 头部的长度字段字节序不一致, OpenFlow 协议头部的长度字段是两字节, `OF_HELLO` 消息长度为 8, 即 `0x0008`, 前面说过在 OpenFlow 协议中要求, 所有的 OpenFlow 消息, 都要用大端序发送. 所以这两个字节用网络序也就是大端序表示应该为 `0x0008`, 即 MSB 在低字节. 显然, 在这一点上 Ryu 的报文是正确的, indigo 的报文是错误的.

那么 `EAGAIN` 错误是怎么出现的呢? 既然 indigo 发出消息的长度字段的端序有问题, 可想而知其对于收到的消息的长度字段的端序理解也很可能有问题. 下面这段是 indigo 读取消息的代码逻辑, 为了清晰起见我做了一些删减:

    /* read header */
    if (READING_HEADER(cxn)) {
        INDIGO_ASSERT(cxn->bytes_needed + cxn->read_bytes ==
                      OF_MESSAGE_HEADER_LENGTH);
        if ((rv = read_from_cxn(cxn)) < 0) {
            return rv;
        }

        msg = (of_message_t)(cxn->read_buffer);
        msg_bytes = of_message_length_get(msg);
        if (msg_bytes < OF_MESSAGE_HEADER_LENGTH) {
            ++ind_cxn_internal_errors;
            return INDIGO_ERROR_PROTOCOL;
        }
        cxn->bytes_needed = msg_bytes - OF_MESSAGE_HEADER_LENGTH;
    }

    if (cxn->bytes_needed == 0) {
        return INDIGO_ERROR_NONE;
    }

    /* read the rest */
    if ((rv = read_from_cxn(cxn)) < 0) {
        return rv;
    }

稍微解释一下这段代码, 在 indigo 的代码中, 与 ryu 通信的 socket 是非阻塞的, 用 poll 系统调用来判断是否可读, indigo 在读 socket 时还有一个字段叫做 `bytes_needed` 用来表示我想要从 socket 中读多少个字节, `read_from_cxn(cxn)` 方法会按照 `bytes_needed` 的值明确读取这些字节数. 当内核第一次通知 indigo 这个 socket 可读时, `bytes_needed` 字段总是 8, 因为 OpenFlow 的消息头最小就是 8, 小于 8 就是非法的. 读取了 8 字节的头之后, `of_message_length_get(msg)` 会解析头部中的两字节的长度字段获知整条 OpenFlow 消息的长度, 用它减去头部的长度来获知我还需要读多少字节.

可想而知, 当 indigo 收到 ryu 的 `OF_HELLO` 时, 它也把 `0x0008` 处理成了 `0x0800`, indigo 以为 ryu 给它发送了 8 x 256 = 2048 个字节, 但显然实际上 ryu 只给它发送了 8 字节. 在 indigo 认为后面应该还有 2040 个字节, 于是继续调用 `read_from_cxn(cxn)`, 但实际上 socket 上没有字节可读了, 于是引发 `EAGAIN` 错误. 

## 问题的解决

综上来看, 问题出在 indigo 对长度字段的解析上, 也就是上面代码段里的 `of_message_length_get()` 方法中, 我层层看下去发现 `of_message_length_get()` 方法的调用链是这样的:
 
    of_message_length_get() >> buf_u16_get() >> U16_NTOH()

这几个方法的含义一看名字就知道, 最后的 `U16_NTOH()` 是个条件编译的宏, 定义如下: 
 
    #if __BYTE_ORDER__ == __ORDER_BIG_ENDIAN__ 
    #define U16_NTOH(val) (val) 
    #else 
    #define U16_NTOH(val) (((val) >> 8 |(((val) & 0xff) << 8)) 
    #endif 

所以现在一切都清晰了, indigo 没有使用 POSIX 标准的系统调用 `htons()`/`ntohs()`, 而是自己定义了一套主机序到网络序的转换方法, 可能是 indigo 想能够编译在更多的平台上而不只是 POSIX 兼容的系统. 只不过这样一来, 我们就需要在编译 indigo 的时候根据我们自己的系统架构来定义 `__BYTE_ORDER__` 宏. 在我上面的实验环境中, 交换机系统架构是小端序的, 所以收到网络上的消息时, 对于二字节的长度字段应该做颠倒高低字节的处理, 然而 `__BYTE_ORDER__` 和 `__ORDER_BIG_ENDIAN__` 这两个宏我都没定义, 所以在预处理阶段 `U16_NTOH(val)` 宏就直接被定义成了 `(val)`.

那么解决办法就是, 视系统架构而定, 在编译 indigo 的 CPPFLAGS (或者你可能用的是 CFLAGS) 加上几个宏定义:

小端序系统:

    CPPFLAGS += -D__ORDER_BIG_ENDIAN__=0 -D__ORDER_LITTLE_ENDIAN__=1 -D__BYTE_ORDER__=__ORDER_LITTLE_ENDIAN__

大端序系统:

    CPPFLAGS += -D__ORDER_BIG_ENDIAN__=0 -D__ORDER_LITTLE_ENDIAN__=1 -D__BYTE_ORDER__=__ORDER_BIG_ENDIAN__

(End)
