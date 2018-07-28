+++
date = "2016-05-27T16:35:19+08:00"
draft = false
title = "reStructuredText语法笔记"
+++

reStructuredText用于排版非常好。但是，和C++一样，越是灵活，越是可扩展，越是复杂。
有些语法，你永远记不住，我经常遇到一些表示方法（比如说超链接），忘记了语法，每次都
要去Google。为了不这么麻烦，我决定记下去，以免下次忘了又要去查。

大部分都来自 [这里](http://sphinx-doc.org/rest.html) 。

代码块
======

在段落的结尾使用 `::` 符号,
然后输入一个空行，然后是代码（必须缩进，空2格或4格），
然后是一个空行。比如

代码如下:

    #include <iostream>

    int
    main(int argc, char *argv[])
    {
        return 0;
    }

引入代码文件
============

对于我等码农来说，这应该算是最常用的了。格式为:

    .. code-include:: source-code-file
    :lexer: language-name

超链接
======

经常要引用一些大牛的博客，链接的格式为:

    `链接文字 <链接>`_

比如， [我的英文博客](http://en.zhengyuanhang.com) 。

斜体和粗体
==========

斜体:

    *这是斜体*

粗体:

    **这是粗体**

图片
====

格式是这样的:

    .. image:: /images/one_piece.gif
     :alt: one_piece
     :width: 200px
     :height: 150px
     :scale: 80 %
     :align: left

align的选项是left, center, right。
