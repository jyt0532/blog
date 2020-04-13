---
layout: post
title: 重構 - 改善既有程式的設計 - Shotgun Surgery
comments: True
subtitle: 如何重構散彈式修改
tags: refacforing
author: jyt0532
excerp: 本文介紹重構Shotgun Surgery
---

這篇文章討論《重構 - 改善既有程式的設計》裡的3.6 - Shotgun Surgery

圖片以及程式碼來源自[重構 - 改善既有程式的設計](https://www.tenlong.com.tw/products/9789861547534)


## 散彈式修改

當你遇到某個變化時 你必須到**許多不同的類別**去進行修改 這就是散彈式修改

如果需要修改的程式碼散落在各處 你不但很難找到他們 也很容易忘記某些重要的修改

## 起因 

原本應該屬於某個類別的責任 被分散在各處

## 解法

### [Move Method](/2020/04/10/feature-envy/#move-method)

移動你的函式還有欄位 到應該去的類別 如果這個類別還不存在就新增一個類別

### Inline Class

![Alt text]({{ site.url }}/public/shotgun-surgery.png)

跟[Extract Class](/2020/04/10/large-class/#extract-class)相反 如果你覺得某個類別根本沒有在承擔責任 
那就把這個類別的所有特性搬移到另一個類別之中 然後移除原本的類別

## 比較

跟[發散式變化](/2020/04/13/divergent-change/)的差別如下

發散式變化 指的是一個類別會受到多種變化的影響 

散彈式修改 指的是一個變化會影響到多個類別


