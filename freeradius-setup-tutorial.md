title: FreeRADIUS + PPTP 系统架设指e
slug: freeradius-setup-tutorial
date: 2014-12-20 23:16
modified: 2014-12-26 16:07
tags: freeradius, mysql, AAA, pptp, linux

搭建这个系统时, 我所使用的服务器是 CentOS 6.5, 下面的步骤如非特别说明, 均以 CentOS 为主.

1. 首先配置好 Poptop 正常工作, 这个可以找我以前的笔记或者博客

2. 安装 freeradius

       # yum install freeradius

   在 Debian 下
 
       # apt-get install freeradius

   > 注意1: 在 Debian 下, 安装完 freeradius 之后, freeradius 就会自动启动, 这点不太好
   > 注意2: 在 Debian 下, 安装 freeradius 时, freeradius-utils 这个包也会被安装上, 所以如果是用的 Debian, 那么下面第三步应该就不用看了

3. (Centos)  安装 freeradius-utils 包, 下面的步骤里我们会用到 radtest 这个程序, 在 centos 源里, 这个程序在 freeradius-utils 包中

       # yum install freeradius-utils

   接下来我们来测试以下我们安装的 freeradius 是否正常工作

4. 首先修改 /etc/raddb/users 文件 (对于 Debian, 这个配置文件位于 /etc/freeradius/users), 添加一个用户, 找到下面这一行, 取消其注释. 

       # steve Cleartext-Password := "testing"

   > 注意: freeradius 项目主页的文档告诉我们, 尽可能的不要修改所有默认的配置, 这些配置一般是会适合你, 除非清除要修改的配置是干什么的. 在参考 1 的链接中, 它还将下面的几行也取消注释了, 这对于你从另一台机器上测试是是有必要的, 但下面一步我们只是想在 freeradius 服务器上测试, 所以只需取消注释这一行即可

5. 启动 freeradius

   CentOS: 

       # radiusd -X

   Debian:

       # freeradius -X

   (在 CentOS 中 man radiusd, 建议我们一开始搭建时应该总是使用 -X 启动 freeradius)

6. 在另一个终端中执行

       $ radtest steve testing localhost 1812 testing123

   这句里面, steve 和 testing 就是刚刚我们配置的用户名和密码, localhost 和 1812 就是 freeradius 运行的 IP 地址和端口号, testing123 则是用于加密 freeradius 服务器与客户端通信的共享 key (虽然在此处我们的 freeradius 服务器端与客户端位于同一台主机上), 这个值是在 /etc/raddb/clients.conf 中定义的. clients.conf 的注释强烈建议这个值一定要被修改, 不要使用默认的 testing123.

   radtest 测试成功后会输出类似如下的信息:

       Sending Access-Request of id 211 to 127.0.0.1 port 1812
       User-Name = "steve"
       User-Password = "testing"
       NAS-IP-Address = 127.0.0.1
       NAS-Port = 1812
       Message-Authenticator = 0x00000000000000000000000000000000
       rad_recv: Access-Accept packet from host 127.0.0.1 port 1812, id=211, length=20

   在你运行 radiusd -X 的终端也会看到相应的信息的输出.

7. 很多 NAS 都自己集成了 radius 客户端的功能, 比如思科华为等公司的路由器交换机等. 但是一般这些产品没有强大到集成了 radius 服务器端. 如果你们的公司是属于这种情况的话, 那么可能你们会直接使用路由器内置的 radius 客户端功能, 这样的话网络拓朴应该是这样的:



                     公网 ------------------ NAS (RADIUS 客户端) ------------ RADIUS Server
                                                       \ 
                                                         \ 
                                                           \
                                                        Devices

   对于处于公网的员工来说, 想要通过 PPTP 连接到内网, 就要首先连接到 NAS (这种情况下, NAS 也要有 PPTP server 的功能), 然后 NAS 会到处于内网的 RADIUS Server 那查询身份验证, 授权以及计费信息.

   如果, 你们的路由器没有 RADIUS 客户端的功能, 那么可以在内网里找一台服务器, 在其上搭建 PPTP server 以及 RADIUS server, 而且, RADIUS client 的功能也得在这台服务器上实现了. 这种情况下, 你需要在路由器上做一个端口映射, 将 1723 (PPTP 端口) 映射到 RADIUS 这台服务器上.

   如果是第一种情况, 也就是 RADIUS Server 和 RADIUS 客户端是分开的, 那么就需要在 RADIUS server 上面配置使其能够不仅接受来自自己本机的连接, 还能接受来自其他主机的连接. 你可以查看 /etc/raddb/clients.conf, 里面有提示告诉你如何去做, 很详细, 比如:

       #
       # You can now specify one secret for a network of clients.
       # When a client request comes in, the BEST match is chosen.
       # i.e. The entry from the smallest possible network.
       #
       # client 192.168.0.0/24 {
       #    secret = testing123-1
       #    shortname = private-network-1
       # }

   取消上面的注释, 然后保存, 重启 radius -X, 就可以在和 RADIUS 处于一个内网的其他主机上连接 RADIUS 服务器了. 可以在那台机器上安装 freeradius-utils 包, 然后运行:

       $  radtest steve testing 192.168.0.9 1812 testing123-1     # 假设 RADIUS 服务器 IP 地址是 0.9

## mysql 与 freeradius 连接

1. 创建 radius 数据库

       $ mysql -uroot -p
       > create database radius default character set utf8;

   然后退出 mysql 就可以了.

