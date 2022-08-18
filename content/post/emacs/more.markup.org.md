---
date: 2022-08-17
title: org-mode 内嵌LaTex
tags: ['emacs', 'elisp']
categories: ['emacs']
---


特殊符号
--------

在表示数学公式上特别有用。比如要表示alpha和beta，则这样：

``` {.example}
\号后面加alpha，\号后面加beta
```

为了补全，原来是绑定在了M-TAB上的，但是这个键我绑定在了切换窗口上，于是
把它改成TAB好了：

``` {.example}
(global-set-key (kbd "TAB") 'pcomplete)
```

但是这样是看不到这个特殊字符的样子的，通过

``` {.example}
C-c C-x \
```

进入可以查看UTF8字符的模式，就可以查看了。再使用一次快捷键，恢复原来的模式。
但是那么多字符，我怎么找呢？可以通过：

``` {.example}
M-x org-entities-help
```

来查看所有的字符！

下标和上标
----------

LaTex的语法中，' \^ '和' \_ '是用于表示上标和下标的。如要表示a~1~ 和x^2^
，则只 需要这样：

``` {.example}
a\_1
x\^2
```

把上面中的\号去掉就行了。 同样，使用

``` {.example}
C-c C-x \
```

进入显示上下标的模式。
如果下标有多个字母，则要用{}把它们括起来，否则只会解析第一个字母，剩下的不作为
下标。

画矩阵
------

直接使用LaTex的语法即可，比如我要画这个矩阵：

``` {.example}
1 2
3 4
```

就写成这样：

``` {.example}
\begin{equation}
\left(
\begin{array}{cc}
1 & 2\\
3 & 4
\end{array}
\right)
\end{equation}
```

先开始一个方程(equation)，再开始左括号，再开始数组（矩阵被看成是一个数组），
后面的cc表示第一列居中(center)对齐，第二列也是居中对齐，如果改成l，则是
左(left)对齐，如果是r则是右(right)对齐。&号用于分隔同一行中的元素（看成
是空格），\
用于换行。 矩阵输出成html之后，加载比较慢。
