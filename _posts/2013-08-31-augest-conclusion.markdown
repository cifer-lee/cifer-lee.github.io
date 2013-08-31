---
layout: post
title: "八月总结"
date: 2013-08-31
---

不知不觉一个月又过去了，又到了月末总结的时候，首先还是把这个月的工作和学到的东西罗列一下

* 我了解到了linux对于tcp连接数的限制问题，linux除了对用户可打开的进程数，文件数有限制，也对本机所能使用的端口数做了限制，
这项限制exist in /etc/sysctl.conf, ipv4.local_port_range

* 对javascript的闭包以及垃圾收集机制有了更深入的了解

* 熟悉了linux下的socket编程

* 深入理解了linux下的阻塞，异步编程模型

* 开发了微信公众帐号的后台，学会了如何做微信公众平台开发

* 学会了如何编写网页模拟登录的程序

* 了解了nodejs模块化编程，javascript面向对象编程

* 对PHP的Session扩展有了更深的认识，也对session和cookie本身有了更深的认识

* 学习了javascript模板引擎这个东西

* 学习了nodejs的模块加载机制，只要模块的搜索路径相同，模块不会被重复加载 

* 学习了bash shell

* 学习了google maps api


还有，以下是这个月所遇到的问题和解决方法，以及一些小tips

* javascript中，判断一个变量是否为数字/整数
```
    function isNumeric(testValue) {
        return !isNaN(parseFloat(testValue)) && isFinite(testValue);
    }
    
    function isInteger(testValue) {
        return !isNaN(parseInteger(testValue)) && isFinite(testValue);
    }
```
* redis 也是单线程的

* date --set=STRING 用来设置日期， STRING可以是"next day"这样的humanble的形式

* nodejs的mqtt库的实现：客户端调用client.end()时，客户端发送一个disconnect消息给服务端，然后客户端断开TCP连接(通过FIN)

* 微信公众平台不够安全，别人只要知道了公众帐号的token，就可以伪装成微信服务器来给我们的公众帐号发情求

* 使用nodejs模拟http post请求时，一定要注意请求体中有中文的问题，请求体中有中文时，请求头中的 Content-Length字段不要简单的
求字符串的长度，因为http协议里计算请求体长度时用的单位是字节，而javascript计算字符串长度时用的单位是字符。可以利用nodejs的Buffer
类来解决这个问题

* vim对用单引号一起来的{}的语法处理处理的不好，vim不能将其识别为引起来的字符串，而是仍然将它看作代码块的结束，
所以使用 vim的语法格式化、自动缩进功能时，{}要用双引号引起来

* 即使我有8G内存，swap分配的比8G大还是有必要的，特别是我很需要休眠功能，我的电脑一般能开半个月不关机的，不用的时候都是
休眠，而休眠是要吃swap空间的，如果有一天我碰瞧把8G内存用了7.5G.。。。所以说，swap大点还是有必要的

* nodejs中，模拟http post请求时，如果想模拟表单请求，那么请求头最好写上 Content-Type: x-www-form-urlencoded，否则就算
你把请求体写成了 name1=value1&name2=value2 的形式，php的$_POST全局数组也不会被populated

* nodejs的writeSync函数的第二个参数是Buffer对象！不是字符串！

* dia 命令行默认以--integrated 参数启动(这启用了选项卡窗口)，这个参数似乎会导致ibus不可以用， 用dia-normal启动就好了(但不会启用选项卡窗口)

* php-fpm运行的当前目录与nginx的root配置项有关，url rewrite并不会影响php-fpm的当前目录，比如 root 为 /home/cifer/www，
那么即使 / 被重写为 /www1 , 在访问 / 时的php-fpm的当前目录依然是 /home/cifer/www 而不是 /home/cifer/www/www1

* bash shell 命令
    * w
    * uptime
    * uname
    
    搜索
    * which 
    * whereis
    * locate
