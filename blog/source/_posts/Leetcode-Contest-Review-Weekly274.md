---
title: Leetcode Contest Review - Weekly 274
date: 2022-01-03 14:40:00
img: /images/LeetcodeIcon.png
toc: true
mathjax: false 
summary: Personal review of Leetcode contest
categories: 
  - Competitive Programming
tags: 
  - Competitive Programming
  - Leetcode
  - Leetcode Contest
---


![](/images/LeetcodeIcon.png)

## Leetcode Weekly Contest 274 Review

這場打得不太好，有點被第四題嚇到了，其實並沒有太難，也不需要SCC的知識。
另外前三題實在是有點太簡單，超過七千人完成前三題，實在是非常不平衡的比賽。


### Q1: LC-2124. Check if All A's Appears Before All B's

簡單，但竟然寫了兩分鐘，另外可以一行解 `return not 'ba' in s`

### Q2: LC-2125. Number of Laser Beams in a Bank

簡單，但也寫了三分鐘，在保留非0的數字有點卡住了

### Q3: LC-2126. Destroying Asteroids

簡單，寫了兩分鐘，完全不知道這題為何放在這

### Q4: LC-2127. Maximum Employees to Be Invited to a Meeting

其實很早就觀察出來只有兩種狀況
1. 一個大的環狀
2. 所有「兩個互相依賴的點並各自延伸出最長的分支」的相加

卡在兩個部分
1. **挑出所有環狀**，由於這題每個點只有一個父親，所以其實隨便從一個點往父節點移動，**最多只會有一個環**便停止。不過我寫了一個找strongly connected component(SCC)的解答，而且我本來還不會寫SCC，花了很多時間在研究SCC怎麼實作。
2. 針對所有只有兩個點的環，怎麼**在兩個點各自找一個最長分支，但是分支不能穿過彼此**，這明顯就兩個點各自DFS就好了，為了不往彼此移動，可以設計`dfs(node, root)`，其中初始的root就放彼此即可，不管其餘分支有多少，都不會導致dfs往彼此移動


### Result

86分鐘完賽，並有**6次WA**，排名381/14086

----

### Summary
- 還是要多觀察題目，很多題目並不需要特殊的演算法
- 程式設計的部分可以多想想，盡量寫的精簡清晰，不但好debug，也好修改
