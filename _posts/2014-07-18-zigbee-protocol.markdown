---
layout: post
title: "Zigbee 协议"
date: 2014-07-18 10:52
---

# 概览

Zigbee 联盟开发的低成本, 低功耗, 双向无线通信标准. 采用 Zigbee 标准的解决方案渗透到各个领域: 消费电子, 智能家控, 工业控制, PC 外设, 医疗传感, 玩具游戏等.

基于 IEEE 802.15.4 标准, 正如你所知道的, 这是一个个人无线区域网构建标准, Zigbee 的目标就是个人无线区域网.

## 协议栈

Zigbee 的协议栈也是很层构成的, 除了最上层之外, 每一层都包含为上层提供的各种服务, 有数据传输服务, 有协议管理服务等. 下层的每一个服务都会暴露给上层服务访问点 (Service Access Point, SAP).

IEEE 802.15.4 标准定义了最底层的两层: 物理层 (PHY) 和介质访问子层 (MAC), 之所以叫(介质访问**子层**是因为它是属于数据链路层 (DLL) 的). Zigbee 联盟在这基础上添加两层: 网络层和应用层. 而其中应用层又包括两个子层: 应用支持子层 (application support sub-layer, APS) 和 Zigbee Device Objects (ZDO).

开发者继承 ZDO, 创建自己的应用程序对象 (开发者可以创建最多 240 个应用程序对象), 同时能够使用 APS 提供的服务.

## 网络拓扑

Zigbee 支持星型, 树状以及 mesh 拓扑. 在星型拓朴中, 整个网络都会受位于中心的设备控制, 这个中心设备叫做 Zigbee Coordinator, 负责初始化以及维护整个网络, 其它所有的设备都是终端设备 (Zigbee End Device), 他们直接与 Zigbee Coordinator 通信.

在树状以及 mesh 拓扑中, Zigbee Coordinator 负责负责初始化网络, 设定主要的参数, 但是网络的节点数可以通过 Zigbee Router 来扩展, 也就是说, 终端设备 (Zigbee End Device) 可以不用直接和 Coordinator 通信, 可以借助 Router 间接的和 Coordinator 通信.

## Zigbee 相关概念

* Active network key

  This is the key used by a ZigBee device to secure outgoing NWK frames and that is available for use to process incoming NWK frames.

* Alternate network key

  This is a key available to process incoming NWK frames in lieu of the active network key.

* Application domain

  This describes a broad area of applications, such as building automation.

* Application key

  This is a master key or a link key transported by the Trust center to a device for the purpose of securing end-to-end communication.

* Application object

  This is a component of the top portion of the application layer defined by the manufacturer that actually implements the application.

* Application profile

  This is a collection of device descriptions, which together form a cooperative application. For instance, a thermostat on one node communicates with a furnace on another node. Together, they cooperatively form a heating application profile

* Application support sub-layer protocol data unit

  This is a unit of data that is exchanged between the application support sub-layers of two peer entities

* Attribute

  This is a data entity which represents a physical quantity or state. This data is communicated to other devices using commands.

* Beacon-enabled personal area network

  This is a personal area network containing at least one device that transmits beacon frames at a regular interval

* Binding

  This is the creation of a unidirectional logical link between a source endpoint/cluster identifier pair and a destination endpoint, which may exist on one or more devices

* Cluster

  This is an application message, which may be a container for one or more attributes. As an example, the ZigBee Device Profile defines commands and responses. These are contained in Clusters with the cluster identifiers enumerated for each command and response. Each ZigBee Device Profile message is then defined as a cluster. Alternatively, an application profile may create sub-types within the cluster known as attributes. In this case, the cluster is a collection of attributes specified to accompany a specific cluster identifier (sub-type messages.)

* Cluster identifier

  This is a reference to an enumeration of clusters within a specific application profile or collection of application profiles. The cluster identifier is a 16-bit number unique within the scope of each application profile and identifies a specific cluster. Conventions may be established across application profiles for common definitions of cluster identifiers whereby each application profile defines a set of cluster identifiers identically. Cluster identifiers are designated as inputs or outputs in the simple descriptor for use in creating a binding table.

* Device application

  This is a special application that is responsible for Device operation. The device application resides on endpoint 0 by convention and contains logic to manage the device’s networking and general maintenance features.

