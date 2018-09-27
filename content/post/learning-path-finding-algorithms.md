---
{
  "title": "Learning Path Finding Algorithms",
  "subtitle": "Generic subtitle",
  "date": "2018-09-27",
  "slug": "learning-path-finding-algorithms"
}
---
<!--more-->


```python
import numpy as np
import matplotlib.pyplot as plt
%matplotlib notebook

N = 10
OBSTALE = '1'
ROAD = '*'
START = 'S'
END = 'E'
game_map = np.array([[ROAD] * 10] * 10)
game_map[1:5, 1] = OBSTALE
game_map[8:10, 1] = OBSTALE
game_map[2:6, 3] = OBSTALE
game_map[7:10, 3] = OBSTALE
game_map[1:5, 5] = OBSTALE
game_map[6:8, 5] = OBSTALE
game_map[0:3, 7] = OBSTALE
game_map[4:6, 7] = OBSTALE
game_map[0, 0] = START
game_map[8, 7] = END

for row in game_map:
    for col in row:
        print(col, end='')
    print('')
```

    S******1**
    *1***1*1**
    *1*1*1*1**
    *1*1*1****
    *1*1*1*1**
    ***1***1**
    *****1****
    ***1*1****
    *1*1***E**
    *1*1******



```python
class Node:
    def __init__(self, x, y):
        self.x = x
        self.y = y
    
    def __eq__(self, other):
        return self.x == other.x and self.y == other.y
    
    def __hash__(self):
        return N * self.x + self.y
    
    def __repr__(self):
        return '({0}, {1})'.format(self.x, self.y)
    
class Edge:
    def __init__(self, node1, node2, weight):
        self.node1 = node1
        self.node2 = node2
        self.weight = weight
    
    def either(self):
        return self.node1
    
    def other(self, node):
        if node == self.node1:
            return self.node2
        else:
            return self.node1
        
class Graph:
    def __init__(self, game_map):
        self.num_nodes = N * N
        self.adj = {}
        for i in range(0, len(game_map)):
            for j in range(0, len(game_map[i])):
                node = Node(i, j)
                self.adj[node] = []
        for i in range(0, len(game_map)):
            for j in range(0, len(game_map[i])):
                if game_map[i][j] != OBSTALE:
                    k = i - 1
                    while k >= 0 and game_map[k][j] != OBSTALE:
                        edge = Edge(Node(i, j), Node(k, j), i - k)
                        self.add_edge(edge)
                        k -= 1
                    k = j + 1
                    while k < N and game_map[i][k] != OBSTALE:
                        edge = Edge(Node(i, j), Node(i, k), k - j)
                        self.add_edge(edge)
                        k += 1
                    k = i + 1
                    while k < N and game_map[k][j] != OBSTALE:
                        edge = Edge(Node(i, j), Node(k, j), k - i)
                        self.add_edge(edge)
                        k += 1
                    k = j - 1
                    while k >= 0 and game_map[i][k] != OBSTALE:
                        edge = Edge(Node(i, j), Node(i, k), j - k)
                        self.add_edge(edge)
                        k -= 1
                    
    def add_edge(self, edge):
        node1 = edge.either()
        node2 = edge.other(node1)
        self.adj[node1].append(node2)
        self.adj[node2].append(node1)
                        
graph = Graph(game_map.tolist())
for key, value in zip(range(3), graph.adj.items()):
    print(value[0], value[1])
```

    (0, 0) [(0, 1), (0, 2), (0, 3), (0, 4), (0, 5), (0, 6), (1, 0), (2, 0), (3, 0), (4, 0), (5, 0), (6, 0), (7, 0), (8, 0), (9, 0), (0, 1), (0, 2), (0, 3), (0, 4), (0, 5), (0, 6), (1, 0), (2, 0), (3, 0), (4, 0), (5, 0), (6, 0), (7, 0), (8, 0), (9, 0)]
    (0, 1) [(0, 0), (0, 2), (0, 3), (0, 4), (0, 5), (0, 6), (0, 0), (0, 2), (0, 3), (0, 4), (0, 5), (0, 6)]
    (0, 2) [(0, 0), (0, 1), (0, 3), (0, 4), (0, 5), (0, 6), (1, 2), (2, 2), (3, 2), (4, 2), (5, 2), (6, 2), (7, 2), (8, 2), (9, 2), (0, 1), (0, 0), (0, 3), (0, 4), (0, 5), (0, 6), (1, 2), (2, 2), (3, 2), (4, 2), (5, 2), (6, 2), (7, 2), (8, 2), (9, 2)]



```python
def floy(graph, start, end):
    distance = [[float('inf') for _ in range(N * N)]] * (N * N)
    next = []
```

