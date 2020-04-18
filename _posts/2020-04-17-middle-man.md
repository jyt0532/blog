---
layout: post
title: 重構 - 改善既有程式的設計 - Middle Man
comments: True
subtitle: 如何重構中間轉手人
tags: refacforing
author: jyt0532
excerp: 本文介紹重構Middle Man
---

這篇文章討論《重構 - 改善既有程式的設計》裡的3.16 - Middle Man

圖片以及程式碼來源自[重構 - 改善既有程式的設計](https://www.tenlong.com.tw/products/9789861547534)


## 中間轉手人

物件的基本特性之一就是封裝(encapsulation) 也就是對外部隱藏內部的細節 而封裝也往往伴隨著delegation

比如說你問你老闆有沒有時間參加一個會議 老闆就把這個訊息delegate給他的記事本 然後再回答你 你不用知道老闆的記事本是電子的還是紙本的

但如果一個類別過度使用delegation 比如果一個類別有超過一半的函式 都是去委託別人做的 那這樣就是過度使用delegation


## 解法

### Remove Middle Man

如果一個類別有一半以上的函式都是在delegate 那就該把中間人移除

![Alt text]({{ site.url }}/public/middle-man.png)

等等等 這個豈不是跟[Hide Delegate](/2020/04/17/message-chains/#hide-delegate)相反的圖嗎

沒錯 當我們發覺訊息鏈太長的時候 應該要封裝delegate 但這是要付出代價的

看回我們的例子 我們在[Hide Delegate](/2020/04/17/message-chains/#hide-delegate)完了之後 就可以帥氣的直接知道誰是Manager

{% highlight java %}
manager = john.getManager();
{% endhighlight %}

但如果有太多東西你都偷吃步 那`Person`就承擔了太多的責任

{% highlight java %}
departmentName = john.getDepartmentName();
departmentCode = john.getDepartmentCode();
departmentColor = john.getDepartmentColor();
{% endhighlight %}

當然這例子有點牽強 你必須在**不斷重構**的過程中 找到最適合的平衡 到底應該加多少中間人 到底應該delegate到什麼程度


## 例外

但有些中間人 是我們刻意為之的 這些就不要隨便移除 

哪些中間人呢 沒錯 就是[Proxy](/2017/10/06/proxy/)和[Decorator](/2017/04/18/decorator/)
