---
{
  "date": "2018-10-31",
  "slug": "longest-common-subsequence",
  "subtitle": "Generic subtitle",
  "title": "Longest common Subsequence"
}
---
<!--more-->

求最长公共子序列的长度，是线性动态规划最基础的内容。现在我自己手写并理解两种实现算法，做此笔记。

## 暴力算法

思路是，双重循环遍历2个数组，当发现2个元素相等时，递归地去找各自数组后面的子序列的最长公共子序列的长度，再加1就是当前最长的公共子序列的长度，然后取最大值。


```python
import random

def lcs_recursive(arr1, arr2, left, right):
    if left >= len(arr1) or right >= len(arr2):
        return 0
    
    max_len = 0
    for i in range(left, len(arr1)):
        for j in range(right, len(arr2)):
            if arr1[i] == arr2[j]:
                length = lcs_recursive(arr1, arr2, i + 1, j + 1)
                max_len = max(max_len, length + 1)
    return max_len

def lcs_bruteforce(arr1, arr2):
    return lcs_recursive(arr1, arr2, 0, 0)
```

## 动态规划

思路是，维护一个二维数组，记录两个下标的当前状态，最长公共子序列的长度。


```python
def lcs_dp(arr1, arr2):
    dp = [[0 for j in range(len(arr2) + 1)] for i in range(len(arr1) + 1)]
    for i in range(len(arr1) - 1, -1, -1):
        for j in range(len(arr2) - 1, -1, -1):
            if arr1[i] == arr2[j]:
                dp[i][j] = dp[i + 1][j + 1] + 1
            else:
                dp[i][j] = max(dp[i][j + 1], dp[i + 1][j])
    return dp[0][0]
```

## 测试

下面来测试一下:


```python
arr1 = [3, 0, 2, 1, 4]
arr2 = [0, 1, 2, 3, 4]
print(lcs_bruteforce(arr1, arr2))
print(lcs_dp(arr1, arr2))


for i in range(0, 10):
    arr1 = [random.randint(0, 9) for i in range(20)]
    arr2 = [random.randint(0, 9) for i in range(20)]
    result1 = lcs_bruteforce(arr1, arr2)
    result2 = lcs_dp(arr1, arr2)
    if result1 != result2:
        print(result1, result2)
```

    3
    3


再做一个性能测试:


```python
import timeit
import random

def test_lcs_bruteforce():
    length = 30
    arr1 = [random.randint(0, 9) for i in range(length)]
    arr2 = [random.randint(0, 9) for i in range(length)]
    lcs_bruteforce(arr1, arr2)
    
def test_lcs_dp():
    length = 30
    arr1 = [random.randint(0, 9) for i in range(length)]
    arr2 = [random.randint(0, 9) for i in range(length)]
    lcs_dp(arr1, arr2)
    
print(timeit.timeit(test_lcs_bruteforce, number=2, setup='from __main__ import test_lcs_bruteforce'))
print(timeit.timeit(test_lcs_dp, number=2, setup='from __main__ import test_lcs_dp'))
```

    12.163799556001322
    0.0012101709944545291



```python
动态规划的速度比暴力算法快太多了！
```

