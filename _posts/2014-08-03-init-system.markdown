---
layout: post
title: "init 系统简史"
date: 2014-08-03 15:30
---

@(Linux 系统管理员笔记)

在基于 Unix 的计算机系统中, init 是系统启动过程中的第一个进程 (在内核态中, 是不存在进程的概念的, init 是由内核态进入用户态后的第一个进程), 它负责启动系统中其它的进程, 系统中所有的进程都是由 init 直接, 或者间接启动的. 并且, 所有的进程在成为孤儿之后, 都会默认的成为 init 的子进程. 同时, init 还是一个 daemon 进程.

init 系统发展于 Unix, 在 Unix 的发展出现分歧时[1], init 的设计思路也出现了分歧, 分为偏 System V 的和偏 BSD 的. Linux 刚出道时, 绝大多数的 Linux 发行版都使用 System V 的风格, 除了个别的例外 --- Slackware 使用 BSD 风格的 init, Gentoo 自成一格. 而如今, 在 Linux 阵营里, 已经出现很多 System V init 的替代品了, 它们比 sysvinit 更强大, 更现代. 不同的发行版也慢慢开始分别采用这些更先进的 init 系统, 比如 systemd, upstart, openrc 等.

# 学术派/BSD 风格的 init 系统

学术派阵营的 init 中, 没有 runlevel 的概念, /etc/rc 脚本决定了 init 进程要做的一切. 这个 init 系统的好处是非常简单, 只要修改 /etc/rc 就能做任何想做的事情; 缺点也是显而易见的, 要添加个新软件就得修改 /etc/rc, 搞得最后人人都要修改 /etc/rc, 不够安全, 一旦有人在 /etc/rc 里写漏了分号什么的, 系统就会无法启动.

学术派阵营中的一方势力, BSD, 在 4.3 版本之后走了一步革新把其它学术派阵营的基友甩了一条街. BSD 4.3 以后, 添加了一个 /etc/rc.local 脚本, 让用户要加东西在 /etc/rc.local 里面加, 不要在 /etc/rc 里面加, 同时在 /etc/rc 里起一个 sub shell 将 /etc/rc.local 作为子进程执行, 这样就算 /etc/rc.local 里面写错了东西少了分号啥的, 也不影响 /etc/rc 的运行.

后来从 BSD 分出去的 NetBSD, 开发了一套更先进的, "模块化" 的 init 系统, 加了一个 /etc/rc.d 目录, 里面包含所有的启动脚本, init 进程会去执行这些启动脚本. 这些脚本的格式是有规定的, 脚本之间有依赖标记, init 在执行这些脚本的时候要参考这些依赖信息[2], FreeBSD 后来也采用了 NetBSD 发明的这套系统.

# System V 风格的 init 系统

sysvinit 引入了运行级别 (runlevel) 的概念, init 进程会先去检查 /etc/inittab 文件, 找到 `:initdefault:` 这一行, 取得默认的运行级别, 如果没有 `:initdefault:` 这一行或者没有定义默认的运行级别, init 就会暂停, 让用户输入要进入的运行级别.

## 运行级别

在 sysvinit 中, 运行级别定义了主机的状态, 不同的发行版实现的运行级别数可能不同, 但是下面三个是 sysvinit 规定的级别:

* 0. Halt
* 1. Single user mode (也叫做 S 级别, 或者 s 级别)
* 6. Reboot

## sysvinit 所能接收的信号

前面说了 init 进程还是一个 daemon 进程, 那么与 daemon 进程的通信, 最简单方便的就是通过信号机制, init 进程能够接受如下的信号:

* SIGHUP

  与 `telinit q` 命令的效果相同.

* SIGUSR1

  收到这个信号后, init 进程会关闭然后再打开 /run/initctl, 这是个管道文件.

* SIGINT

  当按下 CTRL-ALT-DEL 时, 内核会发送这个信号给 init 进程, 进而触发 /etc/inittab 中的 ctrlaltdel 行为.

* SIGWINCH

  当 KeyboardSignal 键被按下时, 内核会发送这个信号给 init 进程, 进而触发 /etc/inittab 中的 kbrequest 行为.

## 与 init 进程有交互的程序

