---
title: 代码风格
---

以下几点关于 C 语言编程的风格是我一直贯彻执行的.

* 常用的逻辑功能写成宏 (如经常性地判断值是否合法, 省去众多冗长的 if)
* 今后的代码风格: 所有变量， 放在最前面声明， 而不是用到时再声明 (C89 风格)
* 变量名尽量短小精悍， 能不用下划线就不用
* Always use struct tag, don't use typedef in struct definition
* Always write pointer form when passing array to function -- from Expert C Programming, section 9.3
* Always using deferencing pointer form when call function by function pointer
* sizeof 的参数不是类型关键字时, 不用加括号
* 对象 (如结构或数组) 在声明时初始化时, 初始化列表最后可以多跟一个逗号. 这在标准里有说到, K&R A.8.7. 原因参见这里: http://stackoverflow.com/questions/7043372/int-a-1-2-weird-comma-allowed-any-particular-reason
* 但是在 enum 声明中, c89 标准中不允许最后一个 enumrator 后多跟一个逗号的. 但是很多编译器 (如 gcc) 扩展准许这么做, 在 gcc 中, 要完全遵循 c89, 需要 -std=c89 -pedantic 两个选项都加上
* 可以学习 FreeRTOS, 以后设计什么模块的时候, 不要给用户提供什么 init 函数, 直接给用户暴露干正事的接口, 模块初始化之类的工作在内部隐式的完成
