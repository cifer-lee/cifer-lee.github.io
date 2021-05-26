---
# Common-Defined params
title: "为什么 bid-ask spread 是做市商的潜在收益"
date: "2021-05-16"
description: "交易 做市商"
categories:
  - "Investment"
tags:
  - "market-marker"
# menu: main # Optional, add page to a menu. Options: main, side, footer

# Theme-Defined params
#thumbnail: "img/placeholder.jpg" # Thumbnail image
#lead: "Gold grade is a term used in gold mining, and should be used as a measure of the quality of gold ore – that is the raw material obtained from mining." # Lead text
---

关于做市商如何挣钱, 这篇文章说的不错: https://www.investopedia.com/terms/m/marketmakerspread.asp

说白了就是低买高卖, bid-ask spread 也就是买一和卖一的价差, 价差越大价差越大, marker 潜在的收益自然就越高. 但是上面文章有一句话没说清楚, 就是这句:

> For example, imagine that a market maker MM in a stock – let’s call it Alpha – shows a bid and ask price with a quote of $10.00 - 10.05. This means that this MM is willing to both buy Alpha shares for $10 and sell it at $10.05.

我查了好几篇文章都和这句话说得类似, 就是现在价差是 10.00 - 10.05. 那么 marker 只要 10.00 买入 10.05 卖出就能赚钱, 所以价差越大做市商赚的越多. 可问题在于, 最低 ask 10.05, marker 凭什么能 10.00 买到?

实际上这种情况做市商赚的只是市价单的钱, 而不是限价单的钱.

marker 的做法是, 看到市场有 0.05 的价差, 它会自卖自买快速成交一个 10.03 的小单, 这样市价就会变成 10.03. 买家看到 10.03 的价格决定买入, 实际上会以 10.05 成交, 因为卖一是 10.05. 这中间 0.02 就被做市商赚去了. 这样一来世价就变成了 10.05, 卖家看到 10.05 的价格决定卖出, 但当他以市价卖出时实际上成交价只是 10.00, 因为买一是 10.00. 这中间的 0.05 也被做市商赚去了. 虽然一单最多只赚 5 分, 但如果数量够多, 收益还是很可观的.

关键是做市商可以不断的自买自卖小单来 “控制” 价格蛊惑用户, 用户如果偏爱市价交易就会想这样被坑. 而且做市商之所以能够不断制造小单, 是因为做市商是被券商免收手续费的.
