---
layout: post
title: Effective Java Item44 - 優先使用標準的函數式接口
comments: True 
subtitle: effective java - 優先使用標準的函數式接口
tags: effectiveJava
author: jyt0532
excerpt: 本篇文章介紹為何要使用標準的函數式接口 以及例外時機
---

這篇是Effective Java - Favor the use of standard functional interfaces章節的讀書筆記 本篇的程式碼來自於原書內容

## Item44: 優先使用標準的函數式接口

[Item42](/2018/08/05/prefer-lambdas-to-anonymous-classes/)介紹了Java的lamdba表達式 從此以後API的編寫產生了很大的變化 記不記得小時候我們在講[Template Pattern](/2017/09/12/template/)的時候 我們覆寫了父類別的方法 為了讓我的子類別對於某些方法有專門的處理方式 這個方法已經不那麼吸引人了

現在常用的作法 是一個可以接受一個函數對象的[靜態工廠](/2017/09/20/static-factory-method/)或是構造方法 也可以達到一樣的效果

### LinkedHashMap

來看一下可愛的LinkedHashMap 裡面有一個方法叫做removeEldestEntry 當這個方法回傳true時 這個map裡面最久遠以前被insert的會被移除

預設的實作是這樣
{% highlight java %}
protected boolean removeEldestEntry(Map.Entry eldest){
    return size() > MAX_SIZE;
}
{% endhighlight %}

這個方法在每次一有entry被insert的時候就會執行 如果回傳true的話就把最久遠以前的insert移除 有這個函式把關 一個map裡面的entry數目就不會超過MAX_SIZE

但你可以複寫它

{% highlight java %}
protected boolean removeEldestEntry(Map.Entry eldest){
    return size() > 100;
}
{% endhighlight %}

那你的map裡面就只能保存最近的100個entry 很多人會把這個用來當cache使用

### 用lambda試試

如果LinkedHashMap現在需要重新實作的話 那對於removeEldestEntry這種簡單的函式 我們會在LinkedHashMap提供一個靜態工廠 或是 構造方法來接受一個函數對象

小複習: 定義**一個函數**的介面或是抽象類別 稱為函數類型(function types) 而函數類型的實例稱為函數對象(funcitonal objects)

那你覺得在removeEldestEntry的例子裡 函數對象相對應的函數類型裡面的那個唯一那個函數應該長什麼樣呢

你可能覺得應該要跟removeEldestEntry一樣 吃一個entry 返回一個boolean但其實不對 因為removeEldestEntry是Map的實例方法(instance method) 你可以存取到Map的資訊 比如說size

可是傳遞給構造方法的函數對象不是map上的實例方法 所以你必須把整個map丟進去

{% highlight java %}
// Unnecessary functional interface; use a standard one instead.
@FunctionalInterface interface EldestEntryRemovalFunction<K,V>{
    boolean remove(Map<K,V> map, Map.Entry<K,V> eldest);
}
{% endhighlight %}

這種寫法可行 但是其實java.util.function包裡面提供了大量標準函數式接口 你應該要去了解目前有了什麼
再看需不需要定義自己的函數式接口 這也是這篇文章的主題

java.util.Function中有43個接口 但你只需要記住其中的六個 其他的都是這六個的變形

| 接口 | 方法 | 範例  |
| --- | --- | --- |
| UnaryOperator      |T apply(T t) |String::toLowerCase |
| BinaryOperator      |T apply(T t1, T t2)      |BigInteger::add |
| Predicate | boolean test(T t)      |Collection::isEmpty |
| Function<T,R> | R apply(T t)      |Arrays::asList |
| Supplier | T get()      |Instant::now |
| Consumer | void accept(T t)      |System.out::println |

其實也很好理解 Operator就是輸出輸入一樣型態 畢竟你只是在operate

Predicate 接受一個任意型態 返回boolean

Function接受A返回B

Supplier就是不接受東西 只返回東西

Consumer就是只接受東西 不返回東西

### 變形

建立在這六個之上的變形 基本上就是對於常用的primitive type各自定義

比如說 LongBinaryOperator 就是輸入一個long 輸出一個long

LongToIntFunction 就是輸入一個long 輸出一個int

BiFunction <T，U，R> 就是輸入A和B 輸出C

建議可以到[這裡](https://docs.oracle.com/javase/8/docs/api/java/util/function/package-summary.html)看一下現在到底提供了什麼 先有個底子

### 還是有特例

但也不是每次只要有得用就用 還是要看情況

比如說大家都很熟的Comparator <T> 其實也是可以用這個函數接口ToIntBiFunction <T, T> 但還是應該乖乖用Comparator <T> 理由如下

1.它的名稱每次在API中使用時都會提供優秀的文檔 且會被廣泛使用

2.強大的契約

3.有很多default的方法可以用

所以只要理由夠強 你就可以用自己定義的專用函數式接口

### 寫自定義函數式接口的注意事項

始終使用@FunctionalInterface註解標註你的函數式接口

這個東西就像Override一樣 清楚的告訴了讀者你的企圖是為了實現lambda而設計 就不會有人之後去裡面再加一個方法

### removeEldestEntry

所以在這篇文章的removeEldestEntry例子 你應該用這個BiPredicate<Map<K,V>, Map.Entry<K,V>>

### 總結


現在你已經是lambda高手 你在設計API的時候都要考慮使用lambda表達式 也就是在輸入型態或是輸出型態 都可以考慮使用標準函數式接口 比如說

{% highlight java %}
void eval(List<Integer> list, Predicate<Integer> predicate);
{% endhighlight %}

{% highlight java %}
BiConsumer<T,U> toDefaultBiConsumer();
{% endhighlight %}
