---
layout: post
title: 重構 - 改善既有程式的設計 - Parallel Inheritance Hierarchies
comments: True
subtitle: 如何重構平行繼承體系
tags: refacforing
author: jyt0532
excerp: 本文介紹重構Parallel Inheritance Hierarchies
---

這篇文章討論《重構 - 改善既有程式的設計》裡的3.11 - Parallel Inheritance Hierarchies

圖片以及程式碼來源自[重構 - 改善既有程式的設計](https://www.tenlong.com.tw/products/9789861547534)


## 平行繼承體系

平行繼承體系是[散彈式修改](/2020/04/13/shotgun-surgery/)的特殊情況 指的是當你為某個類別添加subClass的時候 你也必須同時為另一個類別相應增加一個新的subClass 這就是個不好的味道

比如說你有一個`Engineer`的繼承體系 你也有一個`Milestone`的繼承體系

{% highlight java %}
interface Engineer{
  void A();
}
interface Milestone{
  void B();
}
{% endhighlight %}

每當你想在`Engineer`底下加一個subClass時 你也得同時在`Milestone`的繼承體系加一個相對應的工程師的milestone


{% highlight java %}
class ComputerEngineer implements Engineer{
  
}
class ComputerMilestone implements MileStone{

}
{% endhighlight %}

## 起因 

在設計的時候沒有把類別的責任劃分清楚


## 解法

如果你想消除這個對應關係 基本上就兩個解法

### 解法一: 物件同時實作兩個介面

{% highlight java %}
class ComputerEngineer implements Engineer,MileStone{
  
}
{% endhighlight %}

### 解法二: 把這兩個介面合在一起

{% highlight java %}
interface Engineer{
  void A();
  void B();
}

{% endhighlight %}

## 例外

有時候 這種平行的繼承體系是刻意的 身為部落格的忠實讀者 你知道是什麼情況嗎?


答對了 [抽象工廠](/2017/05/03/abstract-factory/)

當我要在產品族內加一個產品結構的時候 必須所有已經實作過抽象工廠的具體工廠都要做出相對應的變化


