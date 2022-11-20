---
title: Leetcode Contest Review - Weekly 270
date: 2021-12-08 13:43:20
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

## Leetcode Weekly Contest 270 Review

這場打得不太好，除了第四題為**圖論的一筆畫問題**確實不會之外，在前三題的解題過程中都有藏有許多瑕疵需要好好反省


### Q1: LC-2094. Finding 3-Digit Even Numbers

第一題正常來說都要秒解，但這題我不但TLE一次還寫了十分鐘。題目是從pool中任挑三個數字排成一排，並由小到大輸出所有不重複的狀況。

我的程式簡單來說就是把pool跑permutation並只做前三項，第一版繳交時是在6分鐘的時候，程式碼基本上都寫對，也有在idx=3的時候輸出答案，**但是程式碼內竟然沒有在idx=3的時候return**，導致繼續作idx>3的部分，最終TLE，最後找bug並做了一些其實不用的優化，花了10分鐘加上TLE才修正。


### Q2: LC-2095. Delete the Middle Node of a Linked List

從LinkedList中找到特定節點並刪除，這題我覺得寫得還算快，僅花費4分鐘。


### Q3: LC-2096. Step-By-Step Directions From a Binary Tree Node to Another

這題我覺得程式碼寫得很冗，題目希望從一棵樹中，輸出從某個點到另一個點的最短路徑。

這題程式碼的部分特別難看，一堆if區塊，非常瑣碎，導致寫了整整12分鐘。

有看到一種做法，像這種只給樹根的二元樹，而且又不是從樹根為起點，**可以將整棵樹轉換為adjacent matrix**，之後便能透過adj直接從指定起點往外移動。


### Q4: LC-2097. Valid Arrangement of Pairs

這題可說是一筆畫問題的模板題，一筆畫問題是一個圖論經典問題，沒看過算是我不學無術。

值得一提的是，其實我在想得過程中是有點接近正確答案的，當時在思考要如何把某另一條路徑插在目前的路徑上。

仔細想想，這就是divide and conquer或是stack的思想。不馬上輸出目前的答案，先把另一條路徑走完並輸出後再把其他的也補上。


### Result

沒能完賽，26分鐘寫3題，加上1次TLE：總名次436/12931

----

### Summary
- 不要求程式碼逐行檢查，但是判停條件的位置最好再看一下
- 有些二元樹的題目可以轉換成adjacent matrix會更容易寫
- Euler Path&Circle以及Hierholzer's Algorithm需要自己去念書

