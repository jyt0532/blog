---
layout: post
title: 重構 - 改善既有程式的設計 - Temporary Field
comments: True
subtitle: 如何重構令人迷惑的暫時欄位
tags: refacforing
author: jyt0532
excerp: 本文介紹重構令人迷惑的暫時欄位
---

這篇文章討論《重構 - 改善既有程式的設計》裡的3.14 - Temporary Field


圖片以及程式碼來源自[重構 - 改善既有程式的設計](https://www.tenlong.com.tw/products/9789861547534)


## Temporary Field

什麼是暫時的欄位呢 就是一個物件裡面的某些欄位 只有在特定的時候有用 在其他的時候這些欄位是沒有用的 這樣就很容易會讓人迷惑 

常見的例子是 當你某個函式要跑一個演算法時 需要很多變數 可是你又不想把[所有變數都當函式的參數丟給函式](/2020/04/10/long-parameter-list/) 

所以你就把這些變數都塞給某個物件 成為這個物件的欄位們

但在演算法跑完之後 這些欄位就沒有用了 其他人在其他地方看到這些欄位 會非常迷惑


## 解法


### [Extract Class](/2020/04/10/large-class/#extract-class)

把那些只有特定情況會用到的欄位們 提煉到一個新的類別

### [Replace Method with Method Object](/2020/04/09/large-method/#大招-replace-method-with-method-object)

或者你也可以用我們的大招 在同一個類別裡新增一個類別 這樣也比較不會讓人疑惑



