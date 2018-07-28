---
date: 2013-03-10
title: tcpl读书笔记：简介与基础
---

@&lt;font color="red"&gt; 本文为the c programming language一书的笔记
@&lt;/font&gt;

铭记c语言之父
-------------

不要忘记他们的名字，也不要写错他们的名字，他们是：

-   Brian W. Kernighan
-   Dennis M. Ritchie

简介
----

### 历史

-   c语言受BCPL语言和B语言的影响
-   BCPL语言由Martin Richards发明
-   B语言由Ken Thompson发明，他和Dennis Ritchie 并称Unix之父
-   BCPL和B是弱类型的语言

### 符号英文

从另外一篇文章[^1]学习来的：

  symbol   English
  -------- --------------------
  +        plus
  -        minus
           slash
  \        backslash
  \*       asterisk
  %        percent
  &lt;     less-than
  &gt;     greater-than
  &gt;=    greater-than-equal
  "        double-quote
  '        single-quote

变量和算术表达式
----------------

### 华氏温度与摄氏温度互转

-华氏温度，英文为Fahrenheit，冰点为32度，沸点为212度
-设C为摄氏温度，F为华氏温度，则公式为：C = （5 / 9）（F - 32）
-题目为输出1,20,40,...,300华氏温度对应的摄氏温度。虽然很简单，但是还是要写一下。

``` {#fahrenheit to celsius .c}
#include <stdio.h>

#define START 0
#define END 300
#define STEP 20

int main()
{
    int fah;
    float cel;

    printf("fahrenheit \t celsius\n");
    for(fah = START; fah <= END; fah += STEP){
        cel = (5 * (fah - 32)) / 9;
        printf("%6d \t %8g\n",fah,cel);
    }

    return 0;
}
```

  ------------ ---------
  fahrenheit   celsius
  0            -17
  20           -6
  40           4
  60           15
  80           26
  100          37
  120          48
  140          60
  160          71
  180          82
  200          93
  220          104
  240          115
  260          126
  280          137
  300          148
  ------------ ---------

### 字符输入输出

-   代码如下:

``` {#character input and output .c}
#include <stdio.h>

int main()
{
    char c;

    while((c = getchar()) != EOF){
        putchar(c);
    }

    return 0;
}
```

-   输出

\[monkey@itlodge test\]\$ ./t a a -1

书上说char类型存放不了EOF，我觉得有问题，我用char类型一样可以判断，
而且从理论上-1完全可以使用char来存放。
通过查看头文件stdio.h，得到EOF的赋值为：

``` {#EOF .c}
/* End of file character.
   Some things throughout the library rely on this being -1.  */
#ifndef EOF
# define EOF (-1)
#endif
```

-   练习1-9：写一个程序把读入的连续多个空格替换成1个。
-   代码：

``` {#replace .c}
#include <stdio.h>

int main()
{
    char pre_char,cur_char;

    pre_char = '\0';
    while((cur_char = getchar()) != EOF){
        if(!(pre_char == ' ' && cur_char == ' ')){
            putchar(cur_char);
        }
        pre_char = cur_char;    
    }

    return 0;
}
```

-   结果：

\[monkey@itlodge test\]\$ ./t jj hh kk df jj hh kk df

函数
----

### 老式的函数定义和声明

-   如果见到下面这种函数的声明和定义，不要害怕，因为这是曾经的函数：

``` {#old style .c}
int power();    //function prototype

power(base,n)
int base,n;
{
    ...
}
```

[^1]: learn python the hard way
