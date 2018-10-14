---
{
  "title": "Different Implementation of Binary Tree Traversal",
  "subtitle": "Generic subtitle",
  "date": "2018-10-14",
  "slug": "different-implementation-of-binary-tree-traversal"
}
---
<!--more-->

最近在做leetcode时，发现了另外一种二叉树遍历的方法，于是总结回之前掌握的遍历方法，做此笔记。

## 建树

首先定义一棵树，val为元素的值，left是左节点，right是右节点。

并做最基本的建树操作，传入一个数组，构造一棵二分查找树。


```python
class TreeNode:
    def __init__(self, val):
        self.val = val
        self.left = None
        self.right = None

class BinaryTree:
    def __init__(self, arr):
        self.root = TreeNode(arr[0])
        for item in arr[1:]:
            pre = self.root
            cur = self.root
            while cur != None:
                pre = cur
                if item < cur.val:
                    cur = cur.left
                    if cur == None:
                        pre.left = TreeNode(item)
                        break
                else:
                    cur = cur.right
                    if cur == None:
                        pre.right = TreeNode(item)
                        break
```

## 递归法

最简单的树遍历方法就是递归法了，直接就是利用定义，三种遍历方法的区别就是移动一下输出节点的位置。


```python
def preorder_recursive(root, ret):
    if root == None:
        return
    ret.append(root.val)
    preorder_recursive(root.left, ret)
    preorder_recursive(root.right, ret)
    
def inorder_recursive(root, ret):
    if root == None:
        return
    inorder_recursive(root.left, ret)
    ret.append(root.val)
    inorder_recursive(root.right, ret)
 
def postorder_recursive(root, ret):
    if root == None:
        return
    postorder_recursive(root.left, ret)
    postorder_recursive(root.right, ret)
    ret.append(root.val)
```

## 非递归-栈模拟法

比较常见的非递归方法，就是用栈来模拟，这个方法还是有难度的，现在每种遍历都来做一下笔记。

### 前序遍历

这个比较简单，用一个栈来模拟，记录下一个要经过的节点，由于前序遍历是经过时就输出，所以从栈取出之后立即输出就行。这里的技巧就是，先把节点的右节点入栈，再把左节点入栈，这样下次取的时候就是左节点优先，符合遍历的规则。


```python
from collections import deque

def preorder_iterate(root):
    ret = []
    stack = deque([root])
    while len(stack) > 0:
        top = stack.pop()
        ret.append(top.val)
        if top.right != None:
            stack.append(top.right)
        if top.left != None:
            stack.append(top.left)
    return ret
```

### 中序遍历

这个难度中等，除了用一个栈来记录节点，还需要加多一个指针，就叫cur吧，指向当前遍历到的节点。这里的技巧是，每次都将最左边的节点进栈，取出栈顶元素时，就直接输出，然后判断该元素是否有右节点，如果有，再以右子树为整体，将右子树的最左边节点进栈，保证每次从栈顶元素取出的是要输出的元素。


```python
def inorder_iterate(root):
    ret = []
    stack = deque([])
    cur = root
    while cur != None or len(stack) > 0:
        # 每次都将最左边节点进栈
        while cur != None:
            stack.append(cur)
            cur = cur.left
        # 这个时点，cur一定指空指针
        top = stack.pop()
        ret.append(top.val)
        # 当栈顶元素存在右节点时，cur就指向它的右节点，进行新的一轮左节点入栈
        if top.right != None:
            cur = top.right
    return ret
```

### 后序遍历

这个很难，有很多实现，我自己也发明了一种实现，但是看了别人的实现，感觉人家的很巧妙，决定采用别人的思路。

后序遍历是先输出左右节点，再输出根节点，如果采用入栈遍历的形式，根节点需要先不出栈，然后再遍历右子树，等到返回的时候再输出。现在需要个方法来判断什么时候是第一次遍历，什么时候是返回。这里有个技巧，用另外一个指针pre记录上一次遍历过的节点，如果pre节点是当前节点的孩子节点，那说明是在返回过程。


```python
def postorder_iterate(root):
    ret = []
    stack = deque([root])
    pre = None
    while len(stack) > 0:
        top = stack[-1]
        if (top.left == None and top.right == None) or (pre != None and (pre == top.left or pre == top.right)):
            ret.append(top.val)
            pre = top
            stack.pop()
        else:
            if top.right != None:
                stack.append(top.right)
            if top.left != None:
                stack.append(top.left)
    return ret
```

## 非递归-Morris法

还有一种非递归法，可以不使用栈来模拟，不需要额外开辟空间，这就是morris法，对，就是KMP算法中间那个M。

它采用的原理是线索二叉树，通过用右节点为空的节点，将右节点连接到下一个要遍历的节点。

不过，由于这样子会修改二叉树，所以需要重复的遍历一次，将之前修改的右节点重新指向空指针，所以速度上会慢一些。

morris算法刚提出的时候是用于解决中序遍历问题的，但是修改一下，也是可以应用于前序遍历和后序遍历的。

### 前序遍历

原理是这样的，对于遍历到的当前节点cur，去它的左节点开始找最右边的节点，然后连接到当前节点。这个是连接过程。当重新回到当前节点cur时，再走同样的路，将左节点开始找到的最右节点，把之前的连接去掉，以保证二叉树没有被修改。

由于是前序遍历，所以我们在每次连接过程中，将当前节点输出就行。


```python
def preorder_morris(root):
    ret = []
    cur = root
    while cur != None:
        if cur.left == None:
            ret.append(cur.val)
            cur = cur.right
        else:
            p = cur.left
            while p.right != None and p.right != cur:
                p = p.right
            if p.right == None:
                ret.append(cur.val)
                p.right = cur
                cur = cur.left
            else:
                p.right = None
                cur = cur.right
    return ret
```

