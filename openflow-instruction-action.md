title: '关于 OpenFlow 协议中 Instruction, Action 概念的解读'
slug: openflow-instruction-action
category: SDN
tags: openflow, sdn
date: 2016-10-03 21:45:30
modifiedd: 2016-10-23 11:35:30

(首发于 sdnlab: http://www.sdnlab.com/17952.html)

阅读任何一个协议都要注意的一点是这个协议中所定义的专有术语, 对这些术语的理解不到位的话也会造成对协议的理解偏差. 本文想和大家分享几个可能容易混淆的术语.

在 OpenFlow 协议文档中经常会看到这么几个词语: Instruction, Action, Apply-actions, Action Set, Action List, Clear-actions, ... 有点迷惑人, 实际上这里面只有两个实体的概念: Instruction 和 Action. 为了保持后文的易读性, 这两个概念分别用中文 "指令" 和 "动作" 来描述. 下文中的 "指令" 和 "动作" 都特指在 OpenFlow 协议中的含义.

指令这个词, 特指流表表项中的指令, 当某个报文匹配了这个表项之后, 表项中的指令就会被应用于这个报文; 而动作是比指令更细粒度的概念, 但它并不是局限于流表表项的概念, 动作可以独立于指令而存在, 也可以被包含在指令中, 具体说来, 我们在下流表的时候, 可以为某个表项的某种指令指定一些列的动作, 但是动作并不是只有下流表的时候才会被用到.

本文以目前较新的 Openflow 1.4 版本为准, 来分别看一下指令和动作的含义.

<!-- more -->

## 指令

每一个流表的表项都包含一系列的指令, 当报文匹配上了这个表项后, 这些指令就会被执行, 这些指令的执行结果有几种: 改变报文, 改变 action set, 改变 pipeline. 这些指令可以按照其执行结果的不同而分类, 不同的流表的表项包含的指令种类也不同, 前面说了指令可以包含动作, 但也并非所有种类的指令都包含动作, 下面我们一起来看一下指令的分类.

#### 指令的分类

OpenFlow 1.4 中规定了 6 种类型的指令, 但并不要求交换机支持所有的类型. 另外要注意的是, 在 OpenFlow 协议文档中指令的类型名字几乎全都以 "actions" 后缀结尾, 我觉得这是非常容易令人混淆的地方, 我们一定要记住指令类型名中的 "actions" 字样和我们上面说的 "动作" 的概念完全没有关系. 然而 OpenFlow 协议文档的这种写法看起来似乎每种指令都包含了一组动作, 而实际上只有几种指令是真正包含动作的, 下面我们来看一下这 6 种指令与动作的关系.

为了避免我们更加混淆, 6 种指令类型的名字我还是保持和 OpenFlow 1.4 的 Spec 一样. 还有一点需要注意的是, 括号里的 "可选", "必选" 字样指的是交换机是否必须支持这个指令, 而不是说下流表时表项中是否必须包含这个指令.

* (可选指令) Meter *meter_id*, 不包含动作, 行为是将报文送往指定的 meter
* (可选指令) Apply-actions *action(s)*, 这个指令是真正包含动作的指令, 它的行为是立即对报文应用这些指令, 不要改变报文的 action set
* (可选指令) Clear-actions, 这个指令并不包含任何的动作, 它的行为是立即清除报文的 action set 中所有的动作
* (必选指令) Write-actions *actions(s)*, 这个指令真正的包含动作, 它的行为是将自己包含的动作合并到报文的 action set 中
* (可选指令) Write-Metadata *metadata / mask*, 这个也不包含动作, 用的不多
* (必选指令) Goto-Table *next-table-id*, 这个指令也不包含动作, 它表示把报文交给后续的哪张流表处理. OpenFlow 协议要求交换机必须支持这个 action, 但有一个例外是假设你的交换机本身就只支持一张流表, 那可以不支持这个 action.

## 动作

前面说了动作也有分类, 但是相比指令的分类, 动作的分类就比较好理解了, 我们稍加带过, 然后解释下 Action Set 和 Action List 两个概念.

同样, OpenFlow 协议不要求交换机支持所有的动作种类, 我们只看几个常见的:

* (可选) Output, 表示将报文从某个特定的端口送出去
* (必选) Drop, 丢弃报文
* (必选) Group, 表示将报文交给指定的组
* (可选) Change-TTL, 改变报文的 TTL 字段 (可以是 IPv4 TTL, MPLS TTL 或者 Ipv6 Hop Limit)

#### 关于 Action Set

Action set 是一个与报文相关联的概念, 只要提起 action set, 它就一定是报文的 action set, 它包含了当报文离开流表时要附加于这个报文上的动作. 我们前面看到了有一种 Apply-actions 指令, 它是在报文匹配了表项的时候将它包含的动作立即应用到报文上, 而 Write-actions 则是将它包含的动作合并到报文的 action set 中, 另外还有 Clear-actions 指令, 是将报文的 action set 清空. 最终报文走完所有流表时, 其 action set 里面有什么动作, 就执行什么动作, 这就是 action set 的作用了. 

#### 关于 Action List

Action list 实际上就是一系列动作的有序序列, 一定要注意其有序性. 在上面说到的流表中的 Apply-actions 指令中, 以及 OpenFlow 协议中同样能够包含动作的 Packet-out 命令中, 都要求所包含的动作被有序执行. 所以就出来了这么个 action list 的概念, 这是与 action set 的一点区别. 另一个区别是 action list 并不是和报文相关联的概念, action list 可以直接夹带在 controller 发给 agent 的消息中, 比如 Packet-out 消息; 也可以存在于流表表项的指令中, 比如 Apply-actions 指令.

## 协议源代码

说实话, 光看协议 Spec 我是没有理清楚这些个指令与动作的关系的, 真正完全理清楚是看了 OpenFlow 源码之后. 在 OpenFlow 源码中, 指令与动作的结构头分别如下:
    
    struct ofp_instruction_header {
        uint16_t type;          /* One of OFPIT_*. */
        uint16_t len;           /* Length of this struct in bytes. */
    };
    OFP_ASSERT(sizeof(struct ofp_instruction_header) == 4);

    struct ofp_action_header {
        uint16_t type;          /* One of OFPAT_*. */
        uint16_t len;           /* Length of action, including this
                                   header. This is the length of action,
                                   including any padding to make it
                                   64-bit aligned. */
    };
    OFP_ASSERT(sizeof(struct ofp_action_header) == 4);

Meter, Write-metadata, Goto-Table 这三类指令的结构如下, 它们的前两个字段和 `struct ofp_instruction_header` 是相同的, 另外可以看出, 它们都不包含 `struct ofp_action_header` 结构体, 所以这三个指令是不包含动作的.

    /* Instruction structure for OFPIT_METER */
    struct ofp_instruction_meter {
        uint16_t type;            /* OFPIT_METER */
        uint16_t len;             /* Length is 8. */
        uint32_t meter_id;        /* Meter instance. */
    };
    OFP_ASSERT(sizeof(struct ofp_instruction_meter) == 8);
    
    /* Instruction structure for OFPIT_GOTO_TABLE */
    struct ofp_instruction_goto_table {
        uint16_t type;             /* OFPIT_GOTO_TABLE */
        uint16_t len;              /* Length is 8. */
        uint8_t table_id;          /* Set next table in the lookup pipeline */
        uint8_t pad[3];            /* Pad to 64 bits. */
    };
    OFP_ASSERT(sizeof(struct ofp_instruction_goto_table) == 8);
    
    /* Instruction structure for OFPIT_WRITE_METADATA */
    struct ofp_instruction_write_metadata {
        uint16_t type;                    /* OFPIT_WRITE_METADATA */
        uint16_t len;                     /* Length is 24. */
        uint8_t pad[4];                   /* Align to 64-bits */
        uint64_t metadata;                /* Metadata value to write */
        uint64_t metadata_mask;           /* Metadata write bitmask */
    };
    OFP_ASSERT(sizeof(struct ofp_instruction_write_metadata) == 24);

而 Apply-actions, Clear-actions, Write-actions 三种指令则共用如下的结构体, 可以看到它是包含 `struct ofp_action_header` 的, 你可能会奇怪 Clear-actions 指令不是也不包含动作吗, 为什么也用了这个结构体, 实际上对于 Clear-actions 指令来说, `struct ofp_instruction_actions` 结构体的最后一个 actions 字段是大小为 0 的数组.

    /* Instruction structure for OFPIT_WRITE/APPLY/CLEAR_ACTIONS */
    struct ofp_instruction_actions {
        uint16_t type;           /* One of OFPIT_*_ACTIONS */
        uint16_t len;            /* Length is padded to 64 bits. */
        uint8_t pad[4];          /* Align to 64-bits */
        struct ofp_action_header actions[0];        /* 0 or more actions associated with
                                                       OFPIT_WRITE_ACTIONS and
                                                       OFPIT_APPLY_ACTIONS */
    };
    OFP_ASSERT(sizeof(struct ofp_instruction_actions) == 8);

另外上面还说到了一个 Packet-out 消息也是包含动作的, 它的定义如下, actions 字段包含了一个动作列表, 也就是 action list.

    /* Send packet (controller -> datapath). */
    struct ofp_packet_out {
        struct ofp_header header;
        uint32_t buffer_id;              /* ID assigned by datapath (OFP_NO_BUFFER if none). */
        uint32_t in_port;                /* Packet’s input port or OFPP_CONTROLLER. */
        uint16_t actions_len;            /* Size of action array in bytes. */
        uint8_t pad[6];
        struct ofp_action_header actions[0]; /* Action list - 0 or more. */
                                             /* The variable size action list is optionally followed by packet data.
                                              * This data is only present and meaningful if buffer_id == -1. */
        /* uint8_t data[0]; */               /* Packet data. The length is inferred from the length field in the header. */
    };
    OFP_ASSERT(sizeof(struct ofp_packet_out) == 24);

## 参考

1. OpenFlow Spec v1.4.0
