---
title: Persistent Segment Tree
date: 2021-12-13 15:30:00
author: Aaron
img: /images/persistent_seg_tree/tree1.png
top: false
cover: true 
toc: true
summary: Detailed explanation of Persistent Segment Tree
categories: 
  - Data Structure
tags: 
  - Persistent Segment Tree
  - Segment Tree
  - DSA Tutorial Persistent Data Structure
---

# Persistent Segment Tree

中文名為**可持久化線段樹**，也被稱為**主席樹**

## Persistent Data Structure

可持久化資料結構（Persistent data structure）是一種能夠在修改之後其保留歷史版本（即可以在保留原來數據的基礎上進行修改——比如增添、刪除、賦值）的資料結構。這種資料結構實際上是不可變對象，**因為相關操作不會直接修改被保存的數據，而是會在原版本上產生一個新分支**。


## Persistent Segment Tree

> 常見的使用情境像是從區間`nums[l:r]`中取k-th小的數值。

本文假設讀者已知從一個固定的線段樹中取出第k-th小的數值。若我們對nums中的每個前綴陣列都有一棵segment tree，如下圖：
```js
seg_tree(nums[:idx]), where idx = 1 ~ n
```

則對任意子陣列nums[l:r]，便能透過差分的概念組出該子陣列的segment tree，例如
```js
seg_tree(nums[l:r]) = seg_tree(nums[:r]) - seg_tree(nums[:l-1])
```

> 要找[l,r]的統計信息，只要用[0,r]減去[0,l-1]的統計信息即可

### Space Optimization

但如果我們針對每個前綴都建一棵獨立的線段樹，顯然成本過高，但我們能發現，每次在線段樹中多插入一個數值，只會有log(n)的節點被修改。

> 實際上我們每多建立一棵樹，只要新增被修改的節點就好，其餘的部分可接上原本就存在的節點。

下圖是一個統計1~6中，每個數字數量的segment tree，可看到初始的樹(黑)沒有任何數字，若依序放入數字4(藍),3(橘)和2(紫)，各自造成若干節點的改變，並形成四棵線段樹。

<img src="/images/persistent_seg_tree/tree2.png" width="75%" height="75%">

### Complexity

建立所有線段樹，假設nums有n個數值，將所有數值依照大小`離散化`，並建一個初始全為0的segment tree，此時應有`2n-1`個節點，接著依序插入離散化後的數值，每次插入有`log(n)`個節點被更新

- Space Complexity: `O(n*log(n))`
- Time Complexity: 
  - build seg trees: `O(n*log(n))`
  - query: `O(log(n))`


### 程式碼

```python
class Node:
    def __init__(self, cnt= 0):
        self.cnt = cnt
        self.left = None
        self.right = None

class PerSegTree:
    def __init__(self, minval, maxval):
        """Persistent Segment Tree"""
        self.minval = minval
        self.maxval = maxval
        self.versions = [self.build(self.minval, self.maxval)] # versions of tree
        
    def build(self, l, r):
        node = Node()
        if l == r:
            return node
        mid = (l + r) // 2
        node.left = self.build(l, mid)
        node.right = self.build(mid+1,r)
        return node
    
    def _update(self, prenode, l, r, val):
        # 受到影響的節點，創立新的node
        node = Node(prenode.cnt + 1)
        if l == r:
            return node
        mid = (l + r) // 2
        if val <= mid:
            # 往左邊更新，右子樹接上舊的節點
            node.left = self._update(prenode.left, l, mid, val)
            node.right = prenode.right
        else:
            # 往右更新，左子樹接上舊的節點
            node.left = prenode.left
            node.right = self._update(prenode.right, mid+1, r, val)
        return node

    def update(self, val):
        prenode = self.versions[-1]
        self.versions.append(self._update(prenode, self.minval, self.maxval, val))
    
    def _get_kth(self, node1, node2, l, r, k):
        if l == r:
            return l
        mid = (l + r) // 2

        # 差分，計算versions[v1:v2]在左子區間中有多少數量
        left_cnt = node2.left.cnt - node1.left.cnt
        if k <= left_cnt:
            return self._get_kth(node1.left, node2.left, l, mid, k)
        else:
            return self._get_kth(node1.right, node2.right, mid+1, r, k - left_cnt)

    def get_kth(self, idx1, idx2, k):
        return self._get_kth(self.versions[idx1], self.versions[idx2 + 1], self.minval, self.maxval, k)
```




## 參考資料
- [Oi-Wiki: 可持久化線段樹](https://oi-wiki.org/ds/persistent-seg/)
- [主席树（可持久化权值线段树）](https://blog.csdn.net/hzerotole/article/details/109633562)
- [CP-Algorihms: Persistent Segment Tree](https://cp-algorithms.com/data_structures/segment_tree.html#toc-tgt-12)
- [Wiki: Persistent data structure](https://en.wikipedia.org/wiki/Persistent_data_structure)
- [「BZOJ 3673」可持久化并查集 - 可持久化线段树](https://oi.men.ci/bzoj-3673/)


## 題目
- [D-Query](https://www.spoj.com/problems/DQUERY/)
  - 求区间内不重复的数的个数。扫描数列建立可持久化线段树，第i个数若第一次出现，则在线段树中的位置i加1
  - 若不是第一次出现，将上次出现的位置减1，在本次位置加1
  - 对于每个询问的区间 [L,R]，在第R个版本上的线段树只有前R个数，在线段树上查询位置L，对经过的区间中的和进行累计即可
- POJ 2104 K-th Number
- [P2048 [NOI2010] 超级钢琴](https://www.luogu.com.cn/problem/P2048)
  - 给一个长度为n的序列{a_i}，定义一个区间`[l, r]`的价值为这个区间中数的总和，求区间长度在`[L, R]`之间的所有区间中，价值最大的k个区间的价值总和。
    - 將每個左邊界i，在滿足長度下，挑出價值最大的區間`[i, r_i]`放入heap中
    - 從heap中pop價值最大的區間`[idx, r_idx]`
    - 並重新挑選出對idx而言滿足條件，**且不重複挑選下**，價值最大的區間`[idx, r2_idx]`再放入heap中
- [P3919 【模板】可持久化线段树 1（可持久化数组）](https://www.luogu.com.cn/problem/P3919)
- BZOJ 3673: 可持久化并查集 - 可持久化线段树
- 