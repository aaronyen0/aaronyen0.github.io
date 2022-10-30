---
title: Leetcode Contest Review - Weekly 272
date: 2021-12-19 12:00:00
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

## Leetcode Weekly Contest 272 Review

這場比賽實在太簡單，13分鐘完賽排全球51名，雖然成就感十足，但有時候不禁讓人覺得打LC Contest是浪費時間。想想昨天有一場Codeforce比賽沒去參加真的是比較可惜了。

### Q1: Find First Palindromic String in the Array

秒解


### Q2: Adding Spaces to a String

這場最失敗的一題，我看到要push空白字符到string，怕用python的字串相加(`str + str`)會導致複雜度變成n^2，竟然一時之間腦袋當機改成用C++寫，python明明隨便開個list放資料最後再join就好。偏偏C++又不太熟，多花了一點時間在想語法，共花了四分鐘。

值得一提的是，我顯然沒仔細注意spaces的範圍，所以多寫了幾行無意義的邊界判斷，雖然這次沒出錯，但真的該慎記題目要看仔細...


### Q3: Number of Smooth Descent Periods of a Stock

有除過一次bug是沒注意每個價格要剛好遞減1，當時以為只需要遞減。也是題目沒看仔細的緣故...


### Q4: Minimum Operations to Make the Array K-Increasing

這題第一次掃過題目時沒有完全看懂重點是什麼，後來看到測資二才明白根本就是LIS，這題主要卡在英文比大多數參賽者弱，希望能再提升。
另外發現有些rating蠻高的台灣人竟然不知道有nlogn的LIS，真是替他們可惜


### Result

13分鐘完賽：總名次51/15426

----

### Summary
- <font color="LightSalmon" size=5>**題目真的真的真的要看仔細**</font>
- 需要提升英文閱讀的精度和速度
