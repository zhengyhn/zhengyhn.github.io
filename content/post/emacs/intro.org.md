---
date: 2022-08-17
title: org-mode介绍
tags: ['emacs', 'elisp']
categories: ['emacs']
---

什么是org-mode
==============

org-mode是用来写笔记，维护todo列表，写项目计划的。

文档结构
========

大纲
----

-   TAB键绑定在命令org-cycle上，它会在下面3个状态之间循环：

--&gt;折叠--&gt;显示孩子--&gt;显示子树--&gt;折叠...

-   C-u TAB键绑定在命令org-global-cycle上，它会在下面3个状态之间循环：

--&gt;总览--&gt;显示内容--&gt;显示全部--&gt;总览...

标题
----

-   一个\*表示一级标题，二个\*表示二级标题，三个\*表示三级标题，没有多于3个\*的。
-   注意，\*必须顶格写，而且\*后面必须要跟一个空格。后面的字体颜色会改变。

动作
----

  function                      command                            key
  ----------------------------- ---------------------------------- ---------
  next heading                  outline-next-visible-heading       C-c C-n
  previous heading              outline-previous-visible-heading   C-c C-p
  next heading same level       org-forward-same-level             C-c C-f
  previous heading same level   org-backward-same-level            C-c C-b
  backward to higher level      outline-up-heading                 C-c C-u

编辑
----

  function                command                              key
  ----------------------- ------------------------------------ ----------
  insert heading          org-insert-heading                   M-RET
  inseert heading after   org-insert-heading-respect-content   C-RET
  move substree up        org-move-subtree-up                  M-S-UP
  mov substree down       org-move-subtree-down                M-S-DOWN

列表
----

-   无序列表使用-,+,和\*，这里\*不能顶格写
-   有序列表使用1.和1)
-   通过按键shift+左右方向键来改变强调符的类型

脚注
----

-   第一种方法，\[number\]
-   第二种方法，\[\fn:name\]（把\去掉）
-   第三种方法，\[\fn:name:content\]
-   不推荐使用"\[number\]"这种形式，因为数组的访问会出现这种形式

表格
====
