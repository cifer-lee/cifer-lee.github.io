---
title: OpenFlow 协议匹配结构进化与 OXM TLV 解读
date: "2016-11-14"
categories:
  - SDN
tags:
  - openflow
  - SDN
---

在 OpenFlow 流表定义中, 报文匹配表项是个重要的行为, 我们下发的每一个流表表项都可以包含一到多个匹配项, 报文进来时会与这些匹配项比较, 如果匹配成功的话, 表项中动作也相应的被附加到报文上. 典型的匹配项有入端口, 源 MAC 地址, 目的 MAC 地址, VLAN Id, 源 IP 地址, 目的 IP 地址, MPLS 标签等等. 不难想象, 网络中的报文种类繁杂, 想要能够匹配每一种报文, 光匹配项就能列一个长长的列表出来, 并且随着各种网络通信协议的不断演进以及越来越复杂的网络业务, 这些匹配项将来还会可能还会增加. 所以在 OpenFlow 协议中, 选择一种合适的数据结构来描述这些匹配项就显得重要起来.

## 早期的匹配结构

在 OpenFlow 协议早期的版本中, 使用一种固定的数据结构来表述所有的匹配项, 这个数据结构长度固定并且所有的匹配项都包含在里面:

    enum ofp_match_type {
        OFPMT_STANDARD,             /* The match fields defined in the ofp_match
                                        structure apply */
    };

    /* Fields to match against flows */
	struct ofp_match {
	    uint16_t type;              /* One of OFPMT_* */
	    uint16_t length;            /* Length of ofp_match */
	    uint32_t in_port;           /* Input switch port. */
	    uint32_t wildcards;         /* Wildcard fields. */
	    uint8_t dl_src[OFP_ETH_ALEN]; /* Ethernet source address. */
	    uint8_t dl_src_mask[OFP_ETH_ALEN]; /* Ethernet source address mask. */
	    uint8_t dl_dst[OFP_ETH_ALEN]; /* Ethernet destination address. */
	    uint8_t dl_dst_mask[OFP_ETH_ALEN]; /* Ethernet destination address mask. */
	    uint16_t dl_vlan;           /* Input VLAN id. */
	    uint8_t dl_vlan_pcp;        /* Input VLAN priority. */
	    uint8_t pad1[1];            /* Align to 32-bits */
        uint16_t dl_type;           /* Ethernet frame type */
        uint8_t nw_tos;             /* IP ToS */
        uint8_t nw_proto;           /* IP protocol or lower 8 bits of ARP code */
        uint32_t nw_src;            /* IP src address */
        uint32_t nw_src_mask;       /* IP src address mask */
        uint32_t nw_dst;            /* IP dest address */
        uint32_t nw_dst_mask;       /* IP dest address mask */
        uint16_t tp_src;            /* TCP/UDP/SCTP source port */
        uint16_t tp_dst;            /* TCP/UDP/SCTP dest port */
        uint32_t mpls_label;        /* MPLS label */
        uint8_t mpls_tc;            /* MPLS TC */
        uint8_t pad2[3];            /* Align to 64 bit */
        uint64_t metadata;          /* Metadata between tables */
        uint64_t metadata_mask;     /* Mask for metadata */
    }

这个是 OpenFlow v1.1 中的匹配结构体的定义, 其中 `type` 字段的取值只有一个就是 `OFPMT_STANDARD`. `length` 代表整个匹配结构体的长度, 它的值可想而知也是固定的, 就是整个结构体的长度 88 字节. `wildcards` 字段是一个位标志符, 在一次流表下发中, `ofp_match` 中的匹配字段并不都会被用到, 协议规定哪些匹配字段被用到, 其 `wildcards` 中的对应位就被清 0, 否则就置 1. 其余的字段就是各个匹配项了, 其含义想必一目了然, 不需多做解释.

