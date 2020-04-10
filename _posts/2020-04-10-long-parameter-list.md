---
layout: post
title: 重構 - 改善既有程式的設計 - Long Parameter List
comments: True
subtitle: 如何重構過長參數列
tags: refacforing
author: jyt0532
excerp: 本文介紹重構Long Parameter List
---

這篇文章討論《重構 - 改善既有程式的設計》裡的3.4 - Long Parameter List

圖片以及程式碼來源自[重構 - 改善既有程式的設計](https://www.tenlong.com.tw/products/9789861547534)


## 過長參數列

太長的參數列難以理解 太多參數會成前後不一致 不易使用 一但你需要更多資料 你也不得不修改它

## 解法

來聊聊常見的解法

### Replace Parameter with Method

如果被呼叫端也可以自己拿到那個參數 就可以直接讓被呼叫端自己算

比如原本是這樣
{% highlight java %}
int basePrice = _quantity * _itemPrice;
discountLevel = getDiscountLevel();
double finalPrice = discountedPrice (basePrice, discountLevel);
{% endhighlight %}
那其實discountLevel不用丟進去 讓discountedPrice自己算
{% highlight java %}
int basePrice = _quantity * _itemPrice;
double finalPrice = discountedPrice (basePrice);
{% endhighlight %}

特別是OOP `discountLevel`應該要是存在物件層級的變數 不應該還在函式之間傳來傳去

### [Preserve Whole Object](/2020/04/09/large-method/#preserve-whole-object)

### [Introduce Parameter Object](/2020/04/09/large-method/#introduce-parameter-object)

## 注意

本文提及的解法 可能會為你的函式跟類別產生依賴 也就是說這個函式被逼著要去了解這個給進來的類別 

你必須自行判斷值不值得

