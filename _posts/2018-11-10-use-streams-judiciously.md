---
layout: post
title: Effective Java Item45 - 謹慎的使用Stream
comments: True 
subtitle: effective java - 謹慎的使用Stream
tags: effectiveJava
author: jyt0532
excerpt: 本篇文章介紹流運算 以及如何在流運算和迭代運算中取得平衡
---

這篇是Effective Java - Use streams judiciously章節的讀書筆記 本篇的程式碼來自於原書內容

## Item45: 謹慎的使用Stream(流)

在Java8之後 開始支援Stream的API 目的是簡化批量運行的任務

Stream API提供兩個關鍵的抽象: Stream 和 Stream pipeline

Stream(流): 包含一系列元素 常見的來源是集合 數組 文件 常見的數據元素可以是對象引用或是基本類型

Stream Pipeline(流管道): 流管道包含了一個流 搭配一系列的中間操作 然後再加上一個終結操作 

中間操作: 以某種方式轉換流 可能是映射(map) 可能是過濾(filter) 可能是排序(sorted)等等

終結操作: 對於所有中間操作所產生的最終運算流 存到集合中(collect)或是印出所有元素(forEach) 或是藉由流中所有元素計算出一個新值(reduce)

上例子

{% highlight java %}
List<Integer> number = Arrays.asList(1,2,3,4);
List<Integer> square = 
number.stream()
  .map(x -> x*x)
  .collect(Collectors.toList()); 
{% endhighlight %}

這樣square裡面就是[1,4,9,16] 

你也可以多加一層中間運算

{% highlight java %}
List<Integer> squareOver10 = 
number.stream()
  .map(x -> x*x)
  .filter(n -> n > 10)
  .collect(Collectors.toList());
{% endhighlight %}

這樣squareOver19裡面就是[16] 

就是這麼好懂直觀 Stream API 就是有足夠的通用性 幾乎所有的運算都可以用流運算來完成 

### 是否該用流運算的取捨

可以使用流 不代表永遠應該使用

上書中的例子 Anagram指的是兩個單字 是同樣的英文字母但不同順序組合而成

比如說tar跟rat是anagram, night跟thing也是anagram 

以下的程式 目的是要把字典裡的所有單字且是anagram的放進一個map, key是這些anagram裡面照字母排序中最小的 如果這組anagram的數目超過minGroupSize 就全印出來

比如說字典是這樣[auctioned, cautioned, education, state, taste, cat] 且minGroupSize=1

就會印這樣

1: [cat]

2: [state, taste]

3: [auctioned, cautioned, education]

如果minGroupSize=2

就會印這樣

2: [state, taste]

3: [auctioned, cautioned, education]

上程式
{% highlight java %}
public class Anagrams {
  public static void main(String[] args) throws IOException {
    List<String> dictionary = Arrays.asList("auctioned", "cautioned", "education", "state", "taste", "cat");
    int minGroupSize = 1;
    Map<String, Set<String>> groups = new HashMap<>();
    for (int i = 0; i < dictionary.size(); i++) {
        String word = dictionary.get(i);
        groups.computeIfAbsent(alphabetize(word), (unused) -> new TreeSet<>())
          .add(word);
    }
    for (Set<String> group : groups.values())
      if (group.size() >= minGroupSize)
        System.out.println(group.size() + ": " + group);
  }
  private static String alphabetize(String s) {
    char[] a = s.toCharArray();
    Arrays.sort(a);
    return new String(a);
  }
}
{% endhighlight %}

算好理解 但要特別講一下computeIfAbsent

computeIfAbsent吃兩個參數 第一個是key 第二個是函數式接口

當key已經存在map了的話 就回傳value 如果key不存在map裡 你的函數式接口的函數就要回傳value的初始值

ok看懂了 現在用stream來寫寫看


{% highlight java %}
public class Anagrams {
  public static void main(String[] args) {
    List<String> dictionary = Arrays.asList("auctioned", "cautioned", "education", "state", "taste", "cat");
    int minGroupSize = 1;
    dictionary.stream().collect(
        groupingBy(word -> word.chars().sorted()
          .collect(StringBuilder::new,
            (sb, c) -> sb.append((char) c),
            StringBuilder::append).toString()))
        .values().stream()
        .filter(group -> group.size() >= minGroupSize)
        .map(group -> group.size() + ": " + group)
        .forEach(System.out::println);
    }
}
{% endhighlight %}

這就是硬要用流的結果 雖然程式碼短很多 但非常難懂 

第一個程式碼太冗長 第二個程式碼太難懂 要如何找到大家都滿意的中間點 就需要經驗的累積

{% highlight java %}
public class Anagrams {
  public static void main(String[] args) {
    List<String> dictionary = Arrays.asList("auctioned", "cautioned", "education", "state", "taste", "cat");
    int minGroupSize = 1;
    dictionary.stream().collect(groupingBy(word -> alphabetize(word)))
        .values().stream()
        .filter(group -> group.size() >= minGroupSize)
        .forEach(g -> System.out.println(g.size() + ": " + g));
  }
  private static String alphabetize(String s) {
    char[] a = s.toCharArray();
    Arrays.sort(a);
    return new String(a);
  }
}
{% endhighlight %}

說穿了也沒什麼 就是把一個複雜的函數alphabetize提出來

所以在一個用Stream的程式中 最容易看出一個工程師的功力 比如說alphabetize方法的可讀性就變得至關重要 既然你把他提出來了 那別人在讀你的stream時思緒會被中斷 如果你的命名命的好 讀者可以繼續讀下去 而不用先往下看你提出來的方法在做什麼


### 雙層流

其實流可以做得到的事 迴圈都可以做的到 那今天如果要雙重迴圈呢?

比如說發一副牌 雙重迴圈搞定

{% highlight java %}
private static List<Card> newDeck() {
  List<Card> result = new ArrayList<>();
  for (Suit suit : Suit.values())
    for (Rank rank : Rank.values())
      result.add(new Card(suit, rank));
  return result;
}
{% endhighlight %}

用stream 就需要使用flatMap

{% highlight java %}
private static List<Card> newDeck() {
    return Stream.of(Suit.values())
      .flatMap(suit ->
        Stream.of(Rank.values())
          .map(rank -> new Card(suit, rank)))
      .collect(toList());
  }
{% endhighlight %}

揪竟哪一個好 見仁見智 第一個版本好懂 第二個版本優雅

### 總結

有些任務適合用迭代 有些任務適合用Stream 有些任務需要你在兩者之中取平衡 如何才是最好 取決於工程師的經驗以及讀者公司的習慣 沒有標準答案






