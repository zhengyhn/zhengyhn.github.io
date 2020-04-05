---
date: 2020-03-29
title: 一元二次方程求根公式推导
tags: ['Math']
categories: ['Math']
---

一元二次方程 $ax^2 + bx + c = 0$的求根公式为:

$$ x_{x1,x2} = \frac{-b \pm \sqrt{b^2 - 4ac}}{2a} $$

## 推导过程

根据经验，左边如果可以写在平方的形式，通过开方，就可以求得根，所以我们的目标是通过变换，得到左式为平方的形式。

因为 

$$ax^2 + bx + c = 0$$

将c移到右边，得:

$$ax^2 + bx = -c$$

两边都除以a，得:

$$x^2 + \frac{b}{a}x = -\frac{c}{a} $$

两边都加上$(\frac{b}{2a})^2$, 得:

$$x^2 + \frac{b}{a}x + (\frac{b}{2a})^2 = (\frac{b}{2a})^2 - \frac{c}{a} $$

两边化简，得:

$$(x + \frac{b}{2a})^2 = \frac{b^2 - 4ac}{4a^2} $$

两边开方，得:

$$ x + \frac{b}{2a} = \pm \frac{\sqrt{b^2 - 4ac}}{2a} $$

所以:

$$x_{x1,x2} = -\frac{b}{2a} \pm \frac{\sqrt{b^2 - 4ac}}{2a} = \frac{-b \pm \sqrt{b^2 - 4ac}}{2a} $$

## 韦达公式

韦达公式是这样的:

$$ x_1 + x_2 = -\frac{b}{a} $$
$$ x_1x_2 = \frac{c}{a} $$

推导很简单，直接来就行:

$$ x_1 + x_2 = \frac{-b + \sqrt{b^2 - 4ac}}{2a} + \frac{-b - \sqrt{b^2 - 4ac}}{2a} = $$
$$  \frac{-b}{2a} + \frac{-b}{2a} = \frac{b}{a} $$

$$ x_1x_2 = \frac{-b + \sqrt{b^2 - 4ac}}{2a} \times \frac{-b - \sqrt{b^2 - 4ac}}{2a} = $$
$$ \frac{(-b + \sqrt{b^2 - 4ac})(-b - \sqrt{b^2 - 4ac})}{4a^2} = $$
$$ \frac{(-b)^2 - (b^2 - 4ac)}{4a^2} = \frac{c}{a} $$

## 抛物线的顶点坐标

对于抛物线 $ y = ax^2 + bx + c $, 它的顶点坐标是 $(-\frac{b}{2a}, \frac{4ac - b^2}{4a})$

原理很简单，顶点处导数为0，所以有:

$$ y' = 2ax + b = 0 $$
$$ x = -\frac{b}{2a} $$
$$ y = a(-\frac{b}{2a})^2 - b\frac{b}{2a} + c = \frac{4ac - b^2}{4a} $$