* End application

  This is for applications that reside on endpoints 1 through 240 on a Device. The end applications implement features that are non-networking and ZigBee protocol related

* Master key

This is a shared key used during the execution of a symmetric-key key establishment protocol. The master key is the basis for long-term security between the two devices, and may be used to generate link keys.

* Mesh network

This is a network in which the routing of messages is performed as a decentralized, cooperative process involving many peer devices routing on each other’s behalf

# 应用层

如图1 所示, 应用层包括 APS 子层, ZDO (包括 ZDO 管理子层, Endpoint 0), 以及厂商 (也就是开发者, 使用协议栈的人) 定义的应用程序对象 (manufacturer-defined application objects, Endpoint 1-240).

## APS

APS 提供了一个网络层和应用层直接通信的接口, 这些接口既会被 ZDO 使用也会被厂商 (开发者, 使用协议栈的人) 定义的应用程序对象使用. APS 提供的服务由两项实体来表示:

* APS data entity (APSDE), 通过 APSDE service access point (APSDE-SAP).
* APS management entity (APSME), 通过 APSME-SAP.

APSDE 提供了数据通信服务, APSME 提供的是安全机制, 设备绑定, 以及数据库管理服务.

## Application Framework

应用程序框架是 (厂商) 应用程序对象所寄存的地方. Zigbee 支持多达 240 个不同的应用程序对象, 分别标识为 endpoint 1-240. 有两个额外的应用程序对象被定义来给 APSDE 使用: endpoint 0 被保留用于 ZDO, endpoint 255 背包流用作广播消息给所有的应用程序对象 (1-240). endpoint 241-254 被保留未来使用.

### Application Profiles

Application profiles are agreements for messages, message formats, and processing actions that enable developers to create an interoperable, distributed application employing application entities that reside on separate devices. These application profiles enable applications to send commands, request data, and process commands and requests.

### Clusters

Clusters are identified by a cluster identifier, which is associated with data flowing out of, or into, the device. Cluster identifiers are unique within the scope of a particular application profile.

## Zigbee Device Objects

Zigbee device objects (ZDO), 是一个基类, 它提供了一套供应用程序对象, 设备配置, 以及 APS 之间通信的接口. ZDO 位于应用程序框架和 APS 之间. 它提供的接口满足了应用程序对象的所有需求. ZDO 的具体职责如下:

* 初始化 APS, NWK 和 Security Service Provider.
* 从应用程序对象那里收集配置信息, 以供以后使用. 比如如何发现, 安全级别如何, 网络如何管理, 绑定操作怎么做.

The ZDO presents public interfaces to the application objects in the application framework layer for control of device and network functions by the application objects. The ZDO interfaces with the lower portions of the ZigBee protocol stack, on endpoint 0, through the APSDE-SAP for data, and through the APSME-SAP and NLME-SAP for control messages. ZDO 的接口在应用程序框架层面上提供了设备地址管理, 设备发现, 设备绑定, 以及一些安全功能. 

### 设备发现

设备发现是一个 Zigbee 设备发现其它 Zigbee 设备的过程. 有两种形式的设备发现: 一种是请求 IEEE 地址, 一种是请求 NWK 地址. 请求 IEEE 地址是指在已知 NWK 地址的情况下, 向这个 NWK 地址发送单播消息已获得其 IEEE 地址; 请求 NWK 地址是指, 在知道了 IEEE 地址的情况下, 发送广播消息来请求 NWK 地址.

[Cifer: 这和 TCP/IP 里的 arp 协议是有点像的, 但又不完全一样]

### 服务发现

服务发现是某一个设备所能提供的服务被其他设备发现的过程. 服务发现可以通过向某个设备上的每一个 endpoint 发送一条查询消息来完成, 或者是根据某项服务的特性来完成 (比如某个服务可能会监听某个端口, 这就可以通过广播来查询到这个服务).

ZDO 中的相关的服务发现的接口提供了很多服务发现的方法, 直接用就可以.

# 应用层详说

## APS

APS 这一层提供了一些服务, 这些服务是厂商应用程序对象和 ZDO 都能够使用的. APS 提供的服务分两块, 一块是数据服务 (Data service), 另一块是管理服务 (Management service). 

