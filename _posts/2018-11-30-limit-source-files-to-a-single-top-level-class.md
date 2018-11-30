---
layout: post
title: Effective Java Item25 - 讓每個java文件只有一個top-level類別 
comments: True 
subtitle: effective java - 讓每個source file只有一個top-level類別
tags: effectiveJava
author: jyt0532
excerpt: 本篇文章介紹為什麼要讓每個java文件只有一個top-level類
---

這篇是Effective Java - Limit source files to a single top-level class章節的讀書筆記 本篇的程式碼來自於原書內容

## Item25: 讓每個java文件只有一個top-level類別

雖然java檔案允許你在一個檔案裡面定義很多top-level class 但這樣做沒有任何好處 而且風險挺高

舉個例子

{% highlight java %}
//Main.java
public class Main {
  public static void main(String[] args) {
  System.out.println(Utensil.NAME + Dessert.NAME);
  }
}
{% endhighlight %}
{% highlight java %}
//Utensil.java
class Utensil {
  static final String NAME = "pan";
}
class Dessert {
  static final String NAME = "cake";
}
{% endhighlight %}

結果顯而易見 你執行Main的話會印出`pancake`

但今天如果你同事定義了一個新的檔案


{% highlight java %}
//Dessert.java
class Utensil {
  static final String NAME = "pot";
}
class Dessert {
  static final String NAME = "pie";
}
{% endhighlight %}

這樣會如何呢? 這樣就有趣了 不同的compile順序會造成不同的執行結果！

如果你這麼編譯
{% highlight bach %}
> javac Main.java
{% endhighlight %}
或是這麼編譯
{% highlight bach %}
> javac Main.java Utensil.java
{% endhighlight %}

那還是會印出`pancake` 因為當Main一開始看到Utensil 他就會去找Utensil.java 然後找到兩個常數

如果你這麼編譯
{% highlight bach %}
> javac Main.java Dessert.java
{% endhighlight %}

那會編譯錯誤 因為在編譯Main的時候 他先看到Utensil 然後他就去找了Utensil.java這個檔案 找到了之後就把兩個常數讀進JVM 然後要繼續編譯Dessert.java的時候就發現了重複定義的常數

如果你這麼編譯
{% highlight bach %}
> javac Dessert.java Main.java
{% endhighlight %}

精彩了 這會印出`potpie` 因為在編譯Dessert.java的時候 就把那兩個常數讀完了 在編譯Main.java的時候也都看得懂 沒有人會去加載 Utensil.java

### 解決方法

例子講完了 解法也簡單 **每個Class放不同檔案** 如果你真的要放同一個檔案 就用[靜態成員類別](/2018/08/04/favor-static-member-class-over-nonstatic/)

{% highlight java %}
public class Test {
  public static void main(String[] args) {
    System.out.println(Utensil.NAME + Dessert.NAME);
  }
  private static class Utensil {
    static final String NAME = "pan";
  }
  private static class Dessert {
    static final String NAME = "cake";
  }
}
{% endhighlight %}


### 總結

永遠不要將多個頂級類放在同一個源文件中 當你遵循這個規則 你的程式的執行結果就不會隨著你的編譯順序改變而改變







