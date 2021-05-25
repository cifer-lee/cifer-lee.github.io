---
# Common-Defined params
title: "以太坊智能合约快速上手"
date: "2021-05-11"
description: "以太坊 智能合约 solidity ethereum smart contracts"
categories:
  - "Blockchain"
tags:
  - "ethereum"
  - "smart contracts"
  - "solidity"
# menu: main # Optional, add page to a menu. Options: main, side, footer

# Theme-Defined params
#thumbnail: "img/placeholder.jpg" # Thumbnail image
#lead: "Example lead - highlighted near the title" # Lead text
---

这两天学习以太坊合约开发, 不得不说以太坊社区是真 "繁荣", 网上的教程多得数不胜数, 为什么繁荣加引号呢, 因为从这些教程的质量来看就知道, 以太坊社区的繁荣是各自为营, 社区里各式各样的团队各怀鬼胎的搞以太坊的生态建设, 而实际上根本目的都是为了推销自己的 dapp, 试图扩大自己在生态圈里的虚名，做以太坊社区 V 神之下的老二. V 神显然也不想做领袖角色, 社区就让他自由发展吧!

可是官方文档你好歹整好点, 我在官方想找个智能合约的 quickstart 都找不到, 官方文档说两句就丢你给个 remix 或者 truffle 等工具链接让你用那些工具写智能合约. 这不是我想要的, 我想学习的是不借助任何那些花里胡哨的 IDE 怎么把合约部署到主链上, 官方文档却不教你这一点, 没有这个信息, 即使有可能也是某个开发者的个人博客上有介绍(可能也是自己研究之后做个笔记), 而那些花里胡哨的工具的文档只会介绍自己的工具怎么样, 更不会跟你说底层原理.

不得已, 我只能多花了些时间把智能合约的开发部署捋了一遍. 下面我会摈弃虚华的外表, 让你能以最小的学习成本 + 最快的速度完成一个智能合约并发布.

## 开发环境搭建

环境方面我们需要:

