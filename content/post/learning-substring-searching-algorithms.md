---
{
  "date": "2018-10-06",
  "slug": "learning-substring-searching-algorithms",
  "subtitle": "Generic subtitle",
  "title": "Learning SubString Searching Algorithms"
}
---
<!--more-->


```python
def substr_kmp(string, pattern):
    if len(string) == 0 or len(pattern) == 0:
        return -1
    next_table = [-1 for i in range(0, len(pattern))]
    
    i = 0
    j = -1
    while i < len(pattern) - 1:
        if j == -1 or pattern[i] == pattern[j]:
            i += 1
            j += 1
            next_table[i] = j
        else:
            j = next_table[j]
    print(next_table)
    i = 0
    j = 0
    while i < len(string) and j < len(pattern):
        if j == - 1 or string[i] == pattern[j]:
            i += 1
            j += 1
        else:
            j = next_table[j]
    
    print(i, j)
    if j == len(pattern):
        return i - j
    else:
        return -1
    
string = 'ababaababab'
pattern = 'ababab'
print(substr_kmp(string, pattern))
```

    [-1, 0, 0, 1, 2, 3]
    11 6
    5

