---
title: Disjoint Set Union (DSU)
date: 2021-12-15 16:00:00
author: Aaron
img: /images/dsu/dsu1.png
top: false
cover: true 
toc: true
summary: Detailed explanation of Disjoint Set Union
categories: 
  - Data Structure
tags: 
  - Graph
  - Trees
  - Disjoint Set Union
  - DSA Tutorial Graph
---


# Disjoint Set Union (DSU)

**並查集**英文名為Disjoint Set Union)又被稱為Union Find，因為兩個最主要的函式分別就是Union和Find

## Use Case

最常用來檢驗Graph中兩個節點是否連通


## Naive implementation

主要的做法就是透過parents的機制搜索樹的root，將有同樣root的節點視為在同一個連通塊中。

> 初始化會將所有節點的parent都指到結點自己
> - **Find(node)**: 沿著parent找到node的root
> - **Union(node1, node2)**: 連接包含node1的樹以及包含node2的樹
>   - 具體作法是先各自找到root1以及root2
>   - 再將兩棵樹接起來，接起來的方式就是將其中一個root的parent指到另一個

如下圖
![Disjoint Set Union](/images/dsu/dsu1.png)


## Optimization

### Path Compression

有些使用情境中，我們只想簡單使用Union以及Find，而且不在意同一棵樹除了根之外如何連通，就能考慮在Find裡面使用Path Compression。

> 由於在同一棵樹裡面，我們只在意根固定，因此在搜索根的過程中，便能順便改變樹的結構，將所有路過的節點的parent指到根，這樣能很大程度降低樹的深度

如下圖
![Disjoint Set Union](/images/dsu/dsu2.png)

> This simple modification of the operation already achieves the time complexity **`O(logn)` per call** on average

### Union by Rank / Size

Union的操作是在兩棵樹中，將其中一個樹的根結點的parent指向另一個樹的根節點。此時我們應該要考量，哪一個根結點作為兩顆樹合併後的根結點是更為合適的？

基於不同考量，一般有兩種合併樹的方式：
- <font color="LightSalmon" size=5>**Union by Rank**</font>：我們希望**最大深度**在合併之後的變化越少越好，明顯的我們將深度比較小的樹指到深度深的。這樣在最糟情況下，最大深度頂多比原本的多1而已
![Union by Rank](/images/dsu/dsu4.png)


- <font color="LightSalmon" size=5>**Union by Size**</font>：我們希望**節點數**的變化越少越好，顯然節點少的樹指到節點多的樹，會是更有利的變化
![Union by Size](/images/dsu/dsu3.png)


## Time complexity

> 若同時包含**path compression**和**union by size / rank**: `constant time` per queries.

> 若只包含**union by size / rank**: `O(n*logn)` per queries

## 程式碼

```python
class UF:
    def __init__(self, n):
        self.parents = [i for i in range(n)]
        self.rank = [1 for i in range(n)]
        self.size = [1 for i in range(n)]
        self.n = n
        
    def find(self, idx):
        # with path compression
        if idx != self.parents[idx]:
            self.parents[idx] = self.find(self.parents[idx])
        return self.parents[idx]
    
    def naive_find(self, idx):
        if idx == self.parents[idx]:
            return idx
        return self.find(self.parents[idx])
    
    def naive_union(self, idx1, idx2):
        p1 = self.find(idx1)
        p2 = self.find(idx2)
        self.parents[p2] = p1
    
    def union_by_rank(self, idx1, idx2):
        p1 = self.find(idx1)
        p2 = self.find(idx2)
        if p1 != p2:
            if self.rank[p1] < self.rank[p2]:
                p1, p2 = p2, p1
            self.parents[p2] = p1
            # if rank of p1 and p2 are the same -> max height + 1 after merge
            if self.rank[p1] == self.rank[p2]:
                self.rank[p1] += 1

    def union_by_size(self, idx1, idx2):
        p1 = self.find(idx1)
        p2 = self.find(idx2)
        if p1 != p2:
            if self.size[p1] < self.size[p2]:
                p1, p2 = p2, p1
            self.parents[p2] = p1
            self.size[p1] += self.size[p2]
```

## Reference
- [cp-algorihms: Disjoint Set Union](https://cp-algorithms.com/data_structures/disjoint_set_union.html#toc-tgt-2)
 