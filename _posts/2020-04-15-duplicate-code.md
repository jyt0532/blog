---
layout: post
title: 重構 - 改善既有程式的設計 - Duplicated Code
comments: True
subtitle: 如何重構重複的程式碼
tags: refacforing
author: jyt0532
excerp: 本文介紹重構Duplicated Code
---

這篇文章討論《重構 - 改善既有程式的設計》裡的3.1 - Duplicated Code

圖片以及程式碼來源自[重構 - 改善既有程式的設計](https://www.tenlong.com.tw/products/9789861547534)


## 重複的程式碼

就是重複的程式碼 這是比[Alternative Classes with Different Interfaces](/2020/04/13/alternative-classes-with-different-interfaces/)更通用的情況


## 解法 - 重複的程式碼在同一個類別的情況

### [Extract Method](/2020/04/09/large-method/#extract-method)

## 解法 - 重複的程式碼是發生在相同level的兩個不同subclass

### Pull Up Field

![Alt text]({{ site.url }}/public/pull-up-field.png)

就是把欄位提到superClass

### Pull Up Method

![Alt text]({{ site.url }}/public/pull-up-method.png)

就是把方法提到superClass

### Pull Up Constructor

就是把會重複使用的建構子放到superClass
{% highlight java %}
class Manager extends Employee {
  public Manager(String name, String id, int grade) {
    this.name = name;
    this.id = id;
    this.grade = grade;
  }
}
{% endhighlight %}

變成

{% highlight java %}
class Manager extends Employee {
  public Manager(String name, String id, int grade) {
    super(name, id);
    this.grade = grade;
  }
}
{% endhighlight %}

### Form Template Method

兩個subClass有相似的演算法結構 就用[Template method](/2017/09/12/template/)

## 解法 - 重複的程式碼是發生在不同的類別

### [Extract SuperClass](/2020/04/12/refused-bequest/#extract-superclass)

那就可以考慮把相同的部分提煉出一個共同的SuperClass 達到Code reuse

### [Extract Class](/2020/04/10/large-class/#extract-class)

如果無法繼承 就[考慮復合](/2018/05/05/favor-composition-over-inheritance/)

## 解法 - 一般情況

### Consolidate Conditional Expression

把相同結果的if-condition合併

{% highlight java %}
double disabilityAmount() {
  if (seniority < 2) {
    return 0;
  }
  if (monthsDisabled > 12) {
    return 0;
  }
  if (isPartTime) {
    return 0;
  }
}
{% endhighlight %}
變成
{% highlight java %}
double disabilityAmount() {
  if (isNotEligibleForDisability()) {
    return 0;
  }
}
boolean isNotEligibleForDisability() {
  return seniority < 2 || seniority >12 || isPartTime;
}
{% endhighlight %}

### Consolidate Duplicate Conditional Fragments

合併重複的條件片段


{% highlight java %}
if (isSpecialDeal()) {
  total = price * 0.95;
  send();
}
else {
  total = price * 0.98;
  send();
}
{% endhighlight %}
變這樣
{% highlight java %}
if (isSpecialDeal()) {
  total = price * 0.95;
}
else {
  total = price * 0.98;
}
send();
{% endhighlight %}


