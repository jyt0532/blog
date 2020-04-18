---
layout: post
title: 重構 - 改善既有程式的設計 - Message Chain
comments: True
subtitle: 如何重構過度耦合的訊息鏈
tags: refacforing
author: jyt0532
excerp: 本文介紹重構Message Chain
---

這篇文章討論《重構 - 改善既有程式的設計》裡的3.15 - Message Chain

圖片以及程式碼來源自[重構 - 改善既有程式的設計](https://www.tenlong.com.tw/products/9789861547534)


## 過度耦合的訊息鏈

如果模組A跟模組B合作 且模組B跟模組C合作 我們不希望模組A了解模組C的資訊 比如說

{% highlight java %}
a.getB().getC().doSomething()
{% endhighlight %}

我們稱為Message Chain 這意味著物件之間的緊密耦合

這樣的話 如果你未來想在B跟C之間插入一個Q 你會難以進行架構上的變動 你必須找到所有的`a.getB().getC()` 改成`a.getB().getQ().getC()`

## 解法

### Hide Delegate

![Alt text]({{ site.url }}/public/hide-delegate.png)

假設我們有個`Person`和`Department`

{% highlight java %}
class Person {
  Department _department;
  public Department getDepartment() {
    return _department;
  }
  public void setDepartment(Department arg) {
    _department = arg;
  }
}
class Department {
  private String _chargeCode;
  private Person _manager;
  public Department (Person manager) {
    _manager = manager;
  }
  public Person getManager() {
    return _manager;
  }
}
{% endhighlight %}

這種情況下 你想知道一個人的老闆 只能

{% highlight java %}
john.getDepartment().getManager()
{% endhighlight %}

但這樣增加了`Person`跟`Department`的耦合 畢竟`Person`根本就不Care `Department`

那怎麼辦 就在Person裡面加一個Delegate method: `getManager()`

{% highlight java %}
class Person {
  Department _department;
  public void setDepartment(Department arg) {
    _department = arg;
  }
  public Person getManager() {
    return _department.getManager();
  }
}
{% endhighlight %}

注意這時候我就可以拿掉`Person`裡面的`getDepartment()`了