1. 一个趁手的编辑器, 我选 vim 你随意
2. 智能合约的编译器, 我们选择 [solcjs](https://docs.soliditylang.org/en/v0.8.4/installing-solidity.html#npm-node-js)
3. 以太坊节点, 我们选择 geth
4. 需要搭建一个测试或者私有网络, 我们选择 rinkeby 测试网络

关于开发环境方面上述 4 点是我摸索出来成本最低最省事而又最接近在主网部署合约的配置, 诸位如果是第一次接触以太坊智能合约开发大可放心按我的方式来.

反观以太坊的官方文档, 看看它是怎么叫我们搭本地开发环境的:

https://ethereum.org/en/developers/local-environment/
![](local-dev-env.png)

看到了吗, 直接甩给你一堆框架, 谁知道用哪个好, 官方就不能tm多写点自己写个教程吗.
算了, 闲槽少吐, 我们直接开始我们自己的流程.

1, 2 两步应该不用说了, 第 2 步我已经给出了 solcjs 的下载链接, solcjs 直接从 solc 编译器的 C++ 源代码用 Emscripten 生成的.

第 3 步 geth 节点是以太坊的官方 golang 实现, 当然是要用它.

第 4 步之所以我建议用 rinkeby 测试网络是因为第一私有网络搭建和维护相对麻烦, 要自己设计 genesis 状态, 自己在机器上起一到两个节点, 最关键的是市面上就没有完善的针对本地私有网络的区块浏览器, 这可会给调试带来很大的麻烦, 我用这种方式实践的时候创建了智能合约我都不知道进了哪个区块, 只能用得到的智能合约的地址先查出交易的哈希, 再用交易的哈希查处区块高度, 这些都是在 geth console 里敲命令完成的, 过程十分 annoying.

所以我们要选择测试网络, 那么问题又来了, 以太坊那么多测试网络, 为什么要选 rinkeby? 这是因为 rinkeby 网络用的是 PoA 共识而不是 PoW, 这样当你需要测试本地 geth 节点挖矿的话, 你的电脑风扇就不会因为 CPU 算哈希而狂转了.

另外, 当你不需要测试本地挖矿的话, geth 可以以 light 模式启动, 此模式下 geth 会变成一个轻钱包, 启动之后不需要看着那无休止的从 peers 下载历史区块数据的过程.

题外话, 关于 geth 的启动模式, 官方写文档的兄弟写了两句之后发现自己也不知道 light 是什么jb模式, 于是他可能 google 了一下, 找到了 stackexchange 上的这个回答:

https://ethereum.stackexchange.com/questions/11297/what-is-geths-light-sync-and-why-is-it-so-fast

遂直接在[官方文档](https://ethereum.org/en/developers/tutorials/run-light-node-geth/)上链接了这个回答:

![](answer.png)

对于这位兄弟我只想说你真他娘的是个小机灵鬼.

## 智能合约撰写和编译

直接用这个最简单的合约.

```
// SPDX-License-Identifier: GPL-3.0
pragma solidity >=0.4.16 <0.9.0;

contract SimpleStorage {
    uint storedData;

    function set(uint x) public {
        storedData = x;
    }   

    function get() public view returns (uint) {
        return storedData;
    }   
}
```

然后用 solcjs 编译, 我们需要两个编译产物, 分别使用 `solcjs --abi` 和 `solcjs --bin` 命令编译两次源代码, 得到的一是智能合约的 ABI, 一是智能合约的实际字节码.
ABI 的内容, 如我们所愿, 长这样:

```
[
  {
    "inputs":[],
    "name":"get",
    "outputs":[
      {
        "internalType":"uint256",
        "name":"",
        "type":"uint256"
      }
    ],
    "stateMutability":"view",
    "type":"function"
  },
  {
    "inputs":[
      {
        "internalType":"uint256",
        "name":"x",
        "type":"uint256"
      }
    ],
    "name":"set",
    "outputs":[],
    "stateMutability":"nonpayable",
    "type":"function"
  }
]
```

字节码就看不懂了:

```
60806040523480156100115760006000fd5b50610017565b61016b806100266000396000f3fe60806040523480156100115760006000fd5b506004361061003b5760003560e01c806360fe47b1146100415780636d4ce63c1461005d5761003b565b60006000fd5b61005b600480360381019061005691906100b7565b61007b565b005b61006561008b565b60405161007291906100f2565b60405180910390f35b8060006000508190909055505b50565b6000600060005054905061009a565b9056610134565b6000813590506100b081610119565b5b92915050565b6000602082840312156100ca5760006000fd5b60006100d8848285016100a1565b9150505b92915050565b6100eb8161010e565b825250505b565b600060208201905061010760008301846100e2565b5b92915050565b60008190505b919050565b6101228161010e565b811415156101305760006000fd5b505b565bfea26469706673582212208a4ba997696b94e2c3116f2250c36954c3250d29aa10ecf4aa485a568623cc7764736f6c63430008040033
```

## 合约的部署

合约的部署实际上也是一个交易, 既然是交易就要有人为此支付 Gas, 所以首先我们得有一个有 ETH 的账户, ETH 这点不用担心, rinkeby 测试网络已经帮我们考虑到了. 创建好账户后到[这里](https://www.rinkeby.io/#faucet)管 rinkeby 借点钱就行了

```
$ geth --rinkeby account new
```

有了钱以后, 具体怎么发送这个交易呢? 手动构造交易的每个字段当然是很麻烦的, geth console 里提供了个 Contract 对象可以帮我们构造创建合约的交易并广播出去. Contract 对象由 `eth.contract(abi)` 方法创建, 其参数 `abi` 是个 JSON Object, 也就是上面我们编译出来的 ABI.

```
> var simpleStorage = eth.contract([{"inputs":[],"name":"get","outputs":[{"internalType":"uint256","name":"","type":"uint256"}],"stateMutability":"view","type":"function"},{"inputs":[{"internalType":"uint256","name":"x","type":"uint256"}],"name":"set","outputs":[],"stateMutability":"nonpayable","type":"function"}])
```

得到了 Contract 对象后, console 里输入对象名, 可以看到 Contract 对象有如下的属性:

```
> simpleStorage
{
  abi: [{
      inputs: [],
      name: "get",
      outputs: [{...}],
      stateMutability: "view",
      type: "function"
  }, {
      inputs: [{...}],
      name: "set",
      outputs: [],
      stateMutability: "nonpayable",
      type: "function"
  }],
  at: function(address, callback),
  getData: function(),
  new: function()
}
```

`abi` 就是创建 Contract 对象时的输入, 给我们原样又展示出来了, `new` 方法就是用来产生创建实现了此 ABI 的智能合约的交易并广播出去的, `at` 方法则是在合约创建成功之后, 根据合约地址创建一个能够操作此合约的抓手(或者叫做, handle, 句柄或实例), 是为了方便在 geth 中能够直接调用智能合约而存在的.

那么接下来就让我们调用 `new()` 方法来创建这个合约,  顺便把 `new()` 方法的参数也提供了:

```
> var bytecode= "0x60806040523480156100115760006000fd5b50610017565b61016b806100266000396000f3fe60806040523480156100115760006000fd5b506004361061003b5760003560e01c806360fe47b1146100415780636d4ce63c1461005d5761003b565b60006000fd5b61005b600480360381019061005691906100b7565b61007b565b005b61006561008b565b60405161007291906100f2565b60405180910390f35b8060006000508190909055505b50565b6000600060005054905061009a565b9056610134565b6000813590506100b081610119565b5b92915050565b6000602082840312156100ca5760006000fd5b60006100d8848285016100a1565b9150505b92915050565b6100eb8161010e565b825250505b565b600060208201905061010760008301846100e2565b5b92915050565b60008190505b919050565b6101228161010e565b811415156101305760006000fd5b505b565bfea26469706673582212208a4ba997696b94e2c3116f2250c36954c3250d29aa10ecf4aa485a568623cc7764736f6c63430008040033"
> var deploy = {from: "0xc97c9e1f5f842e37544e7790cdda32fef0f8b688", data:bytecode, gas: 2000000}
> var handle = simpleStorage.new(deploy)
```

这里 bytecode 会随广播进入到矿工节点的区块中进而在所有其他节点中存下来, 这就是将来 EVM 要执行的代码.

上面 "0xc97c9e1f5f842e37544e7790cdda32fef0f8b688" 是我在 rinkeby 网络的地址, 并且已经通过 rinkeby faucet 要到钱了.

最后一步 `new()` 会广播交易, 交易成功之后我们的 geth 节点会收到成功的通知, 并将结果反映到 handle 上, 我们立即看一下 handle 的内容:

```
> handle
{
  abi: [{
      inputs: [],
      name: "get",
      outputs: [{...}],
      stateMutability: "view",
      type: "function"
  }, {
      inputs: [{...}],
      name: "set",
      outputs: [],
      stateMutability: "nonpayable",
      type: "function"
  }],
  address: undefined,
  transactionHash: "0x104fb78f40a7265f08cc2566f75707b5140003b264baba06a449497cac81992b"
}
```

可以看到 `transactionHash` 的值, 这个 hash 是广播的时候生成的, 所以自然是立即就能得到. 但是 address 还没有值, 这是因为我们立即交易还没被确认, 稍等 15s, rinkeby 上的矿工们会确认我们的交易, 这时在看:

```
> handle
{
  abi: [{
      inputs: [],
      name: "get",
      outputs: [{...}],
      stateMutability: "view",
      type: "function"
  }, {
      inputs: [{...}],
      name: "set",
      outputs: [],
      stateMutability: "nonpayable",
      type: "function"
  }],
  address: "0xc6c5500ab9dc7fcc7efb6c95033604a7f00a3042",
  transactionHash: "0x104fb78f40a7265f08cc2566f75707b5140003b264baba06a449497cac81992b",
  allEvents: function(),
  get: function(),
  set: function()
}
```

Bingo! 此处无须多言了吧.

etherscan.io 上有针对 rinkeby 的区块浏览器, 可以看到这次交易的详情: https://rinkeby.etherscan.io/tx/0x104fb78f40a7265f08cc2566f75707b5140003b264baba06a449497cac81992b

这要是本地私有网络, 就没有这么方便的浏览器可用了, 这是我花了一个多小时得出的经验.

## 合约的调用

我们可以直接用刚刚的 handle 调用 get/set 方法. 如果你是第二天重新进入的 geth, 则需要先构造出 Contract 对象, 然后再用 at 方法获得 handle.
