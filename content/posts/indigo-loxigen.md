---
title: LoxiGen 与 Indigo 项目介绍
description: "LoxiGen Indigo"
date: 2016-12-31
categories:
  - SDN
tags:
  - Openflow
  - SDN
---

Indigo 是由 Big Switch 开发, 现在托管在 Floodlight 组织下的 OpenFlow agent 开源实现. OpenFlow 控制器的开源实现有很多, 像是 Ryu, ODL, POX 等等, 但是 agent 方面的开源实现可能相对少一些, Indigo 是一个非常精巧的实现, 但是似乎网上 SDN 相关的中文社区对其介绍的文档却很少, 本文将对 Indigo 及其所依附的 LoxiGen 项目做一个简单的介绍.

## 关于 LOXI 与 Loxigen

LOXI (Logical OpenFlow eXtensible Interface), 是一种描述 OpenFlow 协议的逻辑语言.

LoxiGen 项目能够解读 LOXI 语言, 进而用来生成各种编程语言的 OpenFlow 协议库. 所以说 LoxiGen 实际上是一个 “编译器” 项目, 可想而知 LoxiGen 包含一个前端用来解析 LOXI 语言, 以及包含各种编程语言的后端来生成这些编程语言的代码. 目前包含 Java, Python 以及 C 语言的后端, 生成的 C 版本的协议库叫做 LOCI, Java 版本的叫做 OpenFlowJ, Python 版本的叫做 pyloxi.

> LoxiGen is a tool that generate OpenFlow protocol library for a number of languages. It is composed of a frontend that parses wire protocol descriptions and a backend for each supported language (currently C, Python and Java, with an auto-generated wireshark dissector in Lua on the way).

LoxiGen 项目组维护了一个快照, 定期的生成各种语言的协议库, 并将其放在 loxigen-artifacts 仓库下, 这样对于使用者来说只需要定期从 loxigen-artifacts 仓库拿现成的就行了, 连构建都不需要自己构建.

## Indigo 项目源码构成

Indigo 项目 ([https://github.com/floodlight/indigo](https://github.com/floodlight/indigo)) 为我们提供了一些与平台无关的基础功能, 包括:

* IO 复用与定时器管理框架: 提供了通用的 socket 回调注册机制与定时器回调处理机制, 让我们能够在单一线程中同时处理多个 sockets 的读写以及定时器的回调事件.
* OpenFlow 连接管理: 维护每一条与控制器之间的连接, 包括连接的建立, 心跳维持, 消息收发以及连接的终止.
* OpenFlow 状态管理: 包括与控制器之间的各种消息的处理以及交换机状态的上报都会在这里处理
* 配置模块: 提供了平台无关的配置接口

另外 Indigo 中有一些平台相关的功能是需要厂商自己来实现的, 这包括:

* Forwarding 模块: 负责实现将控制器下发的流表下发到交换机芯片中的接口
* Port Manager 模块: 负责实现端口管理的接口

芯片厂商需要负责实现后两个模块, 一般来讲就是将这两个模块中 indigo 所定义的接口用自己芯片的 sdk 来实现. 关于 Indigo 适配有一个很好的例子就是 Broadcom 的 OF-DPA, 我们可以从 Broadcom 的 [OF-DPA](https://github.com/Broadcom-Switch/of-dpa/) 项目仓库中获得实现了 Forwarding 和 Port Manager 模块之后的 indigo 代码.

OF-DPA 项目托管在 `https://github.com/Broadcom-Switch/of-dpa/`, 其对 Indigo 项目做了些许代码上的改动, 从这个地址获得 OF-DPA 的代码, indigo 的代码位于 ofagent/ 目录下:

    ofagent/
	    application/
	    indigo/
	    ofdpadriver/

其中 `indigo/` 目录中就是 indigo 的源码; `ofdpadriver/` 是 OF-DPA 实现好的 Forwarding 和 Port manager 模块的代码, 这两部分会调用芯片 sdk; `application/` 目录中只有一个源文件: `ofagent.c`, 这是一个示例程序, 向我们展示了如何初始化以及启动 indigo.

在 indigo 的项目代码也就是 `indigo/` 目录下, 又有两个子目录: `modules/` 和 `submodules/`, 其中 `modules/` 里面是 indigo 自身的代码, `submodules/` 里是 indigo 所用到的非自身的代码, 目前有 bigcode, infra, loxigen-artifacts.

bigcode 和 infra 都是 Floodlight 开发的通用工具库, 另外你可能注意到了 Indigo 使用了 loxigen-artifacts 中的代码. 下面就看一下 indigo 具体是如何使用 loxigen 的.

## Indigo 与 LoxiGen

Indigo 使用了 LoxiGen 项目生成的 LOCI 协议库, 并且是直接把源代码拿来使用的. 一开始将其放在 `indigo/modules/loci/` 目录下, Indigo 的维护者从 loxigen-artifacts 获得最新的代码, 然后再将其与 `modules/loci/` 下原本的 loxigen 项目代码合并或者直接替换. 因为这样每次都需要维护者手动去 pull loxigen-artifacts, 然后与 Indigo 的 codebase 合并, 后来估计是 Indigo 的维护者也觉得这样太笨了, 于是就干脆不往 `modules/loci/` 里面合了, 而是直接借助了 git submodule 机制, 把 loxigen-artifacts 用 git pull 拉下来放到了 `submodules/loxigen-artifacts/`, 然后 `modules/loci/make.mk` 做了如下的改动, 直接指向 `submodules/loxigen-artifacts/`.

    -loci_INCLUDES := -I $(dir $(lastword $(MAKEFILE_LIST)))inc
    -loci_INTERNAL_INCLUDES := -I $(dir $(lastword $(MAKEFILE_LIST)))src
    +LOCI := $(SUBMODULE_LOXIGEN_ARTIFACTS)/loci

    +loci_INCLUDES := -I $(LOCI)/inc
    +loci_INTERNAL_INCLUDES := -I $(LOCI)/src

    +LIBRARY := loci
    +loci_SUBDIR := $(LOCI)/src
    +include $(BUILDER)/lib.mk

这样一来, 编译 indigo 项目时就会直接从 `submodules/loxigen-artifacts/` 处获取 loxigen 的源码.

(End)
