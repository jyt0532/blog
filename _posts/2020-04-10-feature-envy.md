---
layout: post
title: 重構 - 改善既有程式的設計 - Feature Envy
comments: True
subtitle: 如何重構依戀情節
tags: refacforing
author: jyt0532
excerp: 本文介紹重構Feature Envy
---

這篇文章討論《重構 - 改善既有程式的設計》裡的3.7 - Feature Envy

圖片以及程式碼來源自[重構 - 改善既有程式的設計](https://www.tenlong.com.tw/products/9789861547534)


## 依戀情節

為什麼我們要物件導向 目的就是把 **資料** 和 **加諸其上的操作行為** 包裝在一起

所以一個常見的壞味道 就是類別裡面的函式對於**別的類別**比較有興趣

比如 ClassA 的函式F 一直呼叫 ClassB 的 getter 那就很可能F根本就應該存在ClassB


## 解法

來聊聊常見的解法

### Move Method

這是重構中最常用到的方法

![Alt text]({{ site.url }}/public/move-method.png)

就是把 aMethod 把 Class1 移到 Class2

### [Extract Method](/2020/04/09/large-method/#extract-method)

## 到底要移到哪裡去呢

那如果一個函式又跟ClassA拿東西 又跟ClassB拿東西 又跟自己拿東西 那這個函式到底應該要放到哪裡去呢?

原則就是 哪個Class被這個函式使用的最多 就該去哪

## 例外情況

有時候 我們刻意要把資料本身 跟 行為分開 那就不能refactor 

什麼時候呢 答對了 就是我們想要[動態改](/2017/04/07/strategy/)[變行為](/2017/05/30/state/)的時候
