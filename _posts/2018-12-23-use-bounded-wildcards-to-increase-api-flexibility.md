---
layout: post
title: Effective Java Item31 - 利用限制通配符來提昇API靈活性 
comments: True 
subtitle: effective java - 利用bounded wildcards來提昇API靈活性
tags: effectiveJava
author: jyt0532
excerpt: 本篇文章介紹限制通配符的好處
---

這篇是Effective Java - Use bounded wildcards to increase API flexibility章節的讀書筆記 本篇的程式碼來自於原書內容

本篇是泛型系列文的高潮 請讀者務必要讀懂這篇的內容

本章節可以搭配

[泛型篇章簡介及術語列表](/2018/12/01/generics/) 

[類型參數和通配符的選擇](/2018/12/23/how-to-choose-between-wildcard-and-generic-method/) 

[到底 <T extends Comparable<? super T>>是什麼意思](/2018/12/23/t-extends-comparable-questionmark-super-t/)

服用

## Item31: 利用限制通配符來提昇API靈活性

我們在[Item28](/2018/12/08/prefer-lists-to-arrays/)有說 泛型是**不可變(invariant)** 意思是說 對於任意兩種不同的type `Type1`和`Type2` , `List<Type1>` 既不是`List<Type2>`的子類型 也不是它的父類型

### 最簡單的解釋

看起來真的很違背常理 `List<Dog>`居然不是`List<Animal>`的子類型 但你在靜下心來想一想polymorthism的真諦

`void add(Animal a)`
當我的input參數是Animal 你卻給我Dog 有沒有關係？ 答案是沒關係 因為所有我在函數裡可以對Animal的操作 我都可以對Dog做 

但要是我變成這樣`void add(List<Animal> la)`
我的input參數是`List<Animal>` 你卻給我`List<Dog>` 這樣就有關係了 因為我的函數裡面有可能會有`a.add(new Cat())`的操作 但我卻不應該套用在`List<Dog>`上

所以泛型是invariant

### 例子

我們來複習一下可愛的Stack

{% highlight java %}
public class Stack<E> {
  public Stack();
  public void push(E e);
  public E pop();
  public boolean isEmpty();
}
{% endhighlight %}

今天我們多了一個方法 可以一次push很多個元素
{% highlight java %}
public void pushAll(Iterable<E> src) {
  for (E e : src)
    push(e);
}
{% endhighlight %}

來試試看 push很多Integer進Number裡
{% highlight java %}
Stack<Number> numberStack = new Stack<>();
Iterable<Integer> integers = ... ;
numberStack.pushAll(integers);
{% endhighlight %}
編譯錯誤 因為如同剛剛所說 `Iterable<Integer>`不是`Iterable<Number>`的子類

![Alt text]({{ site.url }}/public/item31-1.png)

那該怎麼辦呢 主角登場 **限制通配符(bounded wildcard type)**

### 限制通配符

`pushAll`的輸入參數不應該是 E的Iterable接口 而應該是

**E的某個子類型的Iterable接口**


{% highlight java %}
public void pushAll(Iterable<? extends E> src) {
    for (E e : src)
        push(e);
}
{% endhighlight %}

搞定 編譯完全沒問題 還是類型安全

### super

再來多加一個方法 `popAll` 他會把stack的所有東西pop並丟進輸入List

第一版
{% highlight java %}
public void popAll(Collection<E> dst) {
  while (!isEmpty())
    dst.add(pop());
}
{% endhighlight %}

如果我們用E的父類型去接他

{% highlight java %}
Stack<Number> numberStack = new Stack<Number>();
Collection<Object> objects = ... ;
numberStack.popAll(objects);
{% endhighlight %}

把numberStack裡的`Number`全部丟進`Collection<Object>` 看起來很ok 但不好意思 泛型是invariant 
`Collection<Object>` 不是`Collection<Number>`的父類

我們會看到跟第一次寫`pushAll`一樣的錯誤

![Alt text]({{ site.url }}/public/item31-2.png)

解法也很像

`popAll`的輸入參數不應該是 E的集合 而應該是

**E的某個父類型的集合**

{% highlight java %}
public void popAll(Collection<? super E> dst) {
  while (!isEmpty())
    dst.add(pop());
}
{% endhighlight %}

Stack類和客戶端都可以輕鬆搞定

### 生產者和消費者

什麼時候該用extend什麼時候該用super呢 決定於輸入參數 

如果輸入參數 是負責*生產*元素 則參數就是extend 

如果輸入參數 是負責*消費*元素 則參數就是super

如果輸入參數 既負責生產又負責消費 那就只能精確的類型匹配 不能用wildcard

### PECS(producer-extends，consumer-super)

這是個幫你記憶的口訣 再回頭看剛剛的例子 `pushAll`提供了元素 是個生產者 所以用extend `popAll`提供了集合要裝元素 所以是消費者 要用super


### 回頭看看之前的程式碼

我們剛剛才學會了一個強大的武器 現在回頭看一下之前寫的東西

