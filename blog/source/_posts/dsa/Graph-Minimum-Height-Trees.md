---
title: Minimum Height Trees (MHTs)
date: 2021-12-16 16:54:00
author: Aaron
top: false
cover: true 
toc: true
summary: Detailed explanation of Minimum Height Trees
categories: 
  - Graph
tags: 
  - Graph
  - Minimum Height Trees
  - DSA Tutorial Graph
---

# Minimum Height Trees (MHTs)

從樹中找到一個或若干根結點，使**所有子樹的最大深度最小**。

## 解說

跟尋找樹重心不同，最大深度問題不能採用dfs的原因在於：即使能判定其母節點方向的節點數有多少，依然無法確定母節點方向的最大深度。

此時可以換個方式思考，假設我將每個**葉節點**(`只有一個edge接觸的點`)都從樹中移除。此時若樹中還有未被選到的點，代表這輪挑選到的葉節點必然不會是根，因此我們重新在當前的樹內重新挑選一次葉節點。

> 實作方法相似拓樸排序，但**注意葉節點的degree是1而不是0**

## 程式碼

```python
from collections import defaultdict, Counter, deque
    
def find_mht_roots(n, edges):
    """find all roots of MHTs
    - n: n nodes labelled from 0 to n - 1
    - edges: list of bidirectional edge where edges[i] = [a_i, b_i]
    """
    # 
    if n == 1:
        return [0]
    adj = collections.defaultdict(list)
    degrees = collections.Counter()
    for x, y in edges:
        degrees[x] += 1
        degrees[y] += 1
        adj[x].append(y)
        adj[y].append(x)

    queue = collections.deque()
    for x in degrees:
        if degrees[x] == 1:
            queue.append(x)

    res = list()
    while queue:
        res = list(queue)
        queue.clear()
        for x in res:
            for y in adj[x]:
                degrees[y] -= 1
                if degrees[y] == 1: #等於1就是找到葉節點
                    queue.append(y)
    return res
```

## 複雜度
- 時間複雜度：`O(V + E)`
- 空間複雜度：`O(V + E)`

