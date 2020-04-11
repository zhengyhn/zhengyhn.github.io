---
date: 2020-04-11
title: 一个重要权限的证明
tags: ['Math']
categories: ['Math']
---

这个权限是:

$$ \lim_{x \to 0}\frac{sinx}{x} = 1 $$

证明方法是用几何方法，这里学到了不少东西。

先看图:

{{< figure src="/images/sinx_x_circle.png" width="50%" title="sinx/x circle" >}}

证明:

做一个半径为 1 的圆，x 为该扇形的弧度，所以弧长 $\widehat{AB} = x, BO = AO = 1 $,

则:

$$ sinx = \frac{BC}{BO} = BC $$

由于:

$$ sinx = \frac{AD}{DO} $$
$$ cosx = \frac{AO}{DO} = \frac{1}{DO} $$

两式相除，得:

$$ tanx = \frac{\frac{AD}{DO}}{\frac{1}{DO}} = AD $$

三角形 AOB 的面积为:

$$ S_{\Delta AOB} = \frac{1}{2}BC \times AO = \frac{1}{2}BC \times 1 = \frac{1}{2}sinx $$

扇形 AOB 的面积为:

$$ S_{\widehat{AOB}} = \frac{1}{2}\widehat{AB} \times AO = \frac{1}{2}\widehat{AB} \times 1 = \frac{1}{2}x $$

三角形 AOD 的面积为:

$$ S_{\Delta AOD} = \frac{1}{2}AD \times AO = \frac{1}{2}AD \times 1 = \frac{1}{2}tanx $$

从图中可以看出，

三角形 AOB 的面积 < 扇形 AOB 的面积 < 三角形 AOD 的面积

所以:

$$ \frac{1}{2}sinx \lt \frac{1}{2}x \lt \frac{1}{2}tanx $$

所以:

$$ sinx \lt x \lt tanx $$

都除以 sinx，得:

$$ 1 \lt \frac{x}{sinx} \lt \frac{1}{cosx} $$

取倒数，得:

$$ cosx \lt \frac{sinx}{x} \lt 1 $$

通过画图，容易得:

$$ \lim_{x \to 0}cosx = 1$$

$$ \lim_{x \to 0}1 = 1 $$

根据夹逼定理，得
$$ \lim_{x \to 0}\frac{sinx}{x} = 1 $$

证毕。
