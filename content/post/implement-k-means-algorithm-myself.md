---
{
  "title": "Implement k-means algorithm myself",
  "subtitle": "Generic subtitle",
  "date": "2018-05-20",
  "slug": "implement-k-means-algorithm-myself"
}
---
<!--more-->

最近在看《集体智慧编程这本书》，再次学习k-means算法，虽然之前接触过很多，但是没有完全自己实现过，这次就试着根据算法思想，手写k-means算法。

这里使用的数据是书里面的博客数据，第一行是表头，第一列是博客名称，后面的列是每个单词出现的次数，大概长这样：


```python
Blog	be	not	your
Signal v. Noise - Medium	46	32	47
Eschaton	13	21	1
```

k-means算法的思想大概是这样子的：

- 先随机初始化k个点，称为中心点
- 对于数据集中的每条记录，计算它与每个中心点的距离，把它归到距离它最近的中心点的分组中
- 对于每个分组，通过均值计算中心点，得到新的中心点
- 重复这个过程，直到每条记录的分组不再变化为止

对于距离的计算，我使用皮尔逊相似度来计算，下面是距离计算的代码:


```python
def get_mean(vec):
    values = [value for value in vec]
    return sum(values) / len(values)


def distance_pearson(left, right):
    # 计算均值
    left_mean = get_mean(left)
    right_mean = get_mean(right)
    # 协方差
    covariant = sum([(left[key] - left_mean) * (right[key] - right_mean) for key in range(len(left))])
    # 标准差
    left_standard = math.sqrt(sum([math.pow(left[key] - left_mean, 2) for key in range(len(left))]))
    right_standard = math.sqrt(sum([math.pow(right[key] - right_mean, 2) for key in range(len(left))]))

    # 1 - 皮尔逊相关度，越接近0距离越近，反之越远
    return 1 - (covariant / (left_standard * right_standard))
```

下面是k-means的实现:


```python
def k_means(rows, distance=distance_pearson, k=4):
    # 共m条记录，n个特征, 也就是n个单词
    rows = np.array(rows)
    m, n = np.shape(rows)
    # 求出每个特征的最小值和最大值
    mins = rows.min(axis=0)
    maxs = rows.max(axis=0)

    # 初始化中心点
    centers = []
    clusters = [[] for i in range(k)]
    for i in range(k):
        # 随机生成的中心点，每个特征必须在最小值和最大值范围内
        center = [random.randrange(mins[i], maxs[i]) for i in range(n)]
        centers.append(center)
    # 一直进行聚类，只有分组不再变化时才停止
    change = True
    while change:
        change = False
        # 对于每条记录，都计算距离
        for i in range(m):
            closest = 0xffff
            closest_center = 0
            # 找到距离最近的中心点
            for j in range(len(centers)):
                d = distance(rows[i], centers[j])
                if d < closest:
                    closest = d
                    closest_center = j
            # 新该记录添加到该分组中，如果已经在了，就不用了
            if i not in clusters[closest_center]:
                clusters[closest_center].append(i)
                change = True
        # 通过均值重新计算分组的中心点
        for i in range(len(clusters)):
            cluster_rows = [rows[cluster] for cluster in clusters[i]]
            centers[i] = np.mean(cluster_rows, axis=0)
    return clusters
```

现在来对博客的数据进行聚类:


```python
col_names, row_names, rows = read_file('./blogdata.txt')
clusters = k_means(rows, k=4)
for i in range(len(clusters)):
    for j in clusters[i]:
        print(row_names[j])
    print('')
```

```python
看一下效果，是这样的：
```

```python
Eschaton
Guy Kawasaki
Google Blogoscoped
The Dish
The Write News
blog maverick
Boing Boing
BuzzMachine
Copyblogger
Latest from Crooks and Liars
Deadspin
kottke.org
Lifehack
TechCrunch
TMZ.com
Seth Godin's Blog on marketing, tribes and respect
Captain's Quarters
Joel on Software
Kotaku
ShoeMoney
SpikedHumor - Today's Videos and Pictures
Signal v. Noise - Medium

John Battelle's Search Blog
Creating Passionate Users
Joi Ito's Web
The Viral Garden
PaulStamatiou.com - Technology, Design and Photography
WIL WHEATON dot NET
43 Folders
456 Berea Street
Joel on Software
Kotaku
Lifehacker
Matt Cutts: Gadgets, Google, and SEO
mezzoblue
Neil Gaiman's Journal
plasticbag.org
Derek Powazek
ProBlogger
SpikedHumor - Today's Videos and Pictures
Steve Pavlina
ongoing by Tim Bray
BuzzMachine
Copyblogger
Guy Kawasaki

Google Operating System
NewsBusters
O'Reilly Radar
Seth Godin's Blog on marketing, tribes and respect
Slashdot
ThinkProgress
Engadget RSS Feed
Gothamist
PerezHilton
Quick Online Tips
Search Engine Roundtable
ShoeMoney
Techdirt.
Wired
The Write News
Schneier on Security
TechCrunch
Captain's Quarters

Signal v. Noise - Medium
Celebslam
The Official Google Blog
Mashable
Autoblog
Captain's Quarters
Schneier on Security
Google Blogoscoped
Lifehacker
Quick Online Tips
Google Operating System
Joel on Software
Search Engine Roundtable
```

看到已经成功聚成了4类，表面看，好像看不出来聚类的效果怎么样，就当作拿来练手，实现的过程中，也加深了对这个算法的理解。