数据服务能够帮助应用程序对象传输数据, 相应的 SAP 叫做 APSDE-SAP; 管理服务提供了比如设备绑定这样的机制, 其 SAP 叫做 APSME-SAP; 

管理服务还维护了一个数据库, 数据库中都是 APS 所为何的应用程序对象, 这个数据库叫做 AIB (APS info base). 

APS 这一层还定义了 APS 层侦格式.

### APSDE

APSDE 提供了数据服务, 使得应用程序对象或者 ZDO 的数据能够传到 NWK 层, 进而传到其它的设备上 --- 当然这些设备要在同一个网络中.

那么具体 APSDE 提供的服务, 我们来看看:

* 产生应用程序层的数据包: APSDE 应该接收应用程序对象传来的数据, 并将其封装加上报头.
* 绑定消息: 一旦两个设备绑定, 一个设备的 APSDE 应该传送消息给另一个设备.
* 分组地址过滤: APSDE 应该能够根据应用程序对象的所出的分组过滤收到的分组消息.
* 可靠的消息传输: APSDE 应该保证从它那发送和接收到的数据是可靠的.
* 没有重复消息: 这和上面那条类似. 同一条消息 APSDE 不会发送两次或以上, APSDE 也不会传送重复的消息给应用程序对象.
* 分片: 消息过长的话, APSDE 应该能够支持对发送的消息分片, 以及对接收的消息重组.

APSME 提供的与管理相关的服务, 我们再看一下 APSME 提供的具体的服务:

* 绑定管理: APSME 能够使得两个设备根据自己的服务与需求进行绑定, 并且在每个设备的 APSME 层会保存这个绑定关系.
* AIB 管理: 上层应用程序对象的信息 (Attribute) 全都存储在 AIB 中, APSME 同样负责管理, 修改它们.
* 安全: APSME 提供安全机制.
* 分组管理: 提供一个能力, 使得可以声明一个地址, 这个地址被多个设备共用, 依次将多个设备加入分组, 以及将设备从分组中移除.

[Cifer: 和 TCP/IP 相比的话, Zigbee 的 APS 层应该相当于 TCP/IP 中的传输层 (TCP, UDP 层), 而 Zigbee 中的 NWK 层, 毫无疑问相当于 TCP/IP 中的网络层 (IP 层)].

下面我们从 APSDE 和 APSME 中挑选几个例子来详细的看一下.

#### APSDE 数据传送

与数据传送相关的操作有三个: 请求, 确认, 通知应用层.

* APSDE-DATA.request

这个操作请求将详细从本设备的应用程序层传送到其他设备的应用程序层. 其消息包含如下字段:

    {
      dst_addr_mode, // integer, 0x00 - 0xff, 表示 *dst_addr* 字段的地址模式, 这个字段 
                     // 0x04 - 0xff 都是被保留的, 只有下面四个值可取:
                     // 0x00 = 表示 dst_addr 和 dst_endpoint 字段不可用
                     // 0x01 = 表示 dst_addr 是 16 位的分组地址, dst_endpoint 字段不可用
                     // 0x02 = 表示 dst_addr 和 dst_endpoint 都可用且都是 16 位地址,
                     //        指定一个特定应用程序对象
                     // 0x03 = dst_addr 和 dst_endpoint 都可用, 且都是 64 位地址
      dst_addr,      // Address, 代表设备地址或者是分组地址, 视 *dst_addr_mode* 字段而定
      dst_endpoint,  // integer, 0x00 - 0xf0, 0xff, 这个参数代表某个特定的应用程序对象,
                     // 它的值与 *dst_addr_mode* 字段有关
      profile_id,    //
      cluster_id,    //
      src_endpoint,  // integer, 0x00 - 0xf0, 不必多解释, 这个值当然不能是 0xff, 那时广播值
      ADSU_length,   // 数据长度
      ADSU,          // 类型是字节流, 表示实际数据
      tx_options,    // 提供了额外的数据发送选项, 有安全保障, 应答机值, 分片保障等
      radius         // unsinged integer, 0x00 - 0xff, 相当于 TCP/IP 里数据包的 TTL 值
    }

* APSDE-DATA.comfirm

