---
{
  "title": "Learning Select Algorithms",
  "subtitle": "Generic subtitle",
  "date": "2018-10-03",
  "slug": "Learning Select Algorithms"
}
---
<!--more-->

最近在看《编程珠玑》，有一道题我很感兴趣。

编写程序，在O(n)时间内从数组 x\[0.. n-1\]中找出第k个最小的元素。

我第一反应，就是利用快排的思想，不断地缩小查找空间，平均可以实现O(n)的时间复杂度，但是想了一下，最坏的情况下(倒序)，需要$O(n^2)$的时间复杂度。难道还有更快的方法吗？去看了一下答案，发现答案就是我想到的这种方法。出于好奇，我就去网上搜了一下，原来这个问题很多人都研究过，并且有好几种方法，而我想到的这个方法，就是快排的作者发明的，叫做quick select，而这种找出第k个最小元素的算法，也叫select算法，利用这些高效的算法来找出中位数，将是初中高中学数学的时候无法想象的，原来可以这样做！

## Quick Select算法

利用快排的思想，随机取一个参照元素，将所有比它小的元素放在左边，比它大的元素放在右边，这个时候看参照元素的下标，如果比k要大，说明我们要找的第k个元素在左边，否则就在右边。这个时候，如果运气好，我们就缩小了很大的搜索范围了，再继续以这种方法找下去，直到找到这个元素为止。

看代码：


```python
import random

def quick_select(arr, k):
    left = 0
    right = len(arr) - 1
    while True:
        if left >= right:
            return arr[left]
        pivot_index = random.randint(left, right)
        arr[right], arr[pivot_index] = arr[pivot_index], arr[right]
        store_index = left
        for i in range(left, right):
            if arr[i] < arr[right]:
                arr[store_index], arr[i] = arr[i], arr[store_index]
                store_index += 1
        arr[right], arr[store_index] = arr[store_index], arr[right]
        if k - 1 == store_index:
            return arr[store_index]
        elif k - 1 < store_index:
            right = store_index - 1
        else:
            left = store_index + 1
    
arr = [random.randint(0, 9) for i in range(0, 10)]
print(arr)
print(quick_select(arr, 3))
```

    [2, 7, 7, 4, 4, 3, 2, 6, 9, 6]
    3


这里，随机生成了10个数字来做测试，看了一下，程序是正确的。

容易得出，平均时间复杂度是$O(n)$，最坏情况下时间复杂度是$O(n^2)$。

## Heap Select算法

还有一种想法是，利用堆排序的思想，先用前k个元素构建一个大根堆，这个时候，堆顶元素是前k个元素里面最大的。然后，对于剩下的每个元素，如果比堆顶元素要小，就将它与堆顶元素交换，重新调整堆，调整后的前k个元素是目前最小的k个元素，一直这样操作到最后，前k个元素就是所有元素中最小的k个元素，由于堆顶的元素是最大的元素，所以堆顶的元素就是第k个最小元素。

看代码：


```python
import random

def heap_select(arr, k):
    build_heap(arr, k)
    
    for i in range(k, len(arr)):
        if arr[i] < arr[0]:
            arr[0], arr[i] = arr[i], arr[0]
            ajust_heap(arr, 0, k - 1)
    return arr[0]

def build_heap(arr, k):
    start = (k - 2) // 2
    while start >= 0:
        ajust_heap(arr, start, k - 1)
        start -= 1

def ajust_heap(arr, root, tail):
    while root * 2 + 1 <= tail:
        left_child = root * 2 + 1
        to_swap = root
        if left_child <= tail and arr[to_swap] < arr[left_child]:
            to_swap = left_child
        right_child = left_child + 1
        if right_child <= tail and arr[to_swap] < arr[right_child]:
            to_swap = right_child
        if to_swap != root:
            arr[to_swap], arr[root] = arr[root], arr[to_swap]
            root = to_swap
        else:
            return

arr = [random.randint(0, 9) for i in range(0, 10)]
print(arr)
print(heap_select(arr, 3))
```

    [8, 6, 4, 9, 1, 1, 8, 4, 4, 7]
    4


从这里可以看出来，堆这种数据结构是非常非常重要的，难怪面试的时候经常问这个。

容易得出，时间复杂度是$O(nlogk)$


## BFPRT算法

还有一种没见过的算法，据说时间复杂度可以达到$O(n)$，是由5个人想出来的，所以就BFPRT算法，另外一个名字叫median of medians 。

它的思想大概是这样的，将整个列表分为5个一组，对于每组，用插入排序或其他的方法快速求出来中位数，再将这些中位数作为一个列表再求出它们的中位数，最后找到一个参照元素，这个时候，跟quick select的思想一样，将比这个参照元素小的放在它左边，大的放在它右边，一样缩小查找范围，直到找出为止。这种思想的核心是，通过中位数的中位数来找到参照元素，不会像quick select那样，运气不好的时候会退化到$O(n^2)$，而是最坏情况下也保持$O(n)$的时间复杂度。

