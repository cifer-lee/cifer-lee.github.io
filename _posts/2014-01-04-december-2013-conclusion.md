---
layout: post
title: 2013 年 12 月总结
date: 2014-01-04 10:32
published: false
---

现在的记性真的越来越差了, 一个月过去, 我竟然几乎记不起这一月发生的事, 唉.

## 学习的东西

*   学习了 pptp vpn 的搭建, 写了有关 vpn 的文章在博客里
*   学习了 linux mtd 对 flash memory 的支持
*   学习了 linux 下的信号机制, 以及 kill 命令
*   在 debian amd64 上交叉编译可在 ar9331 处理器上运行的软件
*   终于明白了串口透传的概念
*   学习了在 C 语言下定义可变参数宏
*   架设 ftp server, vsftpd
*   学习 pseudo-terminal, 写了相关的文章
*   了解了 flash memory 和 eeprom 的关系

## tips

*   设置与清除文件的 sticky bit: `chmod +t file_name`, `chmod -t file_name`
*   当文件夹下有大量的(十万百万级别)文件时, 在该目录下执行 `ls` 可能会导致卡死, 原因可能是 `ls` 命令默认是排序的, 使用 `ls -U` 可能会解决这一问题
*   上面的问题还可以用 `find ./ -type f -print` 来避免卡死, 这个默认应该也是不排序的
*   查看 inode size: `sudo tune2fs -l /dev/sda5`
*   删除从源代码安装的软件: `make uninstall` or `make -n uninstall`(这个命令会显示出安装过程但并不会实际安装, 如果没有提供 uninstall, 我们可以根据这里的输出手动删除之前安装的所有东西)
*   在 debian 源中, php5-mysql 包括了 mysqli, pdo-mysql .... fuck, 坑爹!
*   slim 出错 "Failed to execute login command", 原因是我修改过 /etc/xdg/awesome/rc.lua 下的菜单项, 添加了一个菜单项最后忘了加逗号, 导致登录失败
*   登录失败时可以检查下 root 家目录以及登录用户家目录下的 .xession-errors
*   This will execute $cmd in the background (no cmd window) without PHP waiting for it to finish, on both Windows and Unix. 
        <?php 
        function execInBackground($cmd) { 
            if (substr(php_uname(), 0, 7) == "Windows"){ 
                pclose(popen("start /B ". $cmd, "r"));  
            } else { 
                exec($cmd . " > /dev/null &");   
            } 
        } 
        ?>

## 工作

*   解决了一个相当棘手的问题 - 串口假死
*   把 YeeboxWZ 程序移植到了 openwrt ar9331 架构上
*   又把 YeeboxWZ 移植到了 pcDuino ARM Cortex A8 架构上
*   扫清了代码中可能导致 segv 问题的代码

## 阅读

*   Advanced Programming in The Unix Environment, Terminal IO 章节
*   [http://coolshell.cn/articles/10739.html] 了解了 Lua, 看完了之后, 我不太喜欢这语言
*   老码农故事, 小说, 感觉写的一般

## 其它

*   明白了为什么 /var/log/messages 用 vim 看的时候自动上色

[http://coolshell.cn/articles/10739.html](http://coolshell.cn/articles/10739.html)