2. 安装 freeradius-mysql 包

   安装这个包之后, 你会发现 /etc/raddb/sql 这个目录下多了两个目录, mysql 目录和 ndb 目录 (根据源里面 freeradius 包的不同可能你只有一个 mysql 目录). 其中 ndb 只有两个脚本: schema.sql 和 admin.sql. admin.sql 脚本用来创建管理 radius 数据库的专门用户并为其赋予管理 radius 数据库的全部权限. schema.sql 脚本用来创建 radius 数据库里的表.

   mysql 目录下也有着两个脚本, 只不过 ndb 目录下的 schema.sql 脚本创建数据表的时候会指定使用是 ndb 存储引擎. 而 mysql 目录下的 schema.sql 脚本创建数据库时不会指定存储引擎, 也就是使用默认的存储引擎.

   这两个目录下的 admin.sql 脚本的区别是对权限控制粒度的不同, mysql 目录下的 admin.sql 在创建 radius 专门的管理账户时只对其需要写入的数据表赋予写权限, 其他的表都是读权限. 而 ndb 下的 admin.sql 脚本是对这个专门的账户服务 radius 数据库的全部权限.

   freeradius 不会在乎数据库用的是什么存储引擎, 如果你不熟悉 ndb 引擎或者对引擎没有要求, 那么用 mysql 目录下的脚本即可, 何况, 它对权限粒度把握的还细, 更安全.

   mysql 目录下除了 admin.sql 和 schema.sql 之外, 还包含很多其他的 sql 脚本用于不同用处, 我们暂时不用管它们.

3. 导入 admin.sql 和 schema.sql

   为了简单, 我们就都使用 root 账户导入了:

       $ mysql -uroot -p < admin.sql
       $ mysql -uroot -p radius < schema.sql

   其中 admin.sql 创建的账户的名称和密码是 radius / radpass, 我没有做修改

   这样, msyql 数据方面的工作就完成了, 然后我们需要再在 freeradius 上添加相关的配置.

4. 配置 freeradius 让其知道 mysql

   在 /etc/raddb/radiusd.conf 配置文件中, 找到这一行:

       # $INCLUDE sql.conf

   取消其注释.

   然后编辑 /etc/raddb/sql.conf 文件, 设置数据库类型, 设置访问数据库的账户及密码等, 一般来说下载包的时候源里都会给你做这些事, 如果你没有修改 admin.sql 的话, 那这步应该不用修改什么, 但必须是要确认一下的.

   在强调一遍, freeradius 项目主页有文档说, 建议我们尽可能的不要修改默认参数, 除非你知道修改后的影响.

5. 配置相应的设备, 令其使用 mysql 作为数据存储设备

       # vim /etc/freeradius/sites-available/default

   * 把authorize{}字段下的file注释掉、反注释sql、这里的file指的就是usrs文件、将不再把用户信息写在users而使用mysql来存储用户信息、
   * 把accounting{} 字段下的sql反注释、启用sql来记录统计信息、
   * 把session{}字段下的sql反注释、启用用户同时登录限制功能、这里还需要修改其它地方、一会再说
   * 把post-auth{} 字段的sql反注释、启用用户登录后进行数据记录功能、

   整个文件如下所示:

       authorize {
               ...
               ...
       #        files
               sql
               ...
       }

       ...
       ...
        
       accounting {
               ...
               sql
               ...
       }
       
       ...
       ...
       
       session {
           radutmp
       
           #
           #  See "Simultaneous Use Checking Queries" in sql.conf
           sql
       }
       
       post-auth {
           ...
           sql
           ...
           ...
       }

   如果迩之前如莪一样启动了启用用户同时登录限制功能、那么接下来还要做这一步

   编辑dialup.conf文件

       $ vim /etc/freeradius/sql/mysql/dialup.conf

   找到这几行、将之反注释

       # Uncomment simul_count_query to enable simultaneous use checking
       simul_count_query = "SELECT COUNT(*) \
                                FROM ${acct_table1} \
                                WHERE username = '%{SQL-User-Name}' \
                                AND acctstoptime IS NULL"
   之后整个对mysql的radius配置就已经完成了

## 整合 PPTP 与 FreeRADIUS

首先我们要搭建一个能够正常工作的 PPTP 服务器, 关于 PPTP 服务器的搭建, 可以参考我另外的日志

另外, 要整合的话可定需要一个 radius 客户端, pptp 自己可没有 radius 客户端的功能. 如果你的路由器已经有了 radius 客户端的功能, 那就不需要了.

1. 我们从 CentOS 源里下载 radiusclient-ng 这个包:

       # yum install radiusclient-ng

   安装完之后, 配置文件位于 /etc/radiusclient-ng 目录下.

2. 设置共享密钥

   首先在 /etc/radiusclient-ng/radiusclient.conf 文件中, 你应该可以找到如下几行:

       # RADIUS settings
       
       # RADIUS server to use for authentication requests. this config
       # item can appear more then one time. if multiple servers are
       # defined they are tried in a round robin fashion if one
       # server is not answering.
       # optionally you can specify a the port number on which is remote
       # RADIUS listens separated by a colon from the hostname. if
       # no port is specified /etc/services is consulted of the radius
       # service. if this fails also a compiled in default is used.
       authserver localhost
 
       # RADIUS server to use for accouting requests. All that I
       # said for authserver applies, too.
       #
       acctserver localhost

   这代表 radiusclient-ng 默认认为 radius 服务器是在本机上的, 这和我们的情况相符, 不需改动, 然后我们编辑 /etc/radiusclient-ng/servers 文件, 在其末尾添加:

       localhost         testing123

   其中 localhost 就对应上面的 authserver, acctserver 指定的主机. testing123 是我们前面在 /etc/raddb/clients.conf 中指定的值.

