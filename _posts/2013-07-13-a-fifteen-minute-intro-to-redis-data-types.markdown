---
layout: post
title: "15分钟让你熟悉Redis的数据类型"
date: 2013-07-13
tags: redis 翻译
---

想必你已经知道了，Redis不是一般的key-value类型的数据库，它实际上是一个支持多种数据结构的服务器，就是说，它支持存储多种类型的值。说得再通俗一点就是，给一个key，这个key对应的值可以不仅仅是string类型。下面的这些类型都可以作为值：

*   Binary-safe strings.
*   Lists of binary-safe strings.
*   Sets of binary-safe strings, 这是一种无序的，不重复的元素的集合。
*   Sorted sets，这个与Sets类似，但是这种有序集合类的每一个元素都有一个相关联的score。这些元素按照与其关联的
    score进行排序。
  
#Redis keys

Redis的键(*keys*)是二进制安全的(*binary safe*)的

#The string type

这是Redis里最简单的类型，如果你只用这种类型的话，那Redis就好像是具有持久存储功能的memcached。

尽管strings是Redis里基本的类型，但你可以在这个基本类型上做一些有趣的操作，其中之一就是原子自增(*automic increment*)。

#The List type

##First steps with Redis lists

##Pushing IDS instead of the actual data in Redis lists

    $ redis-cli incr next.news.id  
    (integer) 1
    $ redis-cli set news:1:title "Redis is simple"
    OK
    $ redis-cli set news:1:url "http://code.google.com/p/redis"
    OK
    $ redis-cli lpush submitted.news 1
    OK


#Redis Sets
