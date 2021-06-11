---
title: 也谈 Python 与 Unicode
date: 2017-04-24
categories:
  - Python
tags:
  - python
  - unicode
---

## Unicode 与 UTF8 的关系

unicode 定义了统一的字符集, UTF8 则是一种编码 unicode 字符集的方式.

在 python2 中, `str` 类型, `unicode` 类型都是 `basestring` 的子类, 其中 `str` 类型就好比 C 语言中的字符串, `unicode` 类型就好比 C 语言中的宽字符串. 因为 UTF8 的编码方式不使用 0x0 填充, 一串 UTF8 字符流实际上也是一串合法的 C 字符流 (0x0 结尾) --- 也就是合法的 `str` 类型的字符流.

## 编程相关的编码问题

关于编程语言的编码问题, 可能会涉及到这么几个内容:

1. 源文件自身的编码
2. 编辑器/IDE 所理解的源文件的编码方式
3. 编译器/解释器对源代码中字符常量的处理方式

下文我们会一一说明.

## 源文件自身的编码

源文件自身的编码是由谁决定的呢? 实际是我们自己决定的, 还记得保存文件时弹出的对话框吗? 对话框中有一个选项就是文件的格式, 可能大部分人没有关注, 如果我们不去更改它的话, 那文件就以操作系统的默认编码格式存储了. 通常这在 Linux, MacOS 上是 UTF8, 某些中文 Windows 系统上是 GBK.

在 vim 中我们通过 `set fileencoding=utf8` 选项来告诉 vim 应该把文件保存成 UTF8 格式 --- 也就是将每个字符按照其 UTF8 中定义的 code point 存到磁盘上.

## 编辑器/IDE 所理解的源文件的编码方式

纯文本文件是没有文件头信息的(忽略 BOM 这个异类), 编辑器打开文本文件的时候实际上不知道他是什么格式, 只能以默认格式解读它. 上面说了 Linux 下默认格式是 UTF8, 而有些中文 Windows 系统默认的文本格式是 GBK, 所以在这些中文 Windows 系统中创建的文本文件, 到 Linux 系统中打开可能会发现是乱码, 这是我们就得人为告诉编辑器这个文件的编码格式, 比如在 vim 中可以通过 `:e ++enc=gbk` 命令.

## 屏幕上显示的字符

注意我们最终看到的字符是在显示器看到的, 而因为 vim 只是个编辑器, 他本身并不负责文本在显示器上的显示, 负责将文本输出到显示器(显卡)的是你机器上的 X 服务程序, 而 vim 又是怎么和 X 服务程序通信的呢? 实际上这中间还有个 X 客户程序, X 客户程序是什么呢? 就是虚拟终端程序, 比如 xterm.

虚拟终端为我们和我们的应用程序提供了标准输入和输出, 使得我们能够与应用程序交流. 在我们使用 vim 编辑文本的时候, 输入的字符以及命令通过虚拟终端的标准输入传给 vim, 而 vim 则将结果通过标准输出展示给我们.

另外要注意的是, `:e ++enc=gbk` 只是告诉 vim, 这个文件的格式是 gbk, 这样一来 vim 才能够正确的理解该文件的格式, 但是在 vim 内部对字符的处理, vim 输出到标准输出 xterm, 以及 xterm 到 X 服务程序之间的通信, 全部无一例外走的是 utf8 格式, 这是 Linux 生态传统. 我们当然可以配置 xterm 的字符格式, 也可以更改 vim 内部处理字符的方式 (这两件事我没试过, 我猜是可以的, 至少通过改它们源码一定是可以的), 但这当然是强烈不推荐的.

也就是说, `:e ++enc=gbk` 之后, 只是告诉 vim, 在读入文件的时候按照 gbk 的编码来理解这些字符, 然后这每个被理解的字符在 vim 内部都会转换成对应的 utf8 编码的字符, 后续输出到标准输出 (xterm) 时输出的就是这些 utf8 字符, 接着 xterm 这边只要你没有闲的蛋疼改配置那它一定也是默认理解 utf8 字符的, xterm 会将这些字符, 字符的编码方式 (这里是 utf8), 以及采用的字体, 字号大小等信息拼成一张指令单, 告诉 X 服务程序, 让 X 服务程序按照这个指令单渲染这些字符到显示器上, X 服务程序就会解析出每个字符的 code point, 字号等信息, 然后到字库中找对应这个 code point 的 glyph, 正确的输出到显示器上.

如果你没有 `:e ++enc=gbk`, vim 自然会以为文本格式就是 utf8, 按照 utf8 的编码方式来理解文本内容, 结果巧了 gbk 的编码方式被解读城 utf8 的话, code point 正好落入了 utf8 编码的乱码区, 于是这些乱码信息就被 vim 传给了 xterm 进而传给了 X 服务程序.

上面是输出过程, 下面我们看一看输入过程.

明白了输出过程, 输入过程就很好理解了, 你在 XIM 软件 (输入法软件) 里打字, 一般会有选字框的对吧, 选好字 (这个过程在写输入法的人口中被称作 "上屏幕") 之后, 这些字的 code point 就被发送给 xterm, 还记得吗 Linux 生态系统 utf8 是官定编码格式, XIM 软件也是 X 客户程序之一, 所以发给 xterm 的 code point 也是 utf8 格式. xterm 收到这些 code point 之后给谁呢? 当然是经由标准输入给到了 vim, 然后后面的事情就和上面一样了: vim 将这些字符通过标准输出给回到 xterm, 进而给到 X 服务程序渲染到显示器上.

