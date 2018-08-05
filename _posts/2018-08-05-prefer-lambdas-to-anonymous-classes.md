---
layout: post
title: Effective Java Item42 - lambda表達式優於匿名類 
comments: True 
subtitle: effective java - lambda表達式優於匿名類
tags: effectiveJava
author: jyt0532
excerpt: 本篇文章介紹XX
---

這篇是Effective Java - Prefer lambdas to anonymous classes章節的讀書筆記 本篇的程式碼來自於原書內容

## Item42: lambda表達式優於匿名類

### 匿名類

我們把只定義一個函數的介面或是抽象類別 稱為**函數類型**(function types) 而函數類型的實例稱為函數對象(funcitonal objects)

一直以來 我們創建函數對象的主要方法是[匿名類](/2018/08/04/favor-static-member-class-over-nonstatic/)

看一下我們以前怎麼用匿名類排序

{% highlight java %}
// Anonymous class instance as a function object - obsolete!
Collections.sort(words, new Comparator<String>() {
  public int compare(String s1, String s2) {
    return Integer.compare(s1.length(), s2.length());
  }
});
{% endhighlight %}

sort的第二個參數 是一個有實作Comparator的類別的實例 
這樣的寫法等於是在sort需要一個類別的時候 
隨手寫給你 當然這個你隨手寫的類別需要實作compare

這種寫法使得匿名類適用於一個著名的設計模式 - [策略模式](/2017/04/07/strategy/)

Comparator就是抽象策略(FlyBehavior) 匿名類就是具體策略(FlyWithWings)

### 函數式編程

在Java8中 使用單個抽象方法的接口我們稱為函數式接口 而函數式接口可以用lambda表達式來創建這些接口的實例

上例子

{% highlight java %}
Collections.sort(words, 
  (s1, s2) -> Integer.compare(s1.length(), s2.length()));
{% endhighlight %}

優美 簡潔 我甚至連s1 s2的類型都不用跟它說 它可以從words推斷

你還可以用比較器的構造方法(Comparator construction method)

{% highlight java %}
Collections.sort(words, comparingInt(String::length));
{% endhighlight %}

Java8之後List還支援了sort方法

{% highlight java %}
words.sort(comparingInt(String::length));
{% endhighlight %}

以上的例子 我們看得出來lamdba讓我們程式更加簡潔

### 實用性

lambda不只讓你的程式更加簡潔 還更加實用

我們假設我們有個enum 裡面有4個operation

{% highlight java %}
public enum Operation {
  PLUS, MINUS, TIMES, DIVIDE;

  // Do the arithmetic operation represented by this constant
  public double apply(double x, double y) {
    switch(this) {
      case PLUS:   return x + y;
      case MINUS:  return x - y;
      case TIMES:  return x * y;
      case DIVIDE: return x / y;
    }
    throw new AssertionError("Unknown op: " + this);
  }
}
{% endhighlight %}

用法是這樣

{% highlight java %}
Operation.PLUS.apply(2.0, 3.0); // 5
{% endhighlight %}

用法簡單 可是這個enum的寫法很醜 如果沒有throw Exception會無法編譯

比較好一點的寫法如下

{% highlight java %}
public enum Operation {
  PLUS  {public double apply(double x, double y){return x + y;}},
  MINUS {public double apply(double x, double y){return x - y;}},
  TIMES {public double apply(double x, double y){return x * y;}},
  DIVIDE{public double apply(double x, double y){return x / y;}};
  public abstract double apply(double x, double y);
}
{% endhighlight %}

使用方法一樣 但這個寫法對於每一個enum要做的事 都直接定義一個function 

這時候就是用lambda的好時機

{% highlight java %}
public enum Operations {
  PLUS  ((x, y) -> x + y),
  MINUS ((x, y) -> x - y),
  TIMES ((x, y) -> x * y),
  DIVIDE((x, y) -> x / y);

  private final DoubleBinaryOperator op;

  Operations(DoubleBinaryOperator op) {
    this.op = op;
  }
  public double apply(double x, double y) {
    return op.applyAsDouble(x, y);
  }
}
{% endhighlight %}

這裡的DoubleBinaryOperator是一個函數類型 也就是只有一個函數的介面
{% highlight java %}
@FunctionalInterface
public interface DoubleBinaryOperator {
    /**
     * Applies this operator to the given operands.
     *
     * @param left the first operand
     * @param right the second operand
     * @return the operator result
     */
    double applyAsDouble(double left, double right);
}
{% endhighlight %}

而我們實作這個函數介面 完全不需要匿名類 我們直接給一個符合需求的lambda
也就是輸入兩個double 輸出一個double的函數


### Lamdba限制


1.沒有函數名稱 沒有文檔
所以lambda必須要非常好懂 別人一看就知道在幹嘛 可以一行搞定就一行搞定 不然也不要超過三行
無法做到以上幾點的話 那就乖乖寫一個函式

2.僅限於函數式接口

3.無法使用this來獲得自身對象的引用

### 只能用匿名類而不能用lambda的時機

1.想創建抽象類的實例

2.想創建有多個抽象方法接口的實例

3.想獲得自身對象的引用(可以利用this引用匿名類的實例)


### 序列化

lambda和匿名類都無法可靠的序列化 你也應該不太有機會會要序列化匿名類或lambda 

但如果你真的需要序列化 使用[私有靜態嵌套類的實例](/2018/08/04/favor-static-member-class-over-nonstatic/)

### 結論

lambda是表示小的函數對象的最佳方式 
除非必須創建非函數式接口類型的實例 否則不要使用匿名類作為函數對象

lambda表達式的許多特點 也開始了Java的函數式編程時代