* telinit

  切换运行级别可以借助 telinit 程序, 它是 init 程序的软连接. 所以执行 telinit 就是执行 init 程序自己, 只不过, init 程序会根据 process id 来判断他自己是被内核执行的, 还是被用户执行的, 从而表现出不同的行为. 内核执行的 init 进程的 process id 永远是 1, 用户通过 telinit 执行的 init 进程 process id 不会是 1.

* shutdown

  shutdown 程序通过信号机制与 init 交流. 比如 `shutdown -h` 命令会发信号给 init 进程让它进入 0 运行级别, `shutdown -r` 则会让 init 进程进入 6 运行级别.

# 后生可畏

* busybox

  为嵌入式 Linux 系统定制的, openwrt 用它.

* systemd

  a full replacement for init with parallel starting of services and other features, used by many distributions.

* upstart

  a full replacement of init designed to start processes asynchronously initiated by Ubuntu.

# 参考

1. 这部分可以查一下 Unix 的发展历史, Unix 在发展过程中出现了商业化版本与学术派版本, 学术派的版本仍然是一贯的分享研究精神, 商业化版本则被开源人士所诟病. Unix 的商业化版本主要是 AT&T 在搞, 以 System III, System V 为代表; 学术派版本则是以 VAX, DEC, 以及著名的 BSD 为代表. 这也是为什么有一个说法是 "用 BSD 的看不起用 Linux 的" 的原因, 因为绝大多数的 Linux 发行版在系统设计的很多方面都是和 System V 一个风格...

2. 而在 System V init 系统里, 也有个和 /etc/rc.d 类似的目录, 就是 /etc/init.d, sysvinit 在执行 /etc/init.d 里的脚本时是通过软连接的名字顺序执行, 不是靠依赖关系.

3. Init 维基页: http://en.wikipedia.org/wiki/Init

4. 强烈建议看一下原生的 sysvinit 的 man 手册页, 在下面.

# sysvinit 的 man 手册

由于多数的 Linux 已经不再采用原生的 sysvinit, 所以在一般的 Linux 系统是运行 `man init` 已经看不到原生的 sysvinit 的 manual page 了, 所以我把 sysvinit 原生 man page 发在这里了. 我之所以敢把它挪到这里, 是因为 man page 中的内容是可以自由发布和修改的. 见这里: https://www.kernel.org/doc/man-pages/licenses.html