3. 增加 microsoft 字典

   这一步很重要, 如果不加的话, windows 用户将无法通过 freeradius 的验证. 安装完 radiusclient-ng 包之后, 可以从 /etc/radiusclient-ng/radiusclient.conf 文件看出, 它包含了 /usr/share/radiusclient-ng/dictionary 文件, 这个文件是一个总的字典文件, 要包含其它字典文件, 只需在这个文件包含它们即可, 其他 dictionary 文件也位于 /usr/share/radiusclient-ng/ 目录, 这个目录里有很多字典文件, 但是不幸的是, 唯独没有 microsoft 的字典文件, 具体原因我暂时不清楚. 不过 microsoft 的字典文件可以通过其他方式找到. 在 freeradius 项目的 wiki 里 (参考 1) 有提供这个 microsoft 的字典, 将其拷贝下来即可. 另外这个页面还提供了一个 merit 字典, 不过 /usr/share/radiusclient-ng/ 目录里已经有这个字典了, 就不用再下载了.

   然后我们找到 /usr/share/radiusclient-ng/dictionary 文件, 在末尾添包含 merit 和 microsoft 字典 (/usr/share/radiusclient-ng 目录还包含了很多其他字典, 可能以后会需要, 参考 1 中只指定了这两个字典, 我们也先只添加这两个):

    
       INCLUDE /usr/share/radiusclient-ng/dictionary.merit
       INCLUDE /usr/share/radiusclient-ng/dictionary.microsoft

4. 修改 pptpd 的配置以及 radiusclient-ng 的配置

   /etc/pptpd.conf 需要作如下修改:

   * 确保 noipparam 选项**没有被注释**
   * 确保 delegate 选项**没有被注释**
   * 确保 logwtmp 选项**被注释**

   /etc/ppp/options.pptpd 文件末尾添加如下几行: 

       plugin radius.so
       plugin radattr.so
       radius-config-file /etc/radiusclient-ng/radiusclient.conf         # 如上面所见, 我们的 radiusclient.conf 是位于 /etc/radiusclient-ng 目录下的

   这两个 so 库是 pppd 的插件, 在安装 pppd 的时候就应该已经是安装了. 而 radius-config-file 指定 radiusclient.conf 的位置, 具体看后面的问题 1.

   /etc/radiusclient-ng/radiusclient.conf 这个文件里, 把 bindaddr * 这一行注释掉, 即

        # local address from which radius packets have to be sent
        #bindaddr *

5. 把 pptp 账户信息转移到 mysql radius 数据库

6. 重启 pptpd 和 radiusd 服务

       /etc/init.d/pptpd restart
       radiusd -X

现在你在自己电脑连接 pptp, 就是通过 freeradius 验证了.

## FreeRADIUS 的 web 管理界面

有很多开源的项目能够帮助你从 web 界面管理 FreeRADIUS 系统, 像我随便一搜就搜到下面这么多:

1. http://daloradius.com/
2. http://freeradius.org/dialupadmin.html
3. http://phpradiusadmin.sourceforge.net/
4. http://labs.asn.pl/ara/wiki

不过我又稍微深入的了解了下:

* 第 2 个是 freeradius 官方的, 但是已经不维护了, 最新版本是 2003 年发布的, freeradius 页面 http://wiki.freeradius.org/guide/Dialup-admin 上面明确说了这是 PHP4 写的, 可能在新版本 PHP 上不能好好工作. 弃之.
* 第 3 个最新版本是 1.0, 是 2010 年 12 月发布的.  4 年没更新了. 弃之.
* 第 4 个最新版本是 2009 年发布的, 弃之.

daloradius 最新版本虽然是 2011 年 5 月发布的, 距今也有 3 年多, 但是 daloradius 的社区氛围很好, 很活跃, 相信有问题一定能够及时得到解决. daloradius 代码有 74000 多行, 功能强大. 就是它了!

### 安装过程

下载下来 daloradius 包之后, 看 README, 然后是 INSTALL, 最后可以看看 FAQS.

详细的安装过程都在 INSTALL 文件里, 这里只说重点的. INSTALL 文件和网上的教程一般都是讲的 apache 服务器, 我们用的是 nginx. 有些地方不一样, 重点说一下.

#### 依赖

首先是确保 PHP DB Abstraction Layer (may require PHP Pear) 安装了, 这点 INSTALL 文档说了, 我当时没有注意到这点, 结果搭完 daloradius 登录后页面空白, 看 nginx 日志看到如下内容:

    2014/12/11 15:51:41 [error] 14311#0: *1 FastCGI sent in stderr: "PHP message: PHP Warning: include_once(DB.php): failed to open stream: No such file or directory in /srv/www/daloradius-0.9-9/library/opendb.php on line 84
    PHP message: PHP Warning: include_once(): Failed opening 'DB.php' for inclusion (include_path='.:/usr/share/pear:/usr/share/php') in /srv/www/daloradius-0.9-9/library/opendb.php on line 84
    PHP message: PHP Fatal error: Class 'DB' not found in /srv/www/daloradius-0.9-9/library/opendb.php on line 86" while reading response header from upstream, client: 192.168.0.117, server: dalo.yeedev.com, request: "GET /dologin.php HTTP/1.1", upstream: "fastcgi://unix:/var/run/php-fpm/php-fpm.sock:", host: "dalo.yeedev.com"

