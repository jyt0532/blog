---
layout: post
title: Effective Java Item46 - For-each 優先於傳統的for或while
comments: True 
subtitle: effective java - for-each 優先於傳統的for或while
tags: effectiveJava
author: jyt0532
exerpt: 本篇解釋為何應該多用for-each而不是for或while 還有使用for-each的先決條件
---

這篇是Effective Java - Prefer for-each loops to traditional for loops章節的讀書筆記 本篇的程式碼來自於原書內容

### Item46: For-each 優先於傳統的for或while

Java 1.5版本之前 要loop一個collection 大家這樣做

{% highlight java %}
for (Iterator i = c.iterator(); i.hasNext(); ) {
  doSomething((Element) i.next()); // (No generics before 1.5)
}
{% endhighlight %}

loop數組則是這樣
{% highlight java %}
for (int i = 0; i < a.length; i++) {
  doSomething(a[i]);
}
{% endhighlight %}

這些做法已經比while都好了 但還是太多地方你可能變動到你的索引變量 或是迭代器

所以本書推薦的方式如下

{% highlight java %}
for (Element e : elements) {
  doSomething(e);
}
{% endhighlight %}

冒號代表 對於集合中的每個元素e

不但性能方面一樣(甚至更好) 也不會讓你有機會寫出bug

作者還給出了一個例子 想印出一副牌的每一張

{% highlight java %}
enum Suit { CLUB, DIAMOND, HEART, SPADE }
enum Rank { ACE, DEUCE, THREE, FOUR, FIVE, SIX, SEVEN, EIGHT,
NINE, TEN, JACK, QUEEN, KING }
Collection<Suit> suits = Arrays.asList(Suit.values());
Collection<Rank> ranks = Arrays.asList(Rank.values());
List<Card> deck = new ArrayList<Card>();
for (Iterator<Suit> i = suits.iterator(); i.hasNext(); ){
  for (Iterator<Rank> j = ranks.iterator(); j.hasNext(); ){
    deck.add(new Card(i.next(), j.next()));
  }
}
{% endhighlight %}

原因也很簡單 就是兩個迭代器一起動了 解法是要先暫存外部的那個值

![Alt text]({{ site.url }}/public/item46.png)

長得實在醜 又很容易有bug 如果是用for-each的話
{% highlight java %}
for (Suit suit : suits){
  for (Rank rank : ranks){
    deck.add(new Card(suit, rank));
  }
}
{% endhighlight %}

簡直就是優雅

## Iterable

那對於什麼樣的類別可以用for-each呢 答案是那個類別必須要實作Iterable

{% highlight java %}
public interface Iterable<E> {
  //Returns an iterator over the elements in this iterable
  Iterator<E> iterator()
}
{% endhighlight %}


## 例外

基本上 只要你要循環的這個類別有實作Iterable 你就應該用for-each

1.性能一樣

2.預防bug

3.程式簡潔

但有三種情況無法使用for-each 只能乖乖的用for

1.過濾或刪除: 要刪東西的話 就必須用迭代器(因為你可能需要呼叫到remove())

2.轉換: 就是改變值 for-each只能用來讀取元素

3.平行迭代: 就是剛剛的bug變成feature的時候 當你真的想要內外兩層的迴圈的索引或迭代器一起前進 

