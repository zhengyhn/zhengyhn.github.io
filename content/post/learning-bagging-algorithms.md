---
{
  "date": "2018-11-18",
  "slug": "learning-bagging-algorithms",
  "subtitle": "Generic subtitle",
  "title": "Learning Bagging Algorithms"
}
---
<!--more-->


```python
import random

def max_value_recursive(weights, values, max_weight, i):
    if len(weights) == i:
        return 0
    if max_weight - weights[i] <= 0:
        return 0
    value_not_pick = max_value_recursive(weights, values, max_weight, i + 1)
    value_pick = max_value_recursive(weights, values, max_weight - weights[i], i + 1)
    return max(value_not_pick, values[i] + value_pick)
    
def max_value(weights, values, max_weight):
    return max_value_recursive(weights, values, max_weight, 0)

def max_value_dp(weights, values, max_weight):
    dp = [[0 for i in range(max_weight)] for i in range(len(weights))]
    i = len(weights) - 1
    for j in range(weights[i], max_weight):
        dp[i][j] = values[i]
    for i in range(len(weights) - 2, -1, -1):
        for j in range(weights[i], max_weight):
            dp[i][j] = max(dp[i + 1][j], dp[i + 1][j - weights[i]] + values[i])
    return dp[0][max_weight - 1]

N = 25
W = 50
weights = [random.randint(0, 9) for i in range(N)]
values = [random.randint(0, 9) for i in range(N)]
print(weights)
print(values)
print(max_value(weights, values, W))
print(max_value_dp(weights, values, W))
```

    [3, 0, 2, 6, 5, 5, 8, 1, 5, 4, 8, 8, 5, 5, 7, 4, 8, 8, 1, 3, 0, 3, 3, 6, 8]
    [3, 5, 5, 0, 6, 5, 9, 4, 2, 9, 6, 1, 1, 7, 9, 5, 4, 7, 9, 9, 4, 1, 2, 0, 2]
    88
    88

