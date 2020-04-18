---
layout: post
title: 重構 - 改善既有程式的設計 - Incomplete Library Class
comments: True
subtitle: 如何重構不完美的程式庫類別
tags: refacforing
author: jyt0532
excerp: 本文介紹重構Incomplete Library Class
---

這篇文章討論《重構 - 改善既有程式的設計》裡的3.19 - Incomplete Library Class

圖片以及程式碼來源自[重構 - 改善既有程式的設計](https://www.tenlong.com.tw/products/9789861547534)


## 不完美的程式庫類別

我們常常使用library來達成code reuse的效果 但有的時候 library就沒有我們要的功能 要我們加又加不上去 畢竟是read-only的程式庫



## 解法

### Introduce Foreign Method


{% highlight java %}
class Report {
  void sendReport() {
    Date nextDay = new Date(previousEnd.getYear(),
      previousEnd.getMonth(), previousEnd.getDate() + 1);
    ...
  }
}
{% endhighlight %}

可惡 我們想要的nextDay功能沒有提供怎麼辦 沒關係我們自己加一個函式


{% highlight java %}
class Report {
  void sendReport() {
    Date newStart = nextDay(previousEnd);
    ...
  }
  private static Date nextDay(Date arg) {
    return new Date(arg.getYear(), arg.getMonth(), arg.getDate() + 1);
  }
}
{% endhighlight %}

為什麼要提到這個呢 因為這個函式庫有著大部分你想要的功能 就只差一個函式是你特別常用的 因為差別很細微 所以很多legacy code就直接寫死 不把他單獨獨立成一個新的函式 維護起來測試起來都很困難

### Introduce Local Extension

那當你需要多加的[Foriegn Method](#introduce-foreign-method)太多了 那就不如把這些外加的函式包在一起 變成一個subclass或是wrapper吧

![Alt text]({{ site.url }}/public/introduce-local-extension.png)

就是繼承了程式庫的類別之後 再新增一些你自己要用的功能
