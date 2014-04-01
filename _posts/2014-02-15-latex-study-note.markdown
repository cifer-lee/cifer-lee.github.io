---
layout: post
title: Latex study notes --- floats, figures and captions
published: false
date: 2014-02-15 15:48
---

# Floats

Floats 是一种容器, 用来盛放那些在文档中不能跨页的东西. LaTex 默认能识别 "table" 和 "figure" floats, 不过, 你可以定义自己的 floats. 

Floats 不是正常文本流的一部分, 它们是独立的实体, 他们在页面中的位置由自己决定(top, middle, bottom, left, right, 或者是设计师指定的任何地方). 它们往往有一个标题, 它们也被编号因此可以被引用. 

LaTex 自动浮动 Tables 和 Figures (这里是指 table 和 figure 环境, 而不是指 tabular 环境) --- 根据当前页面还剩多少空间. 如果当前页面没有足够的空间, 那么浮动体就会到下一页的顶部, 这是默认行为, 但是可以通过 Table 和 Figure 的定义来改变.

# Figures

要创建一个 figure 并使其浮动, 使用 figure 环境.

    \begin{figure}[placement specifier]
    ... figure contents ...
    \end{figure}

你可以通过 _placement specifier_ 来控制 floats 的排版.