### 中序遍历

中序遍历的原理一样，只是输出节点的时机不一样，前序遍历是在建立连接的时候输出当前节点，中序遍历则是在删除连接的时候输出。


```python
def inorder_morris(root):
    ret = []
    cur = root
    while cur != None:
        if cur.left == None:
            ret.append(cur.val)
            cur = cur.right
        else:
            p = cur.left
            while p.right != None and p.right != cur:
                p = p.right
            if p.right == None:
                p.right = cur
                cur = cur.left
            else:
                p.right = None
                ret.append(cur.val)
                cur = cur.right
    return ret
```

### 后序遍历

后序遍历，网上有一些实现，但是我看了，实现复杂，而且跟用栈模拟没有两样，那样子实现后序遍历还真的不如用栈来做，而且一点也不优雅。


```python
def postorder_morris(root):
    pass
```

## 测试

现在来测试一下，生成一个随机数组，建一棵树，然后输出上面各个方法的结果。


```python
import random

arr = [random.randint(0, 100) for i in range(0, 10)]
# arr = [63, 69, 30, 8, 66, 79, 38, 5, 23, 85]
print(arr)
tree = BinaryTree(arr)
ret = []
print('preorder:')
preorder_recursive(tree.root, ret)
print(ret)
ret = preorder_iterate(tree.root)
print(ret)
ret = preorder_morris(tree.root)
print(ret)

print('inorder:')
ret = []
inorder_recursive(tree.root, ret)
print(ret)
ret = inorder_iterate(tree.root)
print(ret)
ret = inorder_morris(tree.root)
print(ret)

print('postorder:')
ret = []
postorder_recursive(tree.root, ret)
print(ret)
ret = postorder_iterate(tree.root)
print(ret)
```

    [80, 47, 97, 31, 90, 88, 54, 12, 46, 46]
    preorder:
    [80, 47, 31, 12, 46, 46, 54, 97, 90, 88]
    [80, 47, 31, 12, 46, 46, 54, 97, 90, 88]
    [80, 47, 31, 12, 46, 46, 54, 97, 90, 88]
    inorder:
    [12, 31, 46, 46, 47, 54, 80, 88, 90, 97]
    [12, 31, 46, 46, 47, 54, 80, 88, 90, 97]
    [12, 31, 46, 46, 47, 54, 80, 88, 90, 97]
    postorder:
    [12, 46, 46, 31, 54, 47, 88, 90, 97, 80]
    [12, 46, 46, 31, 54, 47, 88, 90, 97, 80]


看上面的输出，各个方法实现应该是没有问题的。

## 性能测试

来做个性能测试，看下各个算法的效率。


```python
import timeit
import random

N = 10000
arr = [random.randint(0, N) for i in range(0, N)]
tree = BinaryTree(arr)
ret = []

def test_preorder_recursive():
    preorder_recursive(tree.root, ret)
    
def test_preorder_iterate():
    ret = preorder_iterate(tree.root)
    
def test_preorder_morris():
    ret = preorder_morris(tree.root)
        
def test_inorder_recursive():
    inorder_recursive(tree.root, ret)
    
def test_inorder_iterate():
    ret = inorder_iterate(tree.root)
    
def test_inorder_morris():
    ret = inorder_morris(tree.root)
    
def test_postorder_recursive():
    postorder_recursive(tree.root, ret)
    
def test_postorder_iterate():
    ret = postorder_iterate(tree.root)
        
print('test_preorder_recursive:', 
      timeit.timeit(test_preorder_recursive, number=10, setup='from __main__ import test_preorder_recursive'))
print('test_preorder_iterate:', 
      timeit.timeit(test_preorder_iterate, number=10, setup='from __main__ import test_preorder_iterate'))
print('test_preorder_morris:', 
      timeit.timeit(test_preorder_morris, number=10, setup='from __main__ import test_preorder_morris'))
print('')
print('test_inorder_recursive:', 
      timeit.timeit(test_inorder_recursive, number=10, setup='from __main__ import test_inorder_recursive'))
print('test_inorder_iterate:', 
      timeit.timeit(test_inorder_iterate, number=10, setup='from __main__ import test_inorder_iterate'))
print('test_inorder_morris:', 
      timeit.timeit(test_inorder_morris, number=10, setup='from __main__ import test_inorder_morris'))
print('')
print('test_postorder_recursive:', 
      timeit.timeit(test_postorder_recursive, number=10, setup='from __main__ import test_postorder_recursive'))
print('test_postorder_iterate:', 
      timeit.timeit(test_postorder_iterate, number=10, setup='from __main__ import test_postorder_iterate'))
```

    test_preorder_recursive: 0.06207431899383664
    test_preorder_iterate: 0.09082036302424967
    test_preorder_morris: 0.12435238715261221
    
    test_inorder_recursive: 0.08404548815451562
    test_inorder_iterate: 0.06744503695517778
    test_inorder_morris: 0.11024954705499113
    
    test_postorder_recursive: 0.08274320606142282
    test_postorder_iterate: 0.1368340360932052


从上面看出来，morris算法是最慢的，对于morris算法，栈模拟的方法可以算是用空间来换时间，而递归算法，有时候也会比非递归算法快，哈哈。

## 参考资料

- [Morris Traversal方法遍历二叉树（非递归，不用栈，O(1)空间）](https://www.cnblogs.com/AnnieKim/archive/2013/06/15/MorrisTraversal.html)
- [二叉树的非递归遍历](https://www.cnblogs.com/dolphin0520/archive/2011/08/25/2153720.html)
- [Binary Tree Inorder Traversal](https://leetcode.com/problems/binary-tree-inorder-traversal/solution/)
