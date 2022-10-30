---
title: Leetcode Contest Review - Biweekly 68 & Weekly 273
date: 2021-12-27 15:45:00
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


## Leetcode Biweekly Contest 68 Review


### Q1: LC-2114. Maximum Number of Words Found in Sentences

比typing速度


### Q2: LC-2115. Find All Possible Recipes from Given Supplies

這題蠻多英文單字和句子沒看懂，所以一開始亂猜題目，並寫了一個亂猜的版本，浪費了蠻多時間，**如果題目真的不太懂，就更該把測資看完才行**。後來懂了題目後，又花了蠻多時間才意識到是要做topological sort，但如何加速這件事情目前還沒想法，總之這題共花了13分鐘。


### Q3: LC-2116. Check if a Parentheses String Can Be Valid

隨便給一串只包含括號的字串，某些位置被鎖起來不能翻轉，其餘可以自由變換括號方向，問是否存在一個變化方式使字串變成合法的括號形式。
其實這題我不太肯定自己的答案是否正確或只是剛好AC，暫時無法明確想到改善方式，需要再多研究一下這題。

### Q4: LC-2117. Abbreviating the Product of a Range
將數字變成另一種表達法，這個表達法只會保留前後五碼，並喪失中間所有數字的精度，但題目也只要求前後五碼數字正確即可。

後五碼的部分應該不會有什麼問題，主要是前五碼，這題我花了兩個多小時才寫出一個前五碼精度不夠正確的答案。
題目是將所有數字相乘，後來看到**AC的解是將原始數字取log10後相加，之後再還原即可**，畢竟只要保留前五碼的精度，並不是什麼很嚴苛的條件，真是長知識了



### Result

38分鐘寫前三題加上1次WA：總名次383/10622

----

## Leetcode Weekly Contest 273 Review
這場比賽我只有參加virtual contest，不過跟前一天的比賽相比，真的是簡單很多


### Q1: LC-2119. A Number After a Double Reversal

比打字速度的題目，但這題我明顯濫用python數字及字串互轉的方便性，明明只要判斷是否餘十即可。

### Q2: LC-2120. Execution of All Suffix Instructions Staying in a Grid

這題原本想了很久，覺得應該有`O(n)`解，不過最終還是寫了暴力解`O(n^2)`，另外有一個很爛的bug花了很多時間才找到，他放在我自以為絕對不會出錯的小function上，所以debug途中一直沒有去看那個function。

### Q3: LC-2121. Intervals Between Identical Elements

我覺得是個好題目，**沿著數線上的點移動，並計算數線上所有點跟這個點的距離和**，雖然不是什麼難題，但我每次遇到都浪費很多時間在實現他，應該要找一個漂亮的版本理解一下。

### Q4: LC-2122. Recover the Original Array

突破口明顯就是最大或是最小數字，也曾想說有沒有更好的答案，不過顯然暴力解就足以通過，有些一倍delta兩倍delta之類的小細節造成的bug要注意，另外可以再多想想如何輸出解答會更有效。


### Result

是場簡單的比賽，浪費蠻多時間才40分鐘完賽，如果有參加的話應該會落在185/12203上下

----

## Summary

- 如果題目真的不太懂，就更該把測資看完 (critical)
- 英文閱讀能力該提升了，至少要回去把看不懂的問題再多看幾遍 (critical)
- 一些不追求精度的問題，可以考慮數值解，例如乘法改成取log10後相加
- 沿著數線上的點移動，並**計算數線上所有點跟當前位置的距離和**，遇過這種題目超多次了，每次都寫一個不同版本，快去找一個寫得漂亮的解 (critical)
- 有些題目即使無腦解能過，或許簡單想一下實作起來更簡單
- 寫程式時，大方向把握後，就該注意小細節別寫錯