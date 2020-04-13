---
layout: post
title: 重構 - 改善既有程式的設計 - Alternative Classes with Different Interfaces
comments: True
subtitle: 如何重構異曲同工的類別
tags: refacforing
author: jyt0532
excerp: 本文介紹重構異曲同工的類別
---

這篇文章討論《重構 - 改善既有程式的設計》裡的3.18 - Alternative Classes with Different Interfaces

圖片以及程式碼來源自[重構 - 改善既有程式的設計](https://www.tenlong.com.tw/products/9789861547534)


## 異曲同工的類別

就是兩個類別名稱不同但做的是一樣的事

## 起因 

不要懷疑 在legacy code中很常見到 就是工程師在開發函式的時候 並不知道這個功能已經有了

## 解法

如果兩個函式在同一個library 那就選一個留下就好

但如果兩個在不同的library 就很值得仔細觀察這兩個函式 並分離出一個合理的抽象(介面)

怎麼分離呢

### Identical Method Signature

做同樣的事 那函數名稱 參數 回傳值就應該要一樣

### [Extract Superclass](/2020/04/12/refused-bequest/#extract-superclass)

分離出個共同的介面之後呢 就讓這兩個函式都實作它
