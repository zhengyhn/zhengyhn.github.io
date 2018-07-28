+++
date = "2016-05-14T16:35:19+08:00"
draft = false
title = "Implement the sorting algorithms"
+++

The sorting algorithms are very important in programming interview. I have
to be able to write them on a paper without even an error.

Bubble Sort
=================
The immediate thought of bubble sort is that swapping the ajacent elements
if they are of the wrong order in each pass until there are no swapping.
For example, consider the array ``[5, 1, 4, 2, 8]``,

First pass:

[5, 1, 4, 2, 8] => [1, 5, 4, 2, 8] => [1, 4, 5, 2, 8] => [1, 4, 2, 5, 8]
=> [1, 4, 2, 5, 8]

Second pass:

[1, 2, 4, 5, 8] => [1, 2, 4, 5, 8] => [1, 2, 4, 5, 8] => [1, 2, 4, 5, 8]
=> [1, 2, 4, 5, 8]

Third pass:

[1, 2, 4, 5, 8] => [1, 2, 4, 5, 8] => [1, 2, 4, 5, 8] => [1, 2, 4, 5, 8]
=> [1, 2, 4, 5, 8]

Here we found that the array has been sorted after the second pass, but we
have apply the third pass so that we can know the array has been sorted
because there are no swapping in that pass.

There is an optimization that can improve the performance. Observer that
in the second pass, there is no need to compare ``5`` to ``8``, since they
are in the right order in the first pass. Hence, there is no need to compare
the last ``ith`` element in the ``ith`` pass.

[Here](https://github.com/zhengyhn/hugo-blog-code/tree/master/sort/bubble-sort) is the code.

For this test cases, the average run time is **0.9s**.

Select Sort
==================
Just like its name, select sort is to select the smallest or the biggest
element of the rest elements every pass. This algorithm is very easy and
I can write down it with an eyes on.

[Here](https://github.com/zhengyhn/hugo-blog-code/tree/master/sort/select-sort) is the code.

For this test cases, the average run time is **0.4s**.

Insert Sort
=================
Just like playing poker, insert-sort will insert an element in a position
of a sorted subsequence by moving the not-match elements backward. In the
worst case, the time complexity is O(n^2). However, if the list is already
sorted, it requires only O(n) time complexity.

[Here](https://github.com/zhengyhn/hugo-blog-code/tree/master/sort/insert-sort) is the code.

For this test cases, the average run time is also **0.22s**.

Shell Sort
=================
Shell sort is one kind of insert sort. The idea that relatively sorted list can
be sorted more easy is used in shell sort. It will sort the list for several
times and each time with some of the elements. These elements is divided by
a fixed **gap** and the gap will decrease until becoming 1. If the gap is 1,
it's the same as the insert sort. But now it will be more fast than the original
insert sort.

[Here](https://github.com/zhengyhn/hugo-blog-code/tree/master/sort/shell-sort) is the code.

For this test cases, the average run time is also **0.18s**.

Quick Sort
=================
This is the most important algorithm in the interview. I must learn to write
it on a paper with an eye on.

Recursive version
---------------------
Select an element as the one to be compared(so-called **pivot**), and then put
it at the end of the list. Iterate the whole list and let those elements that
are smaller(larger) than the pivot in a part, the others in another part. Do
the same work recursively in the two parts.

[Here](https://github.com/zhengyhn/hugo-blog-code/tree/master/sort/quick-sort) is the code.

For this test cases, the average run time is **0.07s**.

Iterate version
---------------------
The recursive version is just the depth-first search, so we can change it
into stack operation.

[Here](https://github.com/zhengyhn/hugo-blog-code/tree/master/sort/quick-sort) is the code.

For this test cases, the average run time is also **0.07s**.

Merge Sort
==================
Merge sort is in terms of so-called **Divided and Conquered** method. Dividing
each sequence into two subsequences and merge them after they are sorted. Merge
sort is generally faster than Quick sort(As my test cases show) but is slower
than Heap sort.

I implemented three versions of merge sort.

Merge using a temporary array
---------------------------------
In the merge process, I used a buffer to store the sorted array from the two
sorted subarrays and then copy the elements to the original array.

[Here](https://github.com/zhengyhn/hugo-blog-code/tree/master/sort/merge-sort) is the code.

For this test cases, the average run time is **0.042s**.

Merge using Insert sort
---------------------------------
In the merge process, we can using insert sort to merge the two sorted arrays.
Since the two arrays is sorted, it's fast to merge them.

Note this method is [in-place](http://en.wikipedia.org/wiki/In-place_algorithm).

[Here](https://github.com/zhengyhn/hugo-blog-code/tree/master/sort/merge-sort) is the code.

For this test cases, the average run time is also **0.045s**.

Merge by exchanging memory
--------------------------------
How to do it? Suppose we have two subarrays, say ``a`` and ``b`` and ``a``
start from ``begin``, ``b`` start from ``mid``.

1. iterate ``a`` form ``begin`` to ``i`` until ``a[i]`` larger than ``b[mid]``.

2. iterate ``b`` from ``mid`` to ``j`` until ``b[j]`` larger than ``a[i]``.

3. exchange the sub-block ``a[i..mid]`` with ``b[mid..j]``.

4. now replace ``begin`` with the start point of the rest sub-array and repeate
   step 1, 2, 3.

For more information, please visit [this](http://www.cppblog.com/converse/archive/2013/08/08/63008.html).

Note this method is also [in-place](http://en.wikipedia.org/wiki/In-place_algorithm).

[Here](https://github.com/zhengyhn/hugo-blog-code/tree/master/sort/merge-sort) is the code.

For this test cases, the average run time is also **0.045s**.

Heap Sort
=================
A heap is very useful in sorting algorithm or in algorithms that find the top
nth elements. I use a binary tree to build the tree and the binary tree is built
from an array instead of a linked list. This makes me write the heap sort very
easy.

Heap sort is faster than quick sort in many cases, expecially in large data set.

[Here](https://github.com/zhengyhn/hugo-blog-code/tree/master/sort/heap-sort) is the code.

For this test cases, the average run time is also **0.022s**.

Summary
=================
Here is the running time of these sorting algorithm with the same test cases.

<table>
  <th><tr>
    <td>Algorithm</td><td>Best time complexity</td>
    <td>Average time complexity</td><td>Worst time complexity</td>
    <td>Running time</td>
  </tr></th>
  <tr>
    <td>Bubble Sort</td><td>O(n)</td><td>O(n^2)</td><td>O(n^2)</td>
    <td>0.9s</td>
  </tr>
  <tr>
    <td>Select Sort</td><td>O(n^2)</td><td>O(n^2)</td><td>O(n^2)</td>
    <td>0.4s</td>
  </tr>
  <tr>
    <td>Insert Sort</td><td>O(n)</td><td>O(n^2)</td><td>O(n^2)</td>
    <td>0.22s</td>
  </tr>
  <tr>
    <td>Shell Sort</td><td>O(n)</td><td>O(n^2)</td><td>O(n^2)</td>
    <td>0.18s</td>
  </tr>
  <tr>
    <td>Quick Sort</td><td>O(nlog(n))</td><td>O(nlog(n))</td><td>O(n^2)</td>
    <td>0.07s</td>
  </tr>
  <tr>
    <td>Merge Sort</td><td>O(nlog(n))</td><td>O(nlog(n))</td><td>O(nlog(n))</td>
    <td>0.042s</td>
  </tr>
  <tr>
    <td>Heap Sort</td><td>O(nlog(n))</td><td>O(nlog(n))</td><td>O(nlog(n))</td>
    <td>0.022s</td>
  </tr>
</table>