这个操作将请求发送数据 (APSDE-DATA.request) 的结果报告给应用层. 消息体如下:

    {
      dst_addr_mode, // 与其所报告的 request 操作所设置的值一致, 含义同上
      dst_addr,      // 与其所报告的 request 操作的值一致, 含义同上
      dst_endpoint,  // 同上
      src_endpoint,  // 同上
      status,        // request 请求的结果, 可以有如下值:
                     // SUCCESS, NO_SHORT_ADDRESS, NO_BOUND_DEVICE, SECURITY_FAIL, NO_ACK 等
      tx_time        // 传送这个数据包所用的时间, 这个和具体实现有关
    }

* APSDE-DATA.indication

这个行为预示着有数据包发送到本设备的应用层. 消息体如下, 大部分字段的含义和上面相同, 不再多讲.

    {
      dst_addr_mode,
      dst_addr,
      dst_endpoint, 
      src_addr_mode,
      src_addr,
      src_endpoint,
      profile_id,
      cluster_id,
      asdu_length,
      asdu,
      status,
      security_status,
      link_quality,
      rx_time
    }

## 应用程序框架

### 创建一个 Zigbee Profile

想要在不同设备之间通信, 关键要遵守共同的 profile.

一个可以拿来当例子的 profile 是 home automation, Zigbee 联盟网站上能找到它. 这个 Zigbee profile 能够使得很多不同类型的设备之间通信交流, 从而构建一个家庭无线应用. 比如, 光传感器会发送信息给灯控制器, 从而控制电灯的开关; 运动传感器发送报警信号给控制器, 当它检测到门口有人撬门时.

还有一种 profile, 叫做 device profile, 这可能是两个或多个设备之间自己协商的 profile, 不是经过 Zigbee 联盟认可然后世界通用的. 这种 device profile 可以用来实现厂商自己的私有协议. 设备和服务发现就可以在 deivce profile 中定义, 厂商可以通过定制自己的私有协议, 使得只有自家的设备才能搜索到自家的设备.

#### 从 Zigbee 联盟取得一个 Profile (Identifier)

Zigbee 联盟指定的 Profile 分两种, 一种是厂商私有的, 一种是世界共用的. 每一个 Profiler 都会有一个世界唯一的标识符, 这是 Zigbee 联盟负责分配的. 一旦获得了 profile identifier, 拥有者就可以规定这些内容:

* Device descriptions
* Cluster identifiers

Zigbee 联盟官网上所列出的 Home Automation, Light Link 等都是世界通用的 profile.

#### 定义 Device 描述以及 Cluster

上面说了, 获得了 profile identifier 之后, 就可以自己制定 deivce 描述以及 cluster 了. 比如说, 对于 profile 标识符 "1" 来说, 你可以用 16 位的值来指定它所有的设备描述 (你可以指定 65536 项描述), 用另一个 16 位值指定所有的 cluster (也是 65536 项). 而每一个 cluster identifier, 你也可以用 16 位的值制定它所有的属性 (Attribute).

也就是, 每一个 profile 标识符, 可以有 65536 项描述, 可以有 65536 项 cluster, 可以有 65536 * 65536 项 attribute.

对于那些由 Zigbee 联盟制定的公共的 Profile, 其所有的 cluster 以及 attributes 以及 descriptions 也都是公开的.

#### 在 endpoint 上部署 profile

一个 Zigbee 设备可以支持多个 profile, 每一个 endpoint 支持一个,

### Zigbee 描述符

Zigbee 设备使用 descriptor 数据结构来描述他们自己. 实际的数据是由设备自己填写的. 有五个 descriptors: node, node power, simple, complex, 以及 user. 如下所示:

    -----------------------------------------------------------------------------
     Descriptor Name
    -----------------------------------------------------------------------------
          Node                必须              描述的是设备的类型与能力
    -----------------------------------------------------------------------------
       Node power             必须              描述设备电源的特性
    -----------------------------------------------------------------------------
         Simple               必须              关于设备描述的描述
    -----------------------------------------------------------------------------
         Complex              可选              更多关于设备描述的描述
    -----------------------------------------------------------------------------
          User                可选              用户自定义的设备描述
    -----------------------------------------------------------------------------

### 根据设备描述发现设备

