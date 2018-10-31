---
{
  "date": "2018-10-29",
  "slug": "Longest Increasing Subsequence",
  "subtitle": "Generic subtitle",
  "title": "Longest Increasing Subsequence"
}
---
<!--more-->

求最长递增子序列的长度也是动态规划里面的基础问题。记得当时大学的时候碰到这个问题，想破了脑袋也想不出来非暴力的解法，现在写起动态规划的算法还是比较得心应手的，证明这几年我的确是进步了。

下面重新复习这个问题，作此笔记。

## 暴力算法

思路是，遍历每个元素，往后面找到一个比它大的数，递归地调用，计算出来当前的最长递增子序列的长度，再取每个元素计算出来的长度取一个最大值。时间得杂度为$O(n!)$


```python
import random

def lis_recursive(arr, i):
    if i == len(arr) - 1:
        return 1
    max_len = 0
    for j in range(i + 1, len(arr)):
        if arr[j] > arr[i]:
            length = lis_recursive(arr, j)
            if length > max_len:
                max_len = length
    return max_len + 1

def lis_bruteforce(arr):
    max_len = 0
    for i in range(len(arr)):
        length = lis_recursive(arr, i)
        if length > max_len:
            max_len = length
    return max_len
```

## 动态规划

思路是，将上面的递归算法的递归每一步，用一个数组缓存起来结果，以避免重复计算，最后取最大的那个。时间复杂度为$O(n^2)$


```python
def lis_dp(arr):
    max_len = 0
    dp = [1 for j in range(0, len(arr))]
    for i in range(len(arr) - 2, -1, -1):
        for j in range(i + 1, len(arr)):
            if arr[i] < arr[j]:
                dp[i] = max(dp[j] + 1, dp[i])
        max_len = max(max_len, dp[i])
    return max_len
```

## 二分查找法

这是一个非常巧妙的方法，基于的思想是，为了得到最长的递增子序列的长度，那当前元素肯定是越小越好，因为这样子后面才可以有更多的元素跟在后面。

用一个数组min_ends记录在i位置，当最长递增子序列长度为i + 1时，末尾的元素最小的那个，由于递增是有序的，所以可以用二分查找法来找到插入的位置，整个算法的时间复杂度为$O(nlogn)$


```python
def bin_search(arr, start, end, key):
    while start <= end:
        mid = (start + end) // 2
        if key <= arr[mid]:
            end = mid - 1
        else:
            start = mid + 1
    return start

def lis_bin_search(arr):
    min_ends = [0 for i in range(len(arr))]
    min_ends[0] = arr[0]
    length = 1
    for i in range(1, len(arr)):
        pos = bin_search(min_ends, 0, length - 1, arr[i])
        min_ends[pos] = arr[i]
        if pos == length:
            length += 1
    return length
```

## 测试

做一个简单的测试，然后随机生成一些数据用于测试。


```python
arr = [3, 0, 2, 1, 4]
print(lis_bruteforce(arr))
print(lis_dp(arr))
print(lis_bin_search(arr))

for i in range(0, 100):
    arr = [random.randint(0, 9) for i in range(10)]
    result1 = lis_bruteforce(arr)
    result2 = lis_dp(arr)
    result3 = lis_bin_search(arr)
    if result1 != result2 or result2 != result3:
        print(result1, result2, result3)
```

    3
    3
    3


发现3个算法的结果是一致的。

## 性能测试

对3个算法做性能测试:


```python
import timeit
import random

def test_lis_bruteforce():
    length = random.randint(1, 100)
    arr = [random.randint(0, 10) for i in range(length)]
    lis_bruteforce(arr)
    
def test_lis_dp():
    length = random.randint(1, 100)
    arr = [random.randint(0, 10) for i in range(length)]
    lis_dp(arr)
 
def test_lis_bin_search():
    length = random.randint(1, 100)
    arr = [random.randint(0, 10) for i in range(length)]
    lis_bin_search(arr)
    
print(timeit.timeit(test_lis_bruteforce, number=10, setup='from __main__ import test_lis_bruteforce'))
print(timeit.timeit(test_lis_dp, number=10, setup='from __main__ import test_lis_dp'))
print(timeit.timeit(test_lis_bin_search, number=10, setup='from __main__ import test_lis_bin_search'))
```

    2.092267832995276
    0.005408962009823881
    0.0011912080080946907


可以看到暴力算法非常慢，动态规划算法要快个几百倍，而二分查找的算法更快！

## 参考资料
- [长递增子序列 O(NlogN)算法 ](https://www.felix021.com/blog/read.php?1587)
- [leetcode](https://leetcode.com/problems/longest-increasing-subsequence/solution/)
