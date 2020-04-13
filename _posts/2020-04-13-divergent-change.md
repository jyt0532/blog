---
layout: post
title: 重構 - 改善既有程式的設計 - Divergent Change
comments: True
subtitle: 如何重構發散式變化
tags: refacforing
author: jyt0532
excerp: 本文介紹重構Divergent Change
---

這篇文章討論《重構 - 改善既有程式的設計》裡的3.5 - Divergent Change

圖片以及程式碼來源自[重構 - 改善既有程式的設計](https://www.tenlong.com.tw/products/9789861547534)


## 發散式變化

如果一個類別經常因為不同的原因而在不同的方向上發生變化 就需要重構

沒錯 就是[單一職責原則](/2020/03/18/srp/)


## 解法

來聊聊常見的解法

### [Extract Class](/2020/04/10/large-class/#extract-class)

分離類別 讓每個類別只有單一職責

### [Extract SuperClass](/2020/04/12/refused-bequest/#extract-superclass)/[Extract SubClass](/2020/04/10/large-class/#extract-subclass)

善用繼承達到code reuse

## 比較

跟[散彈式修改](/2020/04/13/shotgun-surgery/)的差別如下

發散式變化 指的是一個類別會受到多種變化的影響

散彈式修改 指的是一個變化會影響到多個類別
