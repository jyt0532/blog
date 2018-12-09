---
layout: post
title: Effective Java Item28 - 列表優於數組
comments: True 
subtitle: effective java - List 優於 Arrays
tags: effectiveJava
author: jyt0532
excerpt: 本篇文章介紹為什麼能使用list就要使用list 而不是array
---

這篇是Effective Java - Prefer lists to arrays章節的讀書筆記 本篇的程式碼來自於原書內容

## Item28: 列表優於數組

我們來看看先講泛型跟數組的不同

### covariant 協變 跟 invariant 不可變

Array 是協變 意思是說 如果`Sub` 是`Super` 的子類 那`Sub[]` 也是 `Super[]`的子類

泛型 則是不可變 意思是說 任意兩個Type, List<Type1> 和 List<Type2> 不是彼此的子類型或是父類型

來個例子 以下的代碼compile會過
執行時才會在runtime拋出例外
{% highlight java %}
// Fails at runtime!
Object[] objectArray = new Long[1];
objectArray[0] = "I don't fit in"; // Throws ArrayStoreException
{% endhighlight %}

但下面的程式在編譯時期就會出錯
{% highlight java %}
// Won't compile!
List<Object> ol = new ArrayList<Long>(); // Incompatible types
ol.add("I don't fit in");
{% endhighlight %}

當然這兩個用法都不對 只是用List會在compile-time直接出錯 好的IDE還直接跟你說為什麼錯

![Alt text]({{ site.url }}/public/item28.png)

好處自然清楚 我們希望越早發現錯誤越好

### Reified 具體化 vs Erasure 擦除

Array是具體化的 所以Array會在運行時 才去檢查元素類型的約束

泛型則是透過擦除來實現 這代表只在編譯期執行類型的約束 運行期則是丟棄(擦除)元素類型訊息 
目的是要讓舊的沒有支援泛型的程式[可以相容](/2018/12/02/dont-use-raw-types/)

### 難以混用

基於以上根本上的區別 數組和泛型很難混用 比如說

`new List <E> []`

`new List <String> []`

`new E []` 

全都會拋出泛型數組創建錯誤

為什麼不能創建泛型Array呢 因為他不是type-safe 如果泛型Array合法 那就有可能會在運行時拋出ClassCastException 
那就違反了泛型的保證

上例子


{% highlight java %}
// Why generic array creation is illegal - won't compile!
List<String>[] stringLists = new List<String>[1];  // (1)
List<Integer> intList = List.of(42);               // (2)
Object[] objects = stringLists;                    // (3)
objects[0] = intList;                              // (4)
String s = stringLists[0].get(0);                  // (5)
{% endhighlight %}

(1) 現在假設創建泛型Array合法

(2) 創建泛型

(3) 合法 因為array是協變, `List<String>` 是 `Object`的subtype

(4) 合法 因為泛型是擦除, `List<Integer>`在運行時就只是個List `List<String>[]`在運行時就是個`List[]`

(5) 出大事了 右邊回傳Integer 左邊卻是String 拋出ClassCastException

違反了泛型的保證 所以索性第一步就compile-error

所以當你在可以選擇要使用`List<E>` 還是 `E[]`的時候 通常前者是比較好的

### 簡潔性+性能 vs 類型安全性和互操作性


#### Trade-off 例子

來看一下一個隨機選擇器 `choose()`會回傳choiceArray裡面一個隨機的東西

{% highlight java %}
// Chooser - a class badly in need of generics!
public class Chooser {
  private final Object[] choiceArray;

  public Chooser(Collection choices) {
    choiceArray = choices.toArray();
  }

  public Object choose() {
    Random rnd = ThreadLocalRandom.current();
    return choiceArray[rnd.nextInt(choiceArray.length)];
  }
}
{% endhighlight %}

實在好懂 實在簡潔 麻煩的是當你調用`choose()` 你必須轉換你的類型


{% highlight java %}
Integer arr[] = { 5, 6, 7, 8, 1, 2, 3, 4, 3 };
Set<Integer> a = new HashSet<>(Arrays.asList(arr));
Chooser c = new Chooser(a);
String s = (String)c.choose();
{% endhighlight %}

上面的程式compile會過 但很明顯的運行時會拋出ClassCastException 不是typesafe

現在來改改看 讓他成為泛型

{% highlight java %}
public class Chooser<T> {
  private final T[] choiceArray;

  public Chooser(Collection<T> choices) {
    choiceArray = choices.toArray();
  }
  public Object choose() {
    Random rnd = ThreadLocalRandom.current();
    return choiceArray[rnd.nextInt(choiceArray.length)];
  }
}
{% endhighlight %}
不能被編譯 Constructor出錯
![Alt text]({{ site.url }}/public/item28-2.png)

簡單 強制轉換 加上`(T[])`


{% highlight java %}
public class Chooser<T> {
  private final T[] choiceArray;

  public Chooser(Collection<T> choices) {
    choiceArray = (T[])choices.toArray();
  }
  public Object choose() {
    Random rnd = ThreadLocalRandom.current();
    return choiceArray[rnd.nextInt(choiceArray.length)];
  }
}
{% endhighlight %}

從編譯錯誤 變成警告

![Alt text]({{ site.url }}/public/item28-3.png)

編譯器告訴你在運行時不能保證強制轉換的安全性

除非你能保證使用者不亂用 那你可以照[Item27](/2018/12/02/eliminate-unchecked-warnings/)的方法 
加上註解抑制警告 但是Item27也說 那是逼不得已的做法 

我們試著再改動一次

{% highlight java %}
public class Chooser<T> {
  private final List<T> choiceList;

  public Chooser(Collection<T> choices) {
    choiceList = new ArrayList<>(choices);
  }

  public T choose() {
    Random rnd = ThreadLocalRandom.current();
    return choiceList.get(rnd.nextInt(choiceList.size()));
  }
}
{% endhighlight %}

功德圓滿 這是最冗長也是運行起來最慢的版本 但卻是typesafe的版本

### 結論

數組和泛型有著非常不同的類型規則 數組是協變且可具體化 泛型是不可變且可被擦除

所以數組提供了運行期的typesafe而不是編譯期的typesafe 
泛型提供了編譯期的typesafe而不是運行期的typesafe 

所以當你在猶豫要選哪個時 選擇列表而不是數組
