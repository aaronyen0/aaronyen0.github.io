---
title: Leetcode Contest Review - Biweekly 69 & Weekly 275
date: 2022-01-09 11:40:00
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


## Leetcode Biweekly Contest 69 Review
這兩天LC剛開賽前兩分鐘都進不去，後端的部分可能需要再加強。

### Q1: LC-2129. Capitalize the Title

比typing速度，這題用python絕對比其他都方便許多


### Q2: LC-2130. Maximum Twin Sum of a Linked List

一開始沒有看懂題目，多花了一些時間

### Q3: LC-2131. Longest Palindrome by Concatenating Two Letter Words

以難度而言我覺得放在第二題是比較恰當，就是找到跟自己翻轉後相同的字，再比出現次數而已。有其中一個粗心狀況是忘記乘兩倍導致WA，不但找bug花時間，而且倒扣更是悽慘。

**如何把程式寫的精確，並改正這種粗心的毛病**，是迫切需要加強的


### Q4: LC-2132. Stamping the Grid

~~
這題算是頗有難度的題目。我採用的是暴力解搭配sliding window maximum的優化，寫得很繁瑣，而且還WA三次，分別對應三種不同類型的測資，我認為這三次都是不該出現的倒扣。

WA
1. 沒注意到全部都是障礙時，就不用管H和W，直接通過
2. 離開迴圈之後再圈外做最後一次檢查，忘記把queue超出邊界的部分移除
3. **queue的大小沒數對**，我覺得這個是很嚴重的問題，想的不夠清楚
~~

以上的方法是錯的，請看**prefix sum matrix**就好

其他
1. 花了蠻多時間在研究binary search要挑哪個數字的問題，感覺對bs還是不夠熟練，**沒辦法直覺的確定要用bisect_left還是bisect_right，以及得到的idx要不要減1**。
2. 另外我的做法中，這題顯然適合將邊界再補上一層，這樣就不用在離開for之外又再跑一層，也不會有第二次WA錯誤
3. 這題絕大部分的人都寫**prefix sum matrix(pfsm)**解法，對每個點都尋找是不是郵票的左上角，也就是往右下`W*H`範圍內不能有障礙物，這步可以透過psfm得到。接著有兩個做法(1)對每張郵票都更新`W*H`範圍，這部分可以用BIT2去實作，**區間更新單點查詢**，每次更新花費`O(log(m*n))`；(2)對每個空白點，都搜尋是否有郵票能蓋到他，也就是往**左上** `W*H`範圍內找是否存在一張郵票，這步也可以透過對放置郵票的matrix做pfsm得到。

### Result

68分鐘完賽加上4次WA：總名次346/15120

----

## Leetcode Weekly Contest 275 Review
一樣遇到剛開賽會登不進去

### Q1: LC-2133. Check if Every Row and Column Contains All Numbers

標題都寫row和col了，敘述也有講清楚，結果我只有檢查row，這真是最不應該犯的WA，**題目真的要看仔細**阿

### Q2: LC-2134. Minimum Swaps to Group All 1's Together II

prefix或是sliding window的題目，我覺得寫得還ok

### Q3: LC-2135. Intervals Between Identical Elements

這題算難得算是有第三題的水準，targets內的words是否能透過任一starts裡的word經過兩步運算後得到。

1. 這題寫了12分鐘，題目有點長，應該有花五分鐘在看敘述，**英文閱讀能力及抓重點能力**要加強
2. 中間花了一些時間考慮要變動targets還是starts，其實都可以，反正重點是**英文字母只有26個**，而且這題無關乎字的排序，只考慮每個字母的數量而已，是非常強力的constraint，不要被字串的長度嚇到就好。


### Q4: LC-2136. Earliest Possible Day of Full Bloom

一開始就猜測是排序類Greedy問題，但花了很久的時間才決定要實作。
假設有兩個seed，分別`(p1, g1), (p2, g2)`，哪個seed一定要先種？
> 證明seed1先種比較小還是seed2先種比較小，等同決定一個策略找到下面公式
> `min(max(p1 + g1, p1 + p2 + g2), max(p2 + g2, p1 + p2 + g1))`
> 假設`g1 > g2`，可以發現
> > `p1 + p2 + g1 > p2 + g2`
> > `p1 + p2 + g1 > p1 + p2 + g2`: seed1先種的case2
> > `p1 + p2 + g1 > p1 + g1`: seed1先種的case1
> 也就是如果seed1後種的話，最大值都會超過seed1先種的結果
> 因此必然seed1要先種


### Result

算是場簡單的比賽，而且第四題幾乎直覺猜都會猜對。34分鐘完賽加上1次WA，排名229/16129

----

## Summary

- **題目要看清楚**，以後比賽前務必默念這句
- **英文閱讀能力需要提升**
- **程式寫的清晰精確可以減少粗心**
- **邊界條件要考慮清楚**，包含binary search或是一些sliding window等等
- **離開迴圈後做最後一輪檢查的問題，可以考慮多插入一個數字使之在迴圈內檢查**，否則要小心是否漏掉什麼
- **BIT的區間更新單點查詢**，可以再多練練模板題，**二維**的也要練
- **prefix sum matrix的問題**，可以再多想想
- 英文字只有26個是很強力的限制，不要忘記了

