---
title: 修正hexo-prism-plugin，大括號變為`&#123;`和`&#125`
date: 2021-12-08 11:00:00
author: Aaron
summary: 修正hexo-prism-plugin的大括號顯示問題
categories: 
  - Hexo
tags:
  - Hexo
---

# Issue
使用hexo-prism-plugin，在程式區塊中，左右大括號會自動被轉為html語法，也就是`&#123;`和`&#125`。


## Solution
1. 找到 `/node_modules\hexo-prism-plugin\src\index.js`
2. 將const map修改為

``` js
const map = {
  '&#39;': '\'',
  '&amp;': '&',
  '&gt;': '>',
  '&lt;': '<',
  '&quot;': '"',
  '&#123;': '{',
  '&#125;': '}'
};
```

## Reference
- [hexo-prism-plugin: curly bracket issue](https://github.com/ele828/hexo-prism-plugin/issues/61)