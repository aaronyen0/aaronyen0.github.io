---
title: Leetcode 913. Cat and Mouse
date: 2021-12-09 10:55:23
author: Aaron Yan
cover: true 
toc: true
mathjax: true
summary: Whether the cat can catch the mouse? Stay tuned to find out!
categories: 
  - Dynamic Programming
tags: 
  - Leetcode
  - Dynamic Programming
  - 環狀DP
---


![cat and mouse](https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcQqCjL5wtwnzZg6umbXf9hvGNadI1yVmWSlje7ct3hfeuikITNkUXo3TfNd0YL6nN6xDgQ&usqp=CAU)


# Leetcode 913: Cat and Mouse

## 題目
給一個graph，其中老鼠在節點1，貓在節點2，出口在節點0。老鼠跟貓輪流移動一步，由老鼠先走，且滿足下面規則：
- 貓跟老鼠永遠都做最優行動
- 貓不能走到出口也就是位置0
- 老鼠走到出口，則老鼠贏，回傳1
- 貓跟老鼠在同一個位置，貓贏，回傳2
- 如果遊戲無法結束，則平局，回傳0


## 題解
這一題的難點在於，明明知道一定是DP問題，其中DP狀態可以定義為 (m, c, t)
- m: 老鼠的位置
- c: 貓的位置
- t: 輪到誰移動 (0老鼠，1貓)

但尷尬的是，我們卻無法像是一般DP一樣有一個好的判停方式，這是什麼意思？我們比較下面幾個DP結構


### DP概念
我們將分別展示一般的DP及有環的DP

#### 常見的DP路徑圖

- 以下圖而言我們首先會在V8判定最終結果
- 由於V5, V6, V7都依賴V8，便能進一步得知
- 依此類推直到推得V0的答案

<img src="/images/CatAndMouse_DP1.png" width="30%" height="30%">


- 以下圖而言，首先會判定所有葉節點
- 接著把只依賴葉節點的所有點又再判定一次，以此類推
<img src="/images/CatAndMouse_DP2.png" width="30%" height="30%">

> 不論是哪一種圖，都有一個很強大的性質，那就是
> <font color="LightSalmon" size=5>**依賴路徑中不存在環**</font>

#### 存在環的DP

- F18依賴F16及F17
- F16依賴F15及F19
- F19又依賴F18
<img src="/images/CatAndMouse_DP3.png" width="30%" height="30%">
**此時就形成了一個環狀的DP，導致DP變成一種雞蛋問題**
- 對F16而言，可以得到F15的結果，仍在等待F19
- 結果F19卻在等待F18，而F18卻在等F16


### 解決環狀DP
一般來說，這種情況下確實會無解。但某些情況下，對F16而言，未必非得知道F15及F19才能有答案。

> 以本題來說，若當前狀態為 (m, c, t)
> - **對角色t來說，只要存在一種移動方式確認為t的勝利，縱使不知道往其他方向的移動結果，這個狀態仍然是t的必勝狀態**
> - **若當前狀態的所有移動方式，都是已確認t會輸的狀態，顯然當前狀態也是t必輸的狀態**


#### 策略
> 因應上述的結論，以本題來說可以發展出一個策略
> - 首先挑出能求解的狀態，也就是(x,x,t)為貓的勝利，(0,x,t)為老鼠勝利，注意本題貓不能站在0的位置，將這些有解的狀態放入queue中
> - 從queue中依序取出一種有解的狀態(m, c, t)，檢查該狀態的母節點是否能得到解，注意母節點是輪到對手移動的回合
>   - **母節點有解的方式有 (1)對手走到當前狀態就獲勝, (2)對手已經沒有任何能獲勝的其他路徑**
> - **若母節點能得到解，將其放入queue中**


## 程式碼

### 解法一

實踐上節的策略

