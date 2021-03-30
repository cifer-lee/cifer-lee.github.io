---
title: 比特币的多重签名技术与实践
date: 2018-01-06
categories:
  - Bitcoin
---

> Multisignature scripts set a condition where N public keys are recorded in the script and at least M of those must provide signatures to unlock the funds. This is also known as an M-of-N scheme, where N is the total number of keys and M is the threshold of signatures required for validation
> 
> - <精通比特币>

多重签名首次规范化提出是在 BIP11, 它添加了一种新的交易类型 `OP_CHECKMULTISIG`, 这种交易的锁定脚本是如下这种格式:

```
M <Public Key 1> <Public Key 2> ... <Public Key N> N CHECKMULTISIG
```

其中 N 是密钥总数, M 代表激活这个交易所需要的最少签名数. 比如一个 2/3 多重签名锁定脚本就是如下这样的:

```
2 <Public Key A> <Public Key B> <Public Key C> 3 CHECKMULTISIG
```

对应的解锁脚本是如下这样:

```
0 <Signature B> <Signature C>
```

至于解锁脚本为何最前面有个 0, 是由于 `CHECKMULTISIG` 实现时有个 bug, 脚本执行时, 总是会执行一步出栈操作.

多重签名的锁定脚本中包含了多个地址, 发往这一锁定脚本的币, 可以看成是一个中间状态, 这个锁定脚本上的币可以说是属于其中包含的任一地址 --- 只要任一地址的持有者能取得其它地址的签名并解锁脚本, 就能花费这笔 UTXO.

# 应用场景

## 仲裁

最典型的就是引入仲裁机构的买卖双方交易场景. 在这个场景中, 需要事先创建一个包含买家, 卖家和仲裁三者公钥的 2/3 的 multisig 地址 (锁定脚本). 然后买家创建一笔交易发送到这个地址, 卖家发货, 买家验货没问题后, 仲裁创建一笔发送给卖家的交易, 同时用自己和卖家的签名签署这笔交易.

如果买家验货发现有问题, 则可以向仲裁提出, 由仲裁协商退货并创建一笔发送给买家的交易, 使用仲裁和买家的签名签署.

## 在线钱包

一个更安全的在线钱包服务应该提供多重签名功能. 本地和网站分别掌握一个私钥, 只有两把私钥同时签名才能够动用余额. 这一方面保证了网站不能随意挪用客户资金, 另一方面也保证了即使网站被黑或者用户的电脑被黑, 资金都无法被窃取 --- 除非两边同时被同一个黑客所黑.

其它应用场景比如合伙经营, 遗产管理等可以参考文末.

# 缺点

`CHECKMULTISIG` 虽然安全, 但是也有缺点. 首先就是 `CHECKMULTISIG` 锁定脚本比起 `P2PKH` 锁定脚本来说太长了, 这导致多重签名交易的体积增大, 而每一笔包含 `CHECKMULTISIG` 锁定脚本的 UTXO 都会存在于每个全节点的内存中, 比较耗内存.

另外在 BIP11 提议中, `CHECKMULTISIG` 脚本中密钥的数量被限定为 3 个, 可能这也是为了防止交易体积太大, 然而 3 个密钥常常不够用.

正式由于以上缺点, 才又提出了 `P2SH` 交易类型, 这个以后再说.

# 参考

1. BIP11
2. 比特币多重签名技术的应用场景(https://yilinhut.com/2014/08/08/5115.html)
3. 某款典型的多重签名钱包的使用流程, 可以看出, 多重签名钱包使用体验上还是比较麻烦的: 羽毛币钱包多重签名使用(https://www.sosobtc.com/article/13796.html)
4. 比特币多重签名现状 (2017 年): https://medium.com/@alcio/the-state-of-bitcoin-multisig-82b3bf09b1ca#.5rkhrix00
