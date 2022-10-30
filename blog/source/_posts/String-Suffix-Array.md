---
title: Suffix Array
date: 2021-12-27 17:30:00
author: Aaron
top: false
cover: true 
toc: true
summary: Detailed explanation of Suffix Array
categories: 
  - String
tags: 
  - String
  - Longest Duplicate Substring
  - DSA Tutorial String
---

# Suffix Array

給一個字串，將這個字串所有的後綴(suffix)都拿出來按照字典序排序，定義
- sa[rank] = idx: 排名在rank的後綴從哪個idx開始
- ranks[idx] = rank: 第idx號後綴的排名為rank

如下圖，字串為`"aabaaaab"`，則sa及ranks分別為
<img src="/images/suffix_array/sa1.jpg" width="70%" height="70%">

## 倍增法

為了更快速的對每個後綴排序，一種簡單的方式是使用倍增法。

> 在倍增法的第k輪中，我們對每個SubStr(idx, 2^k)做排序，其中:
> `SubStr(idx, 2^k) = SubStr(idx, 2^(k-1)) + SubStr(idx+2^(k-1), 2^(k-1))`
> 
> 若我們知道第k-1輪每個SubStr(idx, 2^(k-1))的大小關係，則SubStr(idx,2^k)的大小關係便能透過比較其`(ranks[idx], ranks[idx + 2^(k-1)])`得到，如下圖。

<img src="/images/suffix_array/sa2.jpg" width="70%" height="70%">


> 注意`idx+2^(k-1)`有可能超過字串範圍，在此將所有超過字串範圍的rank都視為0