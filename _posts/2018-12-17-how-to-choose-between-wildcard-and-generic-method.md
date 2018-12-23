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

{% highlight java %}
public static <E> void swap(List<E> list, int i, int j);

public static void swap(List<?> list, int i, int j);
{% endhighlight %}


### 類型參數和通配符

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

### 總結

正確的使用通配符類型會讓你的API更加靈活 記住基本的規則PECS 以及`Comparable`和`Comparator`都是消費者
PECS
