---
layout: post
title: Effective Java Item46 - 優先考慮在流的中間操作中使用無副作用的方法
comments: True 
subtitle: effective java - 謹慎的使用Stream
tags: effectiveJava
author: jyt0532
excerpt: 本篇文章介紹如何靈活在流中使用終結操作
---

這篇是Effective Java - Prefer side-effect-free functions in streams章節的讀書筆記 本篇的程式碼來自於原書內容

本篇的原文內容非常的雜亂 筆者認為是寫得最莫名的一個章節 標題跟內文不太符合 硬啃原文書會非常吃力 筆者以自己的筆記方式 希望能以簡單易讀的方式表達作者想法

## Item46: 優先考慮流中無副作用的函數

當你剛開始學 流 的時候 很難掌握他們 因為你已經習慣以前的思考方式 而不是**函數式編程**

在使用流的時候 最重要的思維方式
是將原本的計算結構轉為一連串的小運算(也就是我們[之前](/2018/11/10/use-streams-judiciously/)說的中間操作)

而每個中間操作 都不應該帶有任何副作用 我感覺這有點像是Rest API中的Idempotent

### 思維方式

要把你想做的事情 化為

源流 -> 中間操作1 -> 中間操作2 -> ... -> 中間操作n -> 終結操作

來看個例子

這段程式計算了一個文件中單詞的頻率

{% highlight java %}
Map<String, Long> freq = new HashMap<>();
  try (Stream<String> words = new Scanner(file).tokens()) {
    words.forEach(word -> {
      freq.merge(word.toLowerCase(), 1L, Long::sum);
    });
}
{% endhighlight %}

忘記merge的用法的可以看[這篇](/2018/08/05/prefer-method-reference-to-lambdas/)

非常好懂 非常直觀 但這卻不是函數式編程的思維 問題在哪 

他長得像 流 但卻又不是流

![Alt text]({{ site.url }}/public/item46-2.jpeg)

他並沒有從流的API得到好處 反而讓程式代碼更加冗長難讀 原因只有一個
那就是這個流使用了ForEach來當終結操作 且在這個終結操作中完成所有事情

換個寫法

{% highlight java %}
Map<String, Long> freq = new HashMap<>();
try (Stream<String> words = new Scanner(file).tokens()) {
  freq = words
    .collect(groupingBy(String::toLowerCase, counting()));
}
{% endhighlight %}

這裡的終結操作就是一個collect直接搞定 要寫好流運算就要能精通collect 這通常是你的終結運算 其中丟給collect的最重要的函式是 toList toSet toMap groupingBy join 等等會一一提到

由這個範例可以明白 **ForEach只適合用於報告流計算的結果 而不是用來執行計算**

### toList

上例子 從頻率表中選出出現頻率最高的十個詞
{% highlight java %}
List<String> topTen = freq.keySet().stream()
  .sorted(comparing(freq::get).reversed())
  .limit(10)
  .collect(toList());
{% endhighlight %}


### toMap

toMap是一個重要的收集器 先看例子

{% highlight java %}
List<String> s = ImmutableList.of("apple", "banana", "cat");
Map<Character, String> m = s.stream().collect(
  Collectors.toMap(
    s1 -> s1.charAt(0), 
    s1 -> s1
  )
);
{% endhighlight %}

toMap如果吃兩個參數的話 toMap的第一個參數是如何把流裡面的元素映射到key  第二個參數是如何把流裡面的元素映射到value 很好懂 這兩個參數都很必要

這個程式跑完後 m長這樣

{a=apple, b=banana, c=cat}

那今天如果stream的元素是("apple", "banana", "cat", "ball") 那就有兩個元素跑到同一個key 
那就會拋出`IllegalStateException` 所以toMap提供了吃三個參數的signature
第三個參數叫做merge function 也就是如果key一樣的話 要如何merge

{% highlight java %}
List<String> s = ImmutableList.of("apple", "banana", "cat");
Map<Character, String> m = s.stream().collect(
  Collectors.toMap(
    s1 -> s1.charAt(0),
    s1 -> s1,
    (s1, s2) -> s1 + " + " + s2
  )
);

{% endhighlight %}

m長這樣
{a=apple, b=banana + ball, o=orange}

如果你想要last-write-win 那就改成(s1, s2) -> s2即可

### groupingBy

就像你所有地方看到的groupBy 他會回傳一個map 按照你要分類的方式分類

{% highlight java %}
List<String> s = ImmutableList.of("apple", "banana", "cat", "pet");
Map<Integer, List<String>> m2 = s.stream().collect(
  Collectors.groupingBy(String::length)
);
{% endhighlight %} 

groupBy只吃一個參數的時候 就是你要怎麼map到key 

m長這樣

{3=[cat, pet], 5=[apple], 6=[banana]}

如果吃兩個參數的話 你還可以再對value作運算

{% highlight java %}
List<String> s = ImmutableList.of("apple", "banana", "cat", "ball");
Map<Integer, Long> m2 = s.stream().collect(
  Collectors.groupingBy(String::length, Collectors.counting()));
{% endhighlight %} 

m長這樣
{3=2, 5=1, 6=1}

### join

就是把一個字串流的字串接起來 你可以給delimiter


{% highlight java %}
List<String> s = ImmutableList.of("apple", "banana", "cat", "ball");
String str = s.stream().collect(Collectors.joining());
{% endhighlight %}

什麼參數都沒給的話 那就是硬接 str = "applebananacatball"

只給一個參數的話 `Collectors.joining("|")` 就是delimiter

str = "apple\|banana\|cat\|ball"

給三個參數的話 Collectors.joining("\|", "$", "^")就是delimer, prefix, suffix

str = "$apple\|banana\|cat\|ball^"

### 總結

這篇的標題跟第一段 講的是中間操作要使用無副作用的函數

後段講的都是如何靈活使用終結操作 

