---
title: Leetcode Contest Review - Biweekly 66 & Weekly 269
date: 2021-11-28 22:00:20
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


## Leetcode Biweekly Contest 66 Review


### Q1: LC-2085. Count Common Words With One Occurrence

簡單查數量的題目，比typing的速度而已


### Q2: LC-2086. Minimum Number of Buckets Required to Collect Rainwater from Houses

明顯是greedy的題目，但實作時有點卡住，沒有理好要以房子為單位搜尋還是以空地為單位搜尋，多花了幾分鐘。


### Q3: LC-2087. Minimum Cost Homecoming of a Robot in a Grid

只要看到最短距離，就腦殘想用Dijkstra，反而沒意識到這題所有的最短路徑都是最短距離，隨便挑一條greedy計算就可以了。時間浪費在實作dijkstra外加一個TLE


### Q4: LC-2088. Count Fertile Pyramids in a Land

明顯是DP問題，我的解法重複又冗長的程式碼太多，這部分應該包成function或是翻轉測資就好。

另外我解這種稍微複雜的題目，常常階段性的寫程式碼，例如這題我先寫一段程式，對每個點都找到往左及往右可以抵達的距離，但當下並沒有確定好下一步該做什麼，只是覺得這樣做有幫助，事後也確實是有幫助，並以此為基礎完成這題。值得反省的是，**如果這麼做沒幫助的話，是否不但浪費時間，又限縮了自己後續思考問題的方式**。

### Result

40分鐘完賽加上1次TLE：總名次160/8620

----

## Leetcode Weekly Contest 269 Review

### Q1: LC-2089. Find Target Indices After Sorting Array

比打字速度的題目

### Q2: LC-2090. K Radius Subarray Averages

固定遮罩的總合問題，也是看到就知道怎麼做。值得檢討的是我**題目常常看一半就開始作答**，做到一半的時候發覺欠缺了一些資訊才回去敘述中找，以本例來說我寫到一半才回去找遮罩寬度的定義以及輸出要落在什麼位置，算是蠻不好的習慣，另外也是同一個原因，一些需要事先定義的變數在寫的過程才一直補上，導致程式碼前幾行看起來沒有章法又混亂。

### Q3: LC-2091. Removing Minimum and Maximum From Array

我覺得這題寫得還不錯，只是有些程式碼看起來可以合併成一行才不會顯得有點冗，這場到寫完第三題共花7分鐘。

### Q4: LC-2092. Find All People With Secret

明顯是依照時間處理DSU的問題。只要是時間順序的題目，都要記得「**確認同一個時間內是否可能有多筆事件**」，第一版繳交的答案就是沒意識到這件事情導致WA。為了處理同一時間產生的事件，需要在這個時間點分群以及判斷哪些群該被納入答案中，但我第二版沒處理好分群問題，浪費時間寫了一個現在看來不可思議的答案，當時直到寫完測試才發現問題，最後第三版才改對。因此單單這題花了27分鐘外加一個WA，大把時間浪費在解bug以及莫名其妙的錯誤，明顯也是沒想清楚就開始寫題目的壞習慣導致。


### Result

34分鐘完賽加上1次WA：總名次: 215/10907

----

## Summary

- 題目及測資範圍都看完再作答 (cirtical)
- 做後續步驟前最好再問自己一遍真的想清楚了嗎 (cirtical)
  - 包含確認是否有必要先寫中間步驟的程式碼
- 競速心態要調整，最好把時間都遮起來
- 時間序的題目要注意重複時間問題
- 冗長又重複的程式碼該包成function
