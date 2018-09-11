---
{
  "title": "Learning LSD String Sorting Algorithm",
  "subtitle": "Generic subtitle",
  "date": "2018-09-08",
  "slug": "Learning LSD String Sorting Algorithm"
}
---
<!--more-->

## 场景
假设有一个手机号数组，要求对这个数组进行排序，怎么样排序最快呢？

一般我们会用快排来实现，现场写一个快排:


```python
from collections import deque

def swap(strs, i, j):
    temp = strs[i]
    strs[i] = strs[j]
    strs[j] = temp
    
def quick_sort(strs):
    queue = deque([0, len(strs) - 1])
    while len(queue) > 0:
        start = queue.popleft()
        end = queue.popleft()
        if start >= end:
            continue
        pivot_idx = (start + end) // 2
        swap(strs, pivot_idx, end)
        # 左边最后一个元素位置
        last_left = start - 1
        for i in range(start, end):
            if strs[i] <= strs[end]:
                last_left += 1
                swap(strs, last_left, i)
        last_left += 1
        swap(strs, last_left, end)
        
        if start < last_left - 1:
            queue.append(start)
            queue.append(last_left - 1)
        if last_left + 1 < end:
            queue.append(last_left + 1)
            queue.append(end)    
            
phones = []
for i in range(0, 10):
    phone = ''.join([chr(random.randint(0, 9) + ord('0')) for i in range(0, 11)])
    phones.append(phone)
print(phones)
quick_sort(phones)
print(phones)
```

    ['66561365246', '38002373724', '03228006448', '21864795885', '30174569904', '49813215241', '38227191647', '76392761414', '91683092344', '41287861631']
    ['03228006448', '21864795885', '30174569904', '38002373724', '38227191647', '41287861631', '49813215241', '66561365246', '76392761414', '91683092344']


快排的平均时间复杂度为$O(NlogN)$, 最坏情况下为$O(N^2)$。

如果用python内置的sort函数，来实现，代码如下:


```python
import random

phones = []
for i in range(0, 10):
    phone = ''.join([chr(random.randint(0, 9) + ord('0')) for i in range(0, 11)])
    phones.append(phone)
print(phones)

phones.sort()
print(phones)
```

    ['77394403528', '15857558563', '45028910925', '71613153605', '81278799222', '12345985296', '34263442777', '32325476547', '11424460014', '04413300618']
    ['04413300618', '11424460014', '12345985296', '15857558563', '32325476547', '34263442777', '45028910925', '71613153605', '77394403528', '81278799222']


python的sort的内部实现是TimSort，一种改良的归并排序，平均时间复杂度为$O(NlogN)$, 这个算法非常快，据说内部实现是1000多行的C语言代码。

针对我们这个应用场景，《算法》一书里面讲到另外一种字符串排序算法，叫LSD字符串排序算法。它的时间复杂度可以达到$O(N * W)$，其中W是字符串的长度。

要理解这个算法，首先要学会桶排序，或者叫基数排序。

## 桶排序

桶排序是一种针对特定场景的排序，比如我已知一个数组每个元素的取值范围是0~9，它的思想就是用一个辅助数组来记录每个元素出现的次数，最后回写原数组。看代码:



```python
nums = [random.randint(0, 9) for i in range(10)]

def bucket_sort(arr):
    aux = [0] * 10
    for i in arr:
        aux[i] += 1
    i, j = 0, 0
    for i in range(0, len(aux)):
        while aux[i] > 0:
            arr[j] = i
            aux[i] -= 1
            j += 1

print(nums)
bucket_sort(nums)
print(nums)
```

    [1, 4, 7, 5, 2, 8, 3, 6, 8, 6]
    [1, 2, 3, 4, 5, 6, 6, 7, 8, 8]


可以看到，桶排序的时间复杂度为$O(N + M)$，其中M为取值范围的大小，另外需要增加$O(M)$的空间复杂度。

由于特定字符串的取值范围是固定的，比如手机号就是11个0~9的数字，如果使用桶排序，是不是可以大大提高速度呢？

## LSD字符串排序

LSD(Least-Significant-Digit)字符串排序，它的思想是这样的，从每个字符串的低位开始，对最低位做一次桶排序，完成之后，再往高位做桶排序，一共做W次桶排序，其中W是字符串的长度。注意，在我们这个应用场景里面，每个字符串的长度是一样的。代码实现如下:


