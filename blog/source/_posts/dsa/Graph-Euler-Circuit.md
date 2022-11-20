---
title: Eulerian Circuit and Path
date: 2021-12-07 15:48:00
author: Aaron
img: /images/euler_circuit.png
top: false
cover: true 
toc: true
summary: Detailed explanation of Eulerian circuit and path
categories: 
  - Graph
tags: 
  - Graph
  - Euler Circuit
  - Euler Path
  - DSA Tutorial Graph
---

# Euler Circuit

## 經典問題
### Seven Bridges of Königsberg

![七橋問題](https://upload.wikimedia.org/wikipedia/commons/thumb/5/5d/Konigsberg_bridges.png/300px-Konigsberg_bridges.png)

這是著名的七橋問題，嘗試找一條可以經過七座橋各一次，然後**回到原處的路線**。

由於每座橋只能穿過一次，對圖上的某一個點來看，一旦從某座橋進入，就要從另一座橋走出去。所以，只要看到有個節點有奇數個邊，就表示有一條橋可以走入該節點卻走不出去。


### Eulerian Path
![一筆畫問題](https://upload.wikimedia.org/wikipedia/commons/thumb/1/11/Blender3D_HouseOfStNiclas.gif/83px-Blender3D_HouseOfStNiclas.gif)

又稱**一筆畫問題**，源於七橋問題，對於一個給定的圖，怎樣判斷是否存在著一個恰好包含了所有的邊，並且沒有重複的路徑


----

## 判斷是否滿足Euler
> 滿足歐拉路徑不一定有歐拉回路

![](https://media.geeksforgeeks.org/wp-content/cdn-uploads/Euler1.png)


### 無向圖
- Eulerian Circuit
  1. All vertices with non-zero degree are connected. We don’t care about vertices with zero degree because they don’t belong to Eulerian Cycle or Path (we only consider all edges). 
  2. All vertices have even degree.
- Eulerian Path
  1. Same as condition (1) for Eulerian Circuit 
  2. If zero or two vertices have odd degree and all other vertices have even degree. Note that only one vertex with odd degree is not possible in an undirected graph (sum of all degrees is always even in an undirected graph)

``` python
def test(graph):
    # 無向圖中判定歐拉回路及路徑
    odd = vertex_num_with_odd_degree(graph)
    if odd == 0:
        print("Eulerian Circuit")
    elif odd == 2:
        print("Eulerian Path")
    elif odd > 2:
        print("no Eulerian")
```


### 有向圖
- Eulerian Circuit
  1. All vertices with nonzero degree belong to a single strongly connected component. 
  2. In degree is equal to the out degree for every vertex.
- Eulerian Path
  1. at most one vertex has (out-degree) − (in-degree) = 1
  2. at most one vertex has (in-degree) − (out-degree) = 1
  3. In degree is equal to the out degree for every other vertex, and these vertex should belong to a single connected component of the underlying undirected graph.

----

## 解法

### Hierholzer's Algorithm
<font color="LightSalmon" size=5>**不論有向圖、無向圖、歐拉回路或歐拉路徑都能解**</font>


#### 有向圖中找歐拉回路
1. 先確認該圖滿足歐拉回路
2. 從任意一個點v出發，隨便走一條尚未選過的邊直到卡住，此時必然是卡在點v，因為每個點的入度應該要等同出度，至此所走過的路徑已經是一個歐拉回路，但不一定剛好走過圖中的每個邊
3. 退回曾路過的某個點w，若點w還有未曾走過的邊，則從w往另外一條未走過的邊移動，必然最終也會形成另一組封閉路徑並停在w，不斷做這件事情直到所有的邊都被走過

``` python
class EulerCircuit:
    def find_circuit(self, adj):
        # 假設adj已經滿足歐拉回路
        self.adj = adj
        self.res = []
        
        start = list(adj)[0]
        self.dfs(start, [start])
        return self.res[::-1]

    def dfs(self, x, cur):
        while self.adj[x]:
            y = self.adj[x].pop()
            cur.append(y)
            self.dfs(y, cur)
        self.res.append(cur.pop())

adj = {
    0: [1],
    1: [3, 2],
    2: [0],
    3: [4],
    4: [1]
}

res = EulerCircuit().find_circuit(adj)
# res = [0, 1, 3, 4, 1, 2, 0]
```


> 1. 從0出發沿著紅線走回0，此時是一個封閉路徑為{0,1,2,0}，但藍線還沒走過
> 2. 所以退回到點1並走藍線，又得到另一個封閉路徑{1,3,4,1}
> 3. 將藍色路徑跟紅色路徑接起來就變成 {0,1,3,4,1,2,0}

實作上就是不能走的時候，pop資料回到先前的節點，直到遇到可以走的節點繼續繞其他路徑
![EulerCircuit](/images/euler_circuit.png)


#### 無向圖找出歐拉回路 ([演算法筆記](https://web.ntnu.edu.tw/~algo/Circuit.html#3))


![一個 Euler Circuit ，在某點相交，可拆成兩個 Euler Circuit ](/images/EulerCircuit7.png)

![兩個 Euler Circuit ，可在某點相接，合成一個 Euler Circuit 。](/images/EulerCircuit8.png)


![大的 Euler Circuit 可拆成小的，小的可接成大的](/images/EulerCircuit9.png)

自然想到 Divide-and-Conquer Method，也就是在圖上**隨意走一圈**。未及之處，一定是一個（或數個） Euler Circuit 。

> Divide ：在圖上隨意走一圈。
>
> Conquer：其餘部份遞迴下去。
> 
> Combine：其餘部分的Euler Circuit們，銜接到隨意走的那一圈。


#### 有向圖中找歐拉路徑
1. 根據in-degree和out-degree的差確認是否有唯一起點，其餘相同


#### 無向圖中找歐拉路徑
1. 根據degree的奇偶數量，確認是否能成為起點，其餘相同


----

## 參考資料
- [Geeksforgeeks: Eulerian path and circuit for undirected graph](https://www.geeksforgeeks.org/eulerian-path-and-circuit/)
- [Geeksforgeeks: Euler Circuit in a Directed Graph](https://www.geeksforgeeks.org/euler-circuit-directed-graph/)
- [Wiki: Eulerian path](https://en.wikipedia.org/wiki/Eulerian_path)
- [演算法筆記: Euler Circuit](https://web.ntnu.edu.tw/~algo/Circuit.html#3)