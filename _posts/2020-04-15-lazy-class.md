---
layout: post
title: 重構 - 改善既有程式的設計 - Lazy Class
comments: True
subtitle: 如何重構冗員類別
tags: refacforing
author: jyt0532
excerp: 本文介紹重構Lazy Class
---

這篇文章討論《重構 - 改善既有程式的設計》裡的3.12 - Lazy Class

圖片以及程式碼來源自[重構 - 改善既有程式的設計](https://www.tenlong.com.tw/products/9789861547534)


## 冗員類別

你所創建的類別都需要有人去理解它 維護它 這些都需要花時間心力 

如果一個類別不值得這麼做 就該把它處理掉

## 解法

### [Inline Class](/2020/04/13/shotgun-surgery/#inline-class)

### Collapse Hierarchy

如果subClass和superClass沒有多大差別 就把它合併變成一個

![Alt text]({{ site.url }}/public/collapse-hierarchy.png)

## 例外

有時候 你會看到冗員類別被創建出來 是為了未來的開發先放個placeholder 那你就要評估該不該留它
