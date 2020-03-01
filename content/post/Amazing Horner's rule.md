---
{
  "title": "Amazing Horner's rule",
  "subtitle": "Generic subtitle",
  "date": "2018-07-27",
  "slug": "Amazing Horner's rule"
}
---
<!--more-->

在看《算法导论》，思考题里面提到霍纳规则，出于好奇，查一下这个陌生的名词，结果发现了新大陆，原来在中国这个叫秦九韶算法，好像在中学的时候看过，现在肯定是忘光了，复习一下。

原来是一种计算一元多次函数的高效算法。比如给一个函数$f(x) = 1 + 2x + 3x^2 + 4x^3 + 5x^4$, 让我来写代码来计算，我一定是暴力计算，直接用求幂函数pow来算x，然后加起来。

现在就体现了算法的重要性，如果用上面的暴力算法，时间复杂度是$O(n^2)$，如果应用霍纳规则，时间复杂度居然可以达到$O(n)$!

霍纳法则的思想很简单，就是反过来想，从后面往前面计算，将已经计算的x的幂记录起来，以减少幂运算的重复计算，化简的方法是:
$$ 
f(x) = 1 + 2x + 3x^2 + 4x^3 + 5x^4 
= 1 + x(1 + x(2 + x(3 + x(4 + 5x))))
$$

看一下代码, 这里写了2个方法，brute_force是暴力算法，horner_rule就是霍纳法则，然后用timeit做性能测试


```python
def brute_force():
    """
    return y = 1 + 2x + 3x^2 + 4x^3 + 5x^4
    """
    x = 10
    y = 0
    for i in range(1, 6):
        y = y + i * x**(i - 1)
    return y

def horner_rule():
    """
    return y = 1 + 2x + 3x^2 + 4x^3 + 5x^4
    """
    x = 10
    y = 5
    for i in range(4, 0, -1):
        y = i + y * x
    return y

print(brute_force())
print(horner_rule())

import timeit
print(timeit.repeat('brute_force()', number=10000, setup="from __main__ import brute_force"))

print(timeit.repeat('horner_rule()', number=10000, setup="from __main__ import horner_rule"))
```

    54321
    54321
    [0.030609464971348643, 0.023056003963574767, 0.021249629789963365]
    [0.008943730033934116, 0.008274385938420892, 0.008337561041116714]


这里可以看到，执行1万次，暴力法平均时间大概在0.025s左右，而霍纳法则可以达到0.008s，数据量越大，越能体现2种算法的性能高低。

乍一看，平时好像用不上，用时方悔读书晚！
