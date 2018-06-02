---
layout: post
title: Effective Java Item52 - 慎用重載
comments: True 
subtitle: effective java - 慎用重載
tags: effectiveJava
author: jyt0532
excerpt: 本篇文章介紹如何XXXXXXXXXXXXXXXXXXXXXXXX
---

這篇是Effective Java - Use overloading Judiciously章節的讀書筆記 本篇的程式碼來自於原書內容


## Item52: 慎用重載

### 重載(overload)
來看一下這段程式

{% highlight java %}
public static String classify(Set<?> s) {
  return "Set";
}

public static String classify(List<?> lst) {
  return "List";
}

public static String classify(Collection<?> c) {
  return "Unknown Collection";
}

public static void main(String[] args) {
  Collection<?>[] collections = { 
    new HashSet<String>(),
    new ArrayList<BigInteger>(),
    new HashMap<String, String>().values() };

  for (Collection<?> c : collections)
    System.out.println(classify(c));
}
{% endhighlight %}

這段程式希望根據一個集合是Set還是List還是其他 來進行分類

你可能預期他會印出 

{% highlight java %}
Set
List
Unknown Collection
{% endhighlight %}

但其實他會印出
{% highlight java %}
Unknown Collection
Unknown Collection
Unknown Collection
{% endhighlight %}

為什麼呢

我們重載了classify這個方法 classify的哪一個方法會被調用 是在**compile-time**就決定了

在編譯期 compiler檢查collections的每個元素都是Collection<?> 代表每個collections都有一個可以合法呼叫的classify 也就是classify(Collection<?> c)

### 覆蓋(override)

現在來看看覆蓋(override)的例子


{% highlight java %}
class Wine {
  String name() {
    return "wine";
  }
}
class SparklingWine extends Wine {
  @Override
  String name() {
    return "sparkling wine";
  }
}
class Champagne extends SparklingWine {
  @Override
  String name() {
    return "champagne";
  }
}
public class Overriding {
  public static void main(String[] args) {
    Wine[] wines = { new Wine(), new SparklingWine(), new Champagne() };
    for (Wine wine : wines)
      System.out.println(wine.name());
  }
}
{% endhighlight %}

印出是這樣
{% highlight java %}
wine
sparkling wine
champagne
{% endhighlight %}

差別是什麼呢 因為對於覆蓋 要用哪個方法的選擇是動態的 運行時決定

當new Champagne().name() 他會去看 Champagne 類別有沒有 name 這個方法 有的話就直接執行 沒有的話 去看 Champagne 的父類有沒有 一路往上找

### 那該怎麼辦

既然overload在編譯時期就得決定用哪個方法 那就只能用顯性的instanceof
{% highlight java %}
public static String classify(Collection<?> c) {
  return c instanceof Set ? "Set" :
    c instanceof List ? "List" : "Unknown Collection";
}
{% endhighlight %}


### 覆蓋是規範 重載是例外

覆蓋的使用 完全符合使用者的預期 我們就是因為希望子類別有不同於父類別的實作方式 所以我們覆蓋

但重載 卻是叫你的程式去選一個方法去執行 這就很容易會產生與預期不同的結果

如果你寫出來的程式行為會使程序員感到困惑 那就代表你的程式寫的很差 重載就是一個很常見的例子 而且通常都會等到運行時才發現錯誤

### 謹慎使用重載

什麼是謹慎的定義呢 就是

**永遠不要導出兩個具有相同參數數目的重載方法**

不應該讓程序員去猜哪一個才會被呼叫到 如果真的需要不同類型的input參數 就取另外一個方法名稱就好


比如說ObjectOutputStream類別 對於每一種可能的寫入類型 都導出了一個相對應的方法

writeBoolean(boolean), writeInt(int), writeLong(long)等等 這種方式對於使用者來說 根本不可能用錯

也許你會說 對於構造器來說 你不能選擇方法名稱怎麼辦 

身為我部落格的多年讀者 你一定回答得出來我們除了構造器的另一個選擇 又可以自己定義方法名稱 又不會影響到原本的構造器 給你三秒鐘

3

2

1

答對了 就是[靜態工廠](/2017/09/20/static-factory-method/)

### 總結

**能夠重載 並不代表應該重載** 一般來說 最好可以避免相同參數數目的重載方法

如果以上無法避免 起碼避免其中一種重載方法的輸入型態是另一種的子類

如果以上無法避免 起碼讓這兩個重載方法做到一樣的結果(這樣程序員就不需要了解到底哪個方法被呼叫了 只要結果對就好)

如果以上無法避免 那之後別人在maintain你的程式時遇到困難 <s>就不要怪我囉</s> 是非常合理的
