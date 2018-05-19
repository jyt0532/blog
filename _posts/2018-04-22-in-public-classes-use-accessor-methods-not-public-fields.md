---
layout: post
title: Effective Java Item16 - 在公有類中使用訪問方法而非公有域
comments: True 
subtitle: effective java - 在公有類中使用訪問方法而非公有域
tags: effectiveJava
author: jyt0532
excerpt: 本篇講解Java裡面 為何要在公有類中使用訪問方法來存取變量
---

這篇是Effective Java - In public classes, use accessor methods, not public fields章節的讀書筆記 本篇的程式碼來自於原書內容

### Item16: 在公有類中使用訪問方法而非公有域

在Java裡面 有個東西叫做退化類(degenerated class)

{% highlight java %}
class Point {
  public double x;
  public double y;
}
{% endhighlight %}


為什麼叫退化類呢 因為它沒有方法 就只是把每個變量集合在一起

這種類別沒有提供封裝 而裡面的成員都是public的話 我們根本無法控制誰能對它存取 
對於這種變量 我們應該用公有的訪問方法(getter, setter)來訪問這些變數

{% highlight java %}
// Encapsulation of data by accessor methods and mutators
class Point {
private double x;
private double y;
public Point(double x, double y) {
  this.x = x;
  this.y = y;
}
public double getX() { return x; }
public double getY() { return y; }
public void setX(double x) { this.x = x; }
public void setY(double y) { this.y = y; }
}
{% endhighlight %}

好處如下

1.比較好debug 你的訪問方法是唯一有機會存取你變數的地方 錯就只會在你的訪問方法錯 看log一目瞭然

2.當你之後想改變你內部數據的表示法時 你可以隨意變化: 比如說你想把原本的x, y座標 改成用極座標來存或是運算 你可以完全不讓你的用戶知道的情況下改變 

Java的某些平台類庫就違反了這項規則 比如Java.awt裡面的[Point](https://docs.oracle.com/javase/7/docs/api/java/awt/Point.html)和[Dimension](https://docs.oracle.com/javase/7/docs/api/java/awt/Dimension.html) 
這對於性能和安全性都造成了問題 直到今天都是 這兩個類別是最好的負面教材 戒之慎之

如果你非得要公開你的變量 那至少讓他是不可變的 起碼傷害還少一點

{% highlight java %}
// Public class with exposed immutable fields - questionable
public final class Time {
  private static final int HOURS_PER_DAY = 24;
  private static final int MINUTES_PER_HOUR = 60;
  public final int hour;
  public final int minute;
  public Time(int hour, int minute) {
    if (hour < 0 || hour >= HOURS_PER_DAY)
      throw new IllegalArgumentException("Hour: " + hour);
    if (minute < 0 || minute >= MINUTES_PER_HOUR)
      throw new IllegalArgumentException("Min: " + minute);
    this.hour = hour;
    this.minute = minute;
  }
  ... // Remainder omitted
}
{% endhighlight %}

這樣至少保證你回傳的時間實例都會是合理的 因為你還可以在contructor把關