[Item28](/2018/12/08/prefer-lists-to-arrays/)的`Choose`的構造器

{% highlight java %}
public Chooser(Collection<T> choices)
{% endhighlight %} 

今天以前 這個構造器只能給入T 現在我們稍做修改


{% highlight java %}
public Chooser(Collection<? extends T> choices)
{% endhighlight %}

現在這個構造器可以輸入所有T的子類

再看一下[Item30](/2018/12/15/favor-generic-methods/)的`union`

{% highlight java %}
public static <E> Set<E> union(Set<E> s1, Set<E> s2)
{% endhighlight %}

顯而易見 這是生產者
{% highlight java %}
public static <E> Set<E> union(Set<? extends E> s1,  Set<? extends E> s2)
{% endhighlight %}

注意 return type仍然是`Set<E>` **不要使用限定通配符類型作為返回類型**

有了限定通配符 使用起來輕鬆愉快
{% highlight java %}
Set<Integer>  integers =  Set.of(1, 3, 5);
Set<Double>   doubles  =  Set.of(2.0, 4.0, 6.0);
Set<Number>   numbers  =  union(integers, doubles);
{% endhighlight %}

### 熱身完了

進入難題 來看一下之前[Item30](/2018/12/15/favor-generic-methods/)的`max`

我們套用PECS之前長這樣
{% highlight java %}
public static <E extends Comparable<E>> E max(Collection<E> c)
{% endhighlight %}

複習一下 這個函數的輸入必須要是一個collection of E, 而且E要有implement Comparable

那我們現在有了更強大的武器之後 可以讓這個函式的應用更加廣泛

{% highlight java %}
public static <T extends Comparable<? super T>> T max(List<? extends T> list)
{% endhighlight %}

這可能是本書最複雜的一個函數聲明

在這裏 我們應用了PECS兩次 

第一次是在輸入的參數 因為我們要找最大值 輸入的集合當然是個生產者 無庸置疑 extends

第二個就精彩了 因為Comparable一定是個消費者 (畢竟他需要讀輸入才能比較) 因此 `Comparable<T>`就可以安心被`Comparable<? super T>` 取代

用中文來翻譯一下

{% highlight java %}
<T extends Comparable<T>>
{% endhighlight %}

限制是說 T必須要實作Comparable&lt;T&gt;(只有這樣 T之間才能互相比大小) 比如具體類T是Student 那它必須 implements Comparable&lt;Student&gt;

{% highlight java %}
<T extends Comparable<? super T>> 
{% endhighlight %}

限制是說 T必須要實作Comparable&lt;T或是T的任意父類&gt;(只有這樣 T的實例之間 **或是T和他的父類的實例之間** 才能互相比大小)


Effective Java對於這個聲明給出的範例非常難懂 然後草草結束 用什麼`ScheduledFuture`跟`Delayed`這種沒人知道的東西解釋 
一點意義都沒有 為此我特地開了一篇 [到底<T extends Comparable<? super T>>是什麼意思](/2018/12/23/t-extends-comparable-questionmark-super-t/) 大家可以移駕到那篇去看我用簡單的解說說明兩個聲明的差異

### 類型參數和通配符

類型參數`T` 和通配符`?`具有雙重性 請看下面兩種聲明 第一個是無限制類型參數 第二個是無限制通配符

{% highlight java %}
public static <E> void swap(List<E> list, int i, int j);

public static void swap(List<?> list, int i, int j);
{% endhighlight %}

請注意`?`指的其實是`? extends Object` 所以第二個你可以看成

`public static void swap(List<? extends Object> list, int i, int j);`

那這兩個哪個比較好呢 如果是你要提供公用的API 那第二個好一點 非常好懂 就是把一個List的兩個index交換

通常來說**如果一個類型參數聲明中 類型參數只出現一次
那就把它換成通配符聲明** 這句話對於不論限制類型還是無限制類型都一樣有效

但是我們認為好懂的聲明 卻無法編譯以下簡單的實作

{% highlight java %}
public static void swap(List<?> list, int i, int j) {
    list.set(i, list.set(j, list.get(i)));
}
{% endhighlight %}

聲明雖然看起來好用 但實作上卻是綁手綁腳 你必須要用當初的第一個我們不喜歡的聲明來實作

{% highlight java %}
public static void swap(List<?> list, int i, int j) {
    swapHelper(list, i, j);
}

// Private helper method for wildcard capture
private static <E> void swapHelper(List<E> list, int i, int j) {
    list.set(i, list.set(j, list.get(i)));
}
{% endhighlight %}

搞什麼飛機 你選擇了wildcard當作公開API 但內部卻用generic type來實作 為什麼當初不直接用generic type當作公眾API呢

這時候就可以移駕到這篇文章[類型參數和通配符的選擇](/2018/12/23/how-to-choose-between-wildcard-and-generic-method/) 我在這篇文章有詳細的講解跟比較

### 總結

正確的使用通配符類型會讓你的API更加靈活 記住基本的規則PECS 以及`Comparable`和`Comparator`都是消費者
