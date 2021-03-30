---
title: DPoS 核心概念
date: 2018-10-04
categories:
  - Blockchain
tags:
  - dpos
---

本文基于 BM 的唯二的两篇阐述 DPoS 机制的文章, 第一篇文章是 BM 首次提出 DPoS 共识机制, 第二篇是 BM 后来对 DPoS 机制补充的白皮书. 两篇文章的链接见文末.

最近精读了这两篇文章, 从中提炼出了以下我认为是最核心的内容.

# 见证人选举与调度

DPoS 里最重要的两个部分就是见证人的选举以及调度出块. 

见证人的选举靠持币人的投票, 并且是一币一票的规则, 持币量有多少, 投票权重就多大. 但我觉得这不好, 权重不应当随持币量而无节制的增大.

投票选举是 DPoS 与其它共识机制最大的不同点, DPoS 高度依赖投票机制来剔除不好的见证人, 从来维持链条运行在一个健康稳定的状态. 反观比特币等其它共识, 在发生节点作恶的情况时, 所能采取的措施就捉襟见肘了.

调度这块也是与比特币有着很大的不同, 比特币中是竞争出块, 而在 DPoS 中块由谁出则是被安排的明明白白. 在一个任期内经过选举选出一批见证人, 然后这批见证人就公平的轮流出块了, 任期的长短是可以被协商的. 在任何时候, 持币人都可以重新投票, 下一个任期会根据最新票数重新选举一批见证人. 在最初的 DPoS 机制中, 当这批见证人轮流出完一轮块之后, 他们的顺序还会被打乱, 不过在 EOS 中, 这个规则已经取消了, 后面会说到.

# 最长链准则 (一致性保证)

和比特币一样, DPoS 里见证人出块也遵守最长链准则, 这是所有节点的基本共识, 是一种弱一致性策略.

第二篇文章的前半部分讨论了几种可能的分叉情况, 比如消息延迟, 网络分区, 甚至是存在少量作恶节点等, 并说明了在最长链规则下这些问题会迎刃而解. 但这些基本上都是最长链准则所解决的, 这在比特币时代就是 work 的, 并不是 DPoS 的创新点. 虽然不是 DPoS 的创新, 但 DPoS 却确实改进了这个准则, 在第一篇文章里有这么一句话:

> Ø Because all witnesses are elected, highly accountable, and granted dedicated time slots to produce blocks, there is rarely any situation where two competing chains can exist. From time to time, network latency will prevent one witness from receiving the prior block in time. If this happens, the next witness will resolve the issue by building on whichever block they received first.

不像比特币的矿工可以随意加入退出, DPoS 的见证人都是投票选出来的, 都是可追责的, 每个见证人都有他应该出块的时间点, 所以相比比特币, 显然 DPoS 的最长链更稳固一些. 当然这也是以牺牲了一些去中心为代价的.

另外, 原本 DPoS 中每轮出块见证人都会打乱顺序, 用第二篇文章的原话来说其目的如下:

> Ø This randomization ensures that block producer B doesn’t always ignore block producer A and that anytime there are multiple forks of identical producer counts that ties are eventually broken.

一方面是防止一些节点总是故意的忽略它前面节点的区块, 另外就是减小两个相同长度的分叉链存在的时间. 后来在 EOS 里取消了这一规则, 可能是因为 EOS 的社区监督太强, 存在故意忽略区块的或者故意分叉的节点的话一定会被投票下去, 所以这个规则显得没有必要了, 当然这使得整个链条的出块更清晰更有可预见性, 是好事情.

# 参与度与最新不可逆区块

DPoS 中存在参与度 (participation rate) 的概念, 这是什么概念? 第一篇文章给出了定义:

> Ø The participation rate is calculated by comparing the expected number of blocks produced vs the actual number of blocks produced.

基于参与度的概念, DPoS 又定义了一个最新不可逆区块 (Last Irreversible Block) 的概念: 当某个区块已经被 2/3 的见证人认可 (意味着这个区块后跟着另外 2/3 的见证人出的块), 这个块就会成为最新不可逆区块.

LIB 的思想和比特币的 "6 次确认安全" 是一样的, 但是我认为相对于比特币的 6 次确认, DPoS 的 LIB 确实有所加强, 具体体现在这两点:

1. 比特币的 6 次确认是大众意识, 而 DPoS 中的 LIB 是写到代码里了, 如果一个新块的高度在 LIB 甚至之前, 这个块会被代码拒绝. 相反比特币的代码中没有这个限制逻辑, 只要你算力够, 从第 1 个块开始重新出一条链都可以, 如果能出到现今比特币的区块高度, 现有的所有比特币节点都会转到你的链上来.
2. 由于 DPoS 中的出块节点的身份是可鉴别的, LIB 机制保证了真的是有 2/3 的不同节点确认了 LIB 块. 相反在比特币中, 6 次确认可能是同一个节点确认的.

这两点使得在交易安全性方面 DPoS 比比特币更强一些. 因为在网络分区的情况下, 即便有 6 次确认, 也保证不了在分区消失后你看到的 6 个区块会被作为主链保留, 但是一般这种情况下我们就认为后交易是安全的了. 而在 LIB 的机制下, 如果发生了网络分区, 整条链的参与度会处在一个低水平, 这种情况下我们能够很明确我们的交易是不安全的.

这对于抵御双花攻击也是不错的改进.

# TaPoS

Transactions as Proof of Stake, 是在交易中包含最近的某个区块的哈希, 如果交易所包含的某区块的哈希不存在, 那么这次交易也会被作废. 这一措施可以看作是对 LIB 或者 6 次确认的安全性的补充, 而且这个措施是发生在用户端的.

假设有这么一个场景, 小明和小红约定了要交换一些 BTS 和 STEEM, 小红已经把 BTS 转给了小明, 这一交易被打包进了 a 区块, 然而由于网络分区或者见证人下线等问题导致当前整个链处于一个低参与度的状态, a 区块一直得不到 2/3 确认. 小明可以转 STEEM 给小红, 但是又怕网络稳定后 a 区块被废弃, 怎么办呢? 解决方法就是小明在交易中附带上 a 区块的哈希, 最终可能会有这么几个结果:

1. 网络恢复稳定, 小明的交易被打包进新区块, 但是验证 a 区块已经不在了, 小明的交易无效
2. 网络恢复稳定, 小明的交易被打包进新区块, 验证 a 区块存在, 交易达成
3. 小明的交易被打包进新区块 c, 但是网络依然不稳定. 后来网络稳定后 a, c 区块都存在, 交易达成
4. 小明的交易被打包进新区块 c, 但是网络依然不稳定. 后来网络稳定后 a 区块存在, c 不存在, 小明交易回退
5. 小明的交易被打包进新区块 c, 但是网络依然不稳定. 后来网络稳定后 a, c 都不存在, 小明交易回退

注意在后三种情况中, 不存在网络稳定后, a 不存在 c 存在的情况, 因为 c 里面包含引用 a 的交易.

总之可以看到, 有了 TaPoS 的保证, 小明肯定是不会有损失的.

另外 TaPoS 额外的好处就是, 用户在发起交易的同时相当于也对区块做了确认, 这使得链条更稳固.

# 参考

1. https://bitshares.org/technology/delegated-proof-of-stake-consensus/
2. https://steemit.com/dpos/@dantheman/dpos-consensus-algorithm-this-missing-white-paper
