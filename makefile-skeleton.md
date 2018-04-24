title: 通用 Makefile 模板骨架
slug: makefile-skeleton
tags: Makefile, CMake
date: 2016-11-26 20:31:08

Makefile 仍然是绝大多数 Linux 平台 C 项目用的最多的构建方案, 可能由于 Makefile 本身写起来麻烦, 后来又出现了 CMake 这样的帮助人们自动生成 Makefile 的方案. 但我个人觉得 Makefile 本身写起来也并不麻烦, 反倒是使用 CMake 的话还要再去学习一套语法, 多此一举. 人生苦短, 学习那么多冗余的东西干什么呢.

所以我希望最好是能直接就有一个通用一点的纯 Makefile 模板, 便于我们在开始新项目时快速的拿过来套用上, 但是搜索了一圈也没找到有这样的项目, 所以就自己大概研究一下吧.

于是这篇文章的目的, 是展示如何绘制一个基本的, 方便扩展的, 通用的 Makefile 项目构建模板, 以便于将来需要开展新项目时直接套用.

项目的地址在这里: [https://github.com/cifer-lee/Makefile.skel](https://github.com/cifer-lee/Makefile.skel)

## 暂时本着两点原则

1. 能够自动推断依赖
2. 结构精简, 易读

## 骨架

### 最原始

```
edit : main.o kbd.o command.o display.o \
       insert.o search.o files.o utils.o
        cc -o edit main.o kbd.o command.o display.o \
                   insert.o search.o files.o utils.o

main.o : main.c defs.h
        cc -c main.c
kbd.o : kbd.c defs.h command.h
        cc -c kbd.c
command.o : command.c defs.h command.h
        cc -c command.c
display.o : display.c defs.h buffer.h
        cc -c display.c
insert.o : insert.c defs.h buffer.h
        cc -c insert.c
search.o : search.c defs.h buffer.h
        cc -c search.c
files.o : files.c defs.h buffer.h command.h
        cc -c files.c
utils.o : utils.c defs.h
        cc -c utils.c
clean :
        rm edit main.o kbd.o command.o display.o \
           insert.o search.o files.o utils.o
```

### 引入变量

```
objects = main.o kbd.o command.o display.o \
          insert.o search.o files.o utils.o

edit : $(objects)
        cc -o edit $(objects)
main.o : main.c defs.h
        cc -c main.c
kbd.o : kbd.c defs.h command.h
        cc -c kbd.c
command.o : command.c defs.h command.h
        cc -c command.c
display.o : display.c defs.h buffer.h
        cc -c display.c
insert.o : insert.c defs.h buffer.h
        cc -c insert.c
search.o : search.c defs.h buffer.h
        cc -c search.c
files.o : files.c defs.h buffer.h command.h
        cc -c files.c
utils.o : utils.c defs.h
        cc -c utils.c
clean :
        rm edit $(objects)
```

### 引入隐式规则

隐式规则是 gnu make 中的一个大话题, 包含很多方面, 其中一方面就是 recipe 的自动推导.

```
objects = main.o kbd.o command.o display.o \
          insert.o search.o files.o utils.o

edit : $(objects)
        cc -o edit $(objects)

main.o : defs.h
kbd.o : defs.h command.h
command.o : defs.h command.h
display.o : defs.h buffer.h
insert.o : defs.h buffer.h
search.o : defs.h buffer.h
files.o : defs.h buffer.h command.h
utils.o : defs.h

.PHONY : clean
clean :
        rm edit $(objects)
```

## 自动生成依赖

缺陷

截至目前, 我们拥有了如下的 Makefile

```
# Where are the source files
src_dir = src

# Where the object files go
obj_dir = obj

# The name of the executable file
elf_name = ryuha

CFLAGS =
LDFLAGS =
LDLIBS =

# All the source files ended in '.c' in $(src_dir) directory
srcs := $(wildcard $(src_dir)/*.c)

# Get the corresponding object file of each source file
objs := $(patsubst $(src_dir)/%.c,$(obj_dir)/%.o,$(srcs))

# Get the dependency file of each source file
deps := $(patsubst $(src_dir)/%.c,$(obj_dir)/%.d,$(srcs))

all : $(obj_dir)/$(elf_name) ;

$(obj_dir)/$(elf_name) : $(objs)
	$(CC) $(LDFLAGS) -o $@ $(objs) $(LDLIBS)
	@echo
	@echo $(elf_name) build success!
	@echo

$(obj_dir)/%.o : $(src_dir)/%.c | $(obj_dir)
	$(CC) -c $(CFLAGS) $(CPPFLAGS) -o $@ $<

$(obj_dir)/%.d : $(src_dir)/%.c | $(obj_dir)
	$(CC) -MM $(CFLAGS) $(CPPFLAGS) -MF $@ -MT $(@:.d=.o) $<

-include $(deps)

$(obj_dir) :
	@echo Creating obj_dir ...
	@mkdir $(obj_dir)
	@echo obj_dir created!

clean : 
	@echo "cleanning..."
	-rm -rf $(obj_dir)
	@echo "clean done!"

.PHONY: all clean
```

这个方式看起来工作的不错, 但是有一个缺陷, 假设程序目录如下:

    test/
        src/
            a.c
            a.h
            b.h
        Makefile

a.c:

    #include "a.h"

    int main()
    {
        return 0;
    }

a.h:

    #define A   1

b.h:
    
    #define B   1

现在执行 `make`, 会输出如下信息:

```
cli@sanc:/tmp/test$ make
Creating obj_dir ...
obj_dir created!
cc -MM  -w -pthread -pipe  -MF obj/a.d -MT obj/a.o src/a.c
cc -c  -w -pthread -pipe  -o obj/a.o src/a.c
cc  -o obj/ryuha obj/a.o 

ryuha build success!
```

经过检查, 目录结构变成了

    test/
        src/
            a.c
            a.h
            b.h
        obj/
            a.d
            a.o
            ryuha
        Makefile

而且 obj/a.d 的内容如下:

    obj/a.o: src/a.c src/a.h

工作的不错! 然后这里有一个隐患, 就是在我们的规则中 obj/a.d 会随着 src/a.c 的更新而更新, 然而假如 src/a.c 引用的头文件中又增加了新的头文件, obj/a.d 却不会跟着更新, 但是 src/a.c 的头文件依赖链确实改变了.

我们将 src/a.h 改成如下来验证一下:

    #define A   1

    #include "b.h"

然后再次执行 `make`

```
cli@sanc:/tmp/test$ make
cc -c  -w -pthread -pipe  -o obj/a.o src/a.c
cc  -o obj/ryuha obj/a.o 

ryuha build success!
```

然后再来看一下 obj/a.d 文件, 结果依然是如下内容, 没有任何变化:

    obj/a.o: src/a.c src/a.h

这下隐患揪出来了, a.o 的生成现在绝对依赖 b.h, 但是 a.d 里却没有记录它! 后面的例子就显而易见了, 下面我们修改 b.h 的内容为如下:

b.h:
    
    #define B   3

然后再次执行 `make`, 结果你应该也想到了, 就是 "Nothing to be done" !!!

```
cli@sanc:/tmp/test$ make
make: Nothing to be done for 'all'.
```

那么如何解决呢?

### 让 a.d 的生成也依赖于 a.o 所依赖的头文件

上面的问题, 说白了, 就是 a.d 的依赖只有 a.c, 导致了即使 a.c 引用的头文件变了, 但只要 a.c 不变, a.d 就不会重新生成, 所以解决办法就是让 a.d 也依赖 a.c 的那些头文件. 这里引用 gnu make 官方文档中的一个巧妙的方法, 那就是在生成 a.d 文件之后, 把 a.d 也放进目标字段中, 放在 a.o 的后面, 也就是 a.d 的内容升级成如下的样子:

    obj/a.o obj/a.d: src/a.c src/a.h

怎么实现呢, 就是将 Makefile 中生成 .d 的规则作如下修改, 这就是 gnu make 官方文档的方法:

    $(obj_dir)/%.d : $(src_dir)/%.c | $(obj_dir)
	    @set -e; rm -f $@; \
	    $(CC) -MM $(CFLAGS) $(CPPFLAGS) -MT $(@:.d=.o) $< > $@.$$$$; \
	    sed 's,\($*\)\.o[ :]*,\1.o $@ : ,g' < $@.$$$$ > $@; \
	    rm -f $@.$$$$

这样再重复上面的实验, 就会发现没有问题了.

### 优雅?

上面的写法看似完美, 但实际上还有一点不够优雅的地方. 就在于 `include $(deps)` 指令

    -include $(deps)

我们知道, `make` 在解析 Makefile 文件时碰到 `include` 时, 会把 `include` 后面的每一个文件都作为一个需要更新的目标, 这里 `include $(deps)` 中是所有 .c 文件对应的 .d 文件, 所以只要 .d 不存在, .d 的菜谱就会被执行.

接着上面的实验, 让我们现在执行一下 `make clean`, 很好, 输出如下:

```
cli@sanc:~/Makefile.skel$ make clean
cleanning...
rm -rf obj
clean done!
```

然而如果我们再执行一次 `make clean` 呢?

```
cli@sanc:~/Makefile.skel$ make clean
Creating obj_dir ...
obj_dir created!
cleanning...
rm -rf obj
clean done!
```

嗯? 怎么会多出一个创建 obj dir 的动作? 这就是因为第一次 clean 时将 obj/ 下面的内容清理了, 第二次 clean 时 `include $(deps)` 指令发现 .d 文件没了, 于是执行 .d 的菜谱.

这个要解决这个问题, 一种办法是判断 make 的目标, 如果目标是 clean 的话, 就不调用 `include $(deps)`:

    ifneq ($(MAKECMDGOALS), clean)
    -include $(deps)
    endif

### 更好的解决方法

上述方法的缺点是如果将来加入更多的与生成最终程序无关的目标, 那就需要将那个目标也加入这个条件语句中, 这样又显得难看了.

所以终极解决方法就是,

CPPFLAGS += -MMD

并去掉生成 .d 的规则, `-include $(deps)` 指令保留着前面的 `-`, 这样一来, 在 `make clean` 时, 即使找不到 .d, 但由于没有生成规则, 也就不会执行多余的命令了.

.d 会在 .o 的规则中生成, 第一次 `make` 的时候, .d 和 .o 都是没有的, 因此 .o 只依赖 .c, 一旦菜谱执行, .d 就生成了, 包含了 .o 所有的依赖. 后续只要这些依赖变动过, .o 就会更新 --- 并且顺便把 .d 也更新.

套用第一个问题, 很容易发现这个方法能够解决第一个问题的:

1. 第一次 make 时, 此时 a.d 不存在; a.d 生成, 包含了 a.c, a.h 这两个依赖
2. 在 a.h 中添加 `include "b.h"`
3. 第二次 make, a.d 已经存在, 被包含进来, a.o 的两个依赖 a.c, a.h 也被感知. 而且 a.h 变化了, 所以 a.o 重新生成, 顺便 a.d 也重新生成了, 而 a.d 这次重新生成发觉了 a.h 对 b.h 的依赖, 于是 b.h 也被加到 a.d 中

### 各种自动生成依赖方法的比较

下面这篇文档是 gnu make 的维护者写的, 分析比较了各种自动生成依赖的方式的区别

http://make.mad-scientist.net/papers/advanced-auto-dependency-generation/

## Tips

### `include` 指令的用意

1. 让各个模块各自的 Makefile 使用一些公共的变量以及 pattern rules.
2. 当自动生成目标的依赖时, 生成的依赖可以放在一个单独的文件里, 然后使用 `include` 命令包含这个文件

### Makefile 的重新生成

### make 如何读入多个 Makefiles

1. 通过指定多个 `-f` 选项
2. 通过 `MAKEFILES` 环境变量
3. 通过 `include` 指令

### 二阶段式

阶段一

### 隐式规则

关于隐式规则中, 目标和依赖中的 `/` 问题, 时常看一看 https://www.gnu.org/software/make/manual/make.html#Pattern-Match 就都明白了


## 禁忌

### 不要使用如下的风格

```
objects = main.o kbd.o command.o display.o \
          insert.o search.o files.o utils.o

edit : $(objects)
        cc -o edit $(objects)

$(objects) : defs.h
kbd.o command.o files.o : command.h
display.o insert.o search.o files.o : buffer.h
```

这种风格以依赖为中心, 所有依赖此项目的目标们放在一起写, 可读性非常差

### 不要使用 `FORCE` 规则

这里说的 `FORCE` 规则指的是没有依赖没有菜谱的规则, 这种规则的唯一作用是作为其它规则目标的依赖, 好在执行其它规则时无视目标的新旧, 总是执行菜谱.

这个小 trick 仅用在其它版本的不支持 `.PHONY` 关键字的 make 上, 在 gnu make 中, 我们应该使用表意更明确的 `.PHONY` 关键字.