看代码：


```python
import random

def bfprt_select(arr, k):
    idx = select(arr, 0, len(arr) - 1, k)
    return arr[idx]

def select(arr, left, right, k):
    while True:
        if left >= right:
            return left
        pivot_index = get_pivot(arr, left, right)
        arr[right], arr[pivot_index] = arr[pivot_index], arr[right]
        store_index = left
        for i in range(left, right):
            if arr[i] < arr[right]:
                arr[i], arr[store_index] = arr[store_index], arr[i]
                store_index += 1
        arr[right], arr[store_index] = arr[store_index], arr[right]
        if k - 1 == store_index:
            return store_index
        elif k - 1 > store_index:
            left = store_index + 1
        else:
            right = store_index - 1
        
def get_pivot(arr, left, right):
    if right - left < 5:
        return select5(arr, left, right)
    for i in range(left, right - 4, 5):
        sub_right = i + 4
        if sub_right > right:
            sub_right = right
        median_index = select5(arr, i, sub_right)
        arr[median_index], arr[left + (i - left) // 5] = arr[left + (i - left) // 5], arr[median_index]
    return select(arr, left, left + (right - left) // 5, (right - left) // 10 + 1)
    
def select5(arr, left, right):
    for i in range(left + 1, right + 1):
        temp = arr[i]
        j = i - 1
        while j >= left and arr[j] > temp:
            arr[j + 1] = arr[j]
            j -= 1
        arr[j + 1] = temp
    return left + (right - left) // 2
        
arr = [random.randint(0, 9) for i in range(0, 10)]
print(arr)
print(bfprt_select(arr, 9))
```

    [2, 3, 8, 7, 5, 2, 5, 7, 9, 6]
    8


代码比较多，随机生成一些数据，看起来是正确的。

为了验证程序没有问题，用3个算法都查找一次，如果3个结果都对，那程序不正确的概率接近于0了。测试一下：


```python
for i in range(0, 1000):
    arr = [random.randint(0, 1000) for i in range(0, 1000)]
    k = random.randint(1, 999)
    quick_ret = quick_select(arr[:], k)
    heap_ret = heap_select(arr[:], k)
    bfprt_ret = bfprt_select(arr[:], k)
    if quick_ret != heap_ret or heap_ret != bfprt_ret:
        print(arr, k, quick_ret, heap_ret, bfprt_ret)
```

没有任何输出，说明这3个程序99.99%的概率是对的。现在来看一下对3个算法做性能测试：



```python
import timeit

N = 10000
def test_quick_select():
    arr = [random.randint(0, N) for i in range(0, N)]
    k = random.randint(1, N - 1)
    quick_select(arr, k)

def test_heap_select():
    arr = [random.randint(0, N) for i in range(0, N)]
    k = random.randint(1, N - 1)
    heap_select(arr, k)
    
def test_bfprt_select():
    arr = [random.randint(0, N) for i in range(0, N)]
    k = random.randint(1, N - 1)
    bfprt_select(arr, k)
    

print(timeit.timeit('test_quick_select()', number=10, setup='from __main__ import test_quick_select'))
print(timeit.timeit('test_heap_select()', number=10, setup='from __main__ import test_heap_select'))
print(timeit.timeit('test_bfprt_select()', number=10, setup='from __main__ import test_bfprt_select'))
```

    0.26961762097198516
    0.35226804204285145
    0.5465565109625459


看到这个结果，我惊呆了，bfprt算法居然比quick select慢！而且比heap select也慢！太不可思议了！说好的$O(n)$呢？这真是打脸了！

查了一些资料，的确是，在现实的数据中，quick select是最快的，很多时候，我们运气都没那么差，选到的参照元素都能较快缩小搜索范围。而一般编程语言的标准库实现，会使用quick select和bfprt算法混合的方式，叫做[inroselect](https://en.wikipedia.org/wiki/Introselect)算法，C++的[std::nth_element](https://en.cppreference.com/w/cpp/algorithm/nth_element)就是这样实现的。

总结一下，快排的思想非常有用，堆这种数据结构非常有用，实现方式也很优雅，而学习并实现了bfprt算法，对我个人算法能力的提高帮助很大！

## 参考链接
- [std::nth_element](https://en.cppreference.com/w/cpp/algorithm/nth_element)
- [inroselect](https://en.wikipedia.org/wiki/Introselect)
- [Median of Medians](https://en.wikipedia.org/wiki/Median_of_medians)
- [Quickselect](https://en.wikipedia.org/wiki/Quickselect)
- [Select算法](https://www.cnblogs.com/whensean/p/selection.html)
