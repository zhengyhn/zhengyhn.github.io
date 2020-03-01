+++
date = "2016-05-14T16:35:19+08:00"
draft = false
title = "Implement a singular LinkedList"
+++

LinkedList is a very common data structure. Three years ago, I could write
a singular LinkedList very quickly in C/C++. But the code is ugly and very
ineffecient. Now I am going to implement a singular LinkedList in C++ in order
to cracking the interview.

Here is the specification of my LinkedList.

* ``LinkedList()``, Create an empty LinkedList.
* ``LinkedList(const LinkedList& list)``, Create a LinkedList from another
LinkedList ``list``.
* ``LinkedList(const T& value)``, Create a LinkedList whose first element
is ``value``.
* ``LinkedList(const T values[], size_t size)``, Create a LinkedList from
an array ``values`` of size ``size``.
* ``const LinkedList& operator=(const LinkedList& list)``, Support LinkedList
assignment.
* ``~LinkedList()``, Free all the space requested by the LinkedList.
* ``std::ostream& operator<<(std::ostream&, const LinkedList<T>&);``,
Output the LinkedList in the form **[a, b, c ...]**.
* ``void append(const T& value)``, Append an element ``value`` to the LinkedList.
* ``const T& operator[](size_t index)``, Access the element of the LinkedList,
but it's not random accessible.
* ``void insert(size_t index, const T& value)``, Insert an element ``value``
in the position ``index``.
* ``void remove(size_t index)``, Remove the element in the position ``index``.
* ``size_t size()``, Return the size of the LinkedList.
* ``void reverse()``, Reverse the whole LinkedList.
* ``void sort()``, Sort the whole LinkedList using merge sort.

There are some interview problems about LinkedList.

**2.1** Write code to remove duplicates from an unsorted linked list. How would
you solve this problem if a temporary buffer is not allowed?

I solve it in the method ``rm_dup()``.

**2.2** Implement an algorithm to find the nth to last element of a singly
linked list.

I solve it in the method ``nth_last()``.


[Here](https://github.com/zhengyhn/hugo-blog-code/tree/master/singular-list) is the code.
