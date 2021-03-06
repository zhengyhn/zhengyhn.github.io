---
date: 2013-02-13
title: Linux c编程一站式学习笔记：C语言基础
---

@&lt;font color="red"&gt; 本文为"linux c编程一站式学习"一书的笔记
@&lt;/font&gt;

程序和编程语言
==============

-   平台这个词有很多说法，可以指计算机体系结构，也可以指操作系统，也可以开发平台（

编译器，链接器等）。

-   机器语言称为第一代语言1GL，汇编为2GL，C/C++，JAVA，Python等称为3GL，目前已经有

了4GL和5GL，4GL以后的语言主要描述做什么，而不是一步步怎么做，SQL就是这样的。

-   思考题：解释执行的语言比编译执行的语言有什么优缺点？

1.  优点

-   执行过程简单，不需要通过编译阶段，直接解释执行

1.  缺点

-   速度慢，缺少编译阶段的优化
-   依赖平台

自然语言和形式语言
==================

基本概念
--------

自然语言就是人类讲的语言，不是人为设计的，而是自然进化的。

形式语言为了特定应用而人为设计的语言。例如数字和运算符，化学式，当然还有编程语言。

形式语言有严格的syntax规则，由token的规则和structure的规则组成。token
称为记号，相当于自然语言中的单词和符号，structure称为结构，是指token的排列
方式，token的规则称为lexical规则，structure的规则称为grammar规则。

当我们阅读英语的一个句子的时候，我们不仅要搞清楚每个词(token)的意思，还要
搞清楚整个句子的结构是怎样的（疑问句，定语从句什么的），当我们在处理代码的时候
也会做同样的事情。分析句子的结构的过程叫parse(解析）。比如：

``` {.example}
the other shoe fell
```

上面这个句子一旦解析完成，你就明白了它的意思，知道了shoe是什么意思，fell意味
着什么，是在什么上下文(context)中说的，还理解了它暗示了什么内容，这就是语义
（semantic)。

区别
----

-   歧义(ambiguity)：自然语言充满歧义，人们通过上下文(context)来防止歧义，而形式语言的设计要求不能有歧义，这就是上下文无关的形式语言。
-   冗余(redundancy)：自然语言引入了很多冗余来解决歧义，所以人们说话比较啰嗦，而形式语言更简洁。
-   与字面意思的一致：自然语言充满着隐喻（比喻拟人什么的），而形式语言中字面意思基本就是真实意思。

第一个程序
==========

-   一个好的习惯是打开gcc的-Wall选项，以提示所有的警告信息。

常量、变量和表达式
==================

-   界定符(delimiter)，
-   windows上的文本文件用\r\n做行分隔符，许多应用层的网络协议（如HTTP）也

是用\r\n换行，而Linux和各种Unix上则使用\n来分隔。

-   在格式化字符串中占个位置，并不出现在最终打印结果，如%d，这种叫占位符(placeholder)
-   为什么要规定\?呢，因为C语言中有一些三连符(trigraph)，在一些特殊的终端上缺少某些

字符，需要三连符输入，比如用??=来表示\#字符。

-   下划线(underscore)
-   C89中共有32个关键字，C99中有37个，增加了inline,restrict,~Bool~,~Complex~,~Imaginary~
-   由运算符和操作数组成的算式称为表达式。任何表达式都有值和类型两个基本属性。
-   表示的存储位置的表达式称为左值，放在赋值号左边，放在右边的称为右值。
-   有的表达式既可以做左值也可以做右值，而有的表达式只能做右值。
-   向下取整称为floor,向上取整称为ceiling。
-   C语言中整数除法既不是floor,也不是ceiling，总是把小数部分截掉，在数轴上向0方向

取整。也就是说，当操作数为正数时，使用floor,当操作数为负数时，使用ceiling

-   ASCII，American Standard Code for Information Interchange

简单函数
========

-   printf也有返回值，它返回打印的字符个数，我们调用printf不是为了得到它的返回值，

而是利用它产生的副作用(side effect)——打印

-   改变计算机存储单元里的数据或做输入输出操作都算side effect
-   \#, pound sign or number sign or hash sign
-   尖括号&lt;&gt;,英文为angle bracket
-   gcc必须要加-lm才能使用math.h，因为数学函数位于/lib/libm.so库文件中，-lm告诉

编译器要到这个文件中找，大部分库都位于/lib/libc.so文件中，本来是要加-lc选项的，
但是因为gcc默认就加这个选项，所以可以不加。

-   C标准主要由两部分组成，一部分描述C语法（编译器），一部分描述C标准库。linux中最

