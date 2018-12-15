---
layout: post
title: Effective Java Item30 - 優先考慮泛型方法
comments: True 
subtitle: effective java - 優先考慮generic method
tags: effectiveJava
author: jyt0532
excerpt: 本
---

這篇是Effective Java - Favor generic methods章節的讀書筆記 本篇的程式碼來自於原書內容

本章節可以搭配[泛型篇章簡介及術語列表](/2018/12/01/generics/)服用

## Item30: 優先考慮泛型方法

如同類可以是泛型的 方法也可以是泛型的 

常見的有對於參數化類型(Parameterized type)進行操作的靜態工具方法通常都是泛型 或是集合(Collection)中的大部分algorithm比如`binarySearch` `sort` 都是泛型

來看個例子
{% highlight java %}
// Uses raw types - unacceptable! [Item 26]
public static Set union(Set s1, Set s2) {
  Set result = new HashSet(s1);
  result.addAll(s2);
  return result;
}
{% endhighlight %}

這個方法可以compile 但是會有兩個警告

![Alt text]({{ site.url }}/public/item30-1.png)
![Alt text]({{ site.url }}/public/item30-2.png)

要消除這些警告 必須讓參數泛型
{% highlight java %}
public static <E> Set<E> union(Set<E> s1, Set<E> s2) {
  Set<E> result = new HashSet<>(s1);
  result.addAll(s2);
  return result;
}
{% endhighlight %}

就是這麼簡單 沒有警告 不包含強制轉換 提供類型安全和容易使用

注意 對於`union`這個方法 局限性就是三個set的型態必須要一樣 如果你想要更多彈性 你可以使用限制通配符類型(Bounded wildcard type) 這會在[下一篇](/)再提到

### 泛型單例工廠(generic singleton factory)

有的時候 你需要創建一個不可改變但適用於許多不同類型的對象 而因為泛型是由擦除實現 
所以你可以只用**一個物件** 來達成你所需要的類型參數化 
只是這個物件不能一開始直接宣告直接用 只是你需要一個靜態工廠來幫你在每次有人需要的時候分配對象

來個例子 恆等函數 identity function dispenser (注意 `Function.identity`已經幫你實現好了 你實際上並不需要真的寫一個實現 這裡只是拿來說明)


{% highlight java %}
// Generic singleton factory pattern
private static UnaryOperator<Object> IDENTITY_FN = (t) -> t;
@SuppressWarnings("unchecked")
public static <T> UnaryOperator<T> identityFunction() {
  return (UnaryOperator<T>) IDENTITY_FN;
}
{% endhighlight %}

`IDENTITY_FN` 就是我們的函數對象 也就是我們上一段的**一個物件** 
然後在使用者有需要的時候 把這個物件過一下工廠 幫忙Cast成我們要的類型
至於`@SuppressWarnings("unchecked")` 則是因為把`UnaryOperator<Object>`轉成`UnaryOperator<T>`的時候會有unchecked warning 所以需要抑制

看用法
{% highlight java %}
String[] strings = { "jute", "hemp", "nylon" };
UnaryOperator<String> sameString = identityFunction();
for (String s : strings)
  System.out.println(sameString.apply(s));

Number[] numbers = { 1, 2.0, 3L };
UnaryOperator<Number> sameNumber = identityFunction();
for (Number n : numbers)
  System.out.println(sameNumber.apply(n));

{% endhighlight %}

這就是泛型單例工廠 當你需要某個類型的函數方法 過個工廠後你就可以用了

### 遞歸類型限制(Recursive type bound)

我們知道我們可以用一些限制來限定泛型類型所實際可選擇的類型

比如

`Collection<? extends E>` 代表說這個類型是`E`或是`E`的subtype 

雖然不常見 但我們可以用**這個類型有實作的介面** 來作為一個泛型類型的限制

最常見的例子
{% highlight java %}
// Using a recursive type bound to express mutual comparability

public static <E extends Comparable<E>> E max(Collection<E> c);
{% endhighlight %}

複習一下Comparable
{% highlight java %}
public interface Comparable<T> {
  int compareTo(T o);
}
{% endhighlight %}
這就是說 現在你要用的E 必須要有實現`Comparable` 這樣這個方法就可以光明正大的在實作中呼叫`compareTo`

看個用法

{% highlight java %}
// Returns max value in a collection - uses recursive type bound
public static <E extends Comparable<E>> E max(Collection<E> c) {
  if (c.isEmpty())
    throw new IllegalArgumentException("Empty collection");
  E result = null;
  for (E e : c)
    if (result == null || e.compareTo(result) > 0)
      result = Objects.requireNonNull(e);
  return result;
}
{% endhighlight %}

### 結論

泛型方法就像泛型 使用起來比客戶自己強制Cast要安全的多 所以當你有機會把你的方法泛型化 就把它泛型化 

本篇還提到了泛型單例工廠以及遞歸類型限制 下一章是泛型的重頭戲 這篇文章一直輕描淡寫的帶過`extend`和`super`等等的用法 全都會在下一篇提到