安装可以使用 pear 也可以使用 yum, 建议用 yum 吧, 方式是这样的:

    # yum search php db
    # yum install php-pear-DB

1. 我是将 daloradius-0.9-9.tar.gz 解压到了 /srv/www, 由于打包的人的机器上的用户 ID 和我机器上的用户 ID 不同, 解压之后 /srv/www/daloradius-0.9-9/ 这个目录的权限变成了:

       drwxr-xr-x 11 33 tape 12288 May 6 2011 daloradius-0.9-9/

   看来, 打包 daloradius 的开发者的机器上的用户 ID 是 33, 而我维护的机器上没有编号为 33 的用户, 所以就显示数字了. 后面我们需要让 nginx (以及 php-fpm, 我维护的服务器上运行 php-fpm 的用户和 nginx 一样) 对这个目录有读写权限, 我们将 nginx 用户设为这个目录的属主. (INSTALL 文档里让你把 www-data 设为它的属主, www-data 是默认运行 apache 的用户). 我们运行如下命令:

       # chown -R nginx:nginx daloradius-0.9-9/

   另外, 我们确保一下 library/daloradius.conf.php 文件的权限是 644, 以使得 nginx 用户对它有读写权限 (经过上面的一步, 应该就已经是 664 了)

       # chmod 644 library/daloradius.conf.php

2. 导入 daloradius 数据库

   daloradius 需要使用 freeradius 数据库, 并在里面创建一些自己需要的数据表, freeradius 的数据库我们前面已经创建过了. 那么现在需要的就只是把 daloradius 自己的数据表导进去就可以了. 这些个数据表位于 contrib/db/mysql-daloradius.sql

   还记得我们前面也为 freeradius 的数据库 --- radius, 创建了一个专门的用户吧, 但是当时没有给这个用户赋予 radius 数据库的所有权限, 所以他就没有权限在 radius 数据库里创建数据表, 所以我们还是使用 root 用户导入这些数据表:
   
       # mysql -uroot -p radius < mysql-daloradius.sql

   导完之后, 我一看, 我去, 给我新创建了那么多表! (daloradius 会创建十多张, radius 本来就五六张表)

3. 配置 daloradius 的数据库连接信息

   修改 library/daloradius.conf.php 中的信息, 需要修改的信息有:

       CONFIG_DB_USER = 'radius'
       CONFIG_DB_PASS = 'radpass'    # 这是我们前面配置过的
       CONFIG_DB_NAME = 'radius'
       .....
       ..... 不需改变
       .....
       CONFIG_FILE_RADIUS_PROXY = '/etc/raddb/proxy.conf'    # 默认是 /etc/freeradius/proxy.conf, 这是基于 Debian 系统的, 与我们不符, 需改成我们的
       CONFIG_PATH_DALO_VARIABLE_DATA = '/srv/www/daloradius-0.9-9/var'

   其它保持默认即可. 这个文件还可以配置邮箱通知 smtp 信息, 我们暂时先不配置.

4. 设置 radius 数据库 radius 用户的权限

   第二步的时候导入了很多 daloradius 需要的数据表, 第三步我们配置 daloradius 使用 radius 用户来访问数据库, 但是 radius 用户对新导入那些表没有任何权限  (我们前面的配置中, radius 用户只对 freeradius 的几个表有读写权限). 由于 daloradius 一下子创建了这个么表, 之前 freeradius 有哪些表我也没记住. 我也分不清哪些是 daloradiu 创建的了, 简便起见, 我们直接这样吧:

       GRANT ALL on radius.\* TO 'radius'@'localhost';

5. 收尾

至此就按装完了, INSTALL 文档说了默认的管理员用户和密码: administrator/radius.

最后, INSTALL 文档还建议我们修改 administrator 用户的密码 (daloradius 管理界面就能修改), 以及将 /update.php 文件改个名, 免得被别人不小心启动了升级.

关于 daloradius 的使用在我的另一篇文章中

### 参考

1. daloradius INSTALL 文档推荐的参考文章: http://www.howtoforge.com/authentication-authorization-and-accounting-with-freeradius-and-mysql-backend-and-webbased-management-with-daloradius    

## 问题记录

