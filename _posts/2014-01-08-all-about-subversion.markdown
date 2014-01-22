---
layout: post
title: 关于 svn 的一切
date: 2014-01-08 16:56
---

### 待补充的主题...

### 一些概念

#### 工作目录是怎样工作的 _(How the working copy works)_

对于工作目录中的每一个文件, Subversion 会记录这个文件的许多信息, 其中有两条信息很重要: 

*   What revision your working file is based on ( this is called the file's _working revision_)
*   A timestamp recording when the local copy was last updated by the repository

凭着这两条信息, 再通过与仓库(_repository_)的对话, Subversion 能够区分出一个文件所处的状态, 可能的状态有四种: 

*   Unchanged, and current

    The file is unchanged in the working directory, and no changes to that file have been committed to the repository since its working revision. An svn commit of the file will do nothing, and an svn update of the file will do nothing.

*   Locally changed, and current

    The file has been changed in the working directory, and no changes to that file have been committed to the repository since you last updated. There are local changes that have not been committed to the repository; thus an svn commit of the file will succeed in publishing your changes, and an svn update of the file will do nothing.

*   Unchanged, and out of date

    The file has not been changed in the working directory, but it has been changed in the repository. The file should eventually be updated in order to make it current with the latest public revision. An svn commit of the file will do nothing, and an svn update of the file will fold the latest changes into your working copy.

*   Locally changed, and out of date

    The file has been changed both in the working directory and in the repository. An svn commit of the file will fail with an "out-of-date" error. The file should be updated first; an svn update command will attempt to merge the public changes with the local changes. If Subversion can't complete the merge in a plausible way automatically, it leaves it to the user to resolve the conflict.


#### 推荐的仓库结构

Most projects have a recognizable “main line”, or trunk, of development; some branches, which are divergent copies of development lines; and some tags, which are named, stable snapshots of a particular line of development. 

因此, 首先, 我们建议, 每个项目在仓库中都有一个项目根目录(_project root_), 这个目录包含所有与这个项目相关, 并且只与这个项目相关的版本信息. 第二, 我们建议每一个项目根目录包含一个 trunk 子目录作为项目的主线, 一个 branches 子目录作为分支的创建目录, 还有一个 tags 子目录作为快照的创建目录. 当然, 如果说你的仓库只有一个项目, 那么仓库根目录可以就是项目的根目录.


#### Make Your Changes
你对工作目录的操作可以归纳为两类: file changes and tree changes. 

### svn 管理员

下面我总结一下一个 svn 管理员日常的工作, 英文内容摘自 svn 公开的自由书, 所以不存在版权纠纷, 之所以没有翻译成中文绝对不是因为懒, 而是因为怕自己翻译的不好, 翻译过来会无法完美的表达愿意. (PS: 直接复制英文过来不用翻译真是太爽了! PS2: 绝对不是因为懒才没翻译!)

#### 创建一个仓库

    svnadmin create /var/svn/repos # 创建一个仓库, 默认为 FSFS 存储
    svnadmin create --fs-type bdb /var/svn/repos 

>   Both svnadmin and svnlook are considered server-side utilities—they are used on the machine where the repository resides to examine or modify aspects of the repository, and are in fact unable to perform tasks across a network. A common mistake made by Subversion newcomers is trying to pass URLs (even “local” file:// ones) to these two programs.

#### Dump 仓库的内容至 stdout

    svnadmin dump /var/svn/repos > full.dump
    svnadmin dump /var/svn/repos -r 21 --incremental > incr.dump

#### 将 dump 的内容加载到仓库中

    svnadmin load REPOS_PATH < DUMPED_FILE

#### 仓库热拷贝 (hotcopy)

>   You can run this command at any time and make a safe copy of the repository, regardless of whether other processes are using the repository.

#### 设置仓库的 UUID

>   If you've svnsynced /var/svn/repos to /var/svn/repos-new and intend to use repos-new as your canonical repository, you may want to change the UUID for repos-new to the UUID of repos so that your users don't have to check out a new working copy to accommodate the change:  
>
>     svnadmin setuuid /var/svn/repos-new 2109a8dd-854f-0410-ad31-d604008985ab
>
>   As you can see, svnadmin setuuid has no output upon success.

#### 查看仓库结构

    $ svnlook tree      # 可以查看递归创建的 svn 仓库的结构
    $ svnlook -N tree   # 可以非递归的查看 svn 仓库结构

#### 升级你的仓库

    svnadmin upgrade REPOS_PATH - Upgrade a repository to the latest supported schema version.

#### svnlook

要强调的一点是, svnlook 并不会对仓库做任何改变, 他仅仅是做 "peeking" 操作. 通常 svnlook 被 hooks 使用.

#### 配置 svn 服务器 - svnserve

有很多种方式可以来启动 svnserve 服务器程序

* Run svnserve as a standalone daemon, listening for requests.
* Have the Unix inetd daemon temporarily spawn svnserve whenever a request comes in on a certain port.
* Have SSH invoke a temporary svnserve over an encrypted tunnel.
* Run svnserve as a Microsoft Windows service.
* Run svnserve as a launchd job.

我们来看看这几种方法: 

##### svnserve as daemon

    $ svnserve -d       # svnserve is now running, listening on port 3690, you can use the --listen-port and --listen-host to change the default listening port and hostname

用这条命启动 svnserve 后, 你的系统中的所有的 repositoty 都能够从网络上访问. 客户端指定某一个 repository 的绝对地址就可以访问这个 repository. 比如, 一个 repository 在 /srv/svn/project1, 那么客户端可以通过如下地址访问: svn://your-host-address/var/svn/project1. 你可以指定 -r 选项指定一个目录来仅 "输出" 此目录下的 repositories, 比如:

    $ svnserve -d -r /srv/svn

这样, 客户端就可以通过 svn://your-host-address/project1 来访问了

#### 停止 svnserve 服务器

很不幸, svnserve 命令并没有提供一种"优雅"的方式来终止 svnserve daemon, "优雅"的方式就是 kill -9 or kill -KILL

### 往仓库里导入文件

很多情况下, 在建立仓库之前我们已经事先有了一堆文件了, 那么建立仓库之后我们可以使用 `svn import` 命令来将已有的那堆文件纳入仓库中. 像这样:

    $ svn import project1_dir/ svn://localhost/project1/trunk

仓库中的父级目录会自动创建, 如果我们的项目下有多个子项目, 那么我们的 import 命令可能是这样的:

    $ svn import project1_dir/sub1 svn://localhost/project1/sub1/trunk
    $ svn import project1_dir/sub2 svn://localhost/project1/sub2/trunk

### 附: 我创建 svn 仓库的规则, 以及一般过程

1.  所有的仓库都放在 /srv/svn-repos 目录下
2.  每个项目单独建立一个目录, 比如 /srv/svn-repos/project1
3.  如果项目可分为几个子项目, 不必为子项目建立单独的目录, 而是把子项目的目录树也交给 svn 管理, 比如, 我们不必如此: /srv/svn-repos/project1/subproject1
4.  如果项目肯分为几个子项目, 每个子项目下要至少创建 trunk, tags, branches 这几个分支, 当然这些分支目录也不需要我们创建, 把他们也交给 svn 管理; 如果项目不必分为子项目, 那么直接在项目下创建上述几个分支
5.  svnserve 的根路径设为 /srv/svn-repos, 或者 /srv/svn-repos/project1, 但是不要使用默认的 "/" 路径


#### 创建 svn 仓库的一般过程

*   创建项目目录

        # mkdir /srv/svn-repos/project2

*   建立 svn 目录结构

        # svnadmin create /srv/svn-repos/project1

*   运行 svnserve 服务

        # svnserve -d -r /srv/svn-repos

*   配置

        # anon-access = read
        # auth-access = write
        # password-db = passwd
        前的注释和空格, 并在 conf/passwd 文件 [users] 节下添加一行
        cifer = {password}

#### 配置 apache/dav\_svn
