---
title: linux 日志流
date: 2014-01-01 19:15
---

## klogd

klogd 是一个 system daemon, 平时运行在 user space, 这个进程会提取内核输出的消息并传送给 syslogd 这个 system daemon. 

我们知道 syslogd 这个进程是通过读取 /dev/log 套接字来获取其它程序的输出消息, gnu libc 提供的库函数 syslog 就是往这个套接字里写消息. 但是内核的消息是不往 /dev/log 里发送的, 内核的消息就要通过 klogd 来主动提取了, 在哪里提取呢?

在 linux 中有两个地方可以获得内核中发出的消息: /proc 文件系统和系统调用 sys\_syslog 接口, 尽管他们最终是一样的. Klogd 被设计为从这俩个里选择较合适的那个. 它首先检查 /proc 文件系统是否已被挂载, 是的话就找到 /proc/kmsg 来作为内核消息的源头. 如果 /proc 文件系统没有被挂载那么 klogd 使用系统调用(sys\_log)来获取内核的消息. 引证: 

引自 __Linux Device Drivers 3rd, How Messages Get Logged__
> The printk function writes messages into a circular buffer that is \_\_LOG_BUF_LEN bytes
long: a value from 4 KB to 1 MB chosen while configuring the kernel. The function
then wakes any process that is waiting for messages, that is, any process that is sleep-
ing in the syslog system call or that is reading /proc/kmsg. These two interfaces to the
logging engine are almost equivalent, but note that reading from /proc/kmsg con-
sumes the data from the log buffer, whereas the syslog system call can optionally
return log data while leaving it for other processes as well. In general, reading the
/proc file is easier and is the default behavior for klogd. 

不过, 如果在启动 klogd 的时候指定 -s 选项, 那么 klogd 会总是使用 sys\_syslog 系统调用来获得消息.

klogd 也支持提取消息的优先级, 通过指定 console log level, 也能够控制将消息显示到控制台等.

## circular buffer

看名字就知道, 这个缓冲区是首尾相连的, 它的大小是 \_\_LOG\_BUF\_LEN 字节(根据内核配置, 4KB---1MB), printk 打印出的消息会写到这个缓冲中, 写完后, 那些等待获取消息的进程就会被唤醒, 比如那些调用 sys\_syslog 的, 还有那些读取 /proc/kmsg 的进程, 其实不管是通过 sys\_syslog 还是通过 /proc/kmsg, 它们最终读的都是 circular buffer, 但是要注意的是, 通过读取 /proc/kmsg 会冲掉 circular buffer 中的消息, 而通过 sys\_syslog 则会保留 circular buffer 中的消息供其它进程使用.

## /proc/kmsg

klogd 默认会先读取 /proc/kmsg. 

你可以 stop 掉 klogd, 然后手动获取内核的消息, 比如你 cat /proc/kmsg, 你会发现 /proc/kmsg 就像一个 FIFO, cat 会阻塞, 然后逐行逐行的输出消息. 当然如果你没有停掉 klogd 或其他读取 /proc/kmsg 的进程, 那你就不能从那里读取数据了, 因为你会 contend for it!

## syslogd

这也是个运行在 user space 的守护进程.  

我们知道用户程序通过 gnu libc 的库函数 syslog 可以将自己程序的消息发送给 syslogd, 而内核是不是用 gnu c 库的, 内核要依赖于 klogd 来传送自己的消息.  
klogd 会将获取到的内核的消息发送给 syslogd, 而 syslogd 会检查 /etc/syslog.conf 来决定如何处理这些消息. syslogd 根据消息的 facility 和 priority 来分类这些消息. 对于内核消息, facility 会是 LOG\_KERN 而 priority 则是 printk 时指定的值.

syslogd 通常会将消息输出到 /var/log/messages, 不管是用户程序发来的还是内核发来的, 你可以在启动 klogd 的时候指定 -f 选项来把内核的消息输出到一个文件里, 而不是传送给 syslogd.