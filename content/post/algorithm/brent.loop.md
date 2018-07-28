---
date: 2018-01-06
title: Brent判圈算法学习
tags: ["算法"]
categories: ["算法"]
---

判断一个链接有没有环，很著名的算法是Floyd判圈算法，也叫龟兔算法。但是，原来还有一种算法，可以比Floyd更快一点，这种算法叫做Brent判圈算法。

#### 算法思想
用2个指针rabbit和turtle从链表头出发。

- rabbit先一步一步走，最多走2步，如果走到尽头，则无环，如果和turtle相遇，则有环，否则，本轮结束。
- 这个时候，把turtle放到rabbit当前位置，rabbit继续一步一步走，但是最多走4步，如果走到尽头，则无环，如果和turtle相遇，则有环，否则，本轮结束。
- 然后，把turtle放到rabbit当前位置，rabbit继续一步一步走，但是最多走8步，如果走到尽头，则无环，如果和turtle相遇，则有环，否则，本轮结束。
- ...

跟Floyd算法相比，假设一个5个元素的圆圈链表（尾部元素指回头部），Floyd算法需要走3 * 5 = 15步，而Brent算法只需要走2 + 4 + 5 + 2 = 13步。

#### 代码实现
```
    bool hasCycleBrent(ListNode *head) {
      ListNode *p1 = head;
      ListNode *p2 = head;
      int steps = 0;
      int limit = 2;
      while (p1 != NULL && p2 != NULL) {
        p1 = p1->next;
        if (p1 == p2) {
          return true;
        }
        ++steps;
        if (steps == limit) {
          p2 = p1;
          steps = 0;
          limit *= 2;
        }
      }
      return false;
    }

```
#### 参考
- [brent_x0091_s-cycle-detection](https://iam.tugraz.at/dform/dform_w10/wp-content/uploads/2010/11/brent_x0091_s-cycle-detection1.pdf)
