---
layout: post
title: Effective Java Item43 - 返回零長度的數組或集合 而不是null
comments: True 
subtitle: effective java - 返回零長度的數組或集合 而不是null
tags: effectiveJava 
author: jyt0532
---
這篇是Effective Java - Return empty Arrays or Collections, not nulls章節的讀書筆記 
本篇的程式碼來自於原書內容

## Item43: 返回零長度的數組或集合 而不是null

蠻常有人用null來處理empty的case
{% highlight java %}
//return an array of cheese, or null if no cheese are available
public Cheese[] getCheese() {
  if(cheesesInStock.size() == 0)
    return null;
}
{% endhighlight %}

乍看之下合理 但其實Client需要多做很多不必要的處理

你必須要寫成這樣
{% highlight java %}
Cheese[] cheeses = shop.getCheese();
if(cheeses != null && 
  Arrays.asList(cheeses).contains(Cheese.STILTON))
  System.out.println("ya");
{% endhighlight %}

而不是這樣
{% highlight java %}
Cheese[] cheeses = shop.getCheese();
if(Arrays.asList(cheeses).contains(Cheese.STILTON))
  System.out.println("ya");
{% endhighlight %}

幾乎每一次call你的函式 就必須要特別處理null的情況 這樣很容易出錯 因為很容易忘記要這麼處理

也許有人說這樣可以省點不必要的開銷 如果沒東西 我根本就不需要allocate一個空的數組 直接傳個所有人都看的懂的null搞定 

但這個論點是站不著腳的 因為

1.在這種地方為了微小的performance而影響client使用的方法是不划算的

2.事實上 因為長度是零的數組可以被共享 Collection.emptySet, Collection.emptyList, Collection.emptyMap 
這些函示可以給你不可變的空集合

{% highlight java %}
//The right way to return a copy of a collection
public List<Cheese> getCheeseList() {
  if(cheesesInStock.isEmpty())
    return Collections.emptyList();
  else
    return new ArrayList<Cheese>(cheesesInStock);
}
{% endhighlight %}




### 總結

返回類型是數組或是集合的話 沒有理由返回null 這個習慣應該是從C傳過來的

