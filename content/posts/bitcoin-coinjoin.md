---
title: 比特币隐私加固 - CoinJoin 技术简析
date: 2017-11-28
categories:
  - Bitcoin
tags:
  - bitcoin
---

# 思想

由 @gmaxwell 在 CoinJoin: Bitcoin privacy for the real world 一文提出. 核心思想就是利用比特币的一笔交易中可以有多个输入以及多个输出这一点, 将多笔交易合并, 使得让人难以分辨哪笔输入对应哪笔输出, 进而达到难以追踪某个地址的资金的来源或去向的目的.

CoinJoin 思想的通俗实践是, 当你想要转账时, 可以找到另外一些也想转账的人, 你们分别签名自己的输入共同创建一笔交易. 这就需要掺入一点点中心化来支持, 对于个人钱包来说并不太好实践, 但对于具有中心服务的 web 钱包提供商来说比较容易, web 钱包可以很容易的将多个用户的转账请求合并, 作为一笔交易广播出去, 比如 blockchain.info.

CoinJoin 不需要更改比特币固有协议, 后续的许多旨在提高交易私密性的技术都是基于 CoinJoin 的思想.

# 实现

前面说 blockchain.info 有采用了 CoinJoin, 实际上 blockchain.info 实现的是 Shared Coin --- 一个对 CoinJoin 思想的最简单的实现, 当我们在 blockchain.info 上创建交易时, blockchain.info 会自动将其与其他人发起的交易合并成为一笔.

不过 Shared Coin 后来被研究出交易中的记录仍然是可追溯的, 因此 blockchain.info 下线了 Shared Coin 技术: https://www.coindesk.com/blockchains-sharedcoin-users-can-identified-says-security-expert/

其他对 CoinJoin 思想的实现技术还有 Dark Wallet, CoinShuffle, 达世币中的 PrivateSend 等等.

# 后记

了解到 CoinJoin 是在达世币的白皮书, 后面有机会扒一扒达世币的 PrivateSend 技术, 看看它是如何扩展改进 CoinJoin 的.
