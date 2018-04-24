title: (译)如何使用 C 语言中的 volatile 关键字
slug: c-volatile
date: 2015-11-07 21:01:45
tags: 翻译, C Programming

(*原文: http://www.barrgroup.com/Embedded-Systems/How-To/C-Volatile-Keyword, 已取得翻译许可*)

**很多 C 程序员都不真正懂得 volatile 关键字的用法. 这无需奇怪, 因为大多数的 C 教程对 volatile 的介绍都比较简单. 这篇文章的目的就是告诉你 volatile 的正确使用方式**

你有碰到过下面的几个情形吗?

* 代码编译运行没问题 --- 直到你打开了编译器优化
* 代码运行的很好 --- 直到一个中断发生
* 古怪的硬件驱动程序
* RTOS task 各自单独运行时很好 --- 直到有其它 task 被 spawned

如果你碰到过上述任何一个问题, 那么就可能是你没有使用 volatile 关键字的原因. 你并不孤单, volatile 关键字为很多程序员所不熟悉. 不幸的是, 很多 C 相关的书籍都没有好好的介绍 volatile 关键字.

volatile 关键字和 const 一样, 是一个限定符, 用于一个变量被声明时. 它告诉编译器, 被声明的变量的值可能随时都会被改变 -- 就算使用这个变量的代码的附近 (附近有多近, 要看编译器了, 可能是同一个源文件) 没有任何修改这个变量值的语句也是如此. 给编译器的这个暗示是很严肃的, 在我们继续讲解之前, 我们先来看一下 volatile 的语法.

<!-- more -->

## volatile 关键字的语法

要将一个变量声明为 volatile 的, 需要在声明时将 `volatile` 关键字写到数据类型关键字的前面或后面. 比如, 下面两条语句都将 `foo` 声明为一个 volatile 的整数:

    volatile int foo;
    int volatile foo;

然后下面的例子是声明指向 volatile 变量的指针, 我们经常需要这么做, 尤其是涉及到 memory-mapped I/O 寄存器的时候. 下面两条语句都将 `pReg` 声明为一个指向 volatile 的, 无符号的 8 位整数的指针:

    volatile uint8_t *pReg;
    uint8_t volatile *pReg;

指向 non-volatile 变量的 volatile 指针一般不多见 (我可能曾经用过一次), 不过, 我还是给你展示一下语法:

    int *volatile p;

为了公平起见, 我再展示一个:

    int volatile * volatile p;

Incidentally, for a great explanation of why you have a choice of where to place volatile and why you should place it after the data type (for example, int volatile * foo), read Dan Sak's column "Top-Level cv-Qualifiers in Function Parameters" (Embedded Systems Programming, February 2000, p. 63).

最后, 如果你将 volatile 关键字应用于一个 struct/union, 那么整个 struct/union 中的所有内容都会是 volatile 的. 如果你不想这样, 那么将 volatile 应用于 struct/union 中的单独的成员就可以了.

## volatile 关键字的正确用法

任何时候, 只要这个变量的值可能在不确定的时刻被修改, 那么它都应该被声明为 volatile. 什么是 "不确定的时刻" 呢, 其实总共不过是只有三种情况:

1. Memory-mapped peripheral registers
2. 被 ISR 修改的全局变量
3. 多线程编程中, 被多个 task 共同访问或修改的全局变量

### Peripheral Registers

嵌入式系统包含真实的已经, 一般都有着精细复杂的外围. 这些外围包含一些寄存器, 这些寄存器的值可能会改变, 而且是与程序的执行流毫不相干的. 考虑一个非常简单地例子, 一个 8 bit 的状态寄存器, 被应射到内存地址 0x1234. 现在需要你轮询这个状态寄存器, 直到它的值不是 0. 你可能想当然的这样实现:

    uint8_t *pReg = (uint8_t *)0x1234;
    // Wait for register to become non-zero
    while (*pReg == 0) {} // Do something else

只要你打开编译器的优化选项, 上面的代码基本上在哪个架构哪个编译器上都会得到错误的结果, 因为编译器只会生成类似如下的汇编代码:

        mov ptr, #0x1234
        mov a, @ptr
    
    loop:
        bz loop

编译器这么做的道理很简单: 既然代码里没有任何地方会修改地址 0x1234 处的值, 那么对 0x1234 地址的访问一次就够了, 没必要老是访问. 殊不知, 这只是代码里没有改变 0x1234, 但是能够改变 0x1234 处的值的不光是我们的代码, 还有外围啊. 要解决这个问题我们需要给 pReg 加上 volatile 限定符:

    uint8_t volatile *pReg = (uint8_t volatile *) 0x1234;

这样一来生成的汇编代码将会是下面这样的:

        mov ptr, #0x1234
    
    loop:
        mov a, @ptr
        bz loop

这样便达到我们的要求了.

Subtler problems tend to arise with registers that have special properties. For instance, a lot of peripherals contain registers that are cleared simply by reading them. Extra (or fewer) reads than you are intending can cause quite unexpected results in these cases.

### Interrupt Service Routines

ISRs 常常会设置或修改那些在主代码里的标识变量. 比如, 串行口中断可能会检查每一个接受到的字符, 来看一下它是不是 ETX 字符 (代表消息结束). 如果这个字符是 ETX, 这个 ISR 就会设置一个全局的标识变量. 一个错误的实现是下面这样的:

    int etx_rcvd = FALSE;
    
    void main()
    {
        ...
        while (!ext_rcvd)
        {
            // Wait
        }
        ...
    }

    interrupt void rx_isr(void)
    {
        ...
        if (ETX == rx_char)
        {
            etx_rcvd = TRUE;
        }
        ...
    }

如果编译器优化选项没打开, 这段代码或许能够正常运行. 但现在像样的编译器都会 "破坏" 上面代码的逻辑, 问题在于 "编译器不知道 etx_rcvd 会在一个 ISR 中被改变. 在编译器看来, !ext_rcvd 总是正确的, 所以你将永远无法推出循环. 更甚者, 所有 while() {} 循环体后面的代码会全部被移除 -- 因为它们永远得不到执行. 幸运的话, 你的编译器会警告你的; 不幸运的话, 或者你从不在意编译器的警告的话, 你的代码将会失败的让你很难找到原因.

解决方法也是, 将 etx_rcvd 声明为 volatile.

### 多线程 (tasks) 应用程序

在实时操作系统的通信机制中, 撇开队列, 管道这些高大上, 共享内存 (这里说的就是全局变量) 仍然不失为一个多 tasks 间通信的好办法. 就算你使用了一个抢占式的调度器, 编译器也依然无法知道什么是 context switch, 更不知道 context switch 何时会发生. 因此, 另一个 task 修改一个全局变量, 概念上就和 ISR 修改这个全局变量是一样的. 因此所有的共享的全局变量都应该被声明为 volatile.

    int cntr;
    
    void task1(void)
    {
        cntr = 0;
        
        while (cntr == 0)
        {
            sleep(1);
        }
        ...
    }
    
    void task2(void)
    {
        ...
        cntr++;
        sleep(10);
        ...
    }

同样你需要将 cntr 声明为 volatile, 否则, 编译器优化选项一开, 代码运行就很可能不是你想要的结果.

## 最终幻想

有一些编译器允许你隐式的将所有的变量声明为 volatile 的. 请抵制这种诱惑! 这会让你不再去思考. 同样也会导致产生低效率的程序.

同样, 抵制住想要责备编译器优化或者要关闭它的想法. 当今的编译器都非常优秀, 我已经不记得上次发现编译器优化的 bug 是哪年哪月了.

如果你有一段运行结果令你匪夷所思的代码, 你要修复它, 那么你可以先 grep 一下 volatile, 如果结果是空, 那么就可以按照上面的思路考虑一下是不是没加 volatile 导致的.
