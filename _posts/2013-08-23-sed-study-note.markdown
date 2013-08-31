---
layout: post
title: 'sed 学习笔记'
date: 2013-08-23
---

sed 的语法这是样的：

```
sed [address[,address]][!]command [arguments]
```

```command```包含一个字母或符号

sed 可以指定0个，1个或2个addresses. 在POSIX sed中， address具有如下的形式。\n 可以匹配新行，但是结尾的新行除外。

