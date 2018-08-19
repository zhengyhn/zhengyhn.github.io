---
{
  "date": "2018-08-18",
  "slug": "Learning KD Tree",
  "subtitle": "Generic subtitle",
  "title": "Learning KD Tree"
}
---
<!--more-->

最近在学习KD树，研究了一天，终于是搞懂了！把自己实现的代码记录下来。

KNN的算法思想非常简单，但是暴力的计算距离，计算量非常大，而KD树这种数据结构的使用，可以将KNN的时间复杂度从$O(KN)$降低到$O(KlogN)$，这也是我非常感兴趣的一点。

学习的过程中，发现还有一种叫ball树，是为了解决KD树在高维时计算慢的问题，学无止境啊，这个就得后面慢慢研究了。

原理就不记录了，可以参考[这个链接](https://www.joinquant.com/post/2627), 下面是我实现的代码:


```python
import numpy as np
import timeit
import matplotlib.pyplot as plt
%matplotlib notebook


class Node:
    def __init__(self, val=None, left=None, right=None, split_dim=None):
        # 节点值
        self.val = val
        # 左孩子
        self.left = left
        # 右孩子
        self.right = right
        # 切分维度
        self.split_dim = split_dim


class KDTree:
    def __init__(self, data_set):
        self.root = self._create_node(data_set)

    def _create_node(self, data_set=None):
        # 递归停止条件
        if len(data_set) == 0:
            return None
        # 目前数据集选取切分的维度，使用方差最大的维度做为切分维度
        split_dim = self._get_split_dim(data_set)

        # 在切分的维度上排序，选取中间的点进行切分
        data_set.sort(key=lambda x: x[split_dim])
        mid = len(data_set) // 2
        val = data_set[mid]
        # 递归创建左子树和右子树
        left = self._create_node(data_set[:mid])
        right = self._create_node(data_set[mid + 1:])

        return Node(val=val, left=left, right=right, split_dim=split_dim)

    def _get_split_dim(self, data_set):
        """
        选择方差最大的维度，因为这样划分更均匀
        """
        return np.argmax(np.var(data_set, axis=0))

    def knn(self, target=None, k=3):
        # 记录k个近邻
        # (点，与目标点的距离)
        nearests = [(self.root.val, float("inf"))]
        stack = []
        # 向下搜索目标点，并记录搜索的路径到stack
        # 经过的点，记录到k近邻列表
        self._search_down(root=self.root, stack=stack, nearests=nearests, target=target, k=k)
        # 回溯查找
        while len(stack) > 0:
            node = stack.pop()
            dist = target[node.split_dim] - node.val[node.split_dim]
            # 如果目标点与分隔超平面的距离大于目前k个近邻点与目标点的最大距离，
            # 那这个分隔点的右子树就不需要查找了，因为右子树的距离更远
            if abs(dist) > nearests[len(nearests) - 1][1]:
                continue
            # dist > 0, 则在该切分维度上，目标点大于切分点，说明目标点在切分点的右子树上，所以应该往左子树寻找，看下有没有离目标点更近的
            if dist > 0:
                self._search_down(root=node.left, stack=stack, nearests=nearests, target=target, k=k)
            else:
                self._search_down(root=node.right, stack=stack, nearests=nearests, target=target, k=k)
        return nearests

    def _search_down(self, root=None, stack=[], nearests=[], target=None, k=0):
        node = root
        while node:
            # 记录路径
            stack.append(node)
            # 计算点与目标点的距离
            dist = self._get_dist(node.val, target)
            # 类似插入排序更新nearests
            self._update_nearests(nearests=nearests, point=node.val, dist=dist, k=k)
            # 如果目标点在切分维度上的值小于该点在该维度上的值，往左子树搜索
            if target[node.split_dim] < node.val[node.split_dim]:
                node = node.left
            else:
                node = node.right

    def _update_nearests(self, nearests=[], point=None, dist=0, k=0):
        size = len(nearests)
        tail = nearests[size - 1]
        # 如果距离比nearests里面最大的值还要大，而且nearests已经有k个数了，就没必要再加进去了
        if dist > tail[1] and size >= k:
            return
        # 如果nearests还不够k个数，就往后面加一个
        if size < k and tail[1] != float("inf"):
            nearests.append((point, float("inf")))
        for i in range(len(nearests) - 1, -1, -1):
            # 找到插入的位置
            if dist > nearests[i - 1][1] or i == 0:
                nearests[i] = (point, dist)
                break
            else:
                nearests[i] = nearests[i - 1]

    def _get_dist(self, left, right):
        # L2欧拉距离
        return np.sqrt(np.sum(np.power(np.array(left) - np.array(right), 2)))
```

现在来写一个暴力计算的KNN算法，直接遍历所有数据，计算与目标点的距离: 


```python
def l2_dist(left, right):
    return np.sqrt(np.sum(np.power(np.array(left) - np.array(right), 2)))

def update_nearests(nearests=[], point=None, dist=0, k=0):
    size = len(nearests)
    tail = nearests[size - 1]
    if dist > tail[1]:
        return
    if k > size and tail[1] != float("inf"):
        nearests.append((point, float("inf")))
    for i in range(len(nearests) - 1, -1, -1):
        if dist > nearests[i - 1][1] or i == 0:
            nearests[i] = (point, dist)
            break
        else:
            nearests[i] = nearests[i - 1]

def simple_knn(data_set=[], target=None, k=1):
    nearests = [(None, float('inf'))]
    for item in data_set:
        dist = l2_dist(target, item)
        update_nearests(nearests=nearests, point=item, dist=dist, k=k)
    return nearests
```

然后，我做了一下性能测试，看下两种算法的性能比较:


```python
def kdtree_knn():
    tree = KDTree(data_set)
    for i in range(0, 100):
        target = np.random.normal(loc=10, scale=10, size=5).tolist()
        tree.knn(target=target, k=3)

def brute_force_knn():
    for i in range(0, 100):
        target = np.random.normal(loc=10, scale=10, size=5).tolist()
        simple_knn(data_set=data_set, target=target, k=3)      

# plt.axis((0, 5, 0, 5))
# plt.xlabel('x',size=15)
# plt.ylabel('y', size=15)
# data_array = np.array(data_set)
# plt.scatter(data_array[:,0], data_array[:,1])
# plt.scatter(target[0], target[1], marker='*')

data_set = np.random.normal(loc=10, scale=10, size=200).reshape((-1, 5)).tolist()

print(timeit.repeat('brute_force_knn()', number=1, setup="from __main__ import brute_force_knn"))
print(timeit.repeat('kdtree_knn()', number=1, setup="from __main__ import kdtree_knn"))
```

    [0.06312151486054063, 0.059134894981980324, 0.056395760271698236]
    [0.0651028179563582, 0.07302056299522519, 0.06238660495728254]


可以看出来，在数据量比较小的时候，上面的代码是40个样本，5维，暴力算法还是比KD树要快的。
现在来看一下数据量比较大的时候，


```python
data_set = np.random.normal(loc=10, scale=10, size=10000).reshape((-1, 5)).tolist()

print(timeit.repeat('brute_force_knn()', number=1, setup="from __main__ import brute_force_knn"))
print(timeit.repeat('kdtree_knn()', number=1, setup="from __main__ import kdtree_knn"))
```

    [2.565840309020132, 2.955948404967785, 2.6631450951099396]
    [0.6005713860504329, 0.48381843604147434, 0.472601052839309]


当样本数量达到2000个的时候，暴力算法基本要2.5s左右才能跑完，而KD树只需要0.45s左右。

### 参考链接
- [kd 树算法之思路篇](https://www.joinquant.com/post/2627)
- [kd 树算法之详细篇](https://www.joinquant.com/post/2843)
- [一只兔子帮你理解 kNN](https://www.joinquant.com/post/2227?f=study&m=math)
- [基于kd树的KNN算法的实现](https://www.jianshu.com/p/521f00393504)
