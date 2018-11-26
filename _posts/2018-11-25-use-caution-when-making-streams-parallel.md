---
layout: post
title: Effective Java Item48 - 謹慎使用並行Stream
comments: True 
subtitle: effective java - 謹慎使用流並行
tags: effectiveJava
author: jyt0532
excerpt: 本篇文章介紹並行化Stream的難度以及可能的風險
---

這篇是Effective Java - Use caution when making streams parallel章節的讀書筆記 本篇的程式碼來自於原書內容

## Item48: 謹慎使用並行Stream

要正確快速的編寫並發程序很難 在寫流程序的時候也不例外

{% highlight java %}
public static void main(String... args){
  primes().map(p -> TWO.pow(p.intValueExact()).subtract(ONE))
    .filter(mersenne -> mersenne.isProbablePrime(50))
    .limit(20)
    .forEach(System.out::println);
}

static Stream<BigInteger> primes() {
  return Stream.iterate(TWO, BigInteger::nextProbablePrime);
}
{% endhighlight %}

執行這個程式 會輸出前20個2^n-1且是質數的數字
{% highlight txt %}
3
7
31
127
8191
...
{% endhighlight %}

要跑出前20個 在作者的電腦需要12.5秒 但如果我們在Stream的後面呼叫parallel() 會不會比較快呢
{% highlight java %}
public static void main(String... args){
  primes().parallel()
    .map(p -> TWO.pow(p.intValueExact()).subtract(ONE))
    .filter(mersenne -> mersenne.isProbablePrime(50))
    .limit(20)
    .forEach(System.out::println);
}

static Stream<BigInteger> primes() {
  return Stream.iterate(TWO, BigInteger::nextProbablePrime);
}
{% endhighlight %}

答案是 什麼都不會印出來 因為Stream的library不知道怎麼並行化這個pipeline

### 使用並行時機

如果你的原始數據結構是:

ArrayList、HashMap、HashSet和ConcurrentHashMap, Arrays、int類型範圍的流和long類型的範圍的流

那就比較適合並行 原因是可以精準而且便宜的切割成任意大小的子程序 使得thread劃分工作變得簡單



你的終結操作也很大程度的影響了並行化的效能 如果你在終結操作做了很多工作 而且工作與工作間彼此依賴(比如說下一個操作依賴於上一個操作) 那就很難並行化

比較好的終結操作是reduce min max count sum anyMatch allMatch noneMatch

比較不好的就是collect

### 不可預期的故障

並行化一個流不僅會導致低性能 他還會導致不正確結果和不可預知的行為

即使並行完後 預期行為都正確 也必須測試性能 來決定[值不值得](/2018/06/09/optimize-judiciously/)並行化

### 成功例子

作者也提供了一個成功並行化的例子 
這段程式計算小於n的質數有幾個

{% highlight java %}
static long pi(long n) {
  return LongStream.rangeClosed(2, n)
    .mapToObj(BigInteger::valueOf)
    .filter(i -> i.isProbablePrime(50))
    .count();
}
{% endhighlight %} 
這裡只要輕鬆的加個`.parallel()`
{% highlight java %}
static long pi(long n) {
  return LongStream.rangeClosed(2, n)      
    .parallel()
    .mapToObj(BigInteger::valueOf)
    .filter(i -> i.isProbablePrime(50))
    .count();
}
{% endhighlight %} 
在作者的四核計算機上 效能提升3.7倍

注意這個pipeline的資料結構是long類型的範圍的流 而且終結操作是count

### 總結

不要輕易嘗試並行化流 除非你有充分的理由相信 它將保持正確 而且會提高性能(要仔細分析你的資料結構以及所有流操作)

不恰當的並行化流 不止會讓程式跑錯結果 甚至性能災難












