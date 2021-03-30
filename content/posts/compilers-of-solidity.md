---
title: Solidity 的编译器们
date: 2017-08-20
categories:
  - Blockchain
tags:
  - Ethereum
  - Solidity
---

solidity 是当下 ethereum 最流行的智能合约开发语言, 语法类似于 javascript. 要使用它写智能合约的话, 我们还需要一个编译器, 用来将 solidity 的代码编译为 EVM 字节码.

solidity 的编译器有多种实现, 下面可以一起看一看.

# solc

一般说到 solc 指的都是 ethereum 官方实现的 cpp 版本的 solidity 编译器, 其项目源代码位于 https://github.com/ethereum/solidity. 按照我以往对编译器的印象, 本以为 solc 一定也是像 gcc 那样的庞然大物, 结果发现完全不是, solidity 编译器实际上是个体积很小很简单精巧的工具, 项目源代码算上注释总共也就才 6w 多行. 我们可以克隆这个仓库, 然后自行构建它, 构建出来的主要目标文件就是 `solc`.

# solcjs

solidity 的官方文档中还介绍了使用 npm/nodejs 安装的方法. `npm install -g solc` 之后会得到一个 solcjs 命令, 本来我以为这个 solcjs 实际上是 solc 的前端, 查了一下发现完全不是的: https://github.com/ethereum/solidity/issues/725#issuecomment-233227534, `solcjs` 本身就是整个 solidity 编译器, `solcjsi` 项目代码在 https://github.com/ethereum/solc-js, 这个项目是借助 `Emscripten` 工具直接把 cpp 的代码重写成了 javascript 代码, 于是得到了一个完整的 javascript 版的 solidity 编译器, 我们可以在 nodejs 中直接 `require ('solc')` 使用它.

# Remix

既然编译器都能在 nodejs 里跑了, 那在浏览器里跑也是没问题的了, 没错这就是 Remix 了. Remix 就是一款 web 版的在线 solidity 编译器, 无需下载即可使用. 不过估计它也是直接用了 solcjs 吧.
