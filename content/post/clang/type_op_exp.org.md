---
date: 2013-03-10
title: tcpl读书笔记：类型与表达式
---

@&lt;font color="red"&gt; 本文为the c programming language一书的笔记
@&lt;/font&gt;

数据类型和大小
==============

-   有关short,int,long的大小。

它们的大小由编译器决定，但是有一个规定，short和int至少要16位，long至少要32位。
short必须小于int,int必须小于long

常量
====

-   1234L表示常量1234类型是long，1234u表示类型为unsigned,1.23f表示类型为float，

1.23L表示类型为long double。

-   回顾转义字符：

  character        function
  ---------------- --------------------------
  \a               alert
  \b               backspace
  \f               formfeed,used by printer
  \n               newline
  \r               carriage return
  \t               horizontal tab
  | vertical tab
  \                backslash
  \?               question mark
  '                single quote
  "                double quote
  \ooo             octal number
  \xhh             hexadecimal number

定义
====

-   全局变量和静态变量默认初始化为0（字符类型的为'0'），局部变量（自动变量）默认初始化

为不确定的值。

类型转换
========

-   记得用过微软写的atoi函数，当时我想自己写，一时想不出来，今天看到这份代码，真的觉得

自己很笨，是自己不想思考呢？还是我真的被禁锢了思想。

``` {.c}
#include <stdio.h>
#include <string.h>

int str2i(char *str)
{
    int i,len,result;

    len = strlen(str);
    result = 0;
    for(i = 0; i < len; i++){
        result = result * 10 + str[i] - '0';
    }

    return result;
}

int main()
{
    char *s = "543212345";

    printf("%d\n",str2i(s));

    return 0;
}
```

看到后面才发现，上面这份代码，其实非常水，因为没有考虑负数和其它特殊情况。

-   有符号数和无符号数的比较和机器相关。

``` {.example}
-1L < 1U 返回true，因为在比较时1U会转化为signed long。
而-1L < 1UL 则返回false，因为在比较时-1L会转化为unsigned long，变成了一个很大的
正数，因此实际上-1L > 1UL。
```

-   练习：写一个函数htoi(s)，把十六进制字符串转化为十进制整数。

``` {.c}
int htoi(char *s)
{
    int i,len,result,value;

    len = strlen(s);
    result = 0;
    for(i = 2; i < len; i++){
        if(s[i] >= 'a' && s[i] <= 'z'){
            value = s[i] - 'a' + 10;
        }else if(s[i] >= 'A' && s[i] <= 'Z'){
            value = s[i] - 'A' + 10;
        }else{
            value = s[i] - '0';
        }
        result = result * 16 + value;
    }

    return result;
}
```

自加自减
========

-   ++和--操作的必须是变量，不能是表达式，比如(i + j)++这样是错的

三目运算符? :
=============

-   学习到2个妙用? :的例子：

1.  打印10个每行

``` {.c}
for(i = 0; i < n; i++){
    printf("%d%c",%d,(i % 10 == 9 || i == n - 1) ? '\n' : ' ');
}
```

1.  选择是否打印英文后面的复数

``` {.c}
printf("%d item%s.\n",n,n == 1 ? "" : "s");
```

运算符优先级
============

-   虽然说这种东西不用怎么记忆，但是对于我这种立志要精通C语言的人来说，记忆还是非常

必要的。

  operators                                      assoicativity
  ---------------------------------------------- ---------------
  () \[\] -&gt; .                                left to right
  ! \~ ++ -- + - \* (type) sizeof                right to left
  \* / %                                         left to right
  + -                                            left to right
  left-shift right-shift                         left to right
  &lt; less-than-equal &gt; greater-than-equal   left to right
  == !=                                          left to right
  &                                              left to right
  xor                                            left to right
  or                                             left to right
  &&                                             left to right
  oror                                           left to right
  ?:                                             right to left
  = += -= etc                                    right to left
  ,                                              left to right


