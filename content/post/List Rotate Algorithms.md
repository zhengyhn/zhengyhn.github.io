---
{
  "date": "2018-09-13",
  "slug": "List Rotate Algorithms",
  "subtitle": "Generic subtitle",
  "title": "List Rotate Algorithms"
}
---
<!--more-->

最近在《编程珠玑》上看到一道很有趣的题目:

假设我们在开发一个编辑器，要实现一个功能，将下面代码的2个函数调换位置，该怎么做呢？如果空间复杂度要求$O(1)$，又应该怎么做呢？


```python
texts = [
    'def add(a, b):',
    '  return a + b',
    ' ',
    'def sub(a, b):',
    '  retturn a - b',
    ' '
]
```

其实这就是数组左旋转的问题，将数组的前3个元素移到数组的结尾。我最直接的想法，就是用另外一个数组保存前3个元素，然后将后3个元素往前面移动，
最后把前面保存的3个元素放到结尾，代码也比较简洁:


```python
def left_rotate_normal(texts, rotate_len):
    copy = texts[:rotate_len]
    texts[:len(texts) - rotate_len] = texts[rotate_len:]
    texts[len(texts) - rotate_len:] = copy
    
test_texts = texts[:]
print(test_texts)
left_rotate_normal(test_texts, 3)
print(test_texts)
```

    ['def add(a, b):', '  return a + b', ' ', 'def sub(a, b):', '  retturn a - b', ' ']
    ['def sub(a, b):', '  retturn a - b', ' ', 'def add(a, b):', '  return a + b', ' ']


测试结果，是没有问题的，这个算法的最大问题是，需要额外开辟空间，如果数组很大或要移动的部分很大，就会额外占用内存。

书里面给出了两种不需要额外开辟空间的方法:
    
第一种我叫它跳跃法吧，其实是将整块元素的移动分成一个一个元素的移动，每次移动一个元素，就把后面的补充上来, 以第一个元素为例，
先用一个临时变量temp保存第一个元素$texts[0]$, 然后将$texts[i]$移动到$texts[0]$, $texts[2 * i]$移到到$texts[i]$，直到结束，最后将
temp赋值回末尾空出来的位置。代码实现如下:


```python
def left_rotate_jump(texts, rotate_len):
    i = 0
    while i < rotate_len:
        temp = texts[i]
        j = rotate_len
        k = i
        while j < len(texts):
            texts[k] = texts[j]
            k = j
            j *= 2
        texts[j // 2] = temp
        i += 1
        
test_texts = texts[:]
print(test_texts)
left_rotate_jump(test_texts, 3)
print(test_texts)
```

    ['def add(a, b):', '  return a + b', ' ', 'def sub(a, b):', '  retturn a - b', ' ']
    ['def sub(a, b):', 'def add(a, b):', '  return a + b', ' ', '  retturn a - b', ' ']


还有一种方法，也是我觉得很神奇的算法，就是通过几次反转，就实现了数组的左旋转！原理是, 我们把数组分成2部分a和b，然后有这个恒等式:
$$ ba = (a^rb^r)^r $$
其中r是反转的意思。根据这个恒等式，就可以这么实现：


```python
def left_rotate_reverse(texts, rotate_len):
    reverse(texts, 0, rotate_len - 1)
    reverse(texts, rotate_len, len(texts) - 1)
    reverse(texts, 0, len(texts) - 1)
    
def reverse(texts, start, end):
    while start < end:
        temp = texts[start]
        texts[start] = texts[end]
        texts[end] = temp
        start += 1
        end -= 1

test_texts = texts[:]
print(test_texts)
left_rotate_reverse(test_texts, 3)
print(test_texts)
```

    ['def add(a, b):', '  return a + b', ' ', 'def sub(a, b):', '  retturn a - b', ' ']
    ['def sub(a, b):', '  retturn a - b', ' ', 'def add(a, b):', '  return a + b', ' ']


非常神奇，而且很简洁！三种算法的时间复杂度都是$O(n)$，后面2种方法空间复杂度为$O(1)$，不过最后一种算法最简洁，也容易理解。