1. 找不到 radiusclient.conf 文件

   我在用手机开启 3G, 然后开 pptp 连接的时候, 连接不上, 到服务器开监控这 log 文件再连接一次, 日志如下:

       Dec 10 10:48:21 localhost pptpd[6780]: CTRL: Client 153.119.195.169 control connection started
       Dec 10 10:48:21 localhost pptpd[6780]: CTRL: Starting call (launching pppd, opening GRE)
       Dec 10 10:48:21 localhost pppd[6781]: Plugin radius.so loaded.
       Dec 10 10:48:21 localhost pppd[6781]: RADIUS plugin initialized.
       Dec 10 10:48:21 localhost pppd[6781]: Plugin radattr.so loaded.
       Dec 10 10:48:21 localhost pppd[6781]: RADATTR plugin initialized.
       Dec 10 10:48:21 localhost pppd[6781]: pppd 2.4.5 started by root, uid 0
       Dec 10 10:48:21 localhost pppd[6781]: Using interface ppp0
       Dec 10 10:48:21 localhost pppd[6781]: Connect: ppp0 <--> /dev/pts/9
       Dec 10 10:48:21 localhost pppd[6781]: rc_read_config: can't open /etc/radiusclient/radiusclient.conf: No such file or directory
       Dec 10 10:48:21 localhost pppd[6781]: RADIUS: Can't read config file /etc/radiusclient/radiusclient.conf
       Dec 10 10:48:21 localhost pppd[6781]: Peer birdee failed CHAP authentication
       Dec 10 10:48:21 localhost pptpd[6780]: CTRL: EOF or bad error reading ctrl packet length.
       Dec 10 10:48:21 localhost pptpd[6780]: CTRL: couldn't read packet header (exit)
       Dec 10 10:48:21 localhost pptpd[6780]: CTRL: CTRL read failed
       Dec 10 10:48:21 localhost pppd[6781]: Modem hangup
       Dec 10 10:48:21 localhost pppd[6781]: Connection terminated.
       Dec 10 10:48:21 localhost pppd[6781]: Exit.
       Dec 10 10:48:21 localhost pptpd[6780]: CTRL: Client 153.119.195.169 control connection finished

   可以看出, 建立 ppp 连接之后, pppd 正确加载了 radius.so 和 radattr.so, 但是找不到 radiusclient.conf, rc\_read\_config 应该就是这两个库里的某个函数调用之类的, 猜测可能还得在 /etc/pptpd.conf 里告诉去哪里找 radiusclient.conf.

   可以看出 pppd 这两个 so 库是从 /etc/radiusclient/ 目录找 radiusclient.conf, 而我在安装 radiusclient 的时候, 源里面已经没有 radiusclient 这个包了, 取而代之的是 radiusclient-ng 这个包, 所以, 按理说 CentOS 源的维护者应该会帮我们解决这些问题. 我想可能我系统上 pppd 的版本低了, 升级一下也许就可以了. 结果执行 `yum check-update`, `yum info ppp` 之后发现我系统上的 pppd 就是最新的. 唉 CentOS 源维护者没做好这块啊.

   于是我 google 了一下 "rc\_read\_config: can't open /etc/radiusclient/radiusclient.conf: No such file or directory", 无奈竟然 google 了半天没结果, 最终在参考 2 中看到, 说是在 /etc/options.pptpd 末尾, 也就是 radius.so  radattr.so 这两行后面加上一句:

       radius.so
       radattr.so
       radius-config-file /etc/radiusclient-ng/radiusclient.conf         # 如上面所见, 我们的 radiusclient.conf 是位于 /etc/radiusclient-ng 目录下的

2. 上面那个问题解决重启 pptpd 之后, 又出现一个新的问题, 从我手机连 pptp 依然连不上, 这此服务器上日志是这样:

       Dec 10 11:18:14 localhost pptpd[7045]: MGR: Manager process started
       Dec 10 11:18:51 localhost pptpd[7048]: CTRL: Client 153.119.195.169 control connection started
       Dec 10 11:18:51 localhost pptpd[7048]: CTRL: Starting call (launching pppd, opening GRE)
       Dec 10 11:18:51 localhost pppd[7049]: Plugin radius.so loaded.
       Dec 10 11:18:51 localhost pppd[7049]: RADIUS plugin initialized.
       Dec 10 11:18:51 localhost pppd[7049]: Plugin radattr.so loaded.
       Dec 10 11:18:51 localhost pppd[7049]: RADATTR plugin initialized.
       Dec 10 11:18:51 localhost pppd[7049]: pppd 2.4.5 started by root, uid 0
       Dec 10 11:18:51 localhost pppd[7049]: Using interface ppp0
       Dec 10 11:18:51 localhost pppd[7049]: Connect: ppp0 <--> /dev/pts/9
       Dec 10 11:18:54 localhost pppd[7049]: /etc/radiusclient-ng/radiusclient.conf: line 75: unrecognized keyword: bindaddr
       Dec 10 11:18:54 localhost pppd[7049]: rc_read_dictionary: invalid include entry on line 241 of dictionary /usr/share/radiusclient-ng/dictionary
       Dec 10 11:18:54 localhost pppd[7049]: RADIUS: Can't read dictionary file /usr/share/radiusclient-ng/dictionary
       Dec 10 11:18:54 localhost pppd[7049]: Peer birdee failed CHAP authentication
       Dec 10 11:18:54 localhost pptpd[7048]: CTRL: EOF or bad error reading ctrl packet length.
       Dec 10 11:18:54 localhost pptpd[7048]: CTRL: couldn't read packet header (exit)
       Dec 10 11:18:54 localhost pptpd[7048]: CTRL: CTRL read failed
       Dec 10 11:18:54 localhost pppd[7049]: Modem hangup
       Dec 10 11:18:54 localhost pppd[7049]: Connection terminated.
       Dec 10 11:18:54 localhost pppd[7049]: Exit.
       Dec 10 11:18:54 localhost pptpd[7048]: CTRL: Client 153.119.195.169 control connection finished

   可以看出有两个错误, 一个是 radiusclient.conf 中有个关键字 bindaddr 不被识别, 我依然是看了参考 2 这个仁兄的解决方法: 到 radiusclient.conf 中注释掉 bindaddr 这一行, 再次用手机连接, 这次 bindaddr 不被识别的错误没了:

       Dec 10 11:26:31 localhost pptpd[7102]: CTRL: Client 153.119.195.169 control connection started
       Dec 10 11:26:32 localhost pptpd[7102]: CTRL: Starting call (launching pppd, opening GRE)
       Dec 10 11:26:32 localhost pppd[7103]: Plugin radius.so loaded.
       Dec 10 11:26:32 localhost pppd[7103]: RADIUS plugin initialized.
       Dec 10 11:26:32 localhost pppd[7103]: Plugin radattr.so loaded.
       Dec 10 11:26:32 localhost pppd[7103]: RADATTR plugin initialized.
       Dec 10 11:26:32 localhost pppd[7103]: pppd 2.4.5 started by root, uid 0
       Dec 10 11:26:32 localhost pppd[7103]: Using interface ppp0
       Dec 10 11:26:32 localhost pppd[7103]: Connect: ppp0 <--> /dev/pts/9
       Dec 10 11:26:32 localhost pppd[7103]: rc_read_dictionary: invalid include entry on line 241 of dictionary /usr/share/radiusclient-ng/dictionary
       Dec 10 11:26:32 localhost pppd[7103]: RADIUS: Can't read dictionary file /usr/share/radiusclient-ng/dictionary
       Dec 10 11:26:32 localhost pppd[7103]: Peer birdee failed CHAP authentication
       Dec 10 11:26:32 localhost pptpd[7102]: CTRL: EOF or bad error reading ctrl packet length.
       Dec 10 11:26:32 localhost pptpd[7102]: CTRL: couldn't read packet header (exit)
       Dec 10 11:26:32 localhost pptpd[7102]: CTRL: CTRL read failed
       Dec 10 11:26:32 localhost pppd[7103]: Modem hangup
       Dec 10 11:26:32 localhost pppd[7103]: Connection terminated.
       Dec 10 11:26:32 localhost pppd[7103]: Exit.
       Dec 10 11:26:32 localhost pptpd[7102]: CTRL: Client 153.119.195.169 control connection finished

   看来我们前面在 /usr/share/radiusclient-ng/dictionary 后面加的那两行 INCLUDE 不被识别, 看来 microsoft 和 merit 字典不是这么加的. 本来想 google 的, 但是我无意中在 /usr/share/radiusclient-ng/ 目录下的另一个字典文件: dictionary.ascend 顶部介绍中看到如下的内容:

       #
       # Ascend dictionary.
       #
       # Enable by putting the line "$INCLUDE dictionary.ascend" into
       # the main dictionary file.
       #
       # Version: 1.00 21-Jul-1997 Jens Glaser <jens@regio.net>
       #

   而且, 在参考 1 中, 也提到了用 $INCLUDE 指定来包含其他字典文件, 于是我试了以下, 果然解决了这个问题