流行的C函数库是glic，几乎都实现在libc.so里（虽然我看了一下我的系统是libc.so.6,
而libc.so只是libc.so.6的一个符号链接），数学函数在libm.so中，多线程的函数在
libpthread.so中。

-   在主函数中return 0;然后执行这个程序，再用shell命令：

``` {.bash}
echo $?
```

可以看到返回的状态，刚学到bash shell中的这个东西，刚好用到了。

-   函数如果没有参数而留空，这是old
    style，不推荐使用。可惜我使用了多年，以后一定

要纠正。

-   形参：parameter。实参：argument。
-   可以通过

``` {.bash}
man 3 printf
```

来查看printf的说明 为什么是3呢？我用

``` {.bash}
man man
```

查了一下，发现原来有下面的规定：

  section number   description
  ---------------- -----------------------------------------------
  1                executable programs or shell commands
  2                system calls
  3                library calls
  4                special files(usually found in /dev)
  5                file formats and conventions(eg./etc/passwd)
  6                games
  7                miscellaneous
  8                system administration commands(only for root)
  9                kernel routines

-   FHS,filesystem hierarchy standard
-   原来printf也是一个shell命令来的。
-   书上说全局变量使用变量表达式来初始化是不合法的，比如下面这个：

``` {.example}
double pi = acos(-1.0);
```

我试了一下，发现没有错误，只有警告，而且可以运行。
但是为了规范，以后还是要提醒自己一定要只用常量表达式来初始化全局变量。
而当我用宏来定义的时候，却可以：

``` {.example}
#define PI acos(-1.0)
```

没有错误也没有警告，实践证明宏定义替换部分可以使用任意表达式，因为人
家只是替换而已！

-   gcc的扩展特性居然允许嵌套定义函数！

分支语句
========

