---
layout: post
title: 2013 年 9 月总结
date: 2013-09-30 00:00
published: false
---

### 这个月我获取到的新东西有: 

1. 用bash写了一个测试 yeelink 平台 api 接口的框架, 主要用到了 curl, 三种正则, bash 的替换规则
2. 排除了 [ -n string ]  与 [ string ] 这个坑
3. 学习了一些系统监控的命令的用法, 有 iostat, vmstat, top, uptime 等
4. 仔细的设计了微信公众号的 web 界面, 并上线了微信
5. 了解了使用 google chart
6. 修正了微信绑定过程中的一个严重的 bug, 涉及到 nodejs 的模块共享的问题, 详见记事本
7. 阅读了 _linux network administrator's guide 3rd edition_, 巩固了网络有关的知识
8. 修改了微信用户(粉丝)信息在 redis 中的存储结构, 修改为如下结构: 
        
         fakeid:{openid} => {fakeid}
         nickname:{openid} => {nickname}

9. arp 与 arp proxy 的概念, 了解
10. 很深刻的学习了 MVC 思想, 实现了自己的 php mvc 框架, 简约
11. 学习了 android 开发, 做了一个实验室用的小软件
12. 学习了 bootstrap

### 在计划中但是没有完成的: 

1. 用 bash 写 linux 性能分析与监控脚本, 并生成报表
2. 阅读 coolshell.cn 的浏览器渲染原理这篇文章
3. 了解比特币, 查什么是UGC
4. 买哑铃, 买梳子
5. 学习了解 ssh secure shell 的实现方式, 基于 bash shell 改装? 在bash shell 上又加了一层协议?
6. sed, awk, uniq ....
7. 正则表达式中 [ ] 里面的 -, + 等特殊字符的含义
8. 逆向工程, php?
9. 查为何 route -n 中 Use 字段一直是 0
10. 学会用 php 写系统脚本
11. windows, cmd 字符集的问题一直未解决
12. svn 服务器搭建
13. 了解 Flex
14. 人脸识别算法
15. 搭建类似于 pull request 的平台
16. 用 bash 写装机脚本
17. 理解多核处理器, SMP 相关的概念
18. 翻译 Tom 的两篇 MVC 的文章

### 未解之谜: 

1. artTemplete 渲染 date_timestamp 的问题, 详见记事本
2. 为什么 iostat 的输出中, util 值不高的情况下 await 很大, svctm 很小, avgqu_sz 也不大
3. top 的输出中 VIRT, SHR 两字段的含义

其它的琐事: 

1. 计划这个月开始用牙线, 已经开始了
2. 这个月开始坚持每晚做俯卧撑, 锻炼胸肌
