---
title: Pathinfo 和 Nginx
slug: pathinfo-and-nginx
date: 2015-01-30 16:27:31
categories:
    - Nginx
tags:
    - pathinfo
    - nginx
    - php
---

不知为何, Nginx 中配置 PATH_INFO 似乎一直以来是一件不那么明朗的事情, 在网上搜索的话, 会搜到各种各样的配置方式. 很多都是网友们自己 "发明" 的. 各大发行版安装好了 Nginx 之后, 默认也是没有配置对 PATH_INFO 的支持的, 怎么会这样呢? 难道 Nginx 就没有一个官方的解决方案吗?

自然是有的.

PATH_INFO 是 CGI 1.1 标准中规定的一个变量, 在 www 服务器委托 CGI 脚本执行任务时, 需要传递给 CGI 脚本的信息. 这么重要的一个变量, Nginx 当然是会支持的. 参考一中就是官方的方案. 我们在这里重复一下.

首先我们知道, 在 nginx 中, 是可以使用 nginx 自带的一些命令, 给 CGI 1.1 中规定的那些变量赋值的, 而这些命令默认都位于 /etc/nginx/fastcgi.conf 或者 /etc/nginx/fastcgi_params 文件里, 在配置 fastcgi 程序处理我们的请求时, 只要在 nginx 中包含这个两个文件之一, fastcgi 程序就能够取得所需要的变量. 在我的系统上, /etc/nginx/fastcgi.conf 文件是这样的:

    fastcgi_param  SCRIPT_FILENAME    $document_root$fastcgi_script_name;
    fastcgi_param  QUERY_STRING       $query_string;
    fastcgi_param  REQUEST_METHOD     $request_method;
    fastcgi_param  CONTENT_TYPE       $content_type;
    fastcgi_param  CONTENT_LENGTH     $content_length;
    
    fastcgi_param  SCRIPT_NAME        $fastcgi_script_name;
    fastcgi_param  REQUEST_URI        $request_uri;
    fastcgi_param  DOCUMENT_URI       $document_uri;
    fastcgi_param  DOCUMENT_ROOT      $document_root;
    fastcgi_param  SERVER_PROTOCOL    $server_protocol;
    fastcgi_param  HTTPS              $https if_not_empty;
    
    fastcgi_param  GATEWAY_INTERFACE  CGI/1.1;
    fastcgi_param  SERVER_SOFTWARE    nginx/$nginx_version;
    
    fastcgi_param  REMOTE_ADDR        $remote_addr;
    fastcgi_param  REMOTE_PORT        $remote_port;
    fastcgi_param  SERVER_ADDR        $server_addr;
    fastcgi_param  SERVER_PORT        $server_port;
    fastcgi_param  SERVER_NAME        $server_name;

    # PHP only, required if PHP was built with --enable-force-cgi-redirect
    fastcgi_param  REDIRECT_STATUS    200;

/etc/nginx/fastcgi_params 文件的内容与 /etc/nginx/fastcgi.conf 类似, 只是少了 SCRIPT_FILENAME 变量的赋值 (SCRIPT_FILENAME 变量不是 CGI 1.1 要求的), 不过奇怪的是, 这两个文件中默认都没有对 PATH_INFO 的配置.

这样在我们的 cgi 程序里, 就拿不到 PATH_INFO 变量了, 以 PHP 为例, 假如你访问如下的 URI (下面都是以这个 URI 为例子), 会发现 $_SERVER['PATH_INFO'] 是空的.

    http://localhost/index.php/foo/bar?query=hello

怎么办呢, 好说, 既然默认配置里没给 PATH_INFO 赋值, 那我们就自己加上. Nginx 的 fastcgi 模块, 提供了一条指令 fastcgi_split_path_info, 使用这条指令, 再配上一个正则就能将 PATH_INFO 信息提取出来, 这样我们的 nginx 中的配置如下:

    location ~ ^(.+\.php)(.*)$ {
        fastcgi_split_path_info    ^(.+\.php)(.*)$; 
        if (!-f $document_root$fastcgi_script_name) {
            return 404;
        }
        fastcgi_param  PATH_INFO       $fastcgi_path_info;
        include /etc/nginx/fastcgi.conf;
        fastcgi_pass unix:/run/php-fpm.sock;
    }

