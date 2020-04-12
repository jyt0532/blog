---
layout: post
title: 重構 - 改善既有程式的設計 - Refused Bequest
comments: True
subtitle: 如何重構被拒絕的遺贈
tags: refacforing
author: jyt0532
excerp: 本文介紹重構被拒絕的遺贈
---

這篇文章討論《重構 - 改善既有程式的設計》裡的3.21 - Refused Bequest

圖片以及程式碼來源自[重構 - 改善既有程式的設計](https://www.tenlong.com.tw/products/9789861547534)


## 被拒絕的遺贈

指的是程式碼過度利用繼承

只因為想要reuse code而繼承自錯誤的superclass 導致subclass有些繼承自superclass的方法是完全用不到的


## 起因 

過度的濫用繼承 我們應該遵循[Liskov姊的原則](/2020/03/22/lsp/) 只在正確的時候使用繼承

## 解法

### Replace Inheritance with Delegation

![Alt text]({{ site.url }}/public/refused-bequest-1.png)

一樣 [講過不少次了](/2018/05/05/favor-composition-over-inheritance/)


### Extract Superclass

如果你還是覺得應該用繼承 那你就要仔細把subClass不需要的函式跟變數丟回給SuperClass

![Alt text]({{ site.url }}/public/refused-bequest-2.png)

讓subClass就只擁有它自己需要的函式以及變數


