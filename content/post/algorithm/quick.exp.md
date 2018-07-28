---
date: 2013-03-11
title: 比较高效的整数求幂法
---

最近在看快速求fibonaci数列的方法，需要用到快速求幂法，于是参考了[这篇文章](http://blog.csdn.net/hkdgjqr/article/details/5381028)
， 做下笔记。

最直观的方法
============

如果叫我求一个整数的n次幂，我要么用pow，要么写个循环直接相乘。
比如要求：

``` {.example}
5 ^ 55
```

的结果，很明显，要进行54次乘法。要知道，在目前的计算机体系架构中，乘法的计算
是非常慢的。

效率高的求法
============

有的聪明人（不是我）就想到了减少乘法次数的方法：

``` {.example}
5 ^ 2 = 5 * 5
5 ^ 4 = (5 ^ 2) * (5 ^ 2)
5 ^ 8 = (5 ^ 4) * (5 ^ 4)
5 ^ 16 = (5 ^ 8) * (5 ^ 8)
5 ^ 32 = (5 ^ 16) * (5 ^ 16)

5 ^ 55 = 5 ^ (32 + 16 + 4 + 2 + 1)
       = 5 ^ 32 * 5 ^ 16 * 5 ^ 4 * 5 ^ 2 * 5
```

现在关键是怎么求各个幂了，给出55,你怎么分解它成为32, 16, 4, 2, 1呢？

神奇的二进制
============

看一下55的二进制，为：

``` {.example}
0011 0111
```

可以看到，二进制里面为1的就是要乘的数。于是可以想到，

``` {.example}
我们从最低位开始，如果为1,则乘以这个位的2次幂，否则跳过。
```

现在问题又来的，怎么从最低位开始呢？

倒转任意进制数
==============

这其实是一个非常简单的算法，在我刚学C语言的时候就会了，不过当时比较笨，理解
不够透彻。上代码：

``` {.c}
#include <stdio.h>

/* reverse a number NUM with base BASE,
 * return the reversed number in decimal. */
int reverse_num(int base, int num);

int main(int argc, char *argv[])
{
    printf("%d\n", reverse_num(10, 15));
    printf("%#x\n", reverse_num(16, 0x32));

    return 0;
}

int reverse_num(int base, int num)
{
    int new_num;

    new_num = 0;
    while (num > 0) {
        new_num *= base;
        new_num += (num % base);
        num /= base;
    }
    return new_num;
}
```

而对于二进制来说，求模和除法，还可以优化！
于是我们就可以想到如何求幂了。

较高效的整数求幂法
==================

尔等码农，直接上代码吧：

``` {.c}
#include <stdio.h>

/* calculate NUM ^ EXP，return the result */
long long qpow(int num, int exp);

int main(int argc, char *argv[])
{
    printf("%lld\n", qpow(5, 20));

    return 0;
}

long long qpow(int num, int exp)
{
    long long pow, base;

    pow = 1;
    base = num;        //long may not be able to hold
    while (exp > 0) {
        if ((exp & 1) == 1) {   // (exp % 2) == 1
            pow *= base;
        }
        base *= base;
        exp >>= 1;        // exp /= 2;
    }
    return pow;
}
```