3. 解决了上面两个问题, 还有问题阿, 问题就是这个:

       Dec 10 12:33:33 localhost pptpd[7437]: CTRL: Client 153.119.195.169 control connection started
       Dec 10 12:33:33 localhost pptpd[7437]: CTRL: Starting call (launching pppd, opening GRE)
       Dec 10 12:33:33 localhost pppd[7438]: Plugin radius.so loaded.
       Dec 10 12:33:33 localhost pppd[7438]: RADIUS plugin initialized.
       Dec 10 12:33:33 localhost pppd[7438]: Plugin radattr.so loaded.
       Dec 10 12:33:33 localhost pppd[7438]: RADATTR plugin initialized.
       Dec 10 12:33:33 localhost pppd[7438]: pppd 2.4.5 started by root, uid 0
       Dec 10 12:33:33 localhost pppd[7438]: Using interface ppp0
       Dec 10 12:33:33 localhost pppd[7438]: Connect: ppp0 <--> /dev/pts/9
       Dec 10 12:33:36 localhost pppd[7438]: rc_avpair_new: unknown attribute 11
       Dec 10 12:33:36 localhost pppd[7438]: rc_avpair_new: unknown attribute 25
       Dec 10 12:33:37 localhost pppd[7438]: Peer sqltest failed CHAP authentication
       Dec 10 12:33:37 localhost pptpd[7437]: CTRL: EOF or bad error reading ctrl packet length.
       Dec 10 12:33:37 localhost pptpd[7437]: CTRL: couldn't read packet header (exit)
       Dec 10 12:33:37 localhost pptpd[7437]: CTRL: CTRL read failed
       Dec 10 12:33:37 localhost pppd[7438]: Modem hangup
       Dec 10 12:33:37 localhost pppd[7438]: Connection terminated.
       Dec 10 12:33:37 localhost pppd[7438]: Exit.
       Dec 10 12:33:37 localhost pptpd[7437]: CTRL: Client 153.119.195.169 control connection finished

   这个问题参考 1 中有说到, 并且参考 1 对其解释很全. 我也很清楚了这个问题的原因, 但是我严格按照参考 1的步骤, 以及我还采取了其它巧妙的步骤, 但都是解决不了. 最后只好用了不太合常规的做法解决了.

   建议先看以下参考 1 对这个错误的描述再往下看我的解决过程.

   这个错误的重点在于:

       Dec 10 12:33:36 localhost pppd[7438]: rc_avpair_new: unknown attribute 11
       Dec 10 12:33:36 localhost pppd[7438]: rc_avpair_new: unknown attribute 25

   如果你打开 dictionary.microsoft, 就会看到, 11 和 25 这两个 attribute 分别是 MS-CHAP-Challenge 和 MS-CHAP2-Response, 那么问题就很明了了. 就是 dictionary.microsoft 这个字典文件似乎没有被 radiusclient 读到, 参考 1 中说到的所有的点我都排除了. 参考 1 让将 $INCLUDE 改成 INCLUDE, 但是你知道的, 改成 INCLUDE 就会导致上面第 2 个问题.

   我采取的巧妙地方法是: 我觉得可能是 pppd 的 bug, 虽然在 /etc/options.pptpd 中指定配置文件路径: /usr/share/radiusclient-ng/radiusclient.conf, 但是有可能 pppd 还是没处理好, 于是我注释掉了 /etc/options.pptpd 中的这句:

       radius-config-file /etc/radiusclient-ng/radiusclient.conf

   然后位 /etc/radiusclient-ng 建立了一个软连接:

       ln -s /etc/radiusclient-ng /etc/radiusclient

   但结果也是解决不了