其中, fastcgi_split_path_info 指令, 会将后面正则匹配出来的 \1 赋值给 `$fastcgi_script_name`, \2 赋值给 `$fastcgi_path_info`, 这两个都是 nginx fastcgi 模块的内置变量.

要注意的是, 即使不调用 fastcgi_split_path_info 指令, $fastcgi_script_name 变量默认也是有值的, 而 $fastcgi_path_info 默认却是空值 (我用 add_header X-debug-message $fastcgi_path_info; 调试看过).

取得了 `$fastcgi_path_info`, 下面就使用了 `fastcgi_param PATH_INFO $fastcgi_path_info;` 来给 PATH_INFO 赋值, 经过这之后, $_SERVER['PATH_INFO'] 就能被填充上值了.

另外, 关于上面的这段代码:

    if (!-f $document_root$fastcgi_script_name) {
        return 404;
    }

有人问为什么不使用 `try_files $fastcgi_script_name =404;`, 原因是 try_files 会导致 `$fastcgi_path_info` 变为空, 具体的原因可以参见这两个链接:

http://trac.nginx.org/nginx/ticket/321
http://forum.nginx.org/read.php?2,238825,238825

以上就是 Nginx 官方 Wiki 里给出的方法, 链接在参考 1 里.

事实上在找到上面的官方的方法之前, 我先搜到了 @Laruence 的博文, 在参考链接 2 里. @Laruence 的方法很简单, 不过需要借助 PHP 的 fix_pathinfo. 使用 @Laruence 的方法, 只需要你的 nginx 的配置文件这么写就行了:

    location ~ .php {
        include /etc/nginx/fastcgi.conf
        fastcgi_param    PATH_INFO    $fastcgi_script_name;
        fastcgi_pass unix:/run/php-fpm.sock;
    }

刚看的时候, 让我很疑惑, 怎么能直接把 `$fastcgi_script_name` 赋值给 PATH_INFO 呢? `$fastcgi_script_name` 的值不是应该就只是 /index.php 吗, 翻了 nginx fastcgi 模块的文档, google 了两下子, 都没有说 `$fastcgi_script_name` 变量是否包含 PATH_INFO 信息的, 最后我只好自己在 nginx 配置里加 `add_header X-debug-message $fastcgi_script_name;` 调试了一下才知道, 原来 `$fastcgi_script_name` 的值不是 /index.php, 而是 /index.php/foo/bar.[1]

原来如此, PATH_INFO 里现在的值是 /index.php/foo/bar, 然后 PHP 的 fix_pathinfo 特性修正 PATH_INFO 的值为 /foo/bar, 就是这样. 所以说, 这种方式是把 PATH_INFO 的解析工作交给 fastcgi 程序去做了, 这里也就是 PHP. 而第一种方式中, 这个工作其实是事先让 nginx 做好.

按照 CGI 的规范, PATH_INFO 本来就是要服务器程序准备好, 传给 CGI 程序的, 所以我个人倾向于官方的方式. 而且, 早先 PHP fix_pathinfo 这种方式已被爆出有 bug, 还是不用的为妙.

另外, 网上还有另外的一些方法, 各种转贴, 千篇一律, 这里就不详细说了, 只贴个配置吧:

    location ~ .php {
        include /etc/nginx/fastcgi.conf
        set $script_name $fastcgi_script_name;
        set $path_info    "";
    
        if ($uri ~ "^(.+?.php)(/.*)$") {
            set $script_name $1;
            set $path_info $2;
        }
    
        fastcgi_param    PATH_INFO    $path_info;
        fastcgi_param    SCRIPT_NAME    $script_name;
        fastcgi_pass unix:/run/php-fpm.sock;
    }

可以看出, 这种方式实际上和官方的方式原理是一样的.

## 脚注

1. 如果采用了 Nginx 官方的方式, 在 add_header 输出之前加上了这句: `fastcgi_split_path_info ^(.+.php)(.)$;`, 那么 `$fastcgi_script_name` 的值还是会是 /index.php 的. 这里说的是不用 `fastcgi_split_path_info ^(.+.php)(.)$;` 的方式.

## 参考

1. Nginx Wiki, 官方的方法, 推荐: http://wiki.nginx.org/PHPFcgiExample
2. Laruence 前辈的博文: http://www.laruence.com/2009/11/13/1138.html