> INIT(8)         Linux System Administrator's Manual        INIT(8)
> 
> NAME
> 
> 
>        init, telinit - process control initialization
> 
> SYNOPSIS
> 
> 
>        /sbin/init [ -a ] [ -s ] [ -b ] [ -z xxx ] [ 0123456Ss ]
>        /sbin/telinit [ -t sec ] [ 0123456sSQqabcUu ]
> 
> DESCRIPTION
> 
> 
>   Init
>        Init  is  the  parent  of all processes.  Its primary role is to create
>        processes from a script stored in  the  file  /etc/inittab  (see  inittab(5)
>        ).   This file usually has entries which cause init to spawn get-
>        tys on each line that users can log in.  It  also  controls  autonomous
>        processes required by any particular system.
> 
> RUNLEVELS
> 
> 
>        A  runlevel is a software configuration of the system which allows only
>        a selected group of processes to exist.  The processes spawned by  init
>        for each of these runlevels are defined in the /etc/inittab file.  Init
>        can be in one of eight runlevels: 0-6 and S  or  s.   The  runlevel  is
>        changed  by having a privileged user run telinit, which sends appropri-
>        ate signals to init, telling it which runlevel to change to.
> 
>        Runlevels 0, 1, and 6 are reserved. Runlevel 0 is used to halt the sys-
>        tem, runlevel 6 is used to reboot the system, and runlevel 1 is used to
>        get the system down into single user mode. Runlevel  S  is  not  really
>        meant  to  be used directly, but more for the scripts that are executed
>        when entering runlevel 1. For more information on this,  see  the  man-
>        pages for shutdown(8) and inittab(5).
> 
>        Runlevels  7-9  are  also  valid, though not really documented. This is
>        because "traditional" Unix variants don't use  them.   In  case  you're
>        curious,  runlevels  S and s are in fact the same.  Internally they are
>        aliases for the same runlevel.
> 
> BOOTING
> 
> 
>        After init is invoked as the last step of the kernel boot sequence,  it
>        looks for the file /etc/inittab to see if there is an entry of the type
>        initdefault (see inittab(5)). The initdefault entry determines the ini-
>        tial  runlevel  of  the  system.   If  there  is  no  such entry (or no
>        /etc/inittab at all), a runlevel must be entered at the system console.
> 
>        Runlevel S or s bring the system to single user mode and do not require
>        an /etc/inittab file.  In single user mode, a root shell is  opened  on
>        /dev/console.
> 
>        When entering single user mode, init initializes the consoles stty set-
>        tings to sane values. Clocal mode is set. Hardware speed and  handshak-
>        ing are not changed.
> 
>        When  entering  a multi-user mode for the first time, init performs the
>        boot and bootwait entries to allow file systems to  be  mounted  before
>        users  can  log  in.   Then  all entries matching the runlevel are pro-
>        cessed.
> 
>        When starting a  new  process,  init  first  checks  whether  the  file
>        /etc/initscript  exists.  If  it does, it uses this script to start the
>        process.
> 
>        Each time a child terminates, init records the fact and the  reason  it
>        died  in  /var/run/utmp  and  /var/log/wtmp,  provided that these files
>        exist.
> 
> CHANGING RUNLEVELS
> 
> 
>        After it has spawned all of the processes specified, init waits for one
>        of  its descendant processes to die, a powerfail signal, or until it is
>        signaled by telinit to change the system's runlevel.  When one  of  the
>        above  three  conditions  occurs, it re-examines the /etc/inittab file.
>        New entries can be added to this file at any time.  However, init still
>        waits  for  one of the above three conditions to occur.  To provide for
>        an instantaneous response, the telinit Q or q command can wake up  init
>        to re-examine the /etc/inittab file.
> 
>        If  init  is  not  in  single user mode and receives a powerfail signal
>        (SIGPWR), it reads the file /etc/powerstatus. It then starts a  command
>        based on the contents of this file:
> 
>        F(AIL) Power is failing, UPS is providing the power. Execute the power-
>         wait and powerfail entries.
> 
>        O(K)   The power has been restored, execute the powerokwait entries.
> 
>        L(OW)  The power is failing and the UPS has a low battery. Execute  the
>         powerfailnow entries.
> 
>        If  /etc/powerstatus  doesn't  exist or contains anything else then the
>        letters F, O or L, init will behave as if it has read the letter F.
> 
>        Usage of SIGPWR and /etc/powerstatus is discouraged. Someone wanting to
>        interact  with  init  should use the /dev/initctl control channel - see
>        the source code of the sysvinit package for  more  documentation  about
>        this.
> 
>        When  init  is  requested  to change the runlevel, it sends the warning
>        signal SIGTERM to all processes that are undefined in the new runlevel.
>        It then waits 5 seconds before forcibly terminating these processes via
>        the SIGKILL signal.  Note that init assumes that  all  these  processes
>        (and  their  descendants)  remain  in the same process group which init
>        originally created for them.  If any process changes its process  group
>        affiliation  it will not receive these signals.  Such processes need to
>        be terminated separately.
> 
> TELINIT
> 
> 
>        /sbin/telinit is linked to /sbin/init.  It takes a one-character  argu-
>        ment and signals init to perform the appropriate action.  The following
>        arguments serve as directives to telinit:
> 
>        0,1,2,3,4,5 or 6
>         tell init to switch to the specified run level.
> 
>        a,b,c  tell init to process only those /etc/inittab file entries having
>         runlevel a,b or c.
> 
>        Q or q tell init to re-examine the /etc/inittab file.
> 
>        S or s tell init to switch to single user mode.
> 
>        U or u tell  init  to  re-execute itself (preserving the state). No re-
>         examining of /etc/inittab file happens. Run level should be  one
>         of Ss12345, otherwise request would be silently ignored.
> 
>        telinit can also tell init how long it should wait between sending pro-
>        cesses the SIGTERM and SIGKILL signals.  The default is 5 seconds,  but
>        this can be changed with the -t sec option.
> 
>        telinit can be invoked only by users with appropriate privileges.
> 
>        The  init binary checks if it is init or telinit by looking at its pro-
>        cess id; the real init's process id is always 1.  From this it  follows
>        that instead of calling telinit one can also just use init instead as a
>        shortcut.
> 
> ENVIRONMENT
> 
> 
>        Init sets the following environment variables for all its children:
> 
>        PATH   /bin:/usr/bin:/sbin:/usr/sbin
> 
>        INIT_VERSION
>         As the name says. Useful to determine if a script runs  directly
>         from init.
> 
>        RUNLEVEL
>         The current system runlevel.
> 
>        PREVLEVEL
>         The previous runlevel (useful after a runlevel switch).
> 
>        CONSOLE
>         The  system  console.  This is really inherited from the kernel;
>         however if it is not set init will set  it  to  /dev/console  by
>         default.
> 
> BOOTFLAGS
> 
> 
>        It  is possible to pass a number of flags to init from the boot monitor
>        (eg. LILO). Init accepts the following flags:
> 
>        -s, S, single
>       Single user mode boot. In this mode /etc/inittab is  examined  and
>       the  bootup rc scripts are usually run before the single user mode
>       shell is started.
> 
>        1-5  Runlevel to boot into.
> 
>        -b, emergency
>       Boot directly into a single user shell without running  any  other
>       startup scripts.
> 
>        -a, auto
>       The  LILO  boot loader adds the word "auto" to the command line if
>       it booted the kernel with the default command line  (without  user
>       intervention).  If this is found init sets the "AUTOBOOT" environ-
>       ment variable to "yes". Note that you  cannot  use  this  for  any
>       security  measures - of course the user could specify "auto" or -a
>       on the command line manually.
> 
>        -z xxx
>       The argument to -z is ignored. You can use this to expand the com-
>       mand  line  a  bit, so that it takes some more space on the stack.
>       Init can then manipulate the command line so that ps(1) shows  the
>       current runlevel.
> 
> INTERFACE
> 
> 
>        Init  listens  on  a fifo in /dev, /dev/initctl, for messages.  Telinit
>        uses this to communicate with init. The interface is not very well doc-
>        umented  or  finished. Those interested should study the initreq.h file
>        in the src/ subdirectory of the init source code tar archive.
> 
> SIGNALS
> 
> 
>        Init reacts to several signals:
> 
>        SIGHUP
>       Has the same effect as telinit q.
> 
>        SIGUSR1
>       On receipt of this signals, init closes and re-opens  its  control
>       fifo, /dev/initctl. Useful for bootscripts when /dev is remounted.
> 
>        SIGINT
>       Normally the kernel sends this signal to init when CTRL-ALT-DEL is
>       pressed. It activates the ctrlaltdel action.
> 
>        SIGWINCH
>       The  kernel  sends this signal when the KeyboardSignal key is hit.
>       It activates the kbrequest action.
> 
> CONFORMING TO
> 
> 
>        Init is compatible with the System V init. It  works  closely  together
>        with  the  scripts  in  the  directories  /etc/init.d  and /etc/rc{run-
>        level}.d.  If your system uses  this  convention,  there  should  be  a
>        README  file  in the directory /etc/init.d explaining how these scripts
>        work.
> 
> FILES
> 
> 
>        /etc/inittab
>        /etc/initscript
>        /dev/console
>        /var/run/utmp
>        /var/log/wtmp
>        /dev/initctl
> 
> WARNINGS
> 
> 
>        Init assumes that processes and descendants of processes remain in  the
>        same  process group which was originally created for them.  If the pro-
>        cesses change their group, init can't kill them and you may end up with
>        two processes reading from one terminal line.
> 
> DIAGNOSTICS
> 
> 
>        If  init finds that it is continuously respawning an entry more than 10
>        times in 2 minutes, it will assume that there is an error in  the  com-
>        mand  string,  generate  an  error  message  on the system console, and
>        refuse to respawn this entry until either 5 minutes has elapsed  or  it
>        receives  a  signal.   This prevents it from eating up system resources
>        when someone makes a typographical error in the  /etc/inittab  file  or
>        the program for the entry is removed.
> 
> AUTHOR
> 
> 
>        Miquel  van  Smoorenburg  (miquels@cistron.nl),  initial manual page by
>        Michael Haardt (u31b3hs@pool.informatik.rwth-aachen.de).
> 
> SEE ALSO
> 
> 
>        getty(1), login(1), sh(1),  runlevel(8),  shutdown(8),  kill(1),  inittab(5)
>        , initscript(5), utmp(5)
> 
