---
layout: post
title: Effective Java Item27 - 消除非檢查警告
comments: True 
subtitle: effective java - 消除unchecked warnings
tags: effectiveJava
author: jyt0532
excerpt: 本篇文章介紹為什麼應該消除Unchecked warnings
---

這篇是Effective Java - Eliminate unchecked warnings章節的讀書筆記 本篇的程式碼來自於原書內容

本章節可以搭配[泛型篇章簡介及術語列表](/2018/12/01/generics/)服用

## Item27: 消除非檢查警告

當你開始在寫泛型編程 你會收到很多compiler的警告 比如說

| 術語 | 中文 |
| --- | --- |
| Unchecked cast warnings    | 未經檢查的強制轉換警告|
| Unchecked method invocation warnings  | 未經檢查的方法調用警告|
| Unchecked parameterized varag type warnings  | 未經檢查的參數化可變長度類型警告|
| Unchecked conversion warnings  | 未經檢查的轉換警告|

當你寫泛型的程式越多 你看到的警告會越少 

大多數的警告很容易消除 比如說
{% highlight java %}
Set<Lark> exaltation = new HashSet();
{% endhighlight %}
Compiler就會噴警告 
{% highlight txt %}
Venery.java:4: warning: [unchecked] unchecked conversion
  Set<Lark> exaltation = new HashSet();
                 ^
  required: Set<Lark>
  found:  HashSet
{% endhighlight %}

你需要指定類型 或是最起碼加上`<>` 讓編譯器自動推斷類型

{% highlight java %}
Set<Lark> exaltation = new HashSet<>();
{% endhighlight %}

但是很多警告是很難消除的 本篇章提供幾個這種例子 讓你以後可以盡可能消除每一個unchecked warning 只要你沒有unchecked warning 你在運行時 就不會遇到`ClassCastException`

### SuppressWarnings

如果你無法消除警告 但你很確定引發警告的代碼是類型安全的 那你只能用`@SuppressWarnings("unchecked")`註解來抑制警告

如果你在不確定真的安全的情況下就用`@SuppressWarnings("unchecked")` 那在run-time還是有可能拋出`ClassCastException`

你可能會問 那既然我都很確定引發警告的代碼是類型安全的 為什麼還要SuppressWarnings呢 反正在運行時期不會出錯啊 

作者認為如果你選擇忽略 而不是選擇抑制的話 那麼有新的警告出現時 也可能會被你忽略



所以對於每一個非受檢警告 不是消除它 就是抑制它 不要忽略它

### SuppressWarnings使用範圍

SuppressWarnings註解可以用於任何declaration 小到一個變量的宣告 大到整個類別都可以 但你應該使用規模越小越好 最好就在一個變量宣告或是一個短方法使用 如果你在整個類上使用SuppressWarnings 很可能會忽略一些重要的警告

看個例子
{% highlight java %}
public <T> T[] toArray(T[] a) {
  if (a.length < size)
     return (T[]) Arrays.copyOf(elements, size, a.getClass());
  System.arraycopy(elements, 0, a, 0, size);
  if (a.length > size)
     a[size] = null;
  return a;
}
{% endhighlight %}
這個方法會產生警告

{% highlight txt %}
ArrayList.java:305: warning: [unchecked] unchecked cast
  return (T[]) Arrays.copyOf(elements, size, a.getClass());
                 ^
  required: T[]
  found:  Object[]
{% endhighlight %}

在Intellij長這樣
![Alt text]({{ site.url }}/public/unchecked_warning.png)

就是說 這個`Object[]`要強制轉換到`T[]` 編譯器覺得有風險 但既然我們知道這沒問題 我們就可以加上SuppressWarnings
可惜的是 我們不能在return statement加SuppressWarnings 我們也不希望整個方法都加上SuppressWarnings 所以只好多宣告一行
{% highlight java %}
public <T> T[] toArray(T[] a) {
  if (a.length < size) {
    // This cast is correct because the array we're creating
    // is of the same type as the one passed in, which is T[].
    @SuppressWarnings("unchecked") T[] result =
      (T[]) Arrays.copyOf(elements, size, a.getClass());
    return result;
  }
  System.arraycopy(elements, 0, a, 0, size);
  if (a.length > size)
    a[size] = null;
  return a;
}
{% endhighlight %}

雖然多了一行 但我們把SuppressWarnings的範圍最小化！

### 註解 

當你使用SuppressWarnings的時候 請你加上註釋 說明為什麼這可以被抑制 這會有助於別人了解(通常你的同事不要太差的話 你的code review中使用SuppressWarnings他都應該問你為什麼 而理由也通常不會顯而易見 所以還是乖乖上註釋)

## 總結

未經檢查的警告很重要 不要忽略他們 每個未經檢查的警告代表在運行時出現ClassCastException異常的可能性
你要盡你所能消除這些警告 

如果你無法消除 但你可以證明這個警告是類型安全的 你可以在盡可能小的範圍中 加上`SuppressWarnings("unchecked")`註解來抑制該警告 並加上註釋



