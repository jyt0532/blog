---
layout: post
title: 重構 - 改善既有程式的設計 - Composing Methods
comments: True
subtitle: 重新組織你的函式
tags: refacforing
author: jyt0532
excerp: 本文介紹如何重新組織你的函式
---

這篇文章討論《重構 - 改善既有程式的設計》裡的第六章 - Composing Methods

主要會討論[程式碼的壞味道](/toc/refactoring/)中 沒有被提及的重構方法裡面有關於 重新組織你的函式 的內容

圖片以及程式碼來源自[重構 - 改善既有程式的設計](https://www.tenlong.com.tw/products/9789861547534)

## 重新組織你的函式

### [Extract Method](/2020/04/09/large-method/#extract-method)

你有一段程式碼可以被組織在一起並獨立出來 你就該把那段程式碼分離成單獨的函式

### [Extract Variable](/2020/04/15/comments/#extract-variable)

用好的變數名稱來解釋運算式用途

### [Replace Temp with Query](/2020/04/09/large-method/#replace-temp-with-query)

用一個獨立的函式來取代暫時的區域變數

### Remove Assignments to Parameters

不要對參數進行賦值的動作 比如說

{% highlight java %}
int discount(int inputVal, int quantity) {
  if (inputVal > 50) {
    inputVal -= 2;
  }
}
{% endhighlight %}

`inputVal`是你的輸入變數 盡量不要去給他一個全新的值

{% highlight java %}
int discount(int inputVal, int quantity) {
  int result = inputVal;
  if (inputVal > 50) {
    result -= 2;
  }
}
{% endhighlight %}

你就先用一個變數把結果先記下來 之後愛怎麼玩就怎麼玩

**注意 如果是改動的話 那還可以接受**

{% highlight java %}
void aMethod(Object foo) {
  foo.modifyInSomeWay(); // 還OK
  foo = anotherObject; // 別這麼幹
} 
{% endhighlight %}

### [Replace Method with Method Object](/2020/04/09/large-method/#大招-replace-method-with-method-object)

把區域變數變成物件內的欄位 就不用在函式之間把變數傳來傳去