一般来说, 有两个地方会取得设备的描述信息, 一个是 ZDO Management 层; 一个是在服务发现的时候, 使用 device profile request 原语请求 endpoint 0.

node, node power, complex, user 描述符是适用于整个设备的, 而 simple 描述符是需要对每一个应用程序对象都定义的.

### Node 描述符

Node 描述符包含了 Zigbee 设备的说明信息与其能力, 这个描述符对每一个 Zigbee 设备都是必需的, 而且每一个 Zigbee 设备只能有一个 Node 描述符. Node 描述符有如下的字段:

* Logical type: 3 bit, 000 代表 coordinator, 001 代表 router, 010 掉膘 end device
* Complex descriptor available: 1 bit, 指示 complex descriptor 是否存在
* User descriptor available: 1 bit, 指示 user descriptor 是否存在
* Reserved: 3 bit
* APS flag: 3 bit, 这个字段指示了这个设备的 APS 的能力, 这个字段目前没有使用
* Frequency band: 5 bit, 这个字段指示了这个设备所实现的 IEEE 802.15.4 中规定的频段, 如 868 MHz, 902 MHz, 2.4 GHz
* MAC capability flag: 8 bit, 这个字段是 802.15.4 MAC 子层所需要的, 详情见 Zigbee 协议 2.3.2.3.6 节
* Manufacturer code: 16 bit, 由 Zigbee 联盟分配给厂商的标识码
* Maximum buffer size: 8 bit
* Maximum incoming transfer size: 16 bit
* Server mask: 16 bit, 
* Maximum outgoing transfer size: 16 bit
* Descriptor capability field: 8 bit

### Node power 描述符

Node power 描述符展现了当前设备的动态的电源状态, 这个描述符对每个设备也是必须的, 并且也是惟一的. Node power 描述符有包含如下字段:

* Current power mode: 4 bit, 表示设备当前的电源状态, 休眠还是正常工作模式还是省电模式.
* Aailable power sources: 4 bit, 表示此设备能够使用的电源类型, 如 mains power (城市供电), 可充电电池, 一次性电池
* Current power source: 4 bit, 当前的电源类型
* Current power source level: 4 bit, 电量

### Simple 描述符

Simple 描述符包含一些特定于每个 endpoint 的信息, 这个描述符对每一个 endpoint 都是必须的. 字段如下:

* Endpoint: 8 bit, 0x01 ~ 0xf0
* Application profile identifier: 16 bit, 表示在这个 endpoint 上所支持的 profile, profile 标识符一般是 Zigbee 联盟处获得
* Application device identifier: 16 bit, 表示这个 endpoint 所支持的设备描述符
* Application device version: 4 bit,
* Reserved: 4 bit,
* Application input cluster count: 8 bit,
* Application input cluster list: 
* Application output cluster count: 8 bit,
* Application output cluster list: 

### Complex 描述符

包含了一些扩展的信息, 这个描述符是可选的. 字段有 Language and character set, Manufacturer name, Model name, Serial name, Device URL, Icon, Icon URL 等, 顾名思义.

### User 描述符

User 描述符包含了信息, 允许用户能够以一个易读的方式识别设备, 比如 "Bedroom TV" 或者 "Stairs light". 这个描述符是可选的. 这个描述符只有一个字段, 最多容纳 16 个 ASCII 字符.

* User description: 16 bytes

## Zigbee Device Profile

Device Profile 和 Zigbee Profile 类似, 也是要定义 clusters, 在 Device Profile 中定义的 cluster 需要所有的 Zigbee 设备都支持.

### 设备描述符

Device Profile 有一个单独的设备描述符.

## Zigbee Device Objects

### 状态机

#### Zigbee Coordinator

* 初始化

  Zigbee 协调器初始化时要设定一些参数, 以反应协调器启动的行为. 其中 APS 层的参数 apsDesignatedCoordinator 需要设置为 TRUE. 还需要填充各个 Descriptor, 如 Node Descriptor, Node Power Descriptor, Simple Descriptor. 如果需要的话, 还会填充 Complex Descriptor, User Descriptor, 以及 master key.

#### 启动流程

启动过程中会涉及到一些参数, 如下:

