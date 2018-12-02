---
layout: post
title: Effective Java Item26 - 不要使用原始類型
comments: True 
subtitle: effective java - 不要使用raw type
tags: effectiveJava
author: jyt0532
excerpt: 本篇文章介紹
---

這篇是Effective Java - Don't use raw types章節的讀書筆記 本篇的程式碼來自於原書內容

本章節可以搭配[泛型篇章簡介及術語列表](/2018/12/01/generics/)服用
## Item26: 不要使用原始類型

先定義幾個術語:

一個類或接口 如果聲明裡面有一個或多個類型參數(type parameters) 稱之為泛型類或泛型接口
{% highlight java %}
List<E>
{% endhighlight %}

泛型類和泛型接口統稱泛型類型(generic type)

每個泛型也都定義了原始類型(raw type) 他代表沒有類型參數的泛型 比如說相對於`List<E>`的原始類型就是`List` 就等於是把所有泛型類型的訊息從聲明中拿掉

來看看原始類型的用法 這是Java5以前常見的用法

{% highlight java %}
private final Collection stamps = ... ;
{% endhighlight %}

從Java9開始 雖然還是合法 但已經不推薦這麼用了 為什麼呢?

今天有人如果放了一個錯的東西進去

{% highlight java %}
stamps.add(new Coin( ... ));
{% endhighlight %}

編譯時不會錯 運行時也不會錯 只有在你在運行時需要讀這個collection

{% highlight java %}
for (Iterator i = stamps.iterator(); i.hasNext(); )
    Stamp stamp = (Stamp) i.next(); // Throws ClassCastException
{% endhighlight %}

直到此時才會拋出例外

這本書看到這裡 你應該要有sense 一個錯誤能越早被發越越好 最理想就是在compile time就能發現錯誤 不然在你看到錯誤的時候 不知道已經離錯誤多遠 像這個例子 你就要去找所有曾經呼叫過stamps.add的地方 看哪一個出錯

如果你用的是泛型

{% highlight java %}
private final Collection<Stamp> stamps = ... ;
{% endhighlight %}

這樣compiler就知道stamps裡只能有Stamp 那在你丟coin進去的時候 compile-time就會報錯

{% highlight txt %}
Test.java:9: error: incompatible types: Coin cannot be converted
to Stamp
    c.add(new Coin());
              ^
{% endhighlight %}

### 兼容性

既然缺點這麼顯而易見 為什麼還要支持原始類型 答案是為了兼容Java5之前的程式

總不能在泛型被加入Java時 之前的程式都強制compile-error 為了支援Java5之前的程式和新代碼交互操作是非常重要的

### 小撇步

我們知道了不應該使用原始類型`List` 但如果要使用參數化的類型插入任意對象 還是可以用`List<Object>` 這兩者的差別在哪呢？ 

`List` 逃避了泛型檢查 `List<Object>`則是明確的告訴compiler説 他能持有任意類型的對象 

順道一提 你可以將`List<String>`傳給`Lis`t參數 但你卻不能傳給`List<Object>` 因為`List<String>`是`List`的子類型

來看例子


{% highlight java %}
public static void main(String[] args) {
  List<String> strings = new ArrayList<>();
  unsafeAdd(strings, Integer.valueOf(42));
  String s = strings.get(0); // Has compiler-generated cast
}

private static void unsafeAdd(List list, Object o) {
  list.add(o);
}
{% endhighlight %}

unsafeAdd使用了原始類型 編譯會過 雖然會丟出警告
{% highlight txt %}
Test.java:10: warning: [unchecked] unchecked call to add(E) as a
member of the raw type List
    list.add(o);
            ^
{% endhighlight %}

但在Run-time 會在`strings.get(0)`的地方拋出例外

那如果你在unsafeAdd中使用的是`List<Object>` 那在編譯時期就會噴錯

{% highlight txt %}
Test.java:5: error: incompatible types: List<String> cannot be converted to List<Object>
    unsafeAdd(strings, Integer.valueOf(42));
{% endhighlight %}

第二個例子 你也許會在不知道集合中的元素類型的情況下 使用原始類型

假設你想寫一個方法 從兩個集合中回傳他們共有的元素的數量


{% highlight java %}
static int numElementsInCommon(Set s1, Set s2) {
  int result = 0;
  for (Object o1 : s1)
    if (s2.contains(o1))
      result++;
  return result;
}
{% endhighlight %}

這個寫法會對 但卻很危險 比較安全一點的方法是使用無限制通配符類型(unbounded wildcard types) 從今爾後 如果要使用泛型類型 
但不知道或不關心實際類型參數是什麼 那你可以使用問號來代替 變成List<?> 

{% highlight java %}
static int numElementsInCommon(Set<?> s1, Set<?> s2) {
  int result = 0;
  for (Object o1 : s1)
    if (s2.contains(o1))
      result++;
  return result;
}
{% endhighlight %}


### 例外

有規則必有例外 我們來看看什麼時候應該要用原始類型

1.類文字(class literals): `List.class`, `String[].class`,  `int.class` 都合法 但是`List<String>.class`, `List<?>.class`都不合法

2.instanceof: 當你要使用instanceof 你不能這樣寫

{% highlight java %}
if (obj instanceof List<String>)
{% endhighlight %}

你必須要用無限制通配符類型

{% highlight java %}
if (obj instanceof List<?>)
{% endhighlight %}

但是在運行時期 無限制通配符類型並不會影響到instanceof的結果 所以你可以直接用原始類型

{% highlight java %}
if (obj instanceof List){
  List<?> l = (List<?>) obj;
}
{% endhighlight %}

一但你確定obj對象是一個List 必須將它轉成通配符List<?>

### 結論

使用原始類型可能導致運行時異常 並且在編譯時期不會發現錯誤 所以不要使用它們

List<Object>是一個參數化類型 表示一個可以包含任何類型對象的List

List<?> 是一個通配符類型 表示一個只能包含某些未知類型對象的List

List是一個原始類型 它不在泛型類型系統之列 能不用就盡量不要用
