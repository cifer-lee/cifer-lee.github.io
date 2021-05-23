---
title: "技能树"
menu: main # Optional, add page to a menu. Options: main, side, footer
toc: false
---

## System Design Principles

* KISS

## Networking

### TCP / IP

* Connection establishment: three-way handshake
* Connection termination: four-way handshake
* why TIME_WAIT?
* sliding window
* MSS negotiation

### IPC (Inter-process communication)

* shared memory
* memory mapped file
* pipe (named or anonymous)
* Unix domain socket
* parent children process (what will happen to the child process if parent kill child process?)

### epoll

* differences with `select` and `poll`

### Process profling

* netstat
* lsof
* strace
* pprof
* /proc or /sys

## Web

* HTTP protocol
  * what the meanings of 1xx/2xx/3xx/4xx/5xx
  * How to determine a http response's ending (https://blog.csdn.net/gaoyingju/article/details/9223979)
    * Content-length
    * server closes the tcp connection
    * 1xx, 204, 304 response has no body
    * Transfer-Encoding:chunked，则通过chunk size判断长度
* Load balancing
  * Network layer or application layer?
  * AWS ELB
* Nginx
  * What does nginx 502/504 mean?
  * How nginx workarounds the `TIME_WAIT` state during restart without being blocked by `TIME_WAIT` connections. i.e. smooth restart.
  * How nginx workers listen the same ip:port?
    * 内核 reuse_port 出来以前: https://stackoverflow.com/questions/670891/is-there-a-way-for-multiple-processes-to-share-a-listening-socket
    * 内核 reuse_port 出来以后: https://www.nginx.com/blog/socket-sharding-nginx-release-1-9-1/
* PHP
  * Internals
    * Autoload
    * how array structure is implemented in php
    * how to simulate concurrent in php
    * Opcache / apc
    * are all processes in php-fpm share the same opcache and apcu?
    * how apcu is updated by the fpm processes
  * CGI / FastCGI
  * fpm
    * what's in php slow log?
* Python
  * GIL (Global Interrupt Lock)

## Database

### Mysql

* Cluster

  * Master-slave replica
  * read / write separation
  * database sharding / table sharding

* Index

  * MyISAM vs Innodb

  * B/B+ 树，聚簇索引，复合索引

    * 聚集索引: 索引节点存的就是数据.     InnoDB 的聚集索引一般就是主键索引.
    * 非聚集索引: 索引和数据分开存,     索引节点存的是数据的逻辑指针
    * 联合索引:     思考联合索引结构是怎样的? https://bbs.csdn.net/topics/390985501
    * 次级索引: 这个术语是相对于     InnoDB 的聚集索引来说的, 聚集索引之外的所有索引 (包括联合索引) 都算作次级索引

    > *在**innodb**，每个次级索引都存储的主键**key**。因为**innodb**的数据结构决定了，次级索引存储的是主键的值。次级索引的叶子节点并不存储行数据的物理地址。而是存储的该行的主键值。因此，换句话说，主键扮演了行数据的指针。*

    > *这是让我们推导另一个有趣的推论*

    > *一次级索引包含了两次查找。一次是查找次级索引自身。然后查找主键（聚集索引）*

    - InnoDB (实现为聚集索引, 支持事务, 支持行级锁)
    - MyISAM (非聚集索引, 支持全文索引, 只支持表级锁)
    - 不要使用存储过程, 外键, 触发器. [滴滴的 mysql 存储规范](onenote:编程.one#滴滴的 mysql 存储规范&section-id={A9A445F6-738E-0C42-8C57-5627B99B856D}&page-id={BABCC1A7-BD61-9849-A381-EFF382B4534C}&end&base-path=https://d.docs.live.net/2850108539a65c50/文档/Cifer 的笔记本)

* Performance

  * table lock / row lock / column lock?
  * SQL clause performance analysis: `explain` that clause

### DynamoDB

### Aurora

### Locking

* Optimistic lock vs Pessimistic lock
  * 乐观锁与悲观锁: http://www.open-open.com/lib/view/open1452046967245.html. 乐观锁的实现: 记录版本, 读的时候记录一下版本, 写的时候对比记录的版本和库里实际的版本, 如果库里版本比读的版本新, 说明被修改过. 那本次写操作就回滚 (感觉这不能称之为锁... 其实乐观锁的本名叫做乐观并发控制, 这个名字更恰当)

### Database cache policy

* Write-through
* Read-write back

### Storage as service

* storage as service, not just CRUD, like TAO in facebook

## Cache

### redis

* Basics, data structures
  * String
  * List
  * Set
  * Hmget / Hmset
* redis 的两种持久化方式. 快照方式如何实现? 假如做快照时有个耗时 5s 的写操作, 快照是做 5s 前的还是 5s 后的? 5s 前的话怎么做? 要把5s前的内容拷贝一份以防止耗时操作把内容修改了吗? (答案是 5s 前，用写时拷贝原则)
  * AOF 刷写模式:
    * appendfsync always, 每次收到写命令就立即强制写入磁盘，是最有保证的完全的持久化，但速度也是最慢的, 一般不推荐使用.
    * appendfsync everysec, 每秒钟强制写入磁盘一次，在性能和持久化方面做了很好的折中，是受推荐的方式。
    * appendfsync no, 完全依赖OS的写入，一般为30秒左右一次，性能最好但是持久化最没有保证，不被推荐
* Cluster
  * https://redis.io/topics/cluster-spec
* Redlock (Distributed locks with Redis)

## Message Queue

* Kafka
* AWS SQS

## Microservice

### gRPC / HTTP2

* difference with HTTP

## Online Service Issues Triaging

* CPU - pprof
* Memory - mem prof

### Tools

* Datadog
* Kibana logs
* Lightstep