```python
phones = []
for i in range(0, 10):
    phone = ''.join([chr(random.randint(0, 9) + ord('0')) for i in range(0, 11)])
    phones.append(phone)

def lsd_sort(strs):
    # 从低位往高位
    for i in range(len(strs[0]) - 1, -1, -1):
        # 记录各个字符对应的字符串
        aux = [[] for _ in range(10)]
        for j in range(0, len(strs)):
            idx = ord(strs[j][i]) - ord('0')
            aux[idx].append(strs[j])
        k = 0
        for j in range(0, 10):
            for item in aux[j]:
                strs[k] = item
                k += 1
    
print(phones)
lsd_sort(phones)
print(phones)
```

    ['05768655755', '38865328632', '71828701047', '09656342951', '02847623199', '15963698899', '41078839807', '43612067480', '00632587965', '23790216100']
    ['00632587965', '02847623199', '05768655755', '09656342951', '15963698899', '23790216100', '38865328632', '41078839807', '43612067480', '71828701047']


这个算法，每次M个循环里面，都做了1次N循环和1次10循环，所以总的时间复杂度$O(N * M)$。来写个性能测试来比较一下这3种排序算法哪个更快。


```python
import timeit

phone_len = 11
N = 10000

def lsd_sort_test():
    phones = []
    for i in range(0, N):
        phone = ''.join([chr(random.randint(0, 9) + ord('0')) for i in range(0, phone_len)])
        phones.append(phone)
    lsd_sort(phones)
    
def timsort_test():
    phones = []
    for i in range(0, N):
        phone = ''.join([chr(random.randint(0, 9) + ord('0')) for i in range(0, phone_len)])
        phones.append(phone)
    phones.sort()
    
def quick_sort_test():
    phones = []
    for i in range(0, N):
        phone = ''.join([chr(random.randint(0, 9) + ord('0')) for i in range(0, phone_len)])
        phones.append(phone)
    quick_sort(phones)
    
print(timeit.timeit('lsd_sort_test()', number=10, setup="from __main__ import lsd_sort_test"))
print(timeit.timeit('timsort_test()', number=10, setup="from __main__ import timsort_test"))
print(timeit.timeit('quick_sort_test()', number=10, setup="from __main__ import quick_sort_test"))
```

    2.6599199120000776
    2.1297218710005836
    2.5268346160000874


测试发现，对于10000个随机11位字符串，内置的timsort算法是最快的，然后是快排，然后是LSD排序算法。这下子啪啪啪打脸了，本来以为这个LSD算法$O(N * W)$的时间复杂度是比$O(NlogN)$要快，但是结果是比快排要慢。大概的原因是，由于字符串长度是11，共10000个用例，log2(10000)是13.28，只比11大一点点，由于LSD排序还要开辟新空间并回写到原数组，所以会慢一些，如果把字符串长度调小或者用例数量调大，LSD排序是要比快排快的。
比如字符串长度为3:


```python
phone_len = 3
N = 10000

def lsd_sort_test():
    phones = []
    for i in range(0, N):
        phone = ''.join([chr(random.randint(0, 9) + ord('0')) for i in range(0, phone_len)])
        phones.append(phone)
    lsd_sort(phones)
    
def timsort_test():
    phones = []
    for i in range(0, N):
        phone = ''.join([chr(random.randint(0, 9) + ord('0')) for i in range(0, phone_len)])
        phones.append(phone)
    phones.sort()
    
def quick_sort_test():
    phones = []
    for i in range(0, N):
        phone = ''.join([chr(random.randint(0, 9) + ord('0')) for i in range(0, phone_len)])
        phones.append(phone)
    quick_sort(phones)
    
print(timeit.timeit('lsd_sort_test()', number=10, setup="from __main__ import lsd_sort_test"))
print(timeit.timeit('timsort_test()', number=10, setup="from __main__ import timsort_test"))
print(timeit.timeit('quick_sort_test()', number=10, setup="from __main__ import quick_sort_test"))
```

    0.780066997999711
    0.6570827010000357
    1.2234976199997618


TimSort真的是快，这算法真不是吹牛的，不愧是python和java的内置算法。

## 参考资料
- 《算法》第4版