* nwkExtendedPANID: 这是一个 Zigbee 网络的 extended PANID, 是 64 bit 的值, 当它的值是 0x0000000000000000 时, 代表设备没有加入任何网络.
* apsDesignatedCoordinator: 这个字段标志着设备是否为 Coordinator.
* apsChannelMask: 这个字段标识了设备可以构建或者加入的信道
* apsUseExtendedPANID: 设备要加入或者要构建的网络的 64 bit 地址值
* apsUseInsecureJoin: 是否开启安全机制

任何时候设备启动或重启时, 都会经过这套启动流程. 当然, 也可以在软件中控制设备重启. 当设备启动时, 它应该检查 nwkExtendedPANID 参数的值. 如果不为零

# 网络层

## 网络的构建

### 网络构建原语

* NLME-NETWORK-FORMATION.request
* NLME-NETWORK-FORMATION.confirm

### 扫描信道 (以选出干扰最小的) 原语

当网络层要构建网络, 发出 NLME-NETWORK-FORMATION.request 之后, MAC 层会扫描信道

* MLME-SCAN.request
* MLME-SCAN.confirm

当找到了一个信道之后, NLME 应该从返回的 PAN 标识符列表中选择一个, 然后发起 MLME-SET.request 设置一下 MAC 子层的 macPANID 属性

* MLME-SET.request

之后就是启动网络

* MLME-START.request
* MLME-START.confirm

### 允许设备加入网络

只有 Coordinator 或者 Router 能够使用这个原语

* NLME-PERMIT-JOINING.request

如果在 NLME-PERMIT-JOINING.request 命令中, permitDuration 参数设为 0x00, 那么 NLME 应该将 MAC 子层的 macAssociationPermit 属性设为 FALSE, 通过 MLME-SET.request 命令.

如果在 NLME-PERMIT-JOINING.request 命令中, permitDuration 参数设为 0x01 ~ 0xfe 之间的值, 那么 NLME 要将 macAssociationPermit 属性设为 TRUE, 并启动一个定时器, 当时间到了的时候, 将 macAssociationPermit 设为 FALSE.

如果在 NLME-PERMIT-JOINING.reqeust 命令中, permitDuration 参数设为 0xff, 那么 NLME 要将 macAssociationPermit 的值设为 TRUE, 并一直持续到下一个 NLME-PERMIT-JOINING.request 命令为止.

## 加入网络

### 通过关系加入网络 (Joining a Network Through Association)

这个机制是 MAC 层提供的, 可以叫做 MAC layer association procedure, 

# 安全

Zigbee 协议中的安全服务包括 key 的建立, 传输, 侦保护, 还有设备管理. Zigbee 的安全架构的安全性依赖于密钥的保密性, 对称算法的安全性

## 密钥

Zigbee 网络的安全基于两个密钥: link 密钥, nwk 密钥. 在两个 APL 之间的单播通信用 128 bit 的 link key 来加密, 这个 link key 是只在这两个通信的设备间共享的; 广播消息会使用 128 bit 的 nwk key 来加密, 这个 nwk key 是网络中所有的设备共享的.

设备获得 link key 的途径应该是通过 key-tranport, key-establishment, 或者 pre-installation (比如,出厂期间); 而设备获得 nwk key 的途径应该是通过 key-transport 或者 pre-installation.

通过 key-setablishment 方式得到 link key 是基于一个 "master key" 的方式, 而这个 "master key" 的获得又需要经过 key-transport 或者 pre-installation 过程.

## NWK 层安全

当在 NWK 产生的侦需要被加密, 或者一个在高层产生的侦经过 NWK, 并且 NWK 层的 NIB 里 nwkSecureAllFrames 是 TRUE 时, 侦也会被加密 (除非高层的 NLDA-DATA.request 中 SecurityEnable 参数为 FALSE).

## APL 层安全

当一个在 APL 层产生的侦需要安全传输时, APS 子层就会进行这个操作, APS 允许基于 link key 或者 nwk key 来加密侦.

### key 的建立



# 感悟

1.  Cluster 就是命令, Cluster identifier 就是命令 ID. 一个 Profile 下有关于设备的 Device description, 还有设备可以做的操作 Cluster.

# 不明白的地方

1. 概念中的 Device application 与 Endpoint application 具体是什么?
