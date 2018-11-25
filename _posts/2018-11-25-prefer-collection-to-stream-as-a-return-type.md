---
layout: post
title: Effective Java Item47 - 優先使用Collection而不是Stream來作為函數的回傳類型
comments: True 
subtitle: effective java - 優先使用Collection作為為函數的回傳類型
tags: effectiveJava
author: jyt0532
excerpt: 本篇文章介紹用Collection當作函數回傳類型的好處
---

這篇是Effective Java - Prefer collection to stream as a return type章節的讀書筆記 本篇的程式碼來自於原書內容

## Item47: 優先使用Collection而不是Stream 來作為函數的回傳類型

很多函式的回傳值是一系列的元素 壁如Set, List等等 你甚至在定義函式的時候不用寫死 給個Collection就搞定 

{% highlight java %}
Collection<String> getCollection() {
  return ImmutableList.of("apple", "banana", "cat");
}
{% endhighlight %}

如果你預期你函式的回傳值 應該要被使用者跑for-loop 你可以就讓你的回傳類型為Iterable

{% highlight java %}
Iterable<String> getIterable() {
  return ImmutableList.of("apple", "banana", "cat");;
}
{% endhighlight %}

如果你很不乖 很不聽話 不想用終結操作把你的stream變成Collection 你就是想要讓你的函式回傳一個Stream

{% highlight java %}
Stream<String> getStream() {
  return ImmutableList.of("apple", "banana", "cat").stream();
}
{% endhighlight %}

會有什麼後果呢 就是使用你函數的人如果想要loop你的回傳值 他必須強制轉型
{% highlight java %}
for (String a: (Iterable<String>)getStream()::iterator){
  System.out.println(a);
}
{% endhighlight %}

簡直醜到不能再醜了 想個別的辦法吧

余憶童稚時 能閉者眼睛寫出[Adaptor](/2017/07/14/adaptor/)
{% highlight java %}
public static <E> Iterable<E> iterableOf(Stream<E> stream) {
  return stream::iterator;
}
{% endhighlight %}

有了Adaptor後 人生真輕鬆

{% highlight java %}
for (String a: iterableOf(getStream())){
  System.out.println(a);
}
{% endhighlight %}

太乾淨漂亮了 這就是為什麼[Design Pattern](/toc/design_pattern/)要好好學

別忘了Adaptor也可以是雙向的

{% highlight java %}
public static <E> Stream<E> streamOf(Iterable<E> iterable) {
  return StreamSupport.stream(iterable.spliterator(), false);
}
{% endhighlight %}

### 不如回傳Collection吧

當你相信大部分的使用者都會loop你的回傳值 你可以回傳Iterable 讓少數人自己寫Adaptor去做流運算

又或當你相信大部分的使用者都會繼續做Stream的運算 你可以回傳Stream 讓少數人自己寫Adaptor去做for-loop

可是大部分的情況不是簡單的二分法 要如何從中取捨呢 作者的答案是回傳Collection<T> 或是適當的子類型比如說Set<T>或是List<T>

原因很簡單 

1.Collection具有stream()方法 可以快速的轉成Stream 

2.Collection接口是Iterable的子類型

兩邊都不得罪 兩邊討好 八面玲瓏

## 總結 

這篇後段都是不太相關的廢話 這個篇章(Lambdas and Streams)是第三版中新出現的章節 很多小節感覺都很雜亂 講了很多跟主題不相關的廢話 不要花太多時間讀懂全部的東西 看這個部落格就可以

