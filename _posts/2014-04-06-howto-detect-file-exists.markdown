---
layout: post
title: 在 Linux C 程序中如何正确的判断一个文件/目录文件是否存在
date: 2014-04-06 14:59
---

其实不光是在 Linux 下编程, 在其他平台下我们都会有这样的需求: 我们要为应用程序创建自己的数据或者日志目录, 应用程序在每次启动时会检查文件系统中是否已经有了自己的目录, 没有的话就创建它, 有了的话就跳过这一步. 那么如何去判断文件系统中是否已经存在了要创建的目录呢?

Linux 或者 GNU C 都没有提供一个像 file\_exists() 这样直观的系统调用给我们, 所以我们得通过其它的调用来达成这个目标. 

实际上当我第一次要解决这个问题时, 我先 google 了一下, 这个问题在 stackoverflow 上有人问过而且非常受欢迎, 很多人对这个问题又点赞又收藏的, 自然, 这个问题也收到了不少好的答案, 这篇文章算是对这些好答案的总结和延伸.

我们先来看一个大家都应该知道的方式, 第一种方式:

# fopen()

fopen() 方法是流阶级的方法, 这个方法接收用户提供的文件名, 以及访问方式, 然后尝试着打开文件, 打开成功则返回 handle, 失败则返回 NULL. 因此有人提出了使用这个方法来判断指定的文件是否存在的方案:

{% highlight c %}
    #include <stdio.h>

    ...

    FILE *fp = NULL;
    fp = fopen("/tmp/test/somefile", "r");

    if(fp) {
      // exists
    } else {
      // not exists
    }

    fclose(fp);

    ...
{% endhighlight %}

这也是 stackoverflow 上唯一一个得负分的答案, 这个方案的问题在于它没有考虑到文件权限的问题, 而 fopen() 这个函数又是如此的简单 --- 不管因为什么原因打开文件失败了, 它只是返回 NULL 给你, 不会提供更多的错误信息. 

如果文件存在, 而只是你没有对这个文件的读权限, 那么你同样会得到 NULL 返回值, 而你又不能获得其它导致失败的原因, 于是你想当然的认为这个文件不存在, 于是错误就发生了. 下面的两种情况都能够导致你打开失败:

1.  你对 test 目录没有 x 权限
2.  你对 somefile 没有 r 权限

这两种情况下, 显然文件是存在的, 但是我们却得到了 NULL 返回值. 

# open()

一个改进的放案是使用 open() 系统调用, 这是处于 fopen() 底层的调用, 它提供了丰富的出错信息, 以便于你能够检查出错的原因. 这个方案如下:

{% highlight c %}
    #include <fcntl.h>
    #include <errno.h>

    ...

    fd = open(pathname, O_RDONLY);
    if(fd < 0) {
        switch (errno) {
            case EACCES:  // you don't have permission
            break;
            case ENOENT:  // the file doesn't exists
            break;
            default:
            break;
        }
    } else {
        // use the file
    }

    close(fd);
{% endhighlight %}

使用 open() 调用能够帮你完成很多其它的额外的功能, 比如说在文件不存在的时候创建它, 等等.

看起来 open() 的解决方案已经足够了, 但是, 说到底 open() 是需要打开一个文件的, 可能你只是想检查文件是否存在, 而并不想读取它的内容, 这样打开操作就带来了不必要的工作. 如果仅仅是想检查文件是否存在, 或者是否对文件有读, 写, 执行权限的话, 我们还有另一种更好地选择:

# access()

access() 调用以一种更明朗的方式专门检查文件是否存在, 文件是否可读, 可写, 可执行. 不过需要注意的是, access() 在检查文件是否存在以及程序对文件是否具有读写执行权限时, 使用的是程序的实际用户 ID, 而不是有效用户 ID. 实际上这个特点对于 "setuid 化" 的程序是很有用的, 因为 "setuid 化" 的程序可能常常会检查实际用户对某一文件是否具有响应的权限.

使用 access() 来检查文件是否存在的代码如下:

{% highlight c %}
    if( access( fname, F_OK ) != -1 ) {
        // file exists
    } else {
        switch(errno) {
            case EACCES:
                break;
            case ENOENT:
                break;
        }
    }
{% endhighlight %}

access() 在失败时也会通过 errno 提供错误信息, 当你对要检查的文件的父目录没有 x 权限时, 会产生 EACCES; 当要访问的文件不存在时, 会产生 ENOENT.

# stat()

以上几种方法, 都只是根据我们指定的文件名来判断这个文件是否存在, 而不管它是一般文件还是目录文件, 如果我们不仅要确认一个文件存在, 还要确认它是目录文件, 那上面几种方法就不能满足了, 这时候我们可以用 stat() 调用:

{% highlight c %}
    struct stat st_stat = {0};
    int ret = stat(DBDIR, &st_stat);
    ///
    // 如果 stat 调用失败不是由于文件不存在导致的, 那么直接返回
    //
    if(ret && errno != ENOENT) {
      fprintf(stderr, "Check directory error: %s\n", strerror(errno));
      return 1;
    }
  
    ///
    // 如果 stat() 调用失败是由于目录不存在, 就创建目录
    // 如果 stat() 调用没有失败, 但是已经存在的那个文件不是目录文件, 也创建它
    //
    if((ret && errno == ENOENT) || (! ret && ! S_ISDIR(st_stat.st_mode))) {
      ///
      // 创建目录并赋予其 rwxr-xr-x 权限
      //
      if(mkdir(DBDIR, S_IRWXU | S_IRGRP | S_IXGRP | S_IROTH | S_IXOTH)) {
        fprintf(stderr, "Crate directory error: %s\n", strerror(errno));
  
        return 1;
      }
    }
{% endhighlight %}

# 总结

1.  需要注意的是, 以上四个函数调用都会收到一个共同的限制: 如果要检查的文件/目录文件的父目录没有 x 权限, 那么都会产生 EACCES 错误或者返回 NULL(fopen).

# stackoverflow 上的链接

[http://stackoverflow.com/questions/230062/whats-the-best-way-to-check-if-a-file-exists-in-c-cross-platform](http://stackoverflow.com/questions/230062/whats-the-best-way-to-check-if-a-file-exists-in-c-cross-platform)
