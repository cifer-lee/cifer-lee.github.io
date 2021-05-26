---
# Common-Defined params
title: "CSPRNG 如何做到不可预测"
date: "2021-05-10"
description: "随机数 伪随机数生成器 CSPRNG PRNG"
categories:
  - "Programming"
tags:
  - "CSPRNG"
# menu: main # Optional, add page to a menu. Options: main, side, footer

# Theme-Defined params
#thumbnail: "img/placeholder.jpg" # Thumbnail image
#lead: "Gold grade is a term used in gold mining, and should be used as a measure of the quality of gold ore – that is the raw material obtained from mining." # Lead text
---

[上一篇文章](http://cifer76.github.io/posts/prng/) 我们提到了实现伪随机数生成器 PRNG 的一种经典算法: 线性同余法. 我们也提到了线性同余法由于无法做到不可预测而不能成为密码学安全的算法, 如果对其加以改进使做到不可预测，我们就能够得到密码学安全的随机数生成器 CSPRNG. 本文我们就来看看线性同余法是如何被预测的以及 CSPRNG 是如何克服这一点的.

## 线性同余为什么是可预测的

线性同余的公示如下, A, C, M 是常数:

$$R_{n+1} = (A \times R_n + C) \bmod M$$[^1]

其中

$$R_0 = (A \times seed + C) \bmod M$$

我们说过不可预测的定义:

> 即使给出产生序列的算法或硬件和所有以前产生的位序列, 也不可能通过计算来预测下一个随机位是什么.
> 
> 应用密码学: 协议, 算法与 C 源程序》第 2 版[^2]

线性同余的算法很简单只是一个多项式, 其常数 A, C, M 一般是不公开的, seed 一般也不会让我们知道, 但是显然我们只要知道它生成的 4 个随机数, 就能反推出 A, C, M 以及 seed, 从而整个随机序列我们都能知晓.

## CSPRNG 如何做到不可预测

由上述可知线性同余的问题在于容易被反推, 所以首先我们要解决容易反推的问题, 如果做到不被反推呢? 不难想到单向散列函数, 单向散列函数可以被认为是不可破解的. 所以我们只需要在线性同余之后再串联一级单向散列, 就能防止攻击者从输出的随机序列反推出线性公式中的常数.

然而单纯串联一级单向散列函数就够了吗? 上篇文章我们还提到, PRNG 只是以假乱真, 它一定也是有周期的, 只是可能周期比较大而已, 在线性同余公式中有取模运算, 但不要以为没有取模的公式就能做到单调递增, 计算机是有限的, 当公式输出的数达到一定程度, 超出了寄存器或内存的空间的时候就一定会被截断. 因此即使串联了单向散列函数, 理论上供给者仍然能够穷举出所有序列.

为了解决此问题, CSPRNG 需要不断的改变 seed 来使得难以看出序列的周期性. 并且, CSPRNG 的 seed 往往来源于真随机源[^3], 上篇文章我们说过完全从真随机源获得随机数代价往往较高, 但是如果我们降低从真随机源获得随机数的频次, 只将其随机数用作 seed, 后续的序列全都给予这个真随机的 seed 来生成, 就同时获得了安全与效率.

Fortuna 是经典的 CSPRNG 的实现, 用的就是我们所说的思想:

https://en.wikipedia.org/wiki/Fortuna_(PRNG)


[^1]: https://book.douban.com/subject/26265544/
[^2]: https://book.douban.com/subject/25772389/
[^3]: https://crypto.stackexchange.com/questions/62035/prng-with-a-truly-random-seed
