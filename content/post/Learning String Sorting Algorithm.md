---
{
  "date": "2018-09-08",
  "slug": "Learning String Sorting Algorithm",
  "subtitle": "Generic subtitle",
  "title": "Learning String Sorting Algorithm"
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

    ['25900803422', '82533056198', '36977030221', '82656350178', '50197010257', '55052712489', '80915203421', '78663941583', '72021821280', '23338950918']
    ['23338950918', '25900803422', '36977030221', '50197010257', '55052712489', '72021821280', '78663941583', '80915203421', '82533056198', '82656350178']


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

    ['49993400585', '90540095748', '93341457109', '42794844210', '72054270540', '13814446283', '79563846731', '26819845915', '14973623007', '31301649154']
    ['13814446283', '14973623007', '26819845915', '31301649154', '42794844210', '49993400585', '72054270540', '79563846731', '90540095748', '93341457109']


python的sort的内部实现是TimSort，一种改良的归并排序，平均时间复杂度为$O(NlogN)$, 这个算法非常快，据说内部实现是1000多行的C语言代码。

针对我们这个应用场景，《算法》一书里面讲到另外3种字符串排序算法，一种叫LSD字符串排序算法。它的时间复杂度可以达到$O(N * W)$，其中W是字符串的长度。一种叫MSD字符串排序算法，时间复杂度在$O(N)$至$O(N * W)$之间。还有一种叫三向快速排序，时间复杂度在$O(N)$至$O(N * logN)$之间。

要理解这3个算法，首先要学会桶排序，或者叫基数排序。

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

    [0, 1, 0, 9, 3, 2, 9, 2, 4, 5]
    [0, 0, 1, 2, 2, 3, 4, 5, 9, 9]


可以看到，桶排序的时间复杂度为$O(N + M)$，其中M为取值范围的大小，另外需要增加$O(M)$的空间复杂度。

由于特定字符串的取值范围是固定的，比如手机号就是11个0~9的数字，如果使用桶排序，是不是可以大大提高速度呢？

## LSD字符串排序

LSD(Least-Significant-Digit)低位优先字符串排序，它的思想是这样的，从每个字符串的低位开始，对最低位做一次桶排序，完成之后，再往高位做桶排序，一共做W次桶排序，其中W是字符串的长度。注意，在我们这个应用场景里面，每个字符串的长度是一样的。代码实现如下:


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

    ['04145432564', '86748121667', '26710905823', '51612671904', '86609844213', '30603651543', '51116348132', '48212721677', '70985842642', '46470938975']
    ['04145432564', '26710905823', '30603651543', '46470938975', '48212721677', '51116348132', '51612671904', '70985842642', '86609844213', '86748121667']


这个算法，每次M个循环里面，都做了1次N循环和1次10循环，所以总的时间复杂度$O(N * M)$。

## MSD字符串排序算法

MSD(Most Significant Digital)高位优先字符串排序，它的思想是这样的，从每个字符串的高位开始，做一次桶排序，高位字符可能有很多是相同的，对它每组相同的字符，对它们的子字符串递归地做一次MSD排序，为了提高性能，当每组元素个数少于某个阀值时，做插入排序。这样子，大部分情况下，只需要看前面几个字符就可以完成整个排序。代码实现如下：


```python
phones = []
for i in range(0, 10):
    phone = ''.join([chr(random.randint(0, 9) + ord('0')) for i in range(0, 11)])
    phones.append(phone)

def insert_sort(strs, low, high, index):
    for i in range(low + 1, high + 1):
        temp = strs[i]
        j = i - 1
        while j >= low:
            if temp[index:] < strs[j][index:]:
                strs[j + 1] = strs[j]
            j -= 1
        strs[j + 1] = temp

def msd_sort(strs):
    # 从高位开始
    msd_sort_r(strs, 0, len(strs) - 1, 0)
    
def msd_sort_r(strs, low, high, index):
    if high <= low or index >= len(strs[0]):
        return
    if high - low <= 7:
        insert_sort(strs, low, high, index)
        return
    aux = [[] for _ in range(10)]
    for i in range(low, high + 1):
        idx = ord(strs[i][index]) - ord('0')
        aux[idx].append(strs[i])
    k = low
    for i in range(0, len(aux)):
        for item in aux[i]:
            strs[k] = item
            k += 1
    for i in range(0, len(aux)):
        msd_sort_r(strs, low, low + len(aux[i]) - 1, index + 1)
        low += len(aux[i])
        
print(phones)
msd_sort(phones)
print(phones)
```

    ['08914475530', '36382806030', '17292598106', '79423140725', '61884265921', '96644424367', '48257134818', '92088637322', '12274980975', '99064046401']
    ['08914475530', '12274980975', '17292598106', '36382806030', '48257134818', '61884265921', '79423140725', '99064046401', '96644424367', '99064046401']


## 三向字符串排序

快速排序的原理我们都知道，就是取一个参考元素，把比它小的元素放到左边，比它大的元素放到右边，然后递归下去。

三向字符串排序的原理是这样的，取一个参考元素，从高位字符开始，高位字符比参考元素的小的在左边，比它大的在右边，中间的是高位元素相同的，这个时候，除了递归左右两边的，还要把高位元素相同的一组元素，按下一位继续排序。代码实现如下：


```python
phones = []
for i in range(0, 10):
    phone = ''.join([chr(random.randint(0, 9) + ord('0')) for i in range(0, 11)])
    phones.append(phone)

def quick3_sort(strs):
    # 从高位开始
    quick3_sort_r(strs, 0, len(strs) - 1, 0)
    
def quick3_sort_r(strs, low, high, index):
    if high <= low or index >= len(strs[0]):
        return
    if high - low <= 7:
        insert_sort(strs, low, high, index)
        return
    pivot_idx = low
    i = low + 1
    last_left = low - 1
    first_right = high + 1
    while i < first_right:
        if strs[i][index] < strs[pivot_idx][index]:
            last_left += 1
            temp = strs[last_left]
            strs[last_left] = strs[i]
            strs[i] = temp
            pivot_idx = last_left + 1
        elif strs[i][index] > strs[pivot_idx][index]:
            first_right -= 1
            temp = strs[first_right]
            strs[first_right] = strs[i]
            strs[i] = temp
        else:
            i += 1
    quick3_sort_r(strs, low, last_left, index)
    quick3_sort_r(strs, last_left + 1, first_right - 1, index + 1)
    quick3_sort_r(strs, first_right, high, index)
        
print(phones)
quick3_sort(phones)
print(phones)
```

    ['08104132906', '31709678996', '08273091795', '79987194126', '83552801655', '59365186752', '97600677349', '22396353108', '81547158280', '73500845523']
    ['08273091795', '08273091795', '31709678996', '73500845523', '81547158280', '97600677349', '97600677349', '83552801655', '83552801655', '97600677349']


来写个性能测试来比较一下这几种排序算法哪个更快。


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
   
def msd_sort_test():
    phones = []
    for i in range(0, N):
        phone = ''.join([chr(random.randint(0, 9) + ord('0')) for i in range(0, phone_len)])
        phones.append(phone)
    msd_sort(phones)

def quick3_sort_test():
    phones = []
    for i in range(0, N):
        phone = ''.join([chr(random.randint(0, 9) + ord('0')) for i in range(0, phone_len)])
        phones.append(phone)
    quick3_sort(phones)
    
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
print(timeit.timeit('msd_sort_test()', number=10, setup="from __main__ import msd_sort_test"))
print(timeit.timeit('quick3_sort_test()', number=10, setup="from __main__ import quick3_sort_test"))
print(timeit.timeit('timsort_test()', number=10, setup="from __main__ import timsort_test"))
print(timeit.timeit('quick_sort_test()', number=10, setup="from __main__ import quick_sort_test"))
```

    2.858616407000227
    2.4598556860000826
    2.801925394014688
    2.146542837988818
    2.5571339470043313


测试发现，对于10000个随机11位字符串，内置的timsort算法是最快的，然后是MSD算法，然后是快排，然后是三向快速排序，然后是LSD排序算法。这下子啪啪啪打脸了，本来以为这个LSD算法$O(N * W)$的时间复杂度是比$O(NlogN)$要快，但是结果是比快排要慢。大概的原因是，由于字符串长度是11，共10000个用例，log2(10000)是13.28，只比11大一点点，由于LSD排序还要开辟新空间并回写到原数组，所以会慢一些，如果把字符串长度调小或者用例数量调大，LSD排序是要比快排快的。三向快速排序比普通快速排序慢的原因是，三向快速排序适合有公共前缀或者很多重复元素的情况下。
假设字符串长度为3:


```python
phone_len = 3
N = 10000

def lsd_sort_test():
    phones = []
    for i in range(0, N):
        phone = ''.join([chr(random.randint(0, 9) + ord('0')) for i in range(0, phone_len)])
        phones.append(phone)
    lsd_sort(phones)

def msd_sort_test():
    phones = []
    for i in range(0, N):
        phone = ''.join([chr(random.randint(0, 9) + ord('0')) for i in range(0, phone_len)])
        phones.append(phone)
    msd_sort(phones)
    
def quick3_sort_test():
    phones = []
    for i in range(0, N):
        phone = ''.join([chr(random.randint(0, 9) + ord('0')) for i in range(0, phone_len)])
        phones.append(phone)
    quick3_sort(phones)
    
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
print(timeit.timeit('msd_sort_test()', number=10, setup="from __main__ import msd_sort_test"))
print(timeit.timeit('quick3_sort_test()', number=10, setup="from __main__ import quick3_sort_test"))
print(timeit.timeit('timsort_test()', number=10, setup="from __main__ import timsort_test"))
print(timeit.timeit('quick_sort_test()', number=10, setup="from __main__ import quick_sort_test"))
```

    0.8569560649921186
    0.8670862840081099
    1.2120806089951657
    0.7727651970053557
    1.326724906975869


TimSort真的是快，这算法真不是吹牛的，不愧是python和java的内置算法。

## 参考资料
- 《算法》第4版
