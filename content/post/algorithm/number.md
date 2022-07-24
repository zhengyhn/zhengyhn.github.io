---
date: 2022-07-17
title: 数字的算法
tags: ['Algorithm']
categories: ['Algorithm']
---

# 基本代数

## 加法

十进制有一个很傻逼但是很有趣的性质：

``` {.example}
任意3个个位数相加的和最多是两位数
```

事实上，对于任意进制，都有这个性质。 另外一个很有用的性质：

对于一个基数为b的k位数，它能表示的最大数为$b^{k^ {-1}}$

想要知道一个基数为b的数有多少位k，则：

$$ k = log_b(N + 1) $$

## 乘法和除法

这里出现了一种神奇的乘法算法，对于计算机来说效率非常高，
引用[wikipedia](http://en.wikipedia.org/wiki/Multiplication_algorithm#Quarter_square_multiplication)
的例子：

``` {.example}
Decimal:     Binary:
11   3       1011  11
5    6       101  110
2   12       10  1100   #will delete
1   24       1  11000
   ---          -----
    33         100001
```

算法是这样的：

1.  11除以2,取整为5;3乘以2为6
2.  5除以2,取整为2;6乘以2为12
3.  2除了2,为1;12乘以2为24
4.  去掉2那一行，因为2是偶数。
5.  最后相加：3 + 6 + 24 = 33

当然，最简单的是：

``` {.example}
3 11
1 22
  --
  33
```

从右边的二进制可以看到，左边的数在右移，右边的数在左移，去掉的偶数行对应的
二进制数为0,因此不用相乘。左移和右移在计算机中运算是非常快，这种办法大大
提高了乘法的效率。

马上用代码来实现：

``` {.c}
#include <stdio.h>
#include <stdlib.h>

/* multiple two integer A and B */
int multiple(int a, int b);

int main(int argc, char *argv[])
{
    int i, j;

    for (i = -1000; i < 1000; i++) {
        for (j = -1000; j < 1000; j++) {
            if (multiple(i, j) == i * j) {
                printf("match!!\n");
            } else {
                printf("error!!\n");
                exit(1);
            }
        }
    }
    printf("oh!successfully!!!\n");

    return 0;
}

int multiple(int a, int b)
{
    int sign, temp, sum;

    if (a < 0) {
        sign = -1;
        a = -a;
    } else {
        sign = 1;
    }
    if (b < 0) {
        sign = -sign;
        b = -b;
    }
    if (a > b) {
        temp = a;
        a = b;
        b = temp;
    }
    if (a == 0 || b == 0) {
        return 0;
    }

    sum = 0;
    while (a >= 1) {
        if (a & 1) {
            sum += b;
        }
        a >>= 1;
        b <<= 1;
    }
    if (sign == -1) {
        return -sum;
    } else {
        return sum;
    }
}
```

上面代码中，用自己写的乘法和标准的乘法来判断，看看结果有没有错。

同样，可以推理出来除法的算法（虽然我推不出来...），代码实现如下：

``` {.c}
#include <stdio.h>
#include <stdlib.h>

struct Div {
    int quot;
    int rem;
};

/* calculate DIVIDER / DIVISOR, return the quotient and remainder
 * in struct Div. */   
struct Div divide(int divider, int divisor);

int main(int argc, char *argv[])
{
    int i, j;
    struct Div d;

    for (i = -10; i < 10; i++) {
        for (j = 1; j < 100; j++) {
            d = divide(i, j);
            if (d.quot == i / j && d.rem == i % j) {
                printf("match!!\n");
            } else {
                printf("%d / %d :error!\n", i, j);
                exit(1);
            }
        }
    }

    return 0;
}

struct Div divide(int divider, int divisor)
{
    struct Div div;
    int sign, rem_sign;

    if (divisor == 0) {
        fprintf(stderr, "divisor can't be zero!");
        exit(1);
    }
    if (divider == 0) {
        div.quot = 0;
        div.rem = 0;
        return div;
    }
    if (divider < 0) {
        sign = -1;
        divider = -divider;
        rem_sign = -1;
    } else {
        sign = 1;
        rem_sign = 1;
    }
    if (divisor < 0) {
        sign = -sign;
        divisor = -divisor;
    }
    div.quot = div.rem = 0;
    while (divider > 0) {
        if (divider & 1) {
            div.rem++;
        }
        divider >>= 1;
        div.rem += divider;
        while (div.rem >= divisor) {
            div.quot++;
            div.rem -= divisor;
        }
    }
    if (sign == -1) {
        div.quot = -div.quot;
    }
    if (rem_sign < 0) {
        div.rem = -div.rem;
    }

    return div;
}
```

# 模代数

## 求模运算

同余(congruent modulo)定理：

$$
x ≡ y (mod N) ↔ N divides (x - y)
$$

按照我个人的理解，是：

``` {.example}
x和y除了N有相同的余数当且仅当N整除(x - y)
```

用书上的例子，令x = 253, y = 13, 253除以60余数为13, 13除以60余数也是13,
253 - 13 = 240, 240能被60整除。

下面这个图，能形象地说明这个问题：

``` {.example}
... -6 -3 0 3 6 ...        //余数为0
... -5 -2 1 4 7 ...        //余数为1
... -4 -1 2 5 8 ...        //余数为2
```

对多少取模，就有多少行。

替换规则：

------------------------------------------------------------------------

如果x1 ≡ x2 (mod N),且y1 ≡ y2 (mod N)，则 x1 + y1 ≡ x2 + y2 (mod N), 且
x1 \* y1 ≡ x2 \* y2 (mod N)

------------------------------------------------------------------------

这个规则有非常重要的作用，在一些数论的题中，比如：

------------------------------------------------------------------------

2^345^ ≡ (2^5^)^69^ ≡ (32)^69^ ≡ (mod 31) 因为32 ≡ 1 (mod 31), 32 ≡ 1
(mod 31)，所以 32^2^ ≡ 1^2^ (mod 31), ... 32^69^ ≡ 1^69^ ≡ 1 (mod 31) =
1

------------------------------------------------------------------------

模的幂算法：

比较高效的算法，来自[wikipedia](http://en.wikipedia.org/wiki/Modular_exponentiation)
由下面的定理推出：

------------------------------------------------------------------------

c ≡ (a \* b) (mod N) c ≡ (a \* (b (mod N))) (mod N)

------------------------------------------------------------------------

于是：

------------------------------------------------------------------------

a^2^ (mod N) ≡ (a \* (a (mod N))) (mod N) a^3^ (mod N) ≡ (a \* ((a \* (a
(mod N))) (mod N))) (mod N) ...

------------------------------------------------------------------------

我等码农，还是用代码来实现一下吧：

``` {.c}
#include <stdio.h>

/* calculate BASE^EXP % MOD, return the result */
int mod_pow(int base, int exp, int mod);

int main(int argc, char *argv[])
{
    printf("%d\n", mod_pow(4, 13, 497));

    return 0;
}

int mod_pow(int base, int exp, int mod)
{
    int i, inter_base, result;

    result = 1;
    for (i = 0; i < exp; i++) {
        inter_base = base * result;
        result = inter_base % mod;
    }
    return result;
}
```

还有一种更快的算法，类似于前面的乘法和除法。这时候，其实是
把幂看成一个二进制数，从最低位开始，如果为1则要像上面的算法一样计算结果，
否则保留原来结果，每次向高位前进的时候，基数都要翻一倍，并取模。
用代码来说明问题吧：

``` {.c}
#include <stdio.h>

/* calculate BASE^EXP % MOD, EXP >= 0, return the result */
int mod_pow(int base, int exp, int mod);

int main(int argc, char *argv[])
{
    printf("%d\n", mod_pow(4, 13, 497));

    return 0;
}

int mod_pow(int base, int exp, int mod)
{
    int result;

    result = 1;
    while (exp > 0) {
        if (exp & 1) {
            result = (result * base) % mod;
        }
        exp >>= 1;
        base = (base * base) % mod;
    }

    return result;
}
```

这个算法我的理解不是很透彻，先记下来再说。

后来发现原来这个算法称为“蒙格马利算法”，它要用到下面的这个定理作为引理：

``` {.example}
(a * b) % m = ((a % m) * (b % m)) % m
```

## 欧几里得最大公约数算法

首先，不要忘记这个英文名，Euclid。虽然这是一个非常古老的算法，但是有的时候你还是会
忘记，因此，我要好好记住它。

``` {.example}
对于两个正整数x和y，有x >= y，则gcd(x, y) = gcd(x mod y, y).
```

记住容易，但是要能证明它，才是真正理解了。证明如下：

``` {.example}
有2个正整数x, y, x >= y, 它们的最大公约数是gcd。
令x = ky + r，则(x mod y) = r = x - ky，
假设d为x和y的一个公约数，即d能整除x，d也能整除y，
则d也能整除(x mod y)。
假设d为(x mod y)和y的一个公约数，由于x = ky + r = ky + (x mod y)，
所以d也能整除x。
因此，x和y的某个公约数等于x mod y和y对应的公约数，最大公约数当然也相等。
```

用代码来实现：

``` {.c}
#include <stdio.h>

/* return the greatest common divisor with x, y >= 0,
 * implement using recursive method. */
int gcd_r(int x, int y);

/* return the greatest common divisor with x, y >= 0,
 * implement using non-recursive method. */
int gcd(int x, int y);

int main(int argc, char *argv[])
{
    printf("%d\n", gcd_r(1035, 759));
    printf("%d\n", gcd(1035, 759));

    return 0;
}

int gcd_r(int x, int y)
{
    int temp;

    if (x < y) {
        temp = x;
        x = y;
        y = temp;
    }
    if (y == 0) {
        return x;
    } else {
        return gcd_r(y, x % y);
    }
}

int gcd(int x, int y)
{
    int temp;

    if (x < y) {
        temp = x;
        x = y;
        y = temp;
    }
    while (y > 0) {
        temp = y;
        y = x % y;
        x = temp;
    }
    return x;
}
```

## 扩展欧几里得算法

``` {.example}
对于不全为0的正整数a和b，gcd(a, b)表示a和b的最大公约数，则必然存在整数对
x和y，使得gcd(a, b) = ax + by
```

上面的定义引用自[百度百科](http://baike.baidu.com/view/1478219.htm#2)

它的一个应用是解不定方程。引用自[这里](http://blog.csdn.net/fioman/article/details/2455698)
, [这篇文章](http://blog.csdn.net/lhfight/article/details/7755994)
解决了我的一些迷惑。

``` {.example}
对于不定方程ax + by = c，已经a, b, c的值，求解满足方程的一组x, y。
其中a, b, c, x, y都是整数。
```

判断有无解的方法：如果c不是gcd(a, b)的倍数，则无解，否则有解。

求解的方法：

假设ax~1~ + by~1~ = gcd(a, b), 两边除以gcd(a, b)，得到

ax~1~ / gcd(a, b) + by~1~ / gcd(a, b) = 1，两边再同时乘以c，得到：

ax~1~ \* (c / gcd(a, b)) + by~1~ \* (c / gcd(a, b)) = c = ax + by,

于是:

x = x1 \* (c / gcd(a, b))

y = y1 \* (c / gcd(a, b))

这样，求x, y就转化成为求x1和y1了。

现在再来看看怎么求x1和y1。

如果b = 0, 则gcd(a, b) = a，由ax1 + by1 = gcd(a, b) = a，得

x1 = 1, y1为任意整数，比如y1 = 0。

如果b不为0, 则有：

ax1 + by1 = gcd(a, b) = gcd(b, a % b) = bx2 + (a % b)y2

= bx2 + (a - (a / b) \* b)y2 = bx2 + ay2 - (a / b) \* b \* y2

于是有：

x1 = y2, y1 = x2 - (a / b)y2

因此，只要求出x2和y2，就能求出x1和y1了。这样类推下去，直到b =
0，便能得到 一组解，最后可以求出x1和y1，然后就可以求出x和y了。

废话少说，马上上代码：

``` {.c}
#include <stdio.h>

/* extended euclidean algorithm:gcd(a, b) = ax + by
 * return gcd(a, b) */   
int extended_euclid(int a, int b, int *x, int *y);

/* solve the indefinite equation: ax + by = c,
 * if no solution, return 0, otherwise return 1
 * and set one solution in x and y. */
int indefinite_equation(int a, int b, int c, int *x, int *y);

int main(int argc, char *argv[])
{
    int x, y;

    if (indefinite_equation(2, 3, 8, &x, &y)) {
        printf("x=%d, y=%d\n", x, y);
    }

    return 0;
}

int indefinite_equation(int a, int b, int c, int *x, int *y)
{
    int gcd;

    gcd = extended_euclid(a, b, x, y);
    if (c % gcd != 0) {
        return 0;
    }
    *x = *x * (c / gcd);
    *y = *y * (c / gcd);
    return 1;
}

int extended_euclid(int a, int b, int *x, int *y)
{
    int x1, y1, gcd;

    if ( b == 0) {
        *x = 1;
        *y = 0;
        return a;
    }
    gcd = extended_euclid(b, a % b, &x1, &y1);
    *x = y1;
    *y = x1 - (a / b) * y1;

    return gcd;
}
```

它的另外一个应用是求解模的乘法逆元(multiplicative inverse)。
对于我这种数学白痴，还是先来了解一下乘法逆元是个什么东西好了，引用自[wikipedia](http://en.wikipedia.org/wiki/Modular_multiplicative_inverse)
。

整数a模m的乘法逆元是一个整数x，如果它们满足：

a^-1^ ≡ x (mod m)

也可以写成：

ax ≡ aa^-1^ ≡ 1 (mod m)

a的乘法逆元存在当且仅当a与m互质。

好，了解完之后，再来看看怎么运用扩展欧几里得算法来求a模m的乘法逆元。

要求a模m的乘法逆元x，有：

ax ≡ 1 (mod m)

即ax和1对m同余，且余数肯定为1,于是存在整数q，使得：

ax = qm + 1

于是，有：

ax - mq = 1

这个式子有点类似扩展欧几里得算法里面的

ax + by = gcd(a, b)

这个式子了。

这里，a和m是已知的，gcd(a, m)为右边的1,q又是没有用的。

代码实现非常简单，用到了上例中的扩展欧几里得算法的函数：

``` {.c}
/* calculate the multiplicative inverse of (a mod m), return
 * the inverse */   
int mul_inverse(int a, int m);

int mul_inverse(int a, int m)
{
    int inverse, y;

    extended_euclid(a, m, &inverse, &y);

    return inverse;
}
```

最后，感谢[这篇文章](http://www.cppblog.com/zoyi-zhang/articles/44811.html)
帮助我理解乘法逆元。

# 素性测试

primality testing. 合数的英文是composite.

# 费马小定理

fermat's little theorem：

如果p是质数，则对于所有的1 ≤ a &lt; p, 有

a^p-1^ ≡ 1 (mod p)

如果a是任意整数，则有：

a^p^ ≡ a (mod p), 也可以写成：

a^p^ - a ≡ 0 (mod p)

即a^p^ - a是p的倍数。

这个定理的主要应用是进行素性测试。

先来看一下怎么判断一个正整数是不是素数。

最直接的做法,使用定义来判断：

``` {.example}
typedef unsigned int uint;

/* judge whether an integer N is prime
 * in terms of the definition. */   
int is_prime_direct(uint n);

int is_prime_direct(uint n)
{
    int i;

    if (n < 2) {
        return 0;
    }
    for (i = 2; i < n; i++) {
        if (n % i == 0) {
            return 0;
        }
    }
    return 1;
}
```

这个方法是从2到n-1遍历一次，看n能不能被其中一个数整除，事实上，当遍历
超过n / 2的时候，就可以确定看来的数不可能整除n了，于是，可以提高一倍的
速度。

``` {.c}
typedef unsigned int uint;

/* judge whether an integer N is prime
 * in terms of the definition and stop
 * at n / 2. */   
int is_prime_half(uint n);

int is_prime_half(uint n)
{
    int i;

    if (n < 2) {
        return 0;
    }
    for (i = 2; i <= n / 2; i++) {
        if (n % i == 0) {
            return 0;
        }
    }
    return 1;
}   
```

我们再来想一下，如果要判断53是不是素数，发现：

51不能被2整除，53 / 2 = 26，余数为1

53不能被3整除，53 / 3 = 17，余数为2

...

53 不能被17整除, 53 / 17 = 3，余数为2

...

53不能被26整除，53 / 26 = 2，余数为1

可以看出来有重复了，事实上判断2和判断26是一样的，判断3和判断17也是一样的，
我们做了很多重复工作，事实上只要判断到√53 就行了，因为后面的判断和前面是
重复的。于是，就可以写出来更快的算法，速度又有了质的提高。

``` {.c}
#include <math.h>

typedef unsigned int uint;

/* judge whether an integer N is prime
 * in terms of the definition and stop
 * at sqrt(n). */   
int is_prime_sqrt(uint n);

int is_prime_sqrt(uint n)
{
    int i;

    if (n < 2) {
        return 0;
    }
    for (i = 2; i <= sqrt(n); i++) {
        if (n % i == 0) {
            return 0;
        }
    }
    return 1;
}   
```

现在我们就可以用费马小定理来测试素数了，比如要判断正整数n是不是素数，
我们可以在{1, ..., n - 1}中随机取一个数a来测试，如果

a^n-1^ ≡ 1 (mod n)

则n可能是一个素数。说是可能，因为有一些合数也能通过费马测试，这些数被称为
carmichael number(卡迈克尔数)，但是这些数出现的概率非常低。

这里影响效率的操作主要是计算a^(n-1)^ mod
n，还好我们有前面的快速求幂模法（也 称蒙格马利法）。

继续上代码：

``` {.c}
/* judge whether an integer N is prime
 * in terms of fermat's little theorem */
int is_prime_fermat(uint n);

/* calculate BASE^EXP % MOD, return the result */
int pow_mod(int base, int exp, int mod);

int is_prime_sqrt(uint n)
{
    int i;

    if (n < 2) {
        return 0;
    }
    for (i = 2; i <= sqrt(n); i++) {
        if (n % i == 0) {
            return 0;
        }
    }
    return 1;
}   

int is_prime_fermat(uint n)
{
    int base;

    if (n < 2) {
        return 0;
    }
    base = rand() % (n - 1) + 1;
    if (pow_mod(base, n - 1, n) == 1) {
        return 1;
    } else {
        return 0;
    }
}

int pow_mod(int base, int exp, int mod)
{
    int result;

    result = 1;
    while (exp > 0) {
        if (exp & 1) {
            result = (result * base) % mod;
        }
        base = (base * base) % mod;
        exp >>= 1;
    }
    return result;
}
```

我测试了一下，在判断1000以内的素数时，有12个合数被误以为是素数，说明概率还
是挺大的，因为我们还没有优化。

如果我们随机拿多个数a来测试，则会大大降低错误率。比如我取n /
2个数来测试。

``` {.c}
int is_prime_fermat(uint n)
{
    int base, i;

    if (n < 2) {
        return 0;
    }
    for (i = 0; i < n / 2; i++) {
        base = rand() % (n - 1) + 1;
        if (pow_mod(base, n - 1, n) != 1) {
            return 0;
        }
    }
    return 1;
}
```

这次我测试1000以内的素数就全部正确了。
