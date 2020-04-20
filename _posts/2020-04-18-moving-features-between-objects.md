---
layout: post
title: 重構 - 改善既有程式的設計 - Moving Features between Objects
comments: True
subtitle: 在物件之間移動特性
tags: refacforing
author: jyt0532
excerp: 本文介紹如何在物件之間移動特性
---

這篇文章討論《重構 - 改善既有程式的設計》裡的第七章 - Moving Features between Objects

主要會討論[程式碼的壞味道](/toc/refactoring/)中 沒有被提及的重構方法裡面有關於 在物件之間移動特性 的內容

圖片以及程式碼來源自[重構 - 改善既有程式的設計](https://www.tenlong.com.tw/products/9789861547534)

## 在物件之間移動特性

在重構的過程中 數一數二重要的是 就是**決定把責任放在哪** 
我們有許多重構技巧可以讓我們把責任放在對的類別

### [Move Method](/2020/04/10/feature-envy/#move-method)

你的函式如果一直在跟別的類別一直交流 那就讓他去別的類別吧

### [Extract Class](/2020/04/10/large-class/#extract-class)

你的類別做了兩個類別應該做的事 就要分離出一個類別 記得[SRP](/2020/03/18/srp/)

### [Inline Class](/2020/04/13/shotgun-surgery/#inline-class)

[Move Method](#move-method)的相反 你覺得有一個冗類別的話 就把它刪掉 責任交給別人

### [Hide Delegate](/2020/04/17/message-chains/#hide-delegate)

與其讓客戶層層呼叫服務物件的delegate class 去了解delegate class的細節 不如就在服務物件中提供客戶所需的所有函式 隱藏委託關係

### [Remove Middle Man](/2020/04/17/middle-man/#remove-middle-man)

[Hide Delegate](#hide-delegate)的反向 就是當你的服務物件提供了太多客戶所需的函式 那不如就讓客戶自己呼叫delegate class吧

### [Introduce Foreign Method](/2020/04/17/incomplete-library-class/#introduce-foreign-method)

你所使用的server class需要**一個**新的函式 但你又不能改server class 

做法就是在client class中建立一個函式 並以server class的instance作為參數傳進這個函式

### [Introduce Local Extension](/2020/04/17/incomplete-library-class/#introduce-local-extension)

你所使用的server class需要**多個**新的函式 但你又不能改server class 

做法就是建立一個新類別 讓它去繼承或是Wrap這個server class