通过以上的定义不难看出, 早期的 OpenFlow 协议对于匹配项处理的不是太好, 一个是匹配结构体固定, 所有的匹配项都包含在一起, 下流表时即时不需要这个匹配项, 也要一并下发下来, 加大了网络开销; 另外最重要的是这个定义毫无扩展性, 想要增加新的匹配项就等于再更新一版新的 OpenFlow 协议.

在后续的 OpenFlow 协议中, 采用了另一种新的定义解决了这些问题.

## 新的匹配结构

从 OpenFlow 1.2 开始, 一种新的匹配结构被定义出来, 这种结构被称作 OpenFlow Extensible Match, 简称 OXM, 它采用 Type-length-value 结构, 所以也被称作 OXM TLV.

	enum ofp_match_type {
	    OFPMT_STANDARD = 0,       /* Deprecated. */
	    OFPMT_OXM      = 1,       /* OpenFlow Extensible Match */
	};
	
	/* Fields to match against flows */
	struct ofp_match {
	    uint16_t type;             /* One of OFPMT_* */
	    uint16_t length;           /* Length of ofp_match (excluding padding) */
	    /* Followed by:
	     *   - Exactly (length - 4) (possibly 0) bytes containing OXM TLVs, then
	     *   - Exactly ((length + 7)/8*8 - length) (between 0 and 7) bytes of
	     *     all-zero bytes
	     * In summary, ofp_match is padded as needed, to make its overall size
	     * a multiple of 8, to preserve alignement in structures using it.
	     */
	    uint8_t oxm_fields[0];     /* 0 or more OXM match fields */
	    uint8_t pad[4];            /* Zero bytes - see above for sizing */
	};
	OFP_ASSERT(sizeof(struct ofp_match) == 8);

这个结构体定义以及注释都是取自 OpenFlow 1.4 的源码, 从其注释中可以看到 `OFPMT_STANDARD` 类型的匹配项已经废弃不用了, 所以 `ofp_match` 结构体的 `type` 字段今后的取值将总是 `OFPMT_OXM`. `oxm_fields` 字段表示的是一组 OXM TLV 的集合, 可能是 0 个, 也可能多个, 从这可以看出整个 `ofp_match` 结构是一个变长的结构, 在控制器下发消息时, 只需要包含需要的匹配项, 不需要的匹配项无需包含在消息体中, 省去了不必要的开销.

那么 OXM TLV 的格式到底是如何的呢?

## OXM TLV

每一个 OXM TLV 都一定包含一个 4 字节的头, 对于 OpenFlow 标准所定义的匹配项, `oxm_class` 的取值固定为 `0x8000`. `oxm_field` 表示具体的匹配项, 比如源 MAC, VLAN ID 等. `oxm_length` 表示此 OXM TLV 的值的长度, 以字节为单位. `M` 字段则表示这个 OXM TLV 是否包含掩码, 在 OXM TLV 中, 掩码的长度和值的长度是一样的, 掩码中的某位为 1 表示报文中匹配项对应位必须和值的对应位相同才能匹配, 掩码中的某位为 0 则表示对报文中匹配项对应位的值不做限制. 如果包含了掩码, 那么报文的匹配将会变成先和做掩码按位与操作, 结果再和 OXM TLV 的值进行比较. 如果下发的消息里没有包含掩码, 那就需要报文与 OXM TLV 的值完全匹配才行.


    31                         15          8             0
    ------------------------------------------------------
    |       oxm_class         | oxm_field |M| oxm_length |
    ------------------------------------------------------

