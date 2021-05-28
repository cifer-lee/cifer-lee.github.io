---
title: 通用网关协议 (CGI) 进化史 番外篇
slug: common-gateway-interface-evolution-4
date: 2014-07-11 16:00
categories:
  - Web
tags:
  - CGI
  - SCGI
  - FastCGI
  - WSGI
---

## WSGI

在 python 大红大紫的今天, 除了 FastCGI, 我们听到的最多的可能就要数 WSGI 了, 那么 WSGI 又是啥呢? 它和 CGI, SCGI, FastCGI 又有什么关系呢?

你应该能猜得到, 既然 WSGI 也归到了 "通用网关协议 (CGI) 进化史" 这一系列里面, 那么 WSGI 和 CGI 肯定也是有点关系的.


WSGI 是专为 python 设计的协议, 其包装了 FastCGI 协议, WSGI server 一般也都能够作为 SCGI server, 或者 FastCGI server 运行. 

如果说 FastCGI 传递信息时用的是一种底层字节流的形式, 那么 WSGI 就是将这字节流结构化为 python 对象, 这使得在 python 中进行 web 开发时, 你只要写一个文件上传的表单, 通过 WSGI 你就能直接获得这个文件的对象, 而不必自己去读取 HTTP 请求体来接收上传的文件.

Python 养了很多的框架, 如 Zope, Quixote, Webware, SkunkWeb, PSO 以及 Twisted Web. 如此之多的框架对于 python 用户来说可能反倒是个问题, 因为一般情况下这些框架并不是适配于所有的 webserver, 于是选择某个框架也就意味着你相应的只能使用某几个 webserver, 反之亦然.

然而, 瞧瞧 java, 尽管 java 拥有很多的网络应用程序开发框架, 然而所有的框架以及所有的 java 容器都遵循同一个标准: servlet API. 遵循 servlet api 的框架如 struts, spring, hibernate, 而容器则有 tomcat, jboss 等.

如果在 python 中也存在这样的 api, 那么在 python 开发中, 框架的选择与 webserver 的选择就也可以互不干扰了.

因此, 一种简单通用的接口被定义出来, 旨在统一 webserver 与 web application (或者是框架) 直接的通信接口: the Python Web Server Gateway Interface (WSGI).

## webserver 对 WSGI 的支持

### Apache

apache 里有一个模块叫做 mod\_wsgi, 是一个第三方模块, apache 通过这个模块提供对 wsgi 的支持. 这个模块既能够运行在 embeded 模式下, 也能够运行在 daemon 模式下. 这里我们讨论他运行在 embeded 模式下的情况.

在 embeded 模式下, mod\_wsgi 的配置与 mod\_cgi 的配置十分相似, 首先, 我们创建一个 wsgi 脚本, 叫做 myapp.wsgi:

    def application(environ, start_response):
        status = '200 OK'
        output = 'Hello World!'

        response_headers = [('Content-type', 'text/plain'),
                        ('Content-Length', str(len(output)))]
        start_response(status, response_headers)

        return [output]

可以将其放在 `/usr/local/apache/wsgi-bin` 目录下 (当然你可以随便放, 但要保证相关的权限, 还记的 cgi 脚本放在 cgi-bin 目录下吗?), 这里定义的 callable 的名字是 "application", 这是 WSGI 协议里规定的, 必须就叫这个名字.

然后, 还记得在使用 mod\_cgi 模块时, 指定 cgi 脚本所使用的 ScriptAlias 指令吧? 现在要制定 WSGI 脚本, 我们需要使用 WSGIScriptAlias 指令, 在 apache 的配置文件添加如下这句:

    WSGIScriptAlias /myapp /usr/local/apache/wsgi-bin/myapp.wsgi

打开浏览器, 访问 `http://localhost/myapp`, 就能看到你的 wsgi 脚本的运行结果了.

当然, `WSGIScriptAlias` 指令也是可以这么用的:

    WSGIScriptAlias /myapp /usr/local/apache/wsgi-bin/

这样一来, wsgi-bin/ 整个目录下的文件都能够被当作 wsgi 脚本运行, 这点和 mod\_cgi 模块的 ScriptAlias 指令是一致的.

关于 mod\_wsgi 模块还有很多强大的功能, 详情请看参考2.

#### mod\_wsgi embeded 与 mod\_cgi 的区别

相比 FastCGI, SCGI, WSGI 与 CGI 应该是最像的, 不过当然也是有区别的. 

一方面 WSGI 提供了更加丰富的指令, 第二, wsgi 程序并不是另起一个进程执行的, 而是和 apache 处在同一个进程里 (和 apche 的 worker process 处于同一进程). 这一点通过上面的 wsgi 脚本其实也能看出来, 因为上面的 wsgi 脚本里第一行没有 shebang, 这就意味着不需要调用一个外部程序去执行这个脚本.

#### mod\_wsgi 的另一种模式

刚才说了 mod\_wsgi 还可以运行在 daemon 模式下, 在 embeded 模式下, wsgi 脚本改变了就需要重启 apache 服务器, 而在这种模式下, daemon 进程能够监控 wsgi 脚本的改变, 一旦脚本改变 daemon 脚本就回重启, 而不需要重启 apache.

在这种模式下, mod\_wsgi 与 fastcgi 的方式更像了一些, 但是 mod\_wsgi 的 daemon 进程与 apache worker 进程是父子进程的关系 (我没有查资料, 我是靠 mod\_wsgi daemon 模式下的一个配置指令: SetENV 猜测出来的), 而不是通过什么 socket 通信的, 这点与 fastcgi 不一样.

关于这种模式的详情可以参见 参考1, 参考2.

#### mod\_wsgi 的新一代

mod\_wsgi 是个很不错的项目, 这个项目持续了很久, 感谢它的维护者们, 这个项目从 Google Code 已经迁移到了 Github 上, 开启了新的篇章.

上面所介绍的关于 wsgi 的内容, 全部都是 mod\_wsgi 的开发者们在 Google Code 上就已经做好的了事情, 当项目迁移到 Github 之后, 开发者们又开发了新一代的 mod\_wsgi.

最新的一代 mod\_wsgi 已经不必安装为 apache 的模块了, 它可以直接安装到你的 python 中, 并且能够作为独立的 http server 启动 (好像还是调用了 apache 的可执行程序, 我没有深究, 不重要了), 它有一个新的名字: mod\_wsgi-express.

## 参考

1.  mod\_wsgi 官方的快速配置文档, 强烈推荐: http://code.google.com/p/modwsgi/wiki/QuickConfigurationGuide
2.  mod\_wsgi 官方的详细配置文档, 强烈推荐: http://code.google.com/p/modwsgi/wiki/ConfigurationGuidelines
3.  mod\_wsgi 项目新生代, Github 主页: https://github.com/GrahamDumpleton/mod\_wsgi
4.  PEP 333, WSGI 的标准文档: http://legacy.python.org/dev/peps/pep-0333/#specification-overview
5.  这篇主题很好的介绍了 FastCGI 和 WSGI 的区别: http://stackoverflow.com/questions/1747266/is-there-a-speed-difference-between-wsgi-and-fcgi
6.  这篇主题的作者经历很不错, 提问的很广泛, 回答者们的答案虽然都得票不多, 胆答得都很值得一看: http://stackoverflow.com/questions/2532477/mod-cgi-mod-fastcgi-mod-scgi-mod-wsgi-mod-python-flup-i-dont-know-how-m
7.  这个主题, 回答者很精彩的介绍了 WSGI, CGI 等的关系: http://stackoverflow.com/questions/219110/how-python-web-frameworks-wsgi-and-cgi-fit-together
