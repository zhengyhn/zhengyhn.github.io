---
date: 2020-02-29
title: 三角函数和差化积公式推导
tags: ['Math']
categories: ['Math']
---

### 一道函数单调性问题

遇到一道题: 判断 $y = cosx$在区间$(0, \pi)$上的单调性。

通过几何画图是很快看出来的，关键是怎么证明呢?

证明过程是这样的:

令$x_1,x_2 \in (0, \pi), x_1 < x_2$, 则$ cosx_2 - cosx_1 = -2sin\frac{x_2 + x_1}{2}sin\frac{x_2 - x_1}{2} $.

由于$x_1, x_2 \in (0, \pi)$, 所以$\frac{x_2 + x_1}{2} \in (0, \pi), \frac{x_2 - x_1}{2} \in (0, \pi)$，

所以$sin\frac{x_2 + x_1}{2} > 0,  sin\frac{x_2 - x_1}{2} > 0$,

所以$-2sin\frac{x_2 + x_1}{2}sin\frac{x_2 - x_1}{2}$ < 0,

所以$cosx_2 - cosx_1 < 0 $. 所以$y = cosx$ 在区间$(0, \pi)$上单调递减。

这里唯一不明白的地方就是下面这个公式怎么来的?

$ cosx_2 - cosx_1 = -2sin\frac{x_2 + x_1}{2}sin\frac{x_2 - x_1}{2} $

去网上查了一下，原来这叫做和差化积公式。好吧，高中数学全忘光了。

### 和差化积公式

和差化积公式一共有 4 个。

$ sina + sinb = 2sin\frac{a + b}{2}cos\frac{a - b}{2} $

$ sina - sinb = -2cos\frac{a + b}{2}sin\frac{a - b}{2} $

$ cosa + cosb = 2cos\frac{a + b}{2}cos\frac{a - b}{2} $

$ cosa - cosb = -2sin\frac{a + b}{2}sin\frac{a - b}{2} $

但是这么长的公式，怎么记忆呢? 能不能推导?

### 和差化积公式推导

要推导出来和差化积公式，先要学会和角公式。

#### 和角公式

$ sin(a + b) = sinacosb + cosasinb $

$ sin(a - b) = sinacosb - cosasinb $

$ cos(a + b) = cosacosb - sinasinb $

$ cos(a - b) = cosacosb + sinasinb $

有一个记忆口诀: 正余同余正, 余余反正正。

口诀是这样理解的，

- 对于 sin 的和角公式，是**正余同余正**。比如$sin(a + b)$, 就等于 a 的正弦乘以 b 的余弦，然后符号跟+相同，也就是+，再是 a 的余弦乘以 b 的正弦。

- 对于 cos 的和角公式，是**余余反正正**。比如$cos(a + b)$, 就等于 a 的余弦乘以 b 的余弦，然后符号跟+相反，也就是-，再是 a 的正弦乘以 b 的正弦。

对于和角公式的推导，可以通过几何画图来推导，这个就有点麻烦了，记住就行。

#### 和差化积推导过程

有了上面的知识，就可以推导和差化积公式了。这里用到了一个技巧，将一个数折成 2 个数之和或差。

对于$sina + sinb$ 和 $sina - sinb$两条公式, 推导如下:

$ sina = sin(\frac{a + b}{2} + \frac{a - b}{2}) = sin\frac{a + b}{2}cos\frac{a - b}{2} + cos\frac{a + b}{2}sin\frac{a - b}{2} $

$ sinb = sin(\frac{a + b}{2} - \frac{a - b}{2}) = sin\frac{a + b}{2}cos\frac{a - b}{2} - cos\frac{a + b}{2}sin\frac{a - b}{2} $

上面两式相加，得,

$ sina + sinb = 2sin\frac{a + b}{2}cos\frac{a - b}{2} $

两式相减，得,

$ sina - sinb = -2cos\frac{a + b}{2}sin\frac{a - b}{2} $

对于$cosa + cosb$ 和 $cosa - cosb$两条公式, 推导如下:

$ cosa = cos(\frac{a + b}{2} + \frac{a - b}{2}) = cos\frac{a + b}{2}cos\frac{a - b}{2} - sin\frac{a + b}{2}cos\frac{a - b}{2} $

$ cosb = cos(\frac{a + b}{2} - \frac{a - b}{2}) = cos\frac{a + b}{2}cos\frac{a - b}{2} + sin\frac{a + b}{2}cos\frac{a - b}{2} $

上面两式相加，得,

$ cosa + cosb = 2cos\frac{a + b}{2}cos\frac{a - b}{2} $

两式相减，得,

$ cosa - cosb = -2sin\frac{a + b}{2}sin\frac{a - b}{2} $

上面这一条公式就是文章开头用到的那一条。证毕!
