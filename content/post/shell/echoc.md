---
date: 2013-03-20
title: 一个带颜色输出的echo
---

动机
====

为了找到工作，最近需要多写shell了，有时候我很想输出信息的时候有颜色，比如
成功的信息用绿色，错误的信息用红色，去查了一下echo，发现写起来一大堆麻烦，
甚至是硬编码，没办法，参考自[这篇文章](http://hi.baidu.com/lonelycorn/item/c7472d4bc25127ea1e19bc96)
，自己写了一个给自己用。

基础知识积累
============

通过echo的man手册及网友的帮助，发现，echo用-e来开启转义，设置颜色的格式为：

``` {.example}
\e[background;foreground;高亮m字符串
```

默认的为：

``` {.example}
\e[0m
```

背景色的数字：

  number   color
  -------- -------------
  0        transparent
  40       black
  41       red
  42       green
  43       yellow
  44       blue
  45       purple
  46       dark green
  47       white

前景色的数字：

  number   color
  -------- ------------
  30       black
  31       red
  32       green
  33       yellow
  34       blue
  35       purple
  36       dark green
  37       white

高亮的数字是1,非高亮为0

那为什么后面是m呢？根据[教程](http://www.freeos.com/guides/lsst/misc.htm)
,m是一个标志，表示用来改变文字的颜色和特效的。
原来前面用分号分开的就是m指定的特性，还能用2指定加粗，这里不详细说了。

实现
====

刚学bash shell不久，写了下面的echoc，表示有颜色的输出。

``` {.bash}
#!/bin/bash
# can: echo string with color
# usage: echoc [-color] [string]
# author: yuanhang zheng

if [ $# -lt 2 ]; then
    if [ $# == 1 ]; then
        if [ ${1:0:1} == "-" ]; then
            echo ""
        else
            echo "$1"
        fi
    else
        echo "" 
    fi
elif [ $# == 2 ]; then
    if [ ${1:0:1} != "-" ]; then
        echo "usage: ${0##*/} [-color] [string]"
        echo "e.g: ${0##*/} -red hey! "
        exit 1
    elif [ "$1" == "-red" ]; then
        echo -e "\e[0;31;1m$2\e[0m"
    elif [ "$1" == "-green" ]; then
        echo -e "\e[0;32;1m$2\e[0m"
    elif [ "$1" == "-yellow" ]; then
        echo -e "\e[0;33;1m$2\e[0m"
    elif [ "$1" == "-blue" ]; then
        echo -e "\e[0;34;1m$2\e[0m"
    elif [ "$1" == "-purple" ]; then
        echo -e "\e[0;35;1m$2\e[0m"
    elif [ "$1" == "-white" ]; then
        echo -e "\e[0;37;1m$2\e[0m"
    else
        echo "no such color!"
        echo "you have only 'red', 'green', 'yellow', 'blue',
'purple', 'white'"
        exit 1
    fi
else
    echo "too many parameters!"
    echo "usage: ${0##*/} [-color] [string]"
    echo "e.g: ${0##*/} -red hey! "
    exit 1
fi
```

好了，如果你觉得有用，就直接用吧，如果你觉得里面的语法不太懂，去查吧。
