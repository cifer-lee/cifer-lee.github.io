## Algorithms & Programming

1. 二叉树 Z 字输出每层节点 - 2019 抖音
2. 从有序单链表构造平衡二叉搜索树 - 2019 抖音
3. 外部排序 - 2017 阿里
4. 如何判断本机字节序 - 2017 滴滴

   > int i = 100;
   >
   > char *p = &i;
   >
   > 打印 p[0],p[1],p[2],p[3]

5. 提取 nginx log 中 url 访问 top20 - 2017 全民快乐

   > awk print "{$0}" | sort | uniq -c | sort | head -n 20

6. Singleton design - 2018 融 360, 腾讯视频
   1. prevent construction (private constructor)
   2. thread safe

7. 有序数组中找 target, 找不到返回 -1, 考虑数组元素全部相等的情况 - 2018 融 360
8. 两个有序数组, 求交集, O(n)  - 2018 融 360
9. 一个有序字符数组, 找出第一个只出现一次的字符 (计数法) - 2018 融 360
10. 外部排序, 100T 数据 字符串, 求出现次数最多的字符串 - 2018 融 360
11. 两个大文件, 包含很多数, 内存装不下, 找出两者共同包含的数 - 2018 网心
    1. hash 分区成小文件块, 分区比

12. 整数数组, 字典排序 (重点在字典树的构造) - 2018 头条懂车帝
13. 链表中是否存在环  - 2018 头条懂车帝
14. 设计一个短网址服务 (要点是 hash 存储, redis 分布式存储, proxy 根据 hash 前缀分布式存储 hash key) -   2018 头条懂车帝
15. 设计一个 Path 类 (实现 mkdir, cd, ls 方法) (要点是 Trie 存储结构, 以及当前目录的维护) - 2018 头条懂车帝
16. 二叉树，计算任意两个节点之间的距离 - 2018 头条懂车帝
17. C 语言的字节对齐问题, #progam 声明对其字节, struct 结构体占用空间 - 2018 头条懂车帝
18. 字符串，分割符，返回 token 数组  - 2018 腾讯视频
19. 积蓄雨水, 不同高度的墙, 能存多少雨水. 抽象成一个数组, 不同大小的元素 - 2018 腾讯视频
20. 一个数组，给一个 sum，找出数组中和为 sum 的两个元素的索引，题目保证有唯一解。 O(N^2) 和 O(N) 的方法各想一种 - 2018 腾讯 PCG
21. 快排, 二分查找 - 2018 和风畅想
22. 两个逆序排好序的数组 找出第 k 大的数 (要求 logN) - 2018 头条 效能
23. 设计一个评论系统, 考虑瞬时大量评论, 考虑分页查询  - 2018 头条 效能
24. 旋转过的数组 找出指定 target - 2018 格步
25. 二叉树右视图 - 2018 格步
26. LRU 实现 - 2018 格步

## System Design Principles

* KISS

## Networking

### TCP / IP

* Connection establishment: three-way handshake
* Connection termination: four-way handshake
* why TIME_WAIT?
* sliding window

### IPC (Inter-process communication)

* shared memory
* memory mapped file
* pipe (named or anonymous)
* Unix domain socket
* parent children process (what will happen to the child process if parent kill child process?)

### epoll

* differences with `select`

### Process profling

* netstat
* lsof
* strace
* pprof
* /proc or /sys

## Web

* HTTP protocol
  * what the meanings of 1xx/2xx/3xx/4xx/5xx
  * How to determine a http response ended (https://blog.csdn.net/gaoyingju/article/details/9223979)
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
    * 内核 reuse_port 出来以后: https://www.nginx.com/blog/socket-sharding-nginx-release-1-9-1/)
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

    - InnoDB (实现为聚集索引,     支持事务, 支持行级锁)
    - MyISAM (非聚集索引,     支持全文索引, 只支持表级锁)
    - 不要使用存储过程, 外键,     触发器. [滴滴的 mysql 存储规范](onenote:编程.one#滴滴的 mysql 存储规范&section-id={A9A445F6-738E-0C42-8C57-5627B99B856D}&page-id={BABCC1A7-BD61-9849-A381-EFF382B4534C}&end&base-path=https://d.docs.live.net/2850108539a65c50/文档/Cifer 的笔记本)

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

* redis 的两种持久化方式。 快照方式如何实现？假如做快照时有个耗时 5s 的写操作，快照是做 5s 前的还是 5s 后的？5s 前的话怎么做 ？要把5s前的内容拷贝一份以防止耗时操作把内容修改了吗？（答案是 5s 前，用写时拷贝原则）

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
