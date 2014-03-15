---
layout: post
title: 关于 openvz 的一切
date: 2014-03-14 14:53
---

# 概念

## Container

每一个容器可以承载一个系统

在 OpenVZ 中, 你不能够在 VE 里加载模块, 你必须在 HN 中加载模块, 然后给予 VE 访问权限.
