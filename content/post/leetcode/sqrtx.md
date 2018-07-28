---
date: 2018-01-14
title: sqrtx 求整数平方根
tags: ["leetcode", "算法"]
categories: ["leetcode"]
---

#### 题目
```
Implement int sqrt(int x).

Compute and return the square root of x.

x is guaranteed to be a non-negative integer.
```

#### 我的思路
我能想到的只有暴力方法，从一个位置开始，一步一步加一，直到找到平方根。

#### 代码
```
    int mySqrt(int x) {
      long long times = 0;
      int y = x;
      while (y > 0) {
        y >>= 1;
        ++times;
        if (y <= times) {
          break;
        }
      }
      long long z = x;
      while (true) {
        long long product = times * times;
        if (product == z) {
          return times;
        } else if (product > z) {
          return times - 1;
        }
        ++times;
      }
    }
```

#### 牛顿迭代法
原来有一种牛逼的算法，叫做牛顿-拉弗森方法, 可以用来计算平方根。想求一个数a的平方，其实就是x^2 = a, 写成函数就是f(x) = x^2 - a, 所以我们只要求出让f(x) = 0时x的值就行了。

这种算法的思想是，f(x)是一个抛物线往下方平移a所形成的图形，我们随便选一个点(x0, f(x0)), 做一条切线，那切线方程就是f(x)的导数g(x) = 2x .

导数其实就是切线的斜率t, 设切线与x轴的交点(x1, 0)与(x0, 0) 的距离为d, 根据三角定理，t = f(x) / d = 2x, 则d = f(x) / 2x = (x^2 - a) / 2x .

容易得出，x1 = x0 - d = x0 - (x0^2 - a) / 2x0 = (x0 + a / x0) / 2 .

一直迭代下去，计算出来x2, x3, x4 ... , 就会不断地逼近a的平方根。

#### 代码
```
    int mySqrt(int x) {
      long long ret = x;
      while (ret * ret > x) {
        ret = (ret + x / ret) / 2;
      }
      return ret;
    }
```

#### 参考资料
- [如何通俗易懂地讲解牛顿迭代法求开方？](https://www.zhihu.com/question/20690553)
- [牛顿迭代法快速寻找平方根](http://www.matrix67.com/blog/archives/361)

