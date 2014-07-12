---
layout: post
title: "通用网关协议 (CGI) 进化史 中篇"
date: 2014-07-07 13:34
---

CGI 由于前面提到的性能问题, 越来越无法满足大多数网站的要求. 于是, FastCGI 和 Simple CGI 出现了.

# Simple CGI (SCGI)

Simple CGI 简称 SCGI, 和 FastCGI 一起, 是为了解决原始 CGI 的性能问题而出现的. 它们的解决方式和 "将脚本程序解释器嵌入 webserver (如 mod\_php, mod\_python" 不同, 它们的解决方式是创建一个 long-running 的后台进程, 以处理 webserver 的 forward 过来的请求.

当然, webserver 上仍然需要实现 FastCGI 或者 SCGI 协议的, apache 有 mod\_fastcgi/mod\_fcgid, lighttpd 也有 mod\_fastcgi 和 mod\_scgi.

SCGI 和 FastCGI 基本是一样的, 除了 SCGI 比 FastCGI 更容易实现 --- 正如其名字所暗示的那样.

下面看一下在各个 webserver 对 SCGI 的支持情况与配置方式.

## Apache

最初, Apache 的模块 mod\_scgi 负责实现 scgi 协议, 这个模块不是 Apache 自己开发的, 似乎是 python 用户组开发的, 因为官网就是 python 站上 (参考1). 这个模块在 Apache 2.0+ 上可用, 是非常稳定的模块. 可是现在其开发似乎停滞了, 可能是因为 Apache 上 SCGI 用的少, 而且 mod\_proxy\_scgi 模块出来的原因吧.

mod\_proxy\_scgi 模块是相对较新的模块, 是被包含在 Apache 源代码里的, 内建的模块. 

mod\_scgi 模块的配置大概如下 (详见 参考5):

    # (This actually better set up permanently with the command line
    # "a2enmod scgi" but shown here for completeness)
    LoadModule scgi_module /usr/lib/apache2/modules/mod_scgi.so

    # Set up a location to be served by an SCGI server process
    SCGIMount /dynamic/ 127.0.0.1:4000
    The deprecated way of delegating requests to an SCGI server is as follows:

    <Location "/dynamic">
        # Enable SCGI delegation
        SCGIHandler On
        # Delegate requests in the "/dynamic" path to daemon on local
        # server, port 4000
        SCGIServer 127.0.0.1:4000
    </Location>

mod\_proxy\_scgi 模块的配置大概如下 (详见 参考6):

    ProxyPass /scgi-bin/ scgi://localhost:4000/
    <Proxy balancer://somecluster>
        BalancerMember scgi://localhost:4000
        BalancerMember scgi://localhost:4001
    </Proxy>

前面说了, SCGI 的解决方式是创建一个 long-running 的进程, 或者监听某个 TCP/IP 端口, 或者监听一个 unix 套接字, 以便于和 webserver 通信. 那么现在 webserver 相当于 scgi 客户端, 我们现在还缺少一个 scgi 服务器端.

从上面的 mod\_scgi 和 mod\_proxy\_scgi 的配置也可以看出, SCGI 协议里是还需要一个 scgi 服务器端的.

SCGI 的服务器端是和语言相关的 (与 FastCGI 一样), 不同的语言有不同的实现, 参考8 介绍了这一点.

注: 下面的内容对 FastCGI 也是适用的

如最古老的 CGI 协议一样, SCGI 服务器会将请求递交给它的子进程, 子进程会去执行实际的任务. 不同的地方是, 子进程完成任务后不会退出, 而是 sleep, 等待下一个请求的到来.

SCGI 的另一个好处是, 它不必非得和 webserver 处在同一个机器上.

## Lighttpd

Lighttpd 自带着 mod\_scgi 模块, 其配置方式与 Lighttpd 的 FastCGI 配置方式一样, 可参考 Lighttpd 的 FastCGI 部分.

## Nginx

Nginx 也有自己的 ngx\_http\_scgi\_module 模块以支持 scgi.

# 参考

1. SCGI 的官方网站, 不明白为何是放在 python 官网上的: http://python.ca/scgi/
2. 维基页, 讲得很少: http://en.wikipedia.org/wiki/Simple\_Common\_Gateway\_Interface
3. 强烈推荐: https://docs.python.org/2/howto/webservers.html
4. 讲了 Apache 中的两种 scgi 的模块, 推荐: http://alesteska.blogspot.com/2012/07/scgi-in-apache-http-server-and-in.html
5. 讲了 Apache 中的 mod\_scgi 模块的配置: http://quixote.python.ca/scgi.dev/doc/guide.html
6. Apache 官网上关于 mod\_proxy\_scgi 的介绍: http://httpd.apache.org/docs/trunk/mod/mod\_proxy\_scgi.html
7. http://woof.sourceforge.net/woof-ug/\_woof/docs/ug/apache\_scgi
8. 吐血推荐, 不过这文章有四页, 只看前两页即可: http://www.linuxjournal.com/article/9310
