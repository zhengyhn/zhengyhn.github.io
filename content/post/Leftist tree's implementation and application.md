---
{
  "date": "2018-10-05",
  "slug": "Leftist tree's implementation and application",
  "subtitle": "Generic subtitle",
  "title": "Leftist tree's implementation and application"
}
---
<!--more-->

不小心看到一种数据结构，叫左偏堆，又叫左偏树，据说跟二叉堆一样，可以实现优先队列的功能，而且比二叉堆实现起来简单，于是研究了一下。

## 需求

优先队列的应用是非常广泛的，工作中还没遇到过这种需求，所以我现在制造一个需求，假设现在在开发一个项目管理系统，我们每天都要用的那种，每个任务都有优先级，我们肯定是先做优先级高的任务，再做优先级低的任务，由于插需求的事情经常发生，当一个需求插入时，我们要实现一个功能：优先级最高的排在看板的最顶部，当任务移走时，下一个优先级最高的排在最前面。

## 实现

看板的一列，我们可以看做是一个列表，每个元素，我给它定义成一个类Card，它有2个成员: title和priority，为了遵循习惯，priority越小的优先级越高, 优先级取值范围是1~5。

定义如下：


```python
class Card:
    def __init__(self, title, priority):
        self.title = title
        self.priority = priority
    
    def __gt__(self, b):
        return self.priority > b.priority
    
    def __str__(self):
        return 'card: ' + self.title + ', priority: ' + str(self.priority)
    
    def __repl__(self):
        return 'card: ' + self.title + ', priority: ' + str(self.priority)
```

要实现这个需求，很明显就是使用优先队列，每次插入一个元素或者删除最顶元素时，就重新调整，保证优先级最小的元素在最前面，现在用左倾堆来实现。由于没有要求按优先级排序，所以不需要做堆排序。

先说一下左倾堆的性质：

- 它是一种堆，也就是说，如果是小根堆，每个节点的值比2个孩子的值要小，如果是大根堆，就要大
- 它是左倾的，意思是说，永远保持左子树的深度要比右子树的要大，所以它是不平衡的树
- 建堆需要$O(n)$，插入和删掉都只需要$O(logn)$
- 在最坏情况下，它会退化成一个链表，每个节点都只有左子树，就是输入元素是逆序的情况下
- 只适合删除堆顶元素的情况，不能删除任意一个元素，这种操作也叫Extract-Min

左倾堆实现起来非常简单，原因是，插入和删除操作，都是使用合并操作来实现的。看代码：


```python
import random
from collections import deque

class LeftistNode:
    def __init__(self, val):
        self.val = val
        # distance是节点到右孩子的距离
        self.distance = 0
        self.left_child = None
        self.right_child = None
    def __str__(self):
        return 'node:' + str(self.val)
    
class LeftistTree:
    def __init__(self, arr):
        self.root = None
        for item in arr:
            self.insert(item)
    
    def insert(self, item):
        node = LeftistNode(item)
        self.root = self.merge(self.root, node)
        
    def delete(self):
        # 合并左右子树，相当于删除根节点
        self.root = self.merge(self.root.left_child, self.root.right_child)
        
    def merge(self, left, right):
        if left == None:
            return right
        if right == None:
            return left
        if left.val > right.val:
            left, right = right, left
        # 总是从右子树生长
        left.right_child = self.merge(left.right_child, right)
        # 总是保持左子树深度最大
        if left.left_child == None or left.left_child.distance < left.right_child.distance:
            left.left_child, left.right_child = left.right_child, left.left_child
        # 更新根节点到右子树的距离
        if left.right_child == None:
            left.distance = 0
        else:
            left.distance = left.right_child.distance + 1
        return left
    
    def show(self):
        queue = deque([self.root])
        while len(queue) > 0:
            node = queue.popleft()
            print(node.val)
            if node.right_child != None:
                queue.append(node.right_child)
            if node.left_child != None:
                queue.append(node.left_child)
        print('')
```

现在来生成一些数据，用于测试:


```python
arr = [
    Card('开发注册功能', random.randint(2, 5)),
    Card('开发登录功能', random.randint(2, 5)),
    Card('优化查看详情功能', random.randint(2, 5)),
    Card('重构下载文件功能', random.randint(2, 5)),
    Card('重构项目结构', random.randint(2, 5)),
    Card('开发列表查询功能', random.randint(2, 5)),
    Card('开发用户管理功能', random.randint(2, 5)),
]
for item in arr:
    print(item)
print('')
tree = LeftistTree(arr)
tree.show()

tree.insert(Card('紧急处理用户不能登录的bug', 1))
tree.show()

tree.delete()
tree.show()

tree.delete()
tree.show()
```

    card: 开发注册功能, priority: 3
    card: 开发登录功能, priority: 3
    card: 优化查看详情功能, priority: 2
    card: 重构下载文件功能, priority: 4
    card: 重构项目结构, priority: 5
    card: 开发列表查询功能, priority: 5
    card: 开发用户管理功能, priority: 2
    
    card: 优化查看详情功能, priority: 2
    card: 开发用户管理功能, priority: 2
    card: 重构下载文件功能, priority: 4
    card: 开发注册功能, priority: 3
    card: 开发列表查询功能, priority: 5
    card: 重构项目结构, priority: 5
    card: 开发登录功能, priority: 3
    
    card: 紧急处理用户不能登录的bug, priority: 1
    card: 优化查看详情功能, priority: 2
    card: 开发用户管理功能, priority: 2
    card: 重构下载文件功能, priority: 4
    card: 开发注册功能, priority: 3
    card: 开发列表查询功能, priority: 5
    card: 重构项目结构, priority: 5
    card: 开发登录功能, priority: 3
    
    card: 优化查看详情功能, priority: 2
    card: 开发用户管理功能, priority: 2
    card: 重构下载文件功能, priority: 4
    card: 开发注册功能, priority: 3
    card: 开发列表查询功能, priority: 5
    card: 重构项目结构, priority: 5
    card: 开发登录功能, priority: 3
    
    card: 开发用户管理功能, priority: 2
    card: 开发注册功能, priority: 3
    card: 重构下载文件功能, priority: 4
    card: 开发登录功能, priority: 3
    card: 开发列表查询功能, priority: 5
    card: 重构项目结构, priority: 5
    


可以看出来，建堆之后，优先级最高的是为2的任务，这个时候插入了一个bug，要紧急处理，优先级为1，堆顶元素就变成这个bug了，处理完这个bug之后，移走，相当于删除掉，优先级最高的又变成优先级2的任务了，再移走，又是下一个，说明我们已经实现了优先队列，并且满足了这个需求，更有趣的是，这个左倾树的实现居然这么简单！！

## 参考资料

- [Leftist Tree](https://en.wikipedia.org/wiki/Leftist_tree)