OpenFlow 1.4 中定义的标准匹配项有如下这些:

	/* OXM Flow match field types for OpenFlow basic class. */
	enum oxm_ofb_match_fields {
	    OFPXMT_OFB_IN_PORT        = 0,  /* Switch input port. */
	    OFPXMT_OFB_IN_PHY_PORT    = 1,  /* Switch physical input port. */
	    OFPXMT_OFB_METADATA       = 2,  /* Metadata passed between tables. */
	    OFPXMT_OFB_ETH_DST        = 3,  /* Ethernet destination address. */
	    OFPXMT_OFB_ETH_SRC        = 4,  /* Ethernet source address. */
	    OFPXMT_OFB_ETH_TYPE       = 5,  /* Ethernet frame type. */
	    OFPXMT_OFB_VLAN_VID       = 6,  /* VLAN id. */
	    OFPXMT_OFB_VLAN_PCP       = 7,  /* VLAN priority. */
	    OFPXMT_OFB_IP_DSCP        = 8,  /* IP DSCP (6 bits in ToS field). */
	    OFPXMT_OFB_IP_ECN         = 9,  /* IP ECN (2 bits in ToS field). */
	    OFPXMT_OFB_IP_PROTO       = 10, /* IP protocol. */
	    OFPXMT_OFB_IPV4_SRC       = 11, /* IPv4 source address. */
	    OFPXMT_OFB_IPV4_DST       = 12, /* IPv4 destination address. */
	    OFPXMT_OFB_TCP_SRC        = 13, /* TCP source port. */
	    OFPXMT_OFB_TCP_DST        = 14, /* TCP destination port. */
	    OFPXMT_OFB_UDP_SRC        = 15, /* UDP source port. */
	    OFPXMT_OFB_UDP_DST        = 16, /* UDP destination port. */
	    OFPXMT_OFB_SCTP_SRC       = 17, /* SCTP source port. */
	    OFPXMT_OFB_SCTP_DST       = 18, /* SCTP destination port. */
	    OFPXMT_OFB_ICMPV4_TYPE    = 19, /* ICMP type. */
	    OFPXMT_OFB_ICMPV4_CODE    = 20, /* ICMP code. */
	    OFPXMT_OFB_ARP_OP         = 21, /* ARP opcode. */
	    OFPXMT_OFB_ARP_SPA        = 22, /* ARP source IPv4 address. */
	    OFPXMT_OFB_ARP_TPA        = 23, /* ARP target IPv4 address. */
	    OFPXMT_OFB_ARP_SHA        = 24, /* ARP source hardware address. */
	    OFPXMT_OFB_ARP_THA        = 25, /* ARP target hardware address. */
	    OFPXMT_OFB_IPV6_SRC       = 26, /* IPv6 source address. */
	    OFPXMT_OFB_IPV6_DST       = 27, /* IPv6 destination address. */
	    OFPXMT_OFB_IPV6_FLABEL    = 28, /* IPv6 Flow Label */
	    OFPXMT_OFB_ICMPV6_TYPE    = 29, /* ICMPv6 type. */
	    OFPXMT_OFB_ICMPV6_CODE    = 30, /* ICMPv6 code. */
	    OFPXMT_OFB_IPV6_ND_TARGET = 31, /* Target address for ND. */
	    OFPXMT_OFB_IPV6_ND_SLL    = 32, /* Source link-layer for ND. */
	    OFPXMT_OFB_IPV6_ND_TLL    = 33, /* Target link-layer for ND. */
	    OFPXMT_OFB_MPLS_LABEL     = 34, /* MPLS label. */
	    OFPXMT_OFB_MPLS_TC        = 35, /* MPLS TC. */
	    OFPXMT_OFP_MPLS_BOS       = 36, /* MPLS BoS bit. */
	    OFPXMT_OFB_PBB_ISID       = 37, /* PBB I-SID. */
	    OFPXMT_OFB_TUNNEL_ID      = 38, /* Logical Port Metadata. */
	    OFPXMT_OFB_IPV6_EXTHDR    = 39, /* IPv6 Extension Header pseudo-field */
	    OFPXMT_OFB_PBB_UCA        = 41, /* PBB UCA header field. */
	};

