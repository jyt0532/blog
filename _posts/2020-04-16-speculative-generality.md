---
layout: post
title: 重構 - 改善既有程式的設計 - Speculative Generality
comments: True
subtitle: 如何重構夸夸其談未來性
tags: refacforing
author: jyt0532
excerp: 本文介紹重構Speculative Generality
---

這篇文章討論《重構 - 改善既有程式的設計》裡的3.13 - Speculative Generality

圖片以及程式碼來源自[重構 - 改善既有程式的設計](https://www.tenlong.com.tw/products/9789861547534)


## 夸夸其談未來性

這是什麼意思呢 就是說 

"喔 這個是萬一怎麼樣怎麼樣的話可以用"

"喔 這個在之後會實作 先放著未來再說"

這種程式碼放久了就會有壞味道

## 解法

### [Collapse Hierarchy](2020/04/15/lazy-class/)

把用不到的抽象類別移除

### [Inline Class](/2020/04/13/shotgun-surgery/#inline-class)

非必要的delegation 就把它移除


