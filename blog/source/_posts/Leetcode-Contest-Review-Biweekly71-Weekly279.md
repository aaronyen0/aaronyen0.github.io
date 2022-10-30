---
title: Leetcode Contest Review - Biweekly 71 & Weekly 279
date: 2022-02-06 20:40:00
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


## Leetcode Biweekly Contest 71 Review
這場比賽

### Q1: LC-2160. Minimum Sum of Four Digit Number After Splitting Digits

使用bitmask窮舉，寫了五分鐘有點久。這題有觀察到應該可以 `(a0, a2) + (a1, a3)`但是不敢寫，後來還是寫一個窮舉的版本。


### Q2: LC-2161. Partition Array According to Given Pivot

比typing速度


### Q3: LC-2162. Minimum Cost to Set Cooking Time

實作題，但是題目的數據範圍有一個陷阱，因為這個陷阱卡了應該有二十分鐘。**以後數據範圍應該要深思**


### Q4: LC-2163. Minimum Difference in Sums After Removal of Elements

沒難度。


### Result

47分鐘完賽加上1次WA：總名次155/16927

----

## Leetcode Weekly Contest 279 Review



### Q1: LC-2164. Sort Even and Odd Indices Independently

簡單但是寫起來有點麻煩的題目，不知道怎麼寫比較漂亮

```python
#Lee215的兩行解
def sortEvenOdd(self, A):
    A[::2], A[1::2] = sorted(A[::2]), sorted(A[1::2])[::-1]
    return A
```

### Q2: LC-2165. Smallest Value of the Rearranged Number

寫得有點太複雜，看到一個簡單的方式，從小排到大，找到第一個非零的數字跟最開頭的零交換即可。


### Q3: LC-2166. Design Bitset

沒難度，比閱讀速度的題目


### Q4: LC-2167. Minimum Time to Remove All Cars Containing Illegal Goods

這題還頗有難度，雖然寫對了。但我並沒有真的搞懂自己的DP轉移式


### Result

超過一小時都卡在第四題。這場第四題我覺得應該有不少人亂猜也寫對，整體算是難度適中的一場比賽了

83分鐘完賽加上2次WA：總名次818/18838

----

## Summary

- **英文閱讀能力需要提升**
- **實作題寫之前要想一下有沒有什麼更簡單的方式，不然都花太久了**
- **變數的邊界條件要考慮清楚**

