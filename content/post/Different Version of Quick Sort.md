---
{
  "date": "2018-10-04",
  "slug": "Different Version of Quick Sort",
  "subtitle": "Generic subtitle",
  "title": "Different Version of Quick Sort"
}
---
<!--more-->

最近在看《编程珠玑》，讲到快排的时候，发现除了我之前一直用的划分方法之外，还有一种划分方法，可以在大量元素相等的情况下保持较大的效率，查了一下，原来我以前一直写的划分方法叫做单向扫描法，现在这种方法叫做双向扫描法。于是，自己动手实现一下，涨知识。

因为后面要做性能比较，出于公平，都使用非递归方式来实现，用了python的deque来当作栈来使用。

下面这个是我之前很熟悉的单向扫描法的实现：


```python
import random
from collections import deque

def quick_sort1(arr):
    queue = deque([0, len(arr) - 1])
    while len(queue) > 0:
        left = queue.popleft()
        right = queue.popleft()
        if left >= right:
            continue
        pivot_index = random.randint(left, right)
        arr[pivot_index], arr[right] = arr[right], arr[pivot_index]
        store_index = left
        for i in range(left, right):
            if arr[i] < arr[right]:
                arr[i], arr[store_index] = arr[store_index], arr[i]
                store_index += 1
        arr[store_index], arr[right] = arr[right], arr[store_index]
        queue.extend([left, store_index - 1, store_index + 1, right])
```

而下面的是双向扫描法，不同之处在于划分的代码，使用2个指针，一前一后跟参照元素比较，不会每次都交换，例如最左边的元素，只有遇到比参照元素大的元素时才会停止扫描，这样在很多元素相同的情况下，可以减少很多次交换。

代码如下：


```python
def quick_sort2(arr):
    queue = deque([0, len(arr) - 1])
    while len(queue) > 0:
        left = queue.popleft()
        right = queue.popleft()
        if left >= right:
            continue
        pivot_index = random.randint(left, right)
        arr[pivot_index], arr[right] = arr[right], arr[pivot_index]
        i = left - 1
        j = right
        while i < j:
            i += 1
            j -= 1
            while i <= j and arr[i] < arr[right]:
                i += 1
            while arr[j] > arr[right]:
                j -= 1
            if i < j:
                arr[i], arr[j] = arr[j], arr[i]
        arr[i], arr[right] = arr[right], arr[i]
        queue.extend([left, i - 1, i + 1, right])
```

再试一下，当元素个数比较少时，使用插入排序来代替，看下会不会提高性能。


```python
def quick_sort3(arr):
    queue = deque([0, len(arr) - 1])
    while len(queue) > 0:
        left = queue.popleft()
        right = queue.popleft()
        if left >= right:
            continue
        # 元素个数小于7个时，使用插入排序
        if right - left < 7:
            for i in range(left + 1, right + 1):
                temp = arr[i]
                j = i - 1
                while j >= 0 and arr[j] > temp:
                    arr[j + 1] = arr[j]
                    j -= 1
                arr[j + 1] = temp
            continue
            
        pivot_index = random.randint(left, right)
        arr[pivot_index], arr[right] = arr[right], arr[pivot_index]
        i = left - 1
        j = right
        while i < j:
            i += 1
            j -= 1
            while i <= j and arr[i] < arr[right]:
                i += 1
            while arr[j] > arr[right]:
                j -= 1
            if i < j:
                arr[i], arr[j] = arr[j], arr[i]
        arr[i], arr[right] = arr[right], arr[i]
        queue.extend([left, i - 1, i + 1, right])
```

现在来造一个数据，看下三种算法的实现是否都正确：


```python
arr = [random.randint(0, 9) for i in range(0, 10)]
print(arr)
copy = arr[:]
quick_sort1(copy)
print(copy)
copy = arr[:]
quick_sort2(copy)
print(copy)
copy = arr[:]
quick_sort3(copy)
print(copy)
```

    [8, 9, 4, 9, 2, 9, 3, 9, 0, 6]
    [0, 2, 3, 4, 6, 8, 9, 9, 9, 9]
    [0, 2, 3, 4, 6, 8, 9, 9, 9, 9]
    [0, 2, 3, 4, 6, 8, 9, 9, 9, 9]


看输出，是正确的。最后来做一下性能测试，其中N是元素个数，M是元素取值的最大值，最小值是0。


```python
import timeit

N = 10000
M = N
def test_quick_sort1():
    arr = [random.randint(0, M) for i in range(0, N)]
    quick_sort1(arr)
    
def test_quick_sort2():
    arr = [random.randint(0, M) for i in range(0, N)]
    quick_sort2(arr)

def test_quick_sort3():
    arr = [random.randint(0, M) for i in range(0, N)]
    quick_sort3(arr)
    
print(timeit.timeit('test_quick_sort1()', number=10, setup='from __main__ import test_quick_sort1'))
print(timeit.timeit('test_quick_sort2()', number=10, setup='from __main__ import test_quick_sort2'))
print(timeit.timeit('test_quick_sort3()', number=10, setup='from __main__ import test_quick_sort3'))
```

    0.6726022359216586
    0.6794583379523829
    0.5315109230577946


可以看到，两种划分方式的速度几乎是一样的，这是因为元素的取值足够随机，如果每个元素都不一样，两种算法的速度几乎一致。而混合了插入排序的实现就有较大的速度提高。

再来改一下M值，让随机生成的数据有较多的相等元素，看一下效果怎么样：


```python
N = 10000
M = N / 100

print(timeit.timeit('test_quick_sort1()', number=10, setup='from __main__ import test_quick_sort1'))
print(timeit.timeit('test_quick_sort2()', number=10, setup='from __main__ import test_quick_sort2'))
print(timeit.timeit('test_quick_sort3()', number=10, setup='from __main__ import test_quick_sort3'))
```

    1.1784412029664963
    0.7004980799974874
    0.47143277898430824


可以看出来，效果非常明显，现在元素的取值范围为0~100之间，一共有10000个元素，那随机产生的数组里面，相等的元素会非常多，这个时候，双向扫描的速度大大提高。而基于双向扫描的基础上加上插入排序的实现，速度更快！
