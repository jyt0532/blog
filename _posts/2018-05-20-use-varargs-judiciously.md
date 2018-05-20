---
layout: post
title: Effective Java Item53 - 慎用可變參數
comments: True 
subtitle: effective java - 慎用可變參數
tags: effectiveJava
author: jyt0532
excerpt: 本篇文章介紹如何使用可變參數以及性能問題
---

這篇是Effective Java - Use varargs Judiciously章節的讀書筆記 本篇的程式碼來自於原書內容


## Item53: 慎用可變參數

可變參數是java 1.5引進的方法 它可以讓你的方法吃進任意數量(0至多個)的同類型的變數

{% highlight java %}
static int sum(int... args) {
   int sum = 0;
   for (int arg : args)
      sum += arg;
   return sum;
}
{% endhighlight %}

但有的時候 你的方法想接受一個以上的參數 而不是零個以上怎麼辦呢

也許你可以在方法的一開始檢查參數個數

{% highlight java %}
// The WRONG way to use varargs to pass one or more arguments!
static int min(int... args) {
  if (args.length == 0)
    throw new IllegalArgumentException("Too few arguments");
  int min = args[0];
  for (int i = 1; i < args.length; i++)
  if (args[i] < min)
    min = args[i];
  return min;
}
{% endhighlight %}

這個解法有兩個問題

如果客戶真的沒丟參數進去 你的方法無法在編譯期被偵測 只能在運行期被動的丟例外

第二個問題 很不美觀 除非將min初始化為Integer.MAX_VALUE 否則無法使用for-each

比較好的方法 就是先寫死一個 之後再用可變參數

{% highlight java %}
static int min(int firstArg, int... remainingArgs) {
   int min = firstArg;
   for (int arg : remainingArgs)
      if (arg < min)
         min = arg;
   return min;
}
{% endhighlight %}

這樣就從0個以上 變成 1個以上

其實java把可變參數引進來 最大的用途是為了反射 想了解更多反射的細節 [請關注一本2019年出版即將改變java生態的書](https://www.jyt0532.com/toc/jvm/) 這裡就先不談反射


## 性能

可變參數雖然好用 但性能卻很差 每次可變參數的方法被呼叫 都需要配置一個數組並且對每個元素初始化 身為專業的程序員 要能夠了解你的use case進而最佳化

假設95%的use case都是只有四個參數以下 你可以單獨宣告這四個方法

{% highlight java %}
public void foo() { }
public void foo(int a1) { }
public void foo(int a1, int a2) { }
public void foo(int a1, int a2, int a3) { }
public void foo(int a1, int a2, int a3, int... rest) { }
{% endhighlight %}


## 總結

對於參數數目不定的方法 可以使用可變參數 但是性能不佳 可以的話做一些最佳化






