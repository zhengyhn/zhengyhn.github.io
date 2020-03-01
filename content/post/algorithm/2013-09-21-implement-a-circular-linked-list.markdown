+++
date = "2016-05-14T16:35:19+08:00"
draft = false
title = "Implement a circular linked list"
+++

A circular linked list is the same as a singular linked list except that
there last node points to the first node.

Here is the specification of my CiLinkedList.

* ``CiLinkedList()``, Create an empty CiLinkedList.
* ``CiLinkedList(const CiLinkedList& list)``, Create a CiLinkedList from another
CiLinkedList ``list``.
* ``CiLinkedList(const T& value)``, Create a CiLinkedList whose first element
is ``value``.
* ``CiLinkedList(const T values[], size_t size)``, Create a CiLinkedList from
an array ``values`` of size ``size``.
* ``const CiLinkedList& operator=(const CiLinkedList& list)``, Support
CiLinkedList assignment.
* ``~CiLinkedList()``, Free all the space requested by the CiLinkedList.
* ``std::ostream& operator<<(std::ostream&, const CiLinkedList<T>&);``,
Output the CiLinkedList in the form **[a, b, c ...]**.
* ``const T& operator[](size_t index)``, Access the element of the CiLinkedList,
but it's not random accessible.
* ``size_t size()``, Return the size of the CiLinkedList.
* ``rm_dup()``, Remove duplicate elements int the CiLinkedList.

[Here](https://github.com/zhengyhn/hugo-blog-code/tree/master/circular-list) is the code.