-   C语言规定%两边的操作数必须为整型(char, int,
    long等），两个正数相除余数当然是

正数，如果两个操作数中有负数，则C语言标准规定余数的符号总是与被除数相同。

``` {.c}
#include <stdio.h>

int main()
{
    char c = 'a';

    printf("%d\n", -c % 10L);

    return 0;
}
```

-   &的英文为ampersand，|英文为pipe sign，!英文为exclamation mark

深入理解函数
============

-   log函数默认是以e为底，而以2为底的为log2函数，以10为底的为log10函数。
-   欧几里得法求最大公约数的证明。本来这种东西小学就应该会的，可以我这种白痴

要到现在才明白。

``` {.example}
要求x,y的最大公约数，可以令q = x / y, r = x % y,
则有：x = q * y + r
假设c能够整除y和r，则c一定能整除x，因为：
c / x = c / (q * y + r) = c / (q * y) + c / r
因为c能够同时整除y和r，则加号两边必然是2个整数，所以c / x也必然是整数，
所以c一定能整除x。
所以c能同时整除x和y，于是x和y的最大公约数就转化成了y和x % y的最大公约数。
```

循环语句
========

-   递归思路属于函数式编程，迭代思路属于命令式编程(imperative
    programming)
-   duff's device（去google一下）

数组
====

-   gcc带上-E参数可以打印出来预处理之后的代码。使用工具cpp(c
    preprocesser)也可以。
-   宏是在预处理的阶段进行的，枚举是在编译的阶段进行的。
-   各种类unix的系统都把1970年1月1日00:00:00这一时刻称为epoch（纪元），因为unix
    是在1969年发明。

代码风格
========

-   ,号和;号后面要加空格，这是英文的书写习惯。
-   因为标准的unix终端是24 \* 80的，所以一行达到80个字符一定要折行。
-   函数内的注释尽可能少，写注释是说明代码“能做什么”，而不是说怎么做。只要代码

写得清晰，“怎么做”是一目了然的，如果需要写注释才能解释清楚，说明代码可读性很
差。

-   常用的缩写：

  ---------------------- ------
  count                  cnt
  block                  blk
  length                 len
  window                 win
  number                 nr
  temporary              temp
  internationalization   i18n
  trans                  x
  transmit               xmt
  ---------------------- ------

gdb
===

-   使用命令help可以查看帮助，再也不用去百度google了。
-   直接按回车重复上一个命令
-   start命令是开始执行程序，不是run!
-   使用backtrace(bt)可以查看函数调用的栈帧。
-   使用info(i)可以查看很多信息(locals等)
-   使用finish运行到函数结束
-   display跟踪显示，undisplay取消跟踪显示
-   disable breakpoints可以用来禁用断点，需要时再enable

-   总结gdb命令

  ----------------------- ----------------------------------------------
  backtrace(bt)           查看各级函数调用及参数
  finish                  连续运行到当前函数返回
  frame(f)                选择栈帧，后面带帧编号
  info(i)                 查看信息，后面带查看内容
  list(l)                 列出源码，接着上次位置，每次列10行。
  list(l) line number     后面带行号，则从那里开始
  list(l) function name   列出某个函数的源码
  next(n)                 执行下一语句
  print(p)                打印表达式的值，可以修改变量的值或者调用函数
  quit(q)                 退出
  set var                 改变变量的值
  start                   开始执行，停在第一行
  step(s)                 执行下一语句，如果是函数则进入函数
  break(b)                查看有哪些断点
  break(b) line number    在某一行设置断点
  break function name     在某个函数开头设置断点
  break ... if ...        设置条件断点
  continue(c)             从当前位置开始连续运行程序
  delete(d) bp-number     删除断点，后面的为断点号
  disable bp-number       禁用断点
  enable bp-number        启用断点
  display var             跟踪查看某个变量
  undisplay number        取消跟踪显示，后面的为跟踪显示号
  run(r)                  从头开始连续运行程序
  watch var               设置观察点
  info(i) watchpoints     查看设置了哪些观察点
  x address               打印存储单元的内容
  ----------------------- ----------------------------------------------

排序和查找
==========

-   手刃插入排序

``` {.c}
#include <stdio.h>

void insert_sort(int *arr, int num);

int main(int argc, char *argv[])
{
    int a[5] = { 5, 4, 3, 2, 1 };
    int i;

    insert_sort(a, 5);

    for (i = 0; i < 5; ++i) {
        printf("%d\n", a[i]);
    }

    return 0;
}

void insert_sort(int *arr, int num)
{
    int current, key, back, temp;

    for (current = 1; current < num; current++) {
        back = current - 1;
        key = arr[current];
        while (back >= 0 && key < arr[back]) {
            arr[back + 1] = arr[back];
            back--;
        }
        arr[back + 1] = key;
    }
}
```

-   手刃归并排序

``` {.c}
#include <stdio.h>

#define N 5

void merge_sort(int *arr, int begin, int end);
void merge(int *arr, int begin, int middle, int end);

int main(int argc, char *argv[])
{
    int a[N] = { 5, 4, 3, 2, 1 };
    int i;

    merge_sort(a, 0, N);

    for (i = 0; i < N; ++i) {
        printf("%d\n", a[i]);
    }

    return 0;
}

void merge_sort(int *arr, int begin, int end)
{
    int middle;

    if (begin >= end - 1) {
        return;
    }
    middle = (begin + end) / 2;
    merge_sort(arr, begin, middle);
    merge_sort(arr, middle, end);
    merge(arr, begin, middle, end);
}

void merge(int *arr, int begin, int middle, int end)
{
    int left[N / 2 + 1], right[N / 2 + 1];
    int ai, li, ri;

    for (li = 0, ai = begin; ai < middle; ai++) {
        left[li++] = arr[ai];
    }
    for (ri = 0; ai < end; ai++) {
        right[ri++] = arr[ai];
    }
    li = ri = 0;
    ai = begin;
    while (li < (middle - begin)
           && ri < (end - middle)) {
        if (left[li] < right[ri]) {
            arr[ai++] = left[li++];
        } else {
            arr[ai++] = right[ri++];
        }
    }
    while (li < (middle - begin)) {
        arr[ai++] = left[li++];
    }
    while (ri < (end - middle)) {
        arr[ai++] = right[ri++];
    }
}
```

-   手刃二分查找

``` {.c}
#include <stdio.h>
#include <string.h>
#include <assert.h>

#define N 5

int is_sorted(char *arr[], int len);
char *
binary_search(char *arr[], int begin, int end, char *key);

int main(int argc, char *argv[])
{
    char *a[N] = { "hang", "itfoot", "yuan", "yuanhang", "zheng" };
    char *key = "yuanhang";
    char *found = binary_search(a, 0, N, key);
    if (found) {
        printf("%s\n", found);
    } else {
        printf("not found!\n");
    }

    return 0;
}

char *
binary_search(char *arr[], int begin, int end, char *key)
{
    int middle, result;

    assert(is_sorted(arr, end - begin));

    while (begin < end) {
        middle = (begin + end) / 2;
        result = strcmp(key, arr[middle]);
        if (result < 0) {
            end = middle - 1;
        } else if (result > 0) {
            begin = middle;
        } else {
            return arr[middle];
        }
    }
    return NULL;
}


int is_sorted(char *arr[], int len)
{
    int i;

    for (i = 1; i < len; ++i) {
        if (strcmp(arr[i], arr[i - 1]) < 0) {
            return 0;
        }
    }
    return 1;
}
```