4. 

       Dec 10 15:16:49 localhost pptpd[8445]: MGR: Manager process started
       Dec 10 15:16:56 localhost pptpd[8449]: CTRL: Client 153.119.196.0 control connection started
       Dec 10 15:16:56 localhost pptpd[8449]: CTRL: Starting call (launching pppd, opening GRE)
       Dec 10 15:16:56 localhost pppd[8450]: Plugin radius.so loaded.
       Dec 10 15:16:56 localhost pppd[8450]: RADIUS plugin initialized.
       Dec 10 15:16:56 localhost pppd[8450]: Plugin radattr.so loaded.
       Dec 10 15:16:56 localhost pppd[8450]: RADATTR plugin initialized.
       Dec 10 15:16:56 localhost pppd[8450]: pppd 2.4.5 started by root, uid 0
       Dec 10 15:16:56 localhost pppd[8450]: Using interface ppp0
       Dec 10 15:16:56 localhost pppd[8450]: Connect: ppp0 <--> /dev/pts/2
       Dec 10 15:16:56 localhost pppd[8450]: peer from calling number 153.119.196.0 authorized
       Dec 10 15:16:57 localhost pppd[8450]: MPPE 128-bit stateless compression enabled
       Dec 10 15:17:21 localhost pptpd[8449]: CTRL: EOF or bad error reading ctrl packet length.
       Dec 10 15:17:21 localhost pptpd[8449]: CTRL: couldn't read packet header (exit)
       Dec 10 15:17:21 localhost pptpd[8449]: CTRL: CTRL read failed
       Dec 10 15:17:21 localhost pppd[8450]: Modem hangup
       Dec 10 15:17:21 localhost pppd[8450]: MPPE disabled
       Dec 10 15:17:21 localhost pppd[8450]: Connection terminated.
       Dec 10 15:17:21 localhost pppd[8450]: Connect time 0.5 minutes.
       Dec 10 15:17:21 localhost pppd[8450]: Sent 3656 bytes, received 6726 bytes.
       Dec 10 15:17:21 localhost pppd[8450]: Exit.
       Dec 10 15:17:21 localhost pptpd[8449]: CTRL: Client 153.119.196.0 control connection finished

   我尝试了以下许多种方法:

   1. 修改 radiusd 的 ippool 模块的 ip 地址范围, 改成和以前不用 freeradius 时, 只用 pptpd 分配地址时的范围一样, 不能解决
   2. 再次翻看参考 1中的步骤, 发现我有一步露了, 就是这一步, 但是操作了这一步, 依然不能解决

          ### If it is not available
          # modprobe ppp-compress-18 && echo MPPE Module is ok
          FATAL: Module ppp_mppe not found.
          ### If it is available
          # modprobe ppp-compress-18 && echo MPPE Module is ok
          MPPE Module is ok

   3. google 了一下午, 无果, 主要是上面的 log 信息跟本看不出 freeradius 或者 radiuclient 是否有问题
   4. 最后, 我开始使用排除法, 将 /etc/pptpd.conf 以及 /etc/ppp/options.pptpd 配置文件一步一步, 一个参数一个参数的改, 直到恢复成不和 freeradius 配合时的状态 (这时是能够连接成功的), 然后查看 log 有什么不同
   5. 正所谓, 当你把所有的资源都用尽, 依然解决不了问题的时候, 只要不要气馁, 静下心来继续干, 就会超越瓶顶, 找到解决方案. 在第 4 步的过程中, 我突然想明白一个问题. 突然想明白了 pptpd/pppd 和 freeradius 的关系. 我上面配置的参数, 大多是参考了参考 1 和参考 2 的内容, 这些参数的含义我并没有完全理解. 但是在第 4 步的过程中, 我渐渐的理解了.

   首先看一下, 我们在 /etc/ppp/options.pptpd 中加入的这几行的意义:

       # put plugins here
       # (putting them higher up may cause them to sent messages to the pty)
       plugin radius.so
       plugin radattr.so
       radius-config-file /etc/radiusclient-ng/radiusclient.conf

   不要多想, 这几行其实意思很简单, 前面说了 radius.so 和 radattr.so 是 pppd 的插件, 它们的作用就是让 pppd 能够和 freeradius 通信. 让 pppd 使用 freeradius 的身份验证以及授权机制以及计时功能.

   而我们在 /etc/pptpd.conf 中将 delegate 和 noipparam 取消注释是什么意思呢, 将它们取消注释就会使得 pptpd 不会为连接过来的客户端分配 IP 地址, 转而将这个工作交给 freeradius 去做.

   我在第 4 步中一步一步试的过程中, 发现只要 /etc/pptpd.conf 中的参数是这样: noippram 和 delegate 注释掉, logwtmp 随意. /etc/ppp/options.pptpd 中继续留着这三行: 

       plugin radius.so
       plugin radattr.so
       radius-config-file /etc/radiusclient-ng/radiusclient.conf

   就是可以成功的, 说到底, 只要不把分配 IP 的工作委托给 freeradius, 连接就是可以成功的, 我测试过, 可以正常使用 freeradius 授权, 验证以及计时功能 (mysql radius 数据库里都写入了相关的内容).

   我现在仍有一点不明白的是, pptpd 是怎么把授权和身份验证的过程交给 freeradius 去做的 (其实不和 freeradius 配合时, pptpd 是怎么选择使用 chap 身份认证方式的我也不清楚), 其实相比以前不和 freeradius 配合的方式, 我们的配置的东西只有:

   /etc/pptpd.conf 中的 delegate, noippram 和 logwtmp 选项. /etc/ppp/options.pptpd 中的那三行. 所以委托 freeradius 对客户端进行身份验证以及授权的功能只能是 radius.so 和 radattr.so 这两个模块完成的了.

   但是, pptpd 以前自己是会使用 /etc/ppp/chap-secrets 文件对客户端进行身份验证的, 这个过程又是怎么被 "废除" 的呢? 难道说使用了 radius.so/radattr.so 模块之后, pptpd 就会自动不再使用自己的验证机制吗? 

   这点有待继续研究.

