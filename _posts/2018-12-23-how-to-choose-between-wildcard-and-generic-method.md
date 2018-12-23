---
layout: post
title: 類型參數和通配符的選擇
comments: True 
subtitle: effective java - &lt;?&gt; 和 &lt;T&gt;的差別
tags: effectiveJava
author: jyt0532
excerpt: 本篇文章介紹
---

這篇文章是我在學習[Item31](/2018/12/23/use-bounded-wildcards-to-increase-api-flexibility/)的時候 自己提出來的問題 上網找到解答之後 覺得有必要記錄下來 才產生了這篇文章

### 問題

如同[Item31](/2018/12/23/use-bounded-wildcards-to-increase-api-flexibility/)最後一小章的問題 

這兩個差在哪
{% highlight java %}
public static <E> void swap(List<E> list, int i, int j);

public static void swap(List<?> list, int i, int j);
{% endhighlight %}

還有以下的其他聲明 差在哪
{% highlight java %}
public boolean containsAll(Collection<?> c);

public <T> boolean containsAll(Collection<T> c);
{% endhighlight %}

{% highlight java %}
public boolean addAll(Collection<? extends E> c);

public <T extends E> boolean addAll(Collection<T> c);
{% endhighlight %}

{% highlight java %}
public static <T> void copy(List<T> dest, List<? extends T> src)

public static <T, S extends T> void copy(List<T> dest, List<S> src)
{% endhighlight %}

如果以上的聲明 你都知道差別以及使用時機 那讀者可以跳過這篇文章

### 類型參數和通配符
	
#### 兩者區別與使用時機

1.通配符只能讀取 不能添加

所以要取決於你內部的實作 如果你需要在List裡面加一個東西 就不能用通配符

{% highlight java %}
public static <E> void funct1(final List<E> list1, final E something) {
  list1.add(something);
}
{% endhighlight %}
{% highlight java %}
public static void funct2(final List<?> list, final Object something) {
  list.add(something); // does not compile
}
{% endhighlight %}

記得: 通配符不能用在消費者參數


2.如果你想要在一個聲明裡面表明兩個類別的關係 那就要用類型參數

比如說
{% highlight java %}
public static <T extends Number> void copy(List<T> dest, List<T> src)

{% endhighlight %}

你要明確的表示dest跟src同一個類型 就只能用以上方式 以下方式沒用
{% highlight java %}
public static void copy(List<? extends Number> dest, List<? extends Number> src)

{% endhighlight %}

這樣有可能dest是`List<Integer>`但是src是`List<Double>`

3.你想要**多重限制**一個類型(Multiple bound) 那只能用類型參數

{% highlight java %}
public static <T extends Comparable & Serializable> void copy()
{% endhighlight %}

以下方式不行
{% highlight java %}
public static <? extends Comparable & Serializable> void copy()
{% endhighlight %}

4.你想支持類型**下限**的限制

你用類型參數 你就只能限制上限extends
{% highlight java %}
public <T extends Integer> void genericExtend(List<T> list){}
{% endhighlight %}

不能限制下限
{% highlight java %}
//Compile不過
public <T super Integer> void genericSuper(List<T> list){}
{% endhighlight %}


如果是wildcard 上下限都ok
{% highlight java %}
public void wildCardSuper(List<? super Integer> list){}
public void wildCardExtend(List<? extends Integer> list){}
{% endhighlight %}

5.如果這個類型 在函數裡面也有用到 那就當然只能用類型參數

{% highlight java %}
public void pushAll(Iterable<E> src) {
  for (E e : src)
    push(e);
}
{% endhighlight %}

6.如果一個聲明裡面 類型參數只出現過一次 那就改成通配符 這在[Item31](/2018/12/23/use-bounded-wildcards-to-increase-api-flexibility/)也有提到

### 結論

能用通配符就盡量用通配符 但如果你的函示需要

1.實作中需要寫入(消費者)

2.一個聲明裡面表明兩個類別的關係

3.多重限制(Multiple bound)

4.函數的實作本身 也需要使用到同一個類型

那你就必需要適用類型參數

### 參考資料

[When to use generic methods and when to use wild-card?](https://stackoverflow.com/questions/18176594/when-to-use-generic-methods-and-when-to-use-wild-card)
[Difference between generic type and wildcard type](https://stackoverflow.com/questions/10943137/difference-between-generic-type-and-wildcard-type#)

