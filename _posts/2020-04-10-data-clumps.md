---
layout: post
title: 重構 - 改善既有程式的設計 - Data Clumps
comments: True
subtitle: 如何重構資料泥團
tags: refacforing
author: jyt0532
excerp: 本文介紹重構Data Clumps
---

這篇文章討論《重構 - 改善既有程式的設計》裡的3.8 - Data Clumps

圖片以及程式碼來源自[重構 - 改善既有程式的設計](https://www.tenlong.com.tw/products/9789861547534)


## 資料泥團

data items就像小孩子 喜歡成群結隊的待在一起 你常常可以在很多不同的地方看到相同的幾個 data item 同時出現

這些會同時出現的資訊就該放在同一個物件裡

這個問題同時出現在[過長參數列](/2020/04/10/long-parameter-list/)以及[過長的函式](/2020/04/09/large-method/)

## 判斷方式

那要怎麼判斷哪些人應該一起被放進別的物件裡呢? 你就把其中一個刪掉 找出其他跟著失去意義的資料 
那這些全部都應該一起打包成新物件


## 解法

### [Extract Class](/2020/04/10/large-class/#extract-class)

### [Introduce Parameter Object](/2020/04/09/large-method/#introduce-parameter-object)

### [Preserve Whole Object](/2020/04/09/large-method/#preserve-whole-object)

## 下一步

當把這些資料泥糰都整理好了之後 就可以開始尋找[Feauture Envy] 讓你知道哪些程式碼應該要移到其他class 哪些程式碼應該留下來