## 为什么在和 freeradius 配合时, /etc/pptpd.conf 中的 logwtmp 参数需要注释掉

delegate 和 noipparam 都是不注释, 但是为何 logwtmp 需要注释呢? 我发现, 如果不注释的话, 会有如下错误, 原因我暂时不知道.

但是如果 delegate 和 noipparam 注释了 (不委托 freeradius 给客户端分配 IP), 这时将 logwtmp 不注释, 是不会有这个错误的.

我猜可能是 logwtmp 的内容被 pppd (还是 pptpd?) 读取到引起的. 

    Dec 11 11:09:59 localhost pptpd[13024]: CTRL: Starting call (launching pppd, opening GRE)
    Dec 11 11:09:59 localhost pppd[13025]: Plugin radius.so loaded.
    Dec 11 11:09:59 localhost pppd[13025]: RADIUS plugin initialized.
    Dec 11 11:09:59 localhost pppd[13025]: Plugin radattr.so loaded.
    Dec 11 11:09:59 localhost pppd[13025]: RADATTR plugin initialized.
    Dec 11 11:09:59 localhost pppd[13025]: Plugin /usr/lib64/pptpd/pptpd-logwtmp.so loaded.
    Dec 11 11:09:59 localhost pppd[13025]: pptpd-logwtmp: $Version$
    Dec 11 11:09:59 localhost pppd[13025]: pppd 2.4.5 started by root, uid 0
    Dec 11 11:09:59 localhost pppd[13025]: Using interface ppp0
    Dec 11 11:09:59 localhost pppd[13025]: Connect: ppp0 <--> /dev/pts/8
    Dec 11 11:10:02 localhost pppd[13025]: peer from calling number 153.119.195.144 authorized
    Dec 11 11:10:02 localhost pppd[13025]: MPPE 128-bit stateless compression enabled
    Dec 11 11:10:02 localhost pppd[13025]: Unsupported protocol 'IPv6 Control Protocol' (0x8057) received
    Dec 11 11:10:02 localhost pppd[13025]: Could not determine local IP address
    Dec 11 11:10:02 localhost pppd[13025]: pptpd-logwtmp.so ip-down ppp0
    Dec 11 11:10:02 localhost pppd[13025]: Connect time 0.1 minutes.
    Dec 11 11:10:02 localhost pppd[13025]: Sent 102 bytes, received 116 bytes.
    Dec 11 11:10:02 localhost pppd[13025]: MPPE disabled
    Dec 11 11:10:02 localhost pppd[13025]: Connection terminated.
    Dec 11 11:10:02 localhost pppd[13025]: Connect time 0.1 minutes.
    Dec 11 11:10:02 localhost pppd[13025]: Sent 142 bytes, received 120 bytes.
    Dec 11 11:10:02 localhost pppd[13025]: Exit.
    Dec 11 11:10:02 localhost pptpd[13024]: GRE: read(fd=6,buffer=6124a0,len=8196) from PTY failed: status = -1 error = Input/output error, usually caused by unexpected termination of pppd, check option syntax and pppd logs
    Dec 11 11:10:02 localhost pptpd[13024]: CTRL: PTY read or GRE write failed (pty,gre)=(6,7)
    Dec 11 11:10:02 localhost pptpd[13024]: CTRL: Client 153.119.195.144 control connection finished

## 参考

1. freeradius wiki 主页上的关于集成 pptp 和 freeradius/radiusclient-ng 的介绍: http://wiki.freeradius.org/guide/PopTop-HOWTO
2. 这位博主很全面的介绍了搭建 freeradius 及其配置, 我第一次安装 freeradius 就是看这位博主的文章安装的, 感谢: http://www.cnblogs.com/klobohyz/archive/2012/02/01/2334811.html
3. http://www.cnblogs.com/klobohyz/archive/2012/02/04/2338675.html
4. pptpclient 项目主页
5. freeradius 和 SQL 的介绍: http://wiki.freeradius.org/guide/SQL-HOWTO
