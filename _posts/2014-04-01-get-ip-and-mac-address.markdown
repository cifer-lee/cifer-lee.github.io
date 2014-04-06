---
layout: post
title: 获取接口的 IP 地址的方法
date: 2014-04-01 10:00
---

## iotcl(), SIOCGIFCONF

这应该是属于比较"底层"的方式

## getifaddrs()

{% highlight c %}
    static char *get_local_ip(const char *ifname) {
       struct ifaddrs *ifaddr = NULL;
       getifaddrs(&ifaddr);
           
       struct ifaddrs *p = NULL;
       char *str_if_addr = NULL;
       for(p = ifaddr ; p ; p = p->ifa_next) {
         if(p->ifa_addr == NULL) {
           continue ;
         }
         if(AF_INET == p->ifa_addr->sa_family && !strcmp(ifname, p->ifa_name)) {
           str_if_addr = inet_ntoa(((struct sockaddr_in *)p->ifa_addr)->sin_addr);
         }       
       }
             
       if(ifaddr) { 
         freeifaddrs(ifaddr);
       }       
 
       return str_if_addr;
     }
{% endhighlight %}
                 
这种方式有一个问题, 就是 p->ifa\_addr 可能会是 NULL(参见[这里][LINK2], [这里][LINK3], [这里][LINK4], [这里][LINK5]). 最好先判断一下.

参考资料:

1.  [国外一个使用方式1获取 IP 的文章][LINK1]

[LINK1]: http://blog.markloiseau.com/2012/02/get-network-interfaces-in-c/
[LINK2]: https://lists.debian.org/debian-glibc/2004/03/msg00191.html
[LINK3]: https://lists.debian.org/debian-glibc/2007/02/msg00170.html
[LINK4]: https://github.com/mcproxy/mcproxy/pull/1
[LINK5]: http://sourceforge.net/p/bonding/discussion/77913/thread/03f93486/