```python
class Solution:
    def catMouseGame(self, graph: List[List[int]]) -> int:
        n = len(graph)
        
        # calculate degree of each state
        degrees = [[[0]*2 for j in range(n)] for i in range(n)] #(m,c,t)
        for i in range(n):
            for j in range(n):
                if j == 0:
                    continue
                degrees[i][j][0] += len(graph[i])
                degrees[i][j][1] += len(graph[j]) - (0 in graph[j])
        
        # init dp and queue by known state
        dp = [[[0]*2 for j in range(n)] for i in range(n)] #(m,c,t)
        queue = collections.deque()
        for i in range(1, n):
            states = [(i,i,0),(i,i,1),(0,i,0),(0,i,1)]
            results = [2,2,1,1]
            for (m,c,t), rv in zip(states, results):
                dp[m][c][t] = rv
            queue.extend(states)
        
        while queue:
            # for each known state, traversal their parents, checking whether its parent could come up the answer or not
            m, c, t = queue.popleft()
            rv = dp[m][c][t]
            if t == 0: # mouse
                for pre in graph[c]:
                    if pre == 0 or dp[m][pre][1] != 0:
                        continue

                    # 對貓而言，rv=2，所以只要能走到(m,c,t)就贏了。但若rv=1，則不去走，除非所有可以走的路徑都是rv=1
                    if rv == 2:
                        dp[m][pre][1] = 2
                        queue.append((m, pre, 1))
                    else:
                        degrees[m][pre][1] -= 1
                        if degrees[m][pre][1] == 0:
                            dp[m][pre][1] = 1
                            queue.append((m, pre, 1))
            else: # cat
                for pre in graph[m]:
                    if dp[pre][c][0] != 0:
                        continue
                    
                    # 對老鼠而言剛好跟貓相反
                    if rv == 1:
                        dp[pre][c][0] = 1
                        queue.append((pre, c, 0))
                    else:
                        degrees[pre][c][0] -= 1
                        if degrees[pre][c][0] == 0:
                            dp[pre][c][0] = 2
                            queue.append((pre, c, 0))

        return dp[1][2][0]
```

- 時間複雜度：O(n^3)
  - 狀態有2*n^2種，每個狀態最多有n個節點相鄰
- 空間複雜度：O(n^2)


### 解法二

總共n個點，老鼠如果走了n步還沒到終點也沒被貓逮到，則平局。找不到一個好的證明，不過有看到一個我比較能接受的解釋

> 老鼠至多 n - 1 步就可以到達 0，我們記從 1 到 0 的這條路徑為 path。 如果第 n 步老鼠沒有到達 0 的話，就說明在某個時刻，老鼠偏離了原本的路徑path，為什麼會偏移呢？
> 
> 說明在前面的某一點 p，貓在旁邊候著。貓到點 p 的距離小於等於老鼠到點 p 的距離。 （注意可以有多個這樣的點 p, 無論老鼠選取那條通往 0 的路徑，路上都有一個這樣的點，老鼠會在這個點被貓逮到）。 而老鼠不經過 p 的話又不可能到達 0。

```python
class Solution:
    def catMouseGame(self, graph: List[List[int]]) -> int:
        @lru_cache(None)
        def search(t, x, y):
            if t == int(2*len(graph)): return 0
            if x == y: return 2
            if x == 0:return 1
            if (t % 2==0):
                if any(search(t+1, x_nxt, y)==1 for x_nxt in graph[x]):              
                     return 1
                if any(search(t+1, x_nxt, y)==0 for x_nxt in graph[x]):          
                    return 0
                return 2
            else:
                if any(search(t+1, x, y_nxt)==2 for y_nxt in graph[y] if y_nxt!=0):
                    return 2
                if any(search(t+1, x, y_nxt)==0 for y_nxt in graph[y] if y_nxt!=0):  
                    return 0
                return 1
        return search(0, 1, 2)

```

- 時間複雜度：O(n^4)
  - 個人覺得是狀態n^3再乘以n，不確定為何有人說是 (n^2)*m，如果有人能幫我解惑就好了
- 空間複雜度：O(n^3)


## Reference
- [Leetcode 1728. Cat and Mouse II](https://leetcode.com/problems/cat-and-mouse-ii/)