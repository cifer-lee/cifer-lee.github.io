---
layout: post
title: GNU Make 深入浅出 --- 隐式规则
date: 2014-01-26 17:34
---

## 隐式规则

隐式规则告诉 make 如何根据 target 来自动的推断它的 prerequisites 和 recipe. 拿现有的一条 C 编译的隐式规则来说, 如果 make 发现 target 是 .o 后缀, 而且又发现了相同名字的 .c 文件, 那么 make 就会生成这样一条 recipe: 

    cc -c -o targetname.o targetname.c    # 测试环境 debian 7, make 3.81, cc 4.7.2

而如果 make 没有发现与 target 名字相同的 .c 文件却发现了 .cpp 文件, 那么 make 生成的 recipe 就会是这样的:

    g++ -c -o targetname.o targetname.c   # 测试环境 debian 7, make 3.81, cpp 4.7.2, g++ 4.7.2, 我很奇怪这里为什么是 g++ 而不是 cpp

不过上面那条 recipe 并不完全正确, 因为这里没有考虑到隐式规则使用的那些变量, 是的, 隐式规则会使用一些变量, 如果你改变了这些变量, 那么生成的 recipe 会有所不同. 比如说, 当 make 判断出应该使用 cc 来编译并生成 target 时, 它还会自动应用 CFLAGS 变量, 当 make 判断出应该用 g++ 来编译生成 target 时, 会自动应用 CPPFLAGS 变量.

你可以改变这些规则, 通过修改 _pattern rules_, 以前你可以通过修改 _suffix rules_ 来修改这些隐式规则行为, 不过看名字就可以看出, _suffix rules_ 只是和后缀名有关, 可以猜测你仅仅能够通过它做一些类似于把 .cpp 文件也用 cc 编译器编译之类的事情, _suffix rules_ 已经被更通用的 _pattern rules_ 取代了.

### 使用隐式规则

上面说了一堆, 那到底怎么使用隐式规则呢?

使用隐式规则的方法很简单, 就是你别去给目标写 recipe, 甚至连目标的 rule 都不要去写(当然, 你要为你的最终目标写个 rule, 要不然执行一下 make 会什么也不干就退出了). 比如说, 你的 makefile 可以仅仅这样:

    foo: foo.o bar.o
        cc -o foo foo.o bar.o $(CFLAGS) $(LDFLAGS)

你的最终目标是生成 foo, 你这条 rule 里指定了要生成 foo 需要 foo.o 和 bar.o, 但是 make 在当前目录找不到 foo.o, 这就使得 foo.o 也变成了一个 target, make 要先把 foo.o 搞定才能继续完成 foo 这个 target. 但是你却没有给出 foo.o 的 rule, 那怎么办呢? 

这个时候, make 就会开始找隐式规则, 看看有没有符合 target 为 foo.o 这种情况的规则. 幸运的是, make 找到了一条规则, 这条规则说: 如果 target 以 .o 为后缀, 那就去找找有没有和 target 名字一样的以 .c 为后缀的文件, 有的话, 就执行 `cc -c -o targetname.o targetname.c`, 没有的话, 我也无能为力了, 你接着找别的规则吧. 

(是的, 这就是一条隐式规则的形象的表述, 我斟酌了很久, 终于把它表述的如此形象了, 我都开始佩服我的表诉能力了... 说正题, 这里你应该注意到, 一条隐式规则, 就是在告诉 make 接下来应该怎么做, 它告诉 make 的不仅仅是 target 的 prerequisites, 还有 target 的 recipe, 这一点 GNU Make 手册也是这么说的.)

虽然找到的这条隐式规则告诉了 make 应该去找 .c 文件(prerequisites), 但是隐式规则毕竟是隐式规则, 它只知道 targetname.o 依赖于 targetname.c, 但是它不知道我们的 targetname.c 还包含了一堆 .h 文件, 这也是 targetname.o 应该依赖的, 这一点, 隐式规则帮不了 make 了, 这不能怪 make, 因为 targetname.c 包含哪些 .h 因项目而异, 因程序员而异, make 不可能知道(实际上, make 有办法得出一个源文件包含哪些头文件, 但那个话题超出本文范围了), 所以有时候还是需要我们手动指定所依赖的头文件.

方法很简单, 就是我们为 targetname.o 写一条 rule, 指定**除了 targetname.c 之外**的 prerequisites 就可以了, 但是我们一定不要把 recipe 写上, 如果写上的话, 就成了显示规则了, make 就直接不会去找隐式规则了.

Ok, 上面说的是 targetname.c 存在的情况下, 如果 targetname.c 不存在呢? 规则里也说了, "我也无能为力, 你接着找别的规则吧", 于是 make 就继续找别的规则, 没想到, 还真又被它找到一条规则, 这条规则是这么说的: 如果 target 为 .o 为后缀, 那就去找找有没有和 target 名字一样的以 .cpp 为后缀的文件, 有的话, 就执行 ` g++ -c -o targetname.o targetname.cpp`, 没有的话, 我也无能为力了, 你接着找别的规则吧. 这和第一条规则差不多, 也是 make 就找 .cpp, 找到了就执行 `g++ ...`, 找不到 .cpp 就接着找别的规则...

好了, 现在我们对隐式规则算是有一个形象清晰的认识了, 但是有一个地方是需要我们注意的, 不知你发现没有, 那就是, 从上面的例子我们发现, 有两到多个的规则都适用于 .o 这种后缀的 target, 加入 .c 和 .cpp 文件都存在的话, 那么 make 应该采用哪一条规则呢? 

在 make 内建的隐式规则中, 这些规则是有先后顺序的, 如果两条规则所指定的 prerequisites 都能被满足, 那么排在前面的就会被采用. 还有一点要注意的是, make 在决定了要搜索隐式规则后(一般就是你没有写 recipe 的情况下), 它不会因为你在 rule 里写了与 target 名字相同的 .p 文件(pascal 源文件)就去优先使用 .p 对应的规则, make 严格按照自己内建的规则排序去选该采取的规则. 比如说, 假如你对 foo.o 的rule 是这么写的(没有 recipe, 一定不要写 recipe):

    foo.o: foo.p 

那么 make 还是会先去找有没有 .c 文件, 有的话就执行 `cc -c -o ....`, 因为在 make 内建的规则排序里, C 是排在 pascal 前面的( make 3.81).

### 重写 pattern rule


