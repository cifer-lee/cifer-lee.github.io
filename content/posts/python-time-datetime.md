---
title: 关于 Python 中的 time 与 datetime 模块
date: 2017-11-26
categories:
  - Python
---

在 Python 中处理时间有两个库可用: time 和 datetime, 这两个模块曾经在很长一段时间里困扰着我, 我觉得这是 Python 又一处矛盾的地方 (最大的矛盾是 3 和 2 不兼容), 因为一门对开发者友好的语言应该直接提供最好用的库, 而不是让开发者去做选择.

看看 time 与 datetime 的区别, 以及它们的设计用意, 其实从 Python 标准库手册上就能看出来, time 模块位于 "16. Generic Operating System Services" 这一节, 处于这一节的还有 os, getopt 等模块, 而 datetime则位于 "8. Data Types" 这一节. 因为 time 是被开发来包裹系统调用的 --- 也就是 <time.h> 头文件那些方法, datetime 才是开发来专门作为时间工具库的. 实际上 time 模块是 C 写的, datetime 模块则是 python 写的.

既然是专注于时间处理, 又不需要特意去对应 <time.h>, datetime 模块自然是功能更全一些, 方法更亲民一些, 所以现在的趋势也是使用 datetime 模块.

我一直认为简单的东西才是最好的, 所以我认为最好的实践就是 time 和 datetime 永远只用一个, 我建议是只使用 datetime, 原因正如上所说. 可能唯一需要 time 模块的时候就是要用 time.sleep() 方法的时候, 但天知道为什么 sleep()方法要放到 time 库里呢, 毕竟 <time.h> 里也没有 sleep() 方法啊, 按说放到 threading 之类的库里不是更好..

# datetime 使用的几个 Tips

1. 总是使用 datetime.datetime 类, 不要使用 datetime.date 以及 datetime.time 类
2. 两个 datetime.datetime 对象相减得到 datetime.timedelta 对象
3. 不要纠结 timezone, 总是使用 utc 方法就好了
4. utcnow(), strptime(), strftime() 记住这三个方法就行

# 参考

1. https://mail.python.org/pipermail/python-list/2015-March/688175.html
2. https://mail.python.org/pipermail/python-list/2015-March/688172.html