可以看出相比 OpenFlow 1.1, 这里的匹配项多了很多, 新的匹配结构也提供了方便的扩展匹配项的机制. 前面说了对于 OpenFlow 定义的标准的匹配项, 其 `oxm_class` 字段的值固定为 `0x8000`, 如果是其它厂商或组织定义的匹配项, 则可以使用 `0xFFFF` 这个值. OpenFlow 协议还规定了 `oxm_class` 取值在 `[0x8000, 0xFFFF)` 范围内的都留给 OpenFlow 标准, 以备将来协议更新之用; 而 `[0x0000, 0x7FFF]` 范围内的值则留给 ONF 组织.

### OXM TLV 中 payload 的长度

对于某个 OXM 类别下的某个确定的 OXM TLV, 显然其值的长度是一定的, 如果这个 OXM TLV 不包含掩码, 那么其值的长度就是 payload 的长度; 如果包含掩码, 那么 payload 的长度就是值长度的两倍.

关于值的长度, 有一点需要注意的是, 虽然很多 OXM TLV 的值的长度都按 bit 计算的, 比如 IP 报文的 DSCP 字段其实是 6 bits 字段, Vlan Id 是个 12 bits 字段, 但是 `oxm_length` 字段的单位是字节, 就是说即时 OXM TLV 值用不了一个字节, 也会占用一字节的空间.

这里还有一个细微的问题, 协议 Spec 中没有明确说明的, 就是如果某个 OXM TLV 包含掩码, 并且其值的长度小于 4 bits, 那么这个 OXM TLV 的 payload 最终占用的空间是 1 字节还是 2 字节呢? 这个问题微妙的地方在于值的长度小于 4 bits, 通过上面所说的我们知道如果没有掩码 payload 肯定也是占用 1 字节, 但如果有掩码呢? 在值和掩码加起来还是不超过 1 字节的情况下, 协议会为这种情况做一点空间优化, 让值和掩码 "挤" 进 1 个字节里吗?

答案是不会, 这种情况下 payload 还是会占用 2 字节的空间, 这个问题 OpenFlow 的文档里没有明确的回答, 答案我也是从源码里找到的. 在 OpenFlow 中 OXM TLV 的定义是用 `OXM_HEADER` 以及 `OXM_HEADER_W` 这两个宏来定义的, 前者定义不包含掩码的 OXM TLV, 后者则定义包含掩码的 OXM TLV, 而这两个宏其实又都引用了 `OXM_HEADER__` 这个宏. 可以看到, 定义中并没有对值小于 4 bits 的 OXM TLV 做什么特殊处理, 就算值只有 1 bit, 定义包含掩码时也要对长度乘以 2 变成 2 字节.

	/* Components of a OXM TLV header. */
	#define OXM_HEADER__(CLASS, FIELD, HASMASK, LENGTH) \
	    (((CLASS) << 16) | ((FIELD) << 9) | ((HASMASK) << 8) | (LENGTH))
	#define OXM_HEADER(CLASS, FIELD, LENGTH) \
	    OXM_HEADER__(CLASS, FIELD, 0, LENGTH)
	#define OXM_HEADER_W(CLASS, FIELD, LENGTH) \
	    OXM_HEADER__(CLASS, FIELD, 1, (LENGTH) * 2)

### 扩展 OXM TLV

我们可以通过 Experimenter 特性来扩展 OXM TLV, 前面已经说了当 `oxm_class` 字段取值 `0xFFFF` 时, 就代表这个 OXM TLV 是扩展字段. 另外对于扩展 OXM TLV 最重要的一点就是, 紧跟在 4 字节头后面的一定得是一个 32 bit 的 Experimenter ID, 而不是 OXM TLV 的值. Experimenter ID 可以用来唯一标志厂商或组织, 可以向 ONF 申请分配, 也可以是厂商自己已有的 IEEE 分配的 OUI 号码.

Experimenter ID 之后的内容 OpenFlow 协议就不关心了, 完全由厂商自己来解释.

## 参考

1. OpenFlow Spec v1.4.0
