+++
date = "2016-05-14T16:35:19+08:00"
draft = false
title = "Implement a bidirectional linked list"
+++

A bidirectional linked list is the same as a singular linked list except that
there are two links between two nodes.

Here is the specification of my BiLinkedList.

* ``BiLinkedList()``, Create an empty BiLinkedList.
* ``BiLinkedList(const BiLinkedList& list)``, Create a BiLinkedList from another
BiLinkedList ``list``.
* ``BiLinkedList(const T& value)``, Create a BiLinkedList whose first element
is ``value``.
* ``BiLinkedList(const T values[], size_t size)``, Create a BiLinkedList from
an array ``values`` of size ``size``.
* ``const BiLinkedList& operator=(const BiLinkedList& list)``, Support
BiLinkedList assignment.
* ``~BiLinkedList()``, Free all the space requested by the BiLinkedList.
* ``std::ostream& operator<<(std::ostream&, const BiLinkedList<T>&);``,
Output the BiLinkedList in the form **[a, b, c ...]**.
* ``void append(const T& value)``, Append an element ``value`` to the
BiLinkedList.
* ``const T& operator[](size_t index)``, Access the element of the BiLinkedList,
but it's not random accessible.
* ``void insert(size_t index, const T& value)``, Insert an element ``value``
in the position ``index``.
* ``void remove(size_t index)``, Remove the element in the position ``index``.
* ``size_t size()``, Return the size of the BiLinkedList.

[Here](https://github.com/zhengyhn/hugo-blog-code/tree/master/bidirectional-list) is the code.
