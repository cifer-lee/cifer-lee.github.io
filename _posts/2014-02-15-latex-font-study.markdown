---
layout: post
title: latex 中的字体
date: 2014-02-15 00:50
---

# Introduction

数字字体有一段悠长而错综复杂的历史. 见 [AFM][AFM] 了解详情.

最初 Tex 使用它自己的字体系统, 也就是 MetaFont (Cifer 注: 矢量字体的又一种描述语言), 这是由 Tex 作者设计的字体系统. Tex 的字体系统的默认 font family 叫做 "Computer Modern". 这个字体系统下的字体们有很好微调能力.

Tex 编译们应该允许你使用其它的字体. 除了 Tex 自己的 MetaFont 字体系统, 其它的字体系统着实也不少, 像是 PostScript Type1/Type3 字体, 以及点阵字体. pdftex 引擎一般使用 Type1. 点阵字体实际上是栅格图, 放大时质量不好, 而 Type3 作为 Type1 的升级, 新加了很多特性, 使得 Type3 可以嵌入栅格图. 因此在 Tex 引擎世界中, Type3 常常嵌入到点阵字体中.

字体在被需要时生成, 因此 Tex 编译时间加长.

MetaFont 着实是复杂的字体系统, 随着 Truetype 和 OpenType 的流行, 比较新的一些 Tex 引擎比如 xetex, luatex, 已经可以使用这些新的字体系统了. 如果使用较老的引擎, 那就需要把这些字体转换成 Type1.

# Font families (即 Typefaces)

(Cifer 注: 所谓 font family, 即是 typeface, 翻译成中文, 就是平常所说的"字体"; 而 font, 翻译成中文应当是"字型"; 他们的区别是字型只是字体中的具有同样尺寸的同样粗细和倾斜度的一个子集. 英语中还有一个词叫做 "glyph", 翻译成中文应该是 "字形", 在英文中它只是指某一个字的形状, 形体)

有很多种字体, 比如, Computer Modern, Times, Arial 以及 Courier. 这些字体可以分为三类, serif (或者叫做 roman, rm), sans serif (sf) 和 monospace (tt). 每一种字体都可以归到这三类中; 你可以在每一类中选择喜欢的字体. Computer Modern Roman (Serif) 是 Latex 的默认字体. 同一种字体(family) 中还分多种字型, 它们也有不同的属性, 比如大小, 是否加粗, 是否倾斜等. Family 的存在就是为了一致性, 所以修改文章的字体时, 不建议只挑一两句话修改, 最好是整篇文章的字体修改成同一种.

三类字体由三个变量控制:

*   \rmdefault
*   \sfdefault
*   \ttdefault

默认的 family 存在 \familydefault 中, 它的值应该是上面三个变量的值之一. 如果你要改变默认 family, 你应该这样:

    \renewcommand*{\familydefault}{\sfdefault}

这会使得整个文章都使用 Sans Serif 类的默认字体, 在 LaTex 中也就是 Computer Modern Sans Serif, 如果你没有修改过 \sfdefault 的话.

在 LaTex 中, 你应该按照如下步骤改变字体:

    \renewcommand*{\familydefault}{\sfdefault}
    \renewcommand*\sfdefault{ppl}

上面的 \sfdefault 可以替换为 \rmdefault 或者 \ttdefault.

上面的三个字体默认的变量以及 \familydefault 变量, 都有对应的 switch, 不要弄混了:

*   \normalfont
*   \rmfamily
*   \sffamily
*   \ttfamily

# 强调

>   Notice: 不要再段落中使用粗体字. 这在排版中是不推荐的. 使用粗体字会分散读者的注意, 因为粗体字会使得读者首先注意到它. 同样, 也不要在段落中使用彩色字, 这其实会使得段落很难看.

如果想强调一个词或一个短语, 最简单而且最正确的做法是 \emph{text}.

# 字编码

字符是字节序列, 不要把它和它的展示混淆, 它的展示叫做字形 (glyph), 是读者看的见的东西. 字符 'a' 在不同的字体 (font family) 中或者同一字体的不同字型 (font) 中有不同的字形 (glyph), 比如斜体, 粗体等.

在编译时, tex 会会为每一个字符选择正确的字形 (glyph). 这被叫做字形编码 (font encoding). Latex 默认的 font encoding 是 OT1, 这是 Computer Modern Tex 字形的编码, 它只包含了 128 个字符, 大多都是 ASCII 字符, 很多其他的字符在 OT1 中没有对应的编码. 当需要重音字符时, TeX 使用一个正常字符和一个重音字符来表示, 最终结果可能正确, 但这又很多的弊端:

*   It stops the automatic hyphenation from working inside words containing accented characters.
*   Searches for words with accents in PDFs will fail.
*   Extracting ('e.g.' copy paste) the umlaut 'Ä' via a PDF viewer actually extracts the two characters '"A'.
*   Besides, some of Latin letters could not be created by combining a normal character with an accent, to say nothing about letters of non-Latin alphabets, such as Greek or Cyrillic.

_未完待续..._

# 字形

每种字体都有它自己的字形 (斜体, 粗体), 这也叫做 font styles, 或者 font properties.

Font styles 一般实现在不同的字体文件中. 

控制字形的命令可以参考 wiki, http://en.wikibooks.org/wiki/LaTeX/Fonts#Shapes
控制字体尺寸的命令可以参考 http://en.wikibooks.org/wiki/LaTeX/Fonts#Sizing\_text

# 本地字体选择

你可以局部的修改文章中某一片段的字体, 有如下命令可以用:

\fontencoding
\fontfamily
\fontseries
\fontshape

\selectfont 命令是必须的, 否则字体不会被改变, 强烈建议将命令包围在组中.


[AFM]: http://en.wikipedia.org/wiki/Adobe_Font_Metrics#AFM
