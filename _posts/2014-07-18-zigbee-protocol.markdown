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

The ZigBee device objects (ZDO), represent a base class of functionality that provides an interface between the application objects, the device profile, and the APS. The ZDO is located between the application framework and the application support sub-layer. It satisfies common requirements of all applications operating in a ZigBee protocol stack. The ZDO is responsible for the following: 

* Initializing the application support sub-layer (APS), the network layer (NWK), and the Security Service Provider.
* Assembling configuration information from the end applications to determine and implement discovery, security management, network management, and binding management.

The ZDO presents public interfaces to the application objects in the application framework layer for control of device and network functions by the application objects. The ZDO interfaces with the lower portions of the ZigBee protocol stack, on endpoint 0, through the APSDE-SAP for data, and through the APSME-SAP and NLME-SAP for control messages. The public interface provides address management of the device, discovery, binding, and security functions within the application framework layer of the ZigBee protocol stack. The ZDO is fully described in clause 2.5.

# 不明白的地方

1. 概念中的 Device application 与 Endpoint application 具体是什么?
