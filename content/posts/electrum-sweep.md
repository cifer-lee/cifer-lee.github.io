---
title: Electrum 钱包的 sweep 功能小记
date: 2017-10-17
categories:
  - Bitcoin
tags:
  - Electrum
---

electrum 是款优秀的 bitcoin 轻钱包, bitcoin.org 的文章里经常有提到它, 可见 bitcoin.org 除了自己的 core 钱包之外还是比较推荐 electrum 的.

使用 electrum 钱包时, 建议的生成私钥的方式是从 seed 生成, 这样所有生成的密钥都是可以从 seed 推算出来的, 对钱包的备份也就简化为对 seed 的备份.

# sweep 功能

electrum 有一个 sweep 功能, 刚开始乍一看我惊诧的以为这个功能是销毁钱包, 因为这个功能就在 "Private keys -> sweep" 菜单下, 看着意思就像 "清除所有私钥" 一样, 实际上当然不是的, sweep 存在的目的是让我们把其它客户端生成的私钥导入到我们在 electrum 中使用 seed 生成的钱包中.

导入私钥本应该是个很简单直接的事情, 但是由于 seed 方式生成的钱包的特点就是里面的私钥都是可以由 seed 推导出来的确定性私钥, 随便导入一个我们在其它客户端生成的私钥自然是不行的. 因此就引入了这个 sweep 机制, sweep 的本质就是将你在其他客户端上所生成的私钥的所有余额发送到这个钱包中由 seed 生成的某个私钥上.

所以 "sweep" 指的是 sweep 别的客户端生成的私钥, 而不是 seed 钱包里的私钥.
