---
layout: post
title: Effective Java Item31(補充) - 類型參數和通配符的選擇
comments: True 
subtitle: effective java - 利用bounded wildcards來提昇API靈活性
tags: effectiveJava
author: jyt0532
excerpt: 本篇文章介紹
---

這篇文章是我在學習[Item31]的時候 自己提出來的問題 上網找到解答之後 覺得有必要記錄下來 才產生了這篇文章

### 問題

如同[Item31]最後一小章的問題 

這兩個差在哪
{% highlight java %}
public static <E> void swap(List<E> list, int i, int j);

public static void swap(List<?> list, int i, int j);
{% endhighlight %}

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
	
#### 兩者區別

通配符只能讀取 不能添加

所以要取決於你內部的實作 如果你需要在List裡面加一個東西

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

通配符不能用在消費者參數
#### 使用時機

再來是兩者的使用時機 如果

1.你想要在一個聲明裡面表明兩個類別的關係 那就要用類型參數

比如說
{% highlight java %}
public static <T extends Number> void copy(List<T> dest, List<T> src)

{% endhighlight %}

你要明確的表示dest跟src同一個類型 就只能用以上方式 以下方式沒用
{% highlight java %}
public static void copy(List<? extends Number> dest, List<? extends Number> src)

{% endhighlight %}

這樣有可能dest是List<Integer>但是src是List<Double>

2.你想要**多重限制**一個類型(Multiple bound) 那只能用類型參數

{% highlight java %}
public static <T extends Comparable & Serializable> void copy()
{% endhighlight %}

以下方式不行
{% highlight java %}
public static <? extends Comparable & Serializable> void copy()
{% endhighlight %}

3.你想支持類型**下限**的限制

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



@#@#@@@#@#

類型參數`T` 和通配符`?`具有雙重性 請看下面兩種聲明 第一個是無限制類型參數 第二個是無限制通配符

{% highlight java %}
public static <E> void swap(List<E> list, int i, int j);

public static void swap(List<?> list, int i, int j);
{% endhighlight %}

請注意`?`指的其實是`? extends Object` 所以第二個你可以看成

`public static void swap(List<? extends Object> list, int i, int j);`

那這兩個哪個比較好呢 如果是你要提供公用的API 那第二個好一點 非常好懂 就是把一個List的兩個index交換

通常來說**如果一個類型參數聲明中 類型參數只出現一次 那就把它換成通配符聲明** 這句話對於不論限制類型還是無限制類型都一樣有效

但是我們認為好懂的聲明 卻無法編譯以下簡單的實作

{% highlight java %}
public static void swap(List<?> list, int i, int j) {
    list.set(i, list.set(j, list.get(i)));
}
{% endhighlight %}

聲明雖然看起來好用 但實作上卻是絆手絆腳 你必須要用當初的第一個我們不喜歡的聲明來實作

{% highlight java %}
public static void swap(List<?> list, int i, int j) {
    swapHelper(list, i, j);
}

// Private helper method for wildcard capture
private static <E> void swapHelper(List<E> list, int i, int j) {
    list.set(i, list.set(j, list.get(i)));
}
{% endhighlight %}

搞什麼飛機 你選擇了wildcard當作公開API 但內部卻用generic type來實作 為什麼當初不直接用generic type當作公眾API呢

這時候就可以移駕到這篇文章[類型參數和通配符的選擇](/)

### 參考資料

[](https://stackoverflow.com/questions/18176594/when-to-use-generic-methods-and-when-to-use-wild-card)
[](https://stackoverflow.com/questions/10943137/difference-between-generic-type-and-wildcard-type#)

正確的使用通配符類型會讓你的API更加靈活 記住基本的規則PECS 以及`Comparable`和`Comparator`都是消費者
PECS
