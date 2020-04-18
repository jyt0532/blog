---
layout: post
title: 重構 - 改善既有程式的設計 - Inappropriate Intimacy
comments: True
subtitle: 如何重構不恰當的親密關係
tags: refacforing
author: jyt0532
excerp: 本文介紹重構Inappropriate Intimacy
---

這篇文章討論《重構 - 改善既有程式的設計》裡的3.17 - Inappropriate Intimacy

圖片以及程式碼來源自[重構 - 改善既有程式的設計](https://www.tenlong.com.tw/products/9789861547534)


## 不恰當的親密關係

就是兩個類別過於親密了 花太多時間去研究彼此的private成分

好的類別應該要進少的去了解其他的類別

## 解法

### [Move Method](/2020/04/10/feature-envy/#move-method)

如果一個方法或是欄位屬於Class2但只被Class1存取 那就把這個方法或欄位移過去吧

### Change Bidirectional Association to Unidirectional

![Alt text]({{ site.url }}/public/change-bidirectional-association-to-unidirectional.png)

原本兩個類別之間有雙向關聯 但如果`Order`不再需要了解`Customer`的特性 就該去除不必要的關聯

這樣可以讓依賴關係變成單向

### [Replace Inheritance with Delegation](/2020/04/12/refused-bequest/#replace-inheritance-with-delegation)

繼承往往是造成過度親密的原因 因為subClass對superClass的了解總是超出預期 你可以用delegation來減少兩者的依賴


