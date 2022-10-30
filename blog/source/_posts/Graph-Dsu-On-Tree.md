---
title: DSU On Tree (樹上啟發式合併)
date: 2021-12-17 18:30:00
author: Aaron
img: /images/dsu_on_tree/img1.png
top: false
cover: true 
toc: true
summary: Detailed explanation of DSU on Tree
categories: 
  - Graph
tags: 
  - Graph
  - DSU On Tree
  - DSA Tutorial Graph
---

# 樹上啟發式合併

> In Iran, we call this technique "Guni" (the word means "Sack" in English), instead of "dsu on tree"

對於兩個大小不一樣的集合，我們將小的集合合併到大的集合中，而不是將大的集合合併到小的集合中。


## 理解輕重兒子以及輕重邊

每個節點之下，都有若干兒子子樹，定義**重兒子**為擁有最多節點的兒子子樹，其餘就是輕兒子。同理，重邊就是連接重兒子的邊，其餘就是輕邊

下圖，紅節點就是重兒子，黑粗邊就是重邊

![輕重兒子及輕重邊](/images/dsu_on_tree/img1.png)

## 作法

[CSES: Distinct Colors](https://cses.fi/problemset/task/1139/)

在一棵樹上為每個節點上色，對以每個節點為根的樹，都詢問內含多少種顏色。顯然窮舉法是O(n^2)

> 若我們能先透過前處理將每個輕重兒子區分出來，並在窮舉的過程，在每個節點採用下面策略
> 1. 先各自統計所有輕兒子的顏色，**不保留結果**
> 2. 統計重兒子的顏色，**並保留結果**
> 3. 重新掃過所有輕兒子，**並將結果累加到重兒子的結果上**，便能得到當前節點的顏色


## 複雜度

> 1. 步驟1和2其實就是遍歷每個點一次。
> 
> 2. **根結點到任意節點，經過的輕邊必然不超過log(n)條**。
> 
> 3. 除此之外每個點都還會被若干次步驟3經過，而每個點被步驟3經過的次數，其實就是這個點**往根方向經過的輕邊總數**。
> 
> 4. 因此每個點最多被經過1 + log(n)次，總合為n*(1 + log(n)) = `O(n*log(n))`



## 程式碼1

最標準的啟發式合併，但是有一個測資會TLE，雖然空間複雜度是O(n)，但是整體複雜很多。
- 時間複雜度：`O(n*log(n))`
- 空間複雜度：`O(n)`


> 原本我不是自己維護counter，而是使用 unordered_set去記錄顏色的數量，不過在測資10一直TLE。改成自己維護counter之後，不但通過了，總花費時間還只有版本二的60%左右
> 
> 結論是在考慮效能下，**能使用陣列就盡量就別用雜湊表**


```cpp
// cses1139_distinct_colors.cpp
#include<bits/stdc++.h>
#include<bits/extc++.h>
using namespace std;
using ll = long long int;
using pii = pair<ll, ll>;
#define AC ios::sync_with_stdio(0),cin.tie(0);
 
ll const MOD = 1e9 + 7;
ll const LL_MAX = 1e18 * 4 + 1;
ll const N = 200005;
ll const BLEN = __lg(N) + 1;
 
struct custom_hash {
    static uint64_t splitmix64(uint64_t x) {
        // http://xorshift.di.unimi.it/splitmix64.c
        x += 0x9e3779b97f4a7c15;
        x = (x ^ (x >> 30)) * 0xbf58476d1ce4e5b9;
        x = (x ^ (x >> 27)) * 0x94d049bb133111eb;
        return x ^ (x >> 31);
    }
 
    size_t operator()(uint64_t x) const {
        static const uint64_t FIXED_RANDOM = chrono::steady_clock::now().time_since_epoch().count();
        return splitmix64(x + FIXED_RANDOM);
    }
};
 
 
vector<int> adj[N]; // adjacent matrix
int ans[N]; // answer
int dfn[N]; // dfs order of each node
int big[N]; // big child of each node
int L[N]; // first idx of dfs order starting from node
int R[N]; // last idx of dfs order starting from node
int colors[N]; // color of each node
int node_size[N]; // size of each tree starting from node
int counter[N];

int timestamp = 0;
int total = 0;

void pre_dfs(int node, int parent) {
    /**
     * 1. mark dfn
     * 2. count size of tree with root node
     * 3. find big child of the current node 
     * 4. dfs order starting from the current node (L[node] ~ R[node])
     */
    dfn[++timestamp] = node;
    L[node] = timestamp;
    node_size[node] = 1;
    for(auto nxt : adj[node]) {
        if(nxt == parent) continue;
        pre_dfs(nxt, node);
        node_size[node] += node_size[nxt];
        if(!big[node] || node_size[big[node]] < node_size[nxt]) {
            big[node] = nxt;
        }
    }
    R[node] = timestamp;
}

void dfs(int node, int parent, bool keep) {
    // 1. traversal light childrens
    for(auto nxt : adj[node]) {
        if(nxt == parent || nxt == big[node]) continue;
        dfs(nxt, node, false);
    }

    // 2. traversal big child, and kepp current
    if(big[node]) {
        dfs(big[node], node, true);
    }

    // 3. sum up light sub-trees into current
    for(auto nxt : adj[node]) {
        if(nxt == parent || nxt == big[node]) continue;
        for(int t = L[nxt]; t <= R[nxt]; ++t) {
            if(counter[colors[dfn[t]]]++ == 0) total++;
        }
    }
    if(counter[colors[node]]++ == 0) total++;

    // remove counter if keep is false
    ans[node] = total;
    if(!keep) {
        for(int t = L[node]; t <= R[node]; ++t) {
            counter[colors[dfn[t]]]--;
        }
        total = 0;
    }
}

void solve() {
    int n, a, b, x;
    int idx = 0;
    unordered_map<int, int, custom_hash> color2idx;
    cin >> n;
    for(int i = 1; i <= n; ++i) {
        cin >> x;
        if(!color2idx.count(x)) color2idx[x] = idx++;
        colors[i] = color2idx[x];
    }
    
    for(int i = 0; i < n - 1; ++i) {
        cin >> a >> b;
        adj[a].push_back(b);
        adj[b].push_back(a);
    }
    pre_dfs(1, -1);
    dfs(1 , -1, true);
    for(int i = 1; i <= n; ++i) {
        cout << ans[i] << " ";
    }
    cout << endl;
}
 
int main() {
    AC
    solve();
 
    return 0;
}
```

## 程式碼2


對每個節點都儲存所有顏色，由於顏色數量不會超過節點數，若合併至母節點時，總是小的往大集合合併，總時空間複雜度應為n*log(n)
- 時間複雜度：`O(n*log(n))`
- 空間複雜度：`O(n*log(n))`

```cpp
// cses1139_distinct_colors.cpp
#include<bits/stdc++.h>
#include<bits/extc++.h>
using namespace std;
using ll = long long int;
using pii = pair<ll, ll>;
#define AC ios::sync_with_stdio(0),cin.tie(0);
 
ll const MOD = 1e9 + 7;
ll const LL_MAX = 1e18 * 4 + 1;
ll const N = 200005;
ll const BLEN = __lg(N) + 1;
 
struct custom_hash {
    static uint64_t splitmix64(uint64_t x) {
        // http://xorshift.di.unimi.it/splitmix64.c
        x += 0x9e3779b97f4a7c15;
        x = (x ^ (x >> 30)) * 0xbf58476d1ce4e5b9;
        x = (x ^ (x >> 27)) * 0x94d049bb133111eb;
        return x ^ (x >> 31);
    }
 
    size_t operator()(uint64_t x) const {
        static const uint64_t FIXED_RANDOM = chrono::steady_clock::now().time_since_epoch().count();
        return splitmix64(x + FIXED_RANDOM);
    }
};
 
 
vector<int> adj[N];
int ans[N];
unordered_set<int, custom_hash> colors[N];
 
void dfs(int idx, int pre) {
    for(auto nxt : adj[idx]) {
        if(nxt == pre) continue;
        dfs(nxt, idx);
        
        if (colors[nxt].size() > colors[idx].size()) {
            swap(colors[idx], colors[nxt]);
        }
        
        colors[idx].insert(colors[nxt].begin(), colors[nxt].end());
    }
    ans[idx] = colors[idx].size();
}
 
void solve() {
    int n, a, b, x;
    cin >> n;
    for(int i = 1; i <= n; ++i) {
        cin >> x;
        colors[i].insert(x);
    }
    
    for(int i = 0; i < n - 1; ++i) {
        cin >> a >> b;
        adj[a].push_back(b);
        adj[b].push_back(a);
    }
 
    dfs(1 , -1);
    for(int i = 1; i <= n; ++i) {
        cout << ans[i] << " ";
    }
    cout << endl;
}
 
int main() {
    AC
    solve();
 
    return 0;
}
```



## 參考資料
- [dsu on tree学习笔记](https://www.cnblogs.com/EinNiemand/p/11721101.html#%E5%BA%94%E7%94%A8%E7%AF%87%E5%90%84%E7%A7%8D%E7%81%B5%E6%B4%BB%E8%BF%90%E7%94%A8)
- [Codeforce: [Tutorial] Sack (dsu on tree)](https://codeforces.com/blog/entry/44351)
- [OI-Wiki: 树上启发式合并](https://oi-wiki.org/graph/dsu-on-tree/)

