---
{
  "title": "Different Implementation of Binary Tree Traversal",
  "subtitle": "Generic subtitle",
  "date": "2018-10-07",
  "slug": "different-implementation-of-binary-tree-traversal"
}
---
<!--more-->


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
                else:
                    cur = cur.right
            if pre.left == None:
                pre.left = TreeNode(item)
            else:
                pre.right = TreeNode(item)
```

```python
def preorder_recursive(root, ret):
    if root == None:
        return
    ret.append(root.val)
    preorder_recursive(root->left, ret)
    preorder_recursive(root->right, ret)
    
def inorder_recursive(root, ret):
    if root == None:
        return
    preorder_recursive(root->left, ret)
    ret.append(root.val)
    preorder_recursive(root->right, ret)
 
def postorder_recursive(root, ret):
    if root == None:
        return
    preorder_recursive(root->left, ret)
    preorder_recursive(root->right, ret)
    ret.append(root.val)
```

      File "<ipython-input-2-7bde3e736ad3>", line 5
        preorder_recursive(root->left, ret)
                                ^
    SyntaxError: invalid syntax




```python
import random

arr = [random.randint(0, 9) for i in range(0, 10)]
tree = BinaryTree(arr)
ret = []
preorder_recursive(tree.root, ret)
print(ret)

ret = []
inorder_recursive(tree.root, ret)
print(ret)

ret = []
postorder_recursive(tree.root, ret)
print(ret)
```

