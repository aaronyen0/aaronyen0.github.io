---
title: Leetcode 2117. Abbreviating the Product of a Range
date: 2021-12-28 12:13:00
author: Aaron Yan
cover: true 
toc: true
mathjax: true
summary: Maintain first k-digits for large value? Remember the Logarithm!
categories: 
  - Math
tags: 
  - Leetcode
  - Math
---

# Leetcode 2117. Abbreviating the Product of a Range


## 題目
給你兩個正整數left和right，計算閉區間[left, right]中所有整數的**乘積** 。
假設res為乘積的結果，C為後綴0的個數。將res去掉0後綴後的剩餘數字記為d。

1. 若d的長度大於十，乘積的表達法如下，pre為d的前五碼，suf為d的後五碼:
> `"<pre>...<suf>eC"`

1. 若d的長度在十以內，則乘積的表達法如下，suf就是d:
> `"<suf>eC"`

也就是
> 12345678987600000被表示為: `"12345...89876e5"`
> 345678987600000被表示為: `"3456789876e5"`


## 題解

- C: 可以計算2以及5的因數個數，便能事先得到10的個數。
- suf: 剔除10造成的影響後，簡單對10%10取餘數得到
- pre: 大數乘法顯然無法用正常的整數表達，但題目只要維護5個digits，於是我們轉成log10，將乘法運算變成加法運算，最後再還原即可


## 程式碼

```python
from math import log10

class Solution(object):
    def getfactCounts(self, n, f):
        # 1 ~ n 所有數字累計可以被f整除幾次: n//f + n//(f^2) + n//(f^3) + ...
        res = 0
        while f <= n:
            n //= f
            res += n
        return res

    def abbreviateProduct(self, left, right):
        cnt2 = self.getfactCounts(right, 2) - self.getfactCounts(left - 1, 2)
        cnt5 = self.getfactCounts(right, 5) - self.getfactCounts(left - 1, 5)
        cnt10 = min(cnt2, cnt5)

        # 先得到總共有多少10，等同於知道要消耗多少2和5
        use2, use5 = cnt10, cnt10
        frontlog, suf, mod = 0, 1, 10**10 
        for x in range(left, right + 1):
            while use2 and x % 2 == 0:
                use2 -= 1
                x //= 2
            while use5 and x % 5 == 0:
                use5 -= 1
                x //= 5

            suf *= x
            frontlog += log10(x)
            if suf > mod:
                suf = suf % mod
        
        # frontlog - int(frontlog)會變成0.xxxx，10^(0.xxxx)小數點之前只有一位，也就是x.xxxxx
        # 我們想要方便取前5個digits，若求frontlog - int(frontlog) + 4 = 4.xxxx
        # 則10^(4.xxxx)的小數點前有五位，變成xxxxx.xxxx
        if frontlog < 10:
            return str(suf) + "e" + str(cnt10)
        digit5 = str(10**(frontlog - int(frontlog) + 4))[:5]
        return digit5 + "..." + str(suf)[-5:] + "e" + str(cnt10)
```

## 心得

上了一課，以後大數乘法且精度要求不高的題目，可以考慮轉成log加法