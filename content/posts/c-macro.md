---
title: C 语言宏的展开与字符串化宏和符号连接宏
date: 2018-02-20
categories:
  - Programming
tags:
  - C
---

C 语言由于没什么高级的特性, 所以现有的特性被玩的各种精. 宏展开就是很值得品味的部分.

## 递归展开问题

宏定义语句是这样的:

> #define identifier token-sequence

在具体的宏展开过程中, 遇到标识符时, 此标识符会整个的被使用 token-sequence 展开, 如果 token-sequence 中还包含有其他的被定义的宏标识符, 也都会相应的被展开. 但是显然, 已经展开过的标识符如果再次出现, 则维持原样不变, 不会再次展开, 否则就递归个没完了.

比如说有如下代码段:

```
#define x y
#define y x

x
```

对其执行 `gcc -E` 预处理时得到的结果是 `x`, 这里发生了两步替换, 首先 `x` 被展开成 `y`, 然后因为 `y` 也被定义为宏, `y` 又展开成 `x`. 注意此时, 由于 `x` 这个标识符已经展开过, 这里是第二次出现, 所以不会被再次展开成 `y`. 不然就没完没了了.

这部分可以参见 K&R C, A.12 中有这么一段话:

> In both kinds of macro, the replacement token sequence is repeatedly rescanned for more defined identifiers. However, once a given identifier has been replaced in a given expansion, it is not replaced if it turns up again during rescanning; instead it is left unchanged.

## 字符串化宏与符号连接宏

上面提到了在标识符出现第二次的时候会不再展开. 除此之外还有两种情况也会阻止标识符的展开, 那就是字符串化 (#) 以及符号连接 (##). 看如下两例:

```
#define a                             b
#define STRINGIZE(x)    x _#x
#define CONCATE(x)      x _##x

STRINGIZE(a)
CONCATE(a)
```

执行 `gcc -E` 会得到如下的结果:

```
b _"a"
b _a
```

可以看到, 没带 # 或者 ## 前缀的标识符被展开了, 而带 # 或 ## 的标识符没有展开就直接字符串化或者符号连接化了.

这部分在 K&R C A.12 也有提到:

> Then the token sequence resulting from each argument is substituted for each unquoted occurrence of the corresponding parameter's identifier in the replacement token sequence of the macro. Unless the parameter in the replacement sequence is preceded by #, or preceded or followed by ##, the argument tokens are examined for macro calls, and expanded as necessary, just before insertion.

就是说, 如果宏定义中直接包含了 # 或 ##, 则会阻止标识符展开, 但在实际的编程中, 这其实并不是我们所期望的. 我们所期望的是 `STRINGIZE` 和 `CONCATE` 宏能够为我们递归的展开参数标识符. 怎么做呢?

## 让 STRINGIZE 和 CONCATE 递归展开标识符

其实解决方法也很简单, 就是在覆盖一层宏定义就可以了, 如下:

```
#define a                               b
#define _STRINGIZE(x)    x _#x
#define STRINGIZE(x)      _STRINGIZE(x)

#define _CONCATE(x)      x _##x
#define CONCATE(x)        _CONCATE(x)

STRINGIZE(a)
CONCATE(a)
```

由于现在 `STRINGIZE` 和 `CONCATE` 宏都不直接包含 # 和 ## 了, 因此对传入的参数标识符会递归的展开, 然后调用相应的第二层宏.

这次再执行 `gcc -E` 就会得到如下输出:

```
b _"b"
b _b
```

## 后记

今天看到 boost 库的 BOOST_PP_STRINGIZE 和 BOOST_PP_CAT 宏时突然想起这个问题, 才两年没写 c/c++ 代码感觉宏展开都快忘了. 故复习一下.

boost 库中的这两个宏的实现方式和上面的思路是一样的, 有兴趣可以看 boost 的 stringize.hpp 以及 cat.hpp 源码.