可以看到, 只要你不要闲的没事瞎折腾 X 系统的编码, 那么输入过程是很难出现乱子的, 这也是为什么你可能打开一个文件是乱码, 但是你继续在这个文件中输入内容却并不是乱码的原因:

.. image:: images/messy-code-test.png

## 编译器/解释器对源代码中字符常量的处理方式

(以下内容针对 python2)

前面说的纯文本文件是不包含 meta 信息的, 几乎所有编程语言的源代码文件都是纯文本文件, `.py` 文件当然也不例外, 连 vim 这样的专业编辑器都没法知道源文件的编码格式, python 解释器肯定也是不知道的, 所以 python 也是选用了一种默认的编码方式来解释 `.py` 源文件, 与编辑器程序不同的是, python 默认认为源文件是 ascii 编码的, 因为程序代码一般都是英文字符嘛, 所以你如果写了这么一个 python 源文件:

::

    #!/usr/bin/env python

    a = '你好, 世界'
    print a

并保存为 utf8 格式, 然后执行, 会发生下面的错误:

::

    $ ls
    test.py
    $ python test.py
      File "test.py", line 3
    SyntaxError: Non-ASCII character '\xe4' in file test.py on line 3, but no encoding declared; see http://python.org/dev/peps/pep-0263/ for details

这就是因为 python 解释器按照 ascii 的编码方式解读源文件, 结果突然读到一个字节 `\xe4` 发现这个字节并不在 ascii 范围内, 与就报错退出了. 这个过程和 vim 其实是类似的, 只不过 vim 不会报错退出, 而是 "将错就错", 把乱码呈现给我们.

那么如何解决这个问题呢? 很简单, 上面错误信息里已经给出了, 让我们参考 pep0263 就好了. 看过 pep0263 之后我们就知道了, 原来 python 是采用了在源文件中写明文件格式的办法, 所以我们只要在 shell-bang 下面一行加上如下内容就行了:

::

    #!/usr/bin/env python
    # -*- coding: utf-8 -*-

    a = '你好, 世界'
    print a

然后再执行就能够正确输出 "你好, 世界" 了.

误区
----

经过上文, 我们应该知道要在程序源代码中使用 utf8 字符, 首先源文件顶部得有这么一行了: `# -*- coding: utf-8 -*`, 否则解释器直接报错.

然后看这么一个问题, 当我们写下这么一段程序时:

::

    #!/usr/bin/env python
    # -*- coding: utf-8 -*-

    a = '你好, 世界'
    b = u' => hello, world'
    print a + b

我们会得到这么一个错误:

::

    Traceback (most recent call last):
      File "test.py", line 6, in <module>
        print a + b
    UnicodeDecodeError: 'ascii' codec can't decode byte 0xe4 in position 0: ordinal not in range(128)

这个错误是由于没有分清 "unicode 对象" 和 "包含 unicode 序列的 str 对象" 的区别导致的. 前面我们说了, 在 python 中, `str` 和 `unicode` 都是 `basestring` 的子类, `str` 对象就相当于 C 语言的字符串类型, `unicode` 则相当于 C 语言中的宽字符序列.

在上面的代码中, 虽然 `a` 的值是 '你好, 世界', 但是 `a` 实际上是 `str` 对象, '你好, 世界' 之所以能够显示出来, 是因为源文件格式是 utf8 而且编辑器和 X 服务程序也都支持 utf8 字符集. 但是变量 `a` 中存储的实际上是 '\xe4\xbd\xa0\xe5\xa5\xbd, \xe4\xb8\x96\xe7\x95\x8c' (使用 a.encode('string_escape') 或者 a.encode('hex_codec') 方法查看).

由于 `b` 是 unicode 类型而 `a` 是 str 类型, 而 python 又是强类型, 所以上面的 `a + b` 的时候, 所以要先对 `a` 做一次 decode 目的是将其转化成 unicode 对象, 然后再将它和 `b` 相加. 然而对 str 类型 decode 实际上是没什么意义的, decode 时所选用的 encoding 默认就是 'ascii' 或者是 `sys.getdefaultencoding()` 的返回值, 当然这个方法的返回值默认也是 'ascii'. 然而采用 'ascii' 去 decode `a` 时, 自然就会因为 `a` 里面包含了 `\xe4` 这样的非 ascii 字符而导致 decode 失败报错.

Python3 的进步
--------------

python2 时代对 unicode 的支持问题一直广为诟病, `str`, `unicode`, `encode`, `decode` 等概念也是十分混乱, 在 python3 中这些概念的清晰度有了很大的提升.

比如在 python3 中 `str` 不再相当于 C 语言中的 `char *`, 而是像 C 语言的宽字符类型或者 Java 的 String 类型那样, 本身就是支持宽字符的类型. 而且增加还增加了 `bytes` 类型, `bytes` 类型这下才是相当于 C 语言的 `char *` 类型. 同时取消了 `str` 类型的 decode 方法, 以及取消了 bytes 类型的 encode 方法.

因为 decode 就是讲字节流解析成对象, encode 就是讲对象编码成字节流, 所以对 bytes 进行 encode, 和对已经表示对象的 str 进行 decode, 都是没有意义的. pyton2 时代 str 类型作为字节流类型时, 存在 str.encode(), 以及 unicode.decode 这两个方法其实都是无意义的.

参考
----

* https://docs.python.org/2/howto/unicode.html
* https://pythonhosted.org/kitchen/unicode-frustrations.html
