---
layout: post
title: "阿里云服务器配置日志"
date: 2013-07-31
---

1. 配置4台mqtt server

* 将我的测试代码mqtt-server.js通过scp传到每个mqtt server的/home/cifer目录中
* 在每个mqtt server的/home/cifer目录下运行npm install mqtt redis

2. 配置1台redis server

* apt-get install redis-server
* 修改配置文件/etc/redis/redis.conf，注释bind 127.0.0.1，在它下面重新bind redis server的内网地址
* 运行redis-server /etc/redis/redis.conf启动redis server

3. 配置2台mqtt client

* scp mqtt-subclient.js mqtt-test.js到两台mqtt client上
* npm install mqtt
