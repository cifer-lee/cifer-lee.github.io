---
layout: post
title: "通用网关协议 (CGI) 进化史 下篇"
date: 2014-07-11 16:00
---

# FastCGI

FastCGI 和 SCGI 是比较类似的, 但 FastCGI 明显要比 SCGI 流行很多, 可以看一下参考1 中维基页面, 实现 FastCGI 的 webserver, 以及语言 binding 明显要比 SCGI 的维基页中介绍的多.

FastCGI 也有自己的官方网站, 其官方网站可以看出 FastCGI 的一系列优点 (这些优点对 SCGI 也适用):

* FastCGI 够简单, 它实际只是 CGI 以及一些扩展.
* 如 CGI 一样, FastCGI 也是和语言无关的. 
* 如 CGI 一样, FastCGI 进程与 webserver 的进程是隔离的, 这相比模块化的方案来说, 提高了安全性. (mod\_perl, mod\_php 这样的方案, 如果模块中有 bug, 会导致 webserver 受影响)
* 如 CGI 一样, FastCGI 并不是与 webserver 的架构结合在一起的, 而模块化的方式, 是与 webserver 的架构结合在一起的.

当然, 除了兼具 CGI 的好处, FastCGI 还具备以下两点主要好处:

* 分布式计算: FastCGI 进程不必和 webserver 运行在同一台机器上.
* 多角色: CGI 脚本能够控制某一个 HTTP 请求的响应值, 而 FastCGI 能做的事情更多, 比如 HTTP 的验证机制.

按照惯例, 我们看一下在各个 webserver 里配置 FastCGI 的方式.

## Apache

(这一节的内容可以参见参考2)

Apache 最早通过 mod\_fcgid 实现的 FastCGI, 这是一个第三方的后来也被 ASF 组织承认的模块. 不过, 它只支持 unix socket 模式, 不支持 tcp socket.

另一个第三方的模块是 mod\_fastcgi, 曾经不能很好的被编译为 apache 2.4.x 的模块, 后来被修复了.

apache 2.4.x 推出了另一个 mod\_proxy\_fcgi 模块, 这是最新的支持 FastCGI 的模块, 类似于 mod\_proxy\_scgi.

至于这三个模块的配置文件怎么写, 就不在这里多费口舌了, 自行 google 之.

## Lighttpd

自己带着 mod\_fastcgi 模块

## Nginx

也有自己的 ngx\_http\_fastcgi\_module 模块

# FastCGI Server

这才是最重要的部分吧, 和 SCGI 一样, FastCGI server 的是实现也是和语言相关的. 因为我对 PHP 最为熟悉, 所以从 PHP 开始说起.

## PHP

PHP 可以作为其它 webserver (如 apache) 的一个模块工作, 也可以工作在 CLI 模式下. 当然我们这里要说的是 CGI 模式, PHP 有可以以两种方式工作在 CGI 模式下, 一是开箱即用的 php-cgi, 另一是 php-fpm.

### php-cgi

php-cgi 是开箱即用的, 当你编译完了 php 源代码, php-cgi 也就编译完了. 

### php-fpm

php-fpm 是更强大的 FastCGI 实现, 它实际上是包装了 php-cgi, php-cgi 启动后就一个进程, 而 php-fpm 则是能够根据负载量动态的创建/销毁进程 (叫做 workder 进程), 而且在创建这些 workder 进程的时候, 是可以指定不同的用户, 用户组的, 这都是单纯的 php-cgi 不能实现的.  
php-fpm 最开始作为一个第三方的实现, 目前已经被包含到 php 核心里了, 现在编译 php 源码, 默认就会编译 php-fpm.

### spawn-fcgi + run\_php

实际上, 还有第三种方式 --- spawn-fcgi, 之所以上面只说 php 有两种运行在 FastCGI 的方式没有包括它, 是因为 spawn-fcgi 并不是专用于 php 的, 它还可以和别的 fastcgi 程序配合使用, 比如可以和 rails 搭配使用.

spawn-fcgi 先前是 lighttpd 的项目, 现在独立了出来, 它的作用就是将别的 fastcgi 程序进程化, 通过它的调用方式你就能猜测到它实际上所干的事:

这是 spawn-fcgi 的 Lighttpd 官网文档里, 启动 php-cgi 的脚本, 脚本的名字叫 run\_php:

    #!/bin/sh
    # Use this as a ./run script with daemontools or runit
    # You should replace xxx with the user you want php to run as (and www-data with the user lighty runs as)

    exec 2>&1
    PHP_FCGI_CHILDREN=2 \
    PHP_FCGI_MAX_REQUESTS=1000 \
    exec /usr/bin/spawn-fcgi -n -s /var/run/lighttpd/php-xxx.sock -n -u xxx -U www-data -- /usr/bin/php5-cgi

spawn-fcgi 还有一个脚本的名字叫 run\_rails, 还有一个叫 run\_generic, 你能够猜出它们的作用是什么. 具体可以见参考3.

最后, FastCGI 官网上有关于 php-fpm, php-cgi, spawn-fcgi 的比较, 写的非常好. 可以看一看.

[未完待续...]

# 参考

1.  http://en.wikipedia.org/wiki/FastCGI
2.  FastCGI 官网: http://www.fastcgi.com/drupal/node/2
3.  spawn-fcgi 项目官网: http://redmine.lighttpd.net/projects/spawn-fcgi/wiki
4.  FastCGI 官网上, 关于 php-fpm, php-cgi, spawn-fcgi 的比较, 强烈推荐: http://php-fpm.org/about/
5.  一篇中文文章, 介绍了 php-cgi, php-fpm, spawn-fcgi, 还不错: http://www.joyphper.net/article/201310/237.html
