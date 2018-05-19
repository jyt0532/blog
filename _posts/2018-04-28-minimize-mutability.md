---
layout: post
title: Effective Java Item17 - 使可變性最小化
comments: True 
subtitle: effective java - 使可變性最小化
tags: effectiveJava
author: jyt0532
excerpt: 本篇講解不可變類的好處與如何實作不可變類
---

這篇是Effective Java - Minimize mutability章節的讀書筆記 本篇的程式碼來自於原書內容

### Item17: 使可變性最小化

不可變類 指的是實例不可以被修改 每個實例中包含的所有訊息必須在創建的時候就提供 直到生命週期結束前都不能變化

Java類庫裡有許多不可變類 比如說String, BigInteger等等 這類型的好處是容易設計 好用 安全

要設計一個不可變類 有五個規則要遵守

1.不提供會修改狀態的方法(mutator)

2.保證類不會被擴展: 這樣可以防止惡意的子類去改變狀態 最簡單的就是讓這個類別是final 比較好的方法就是讓所有構造器是私有或是包級私有 然後用靜態工廠來取代構造器 等等再來細談

3.所有變量都是final: 

4.使所有的變量都是私有: 防止客戶拿到變量的引用 雖然你還是可能讓你的變量public static final 只要這些變量指到基本類型的值或是不可變對象 但是不建議這麼做 
因為這樣會讓往後的版本[無法再改變內部的表示法](/2018/04/22/in-public-classes-use-accessor-methods-not-public-fields/)

5.確保客戶不能存取到可變對象: 說起來容易 但很容易出錯 
比如說不要直接拿客戶提供的引用 assign給自己的變量 
而且要記得在構造器 訪問方法和readObject方法中使用[保護型拷貝](2017/09/26/make-defensive-copies-when-needed/)

比如說 看一下這個複數(包含實部和虛部)的example

{% highlight java %}
public final class Complex {
  private final double re;
  private final double im;
  public Complex(double re, double im) {
    this.re = re;
    this.im = im;
  }
  // Accessors with no corresponding mutators
  public double realPart() { return re; }
  public double imaginaryPart() { return im; }
  public Complex add(Complex c) {
    return new Complex(re + c.re, im + c.im);
  }
  public Complex subtract(Complex c) {
    return new Complex(re - c.re, im - c.im);
  }
  public Complex multiply(Complex c) {
    return new Complex(re * c.re - im * c.im, re * c.im + im * c.re);
  }
  public Complex divide(Complex c) {
    double tmp = c.re * c.re + c.im * c.im;
    return new Complex((re * c.re + im * c.im) / tmp,
    (im * c.re - re * c.im) / tmp);
  }
}
{% endhighlight %}	

值得注意的是每個加減乘除的操作 都是返回一個**新的複數**

## 優點

1.簡單: 所有不可變類別的實例就只有一種狀態 就是剛創建時的狀態 所以你有任何的約束 都只要寫在構造器裡就可以 不用出現在每個setter裡

2.線程安全: 當多個線程同時訪問這個物件 因為不能修改 所以也沒有race condition的問題 所以代表說**不可變對象可以自由地被共享**
這也代表說 對於常用的值 你可以放心地提供public static final 比如說 複數類就可以提供幾個常用的

{% highlight java %}
public static final Complex ZERO = new Complex(0, 0);
public static final Complex ONE = new Complex(1, 0);
public static final Complex I = new Complex(0, 1);
{% endhighlight %}

還可以再進一步擴展 不可變的類可以再提供一個靜態工廠 把被頻繁的請求的實例緩存 也降低了內存和垃圾回收的成本

永遠記得在設計一個類別時 選擇用靜態工廠取代公有構造器可以讓你有添加緩存的靈活性 而不影響客戶

3.不需進行[保護型拷貝](2017/09/26/make-defensive-copies-when-needed/) 也不需要提供clone 或是copy constructor

4.不僅可以共享不可變對象 還可以共享訊息 比如說BigInteger的negate

{% highlight java %}
/**
 * Returns a BigInteger whose value is {@code (-this)}.
 *
 * @return {@code -this}
 */
public BigInteger negate() {
    return new BigInteger(this.mag, -this.signum);
}
{% endhighlight %}
新的回傳的BigInteger的mag指到的Array跟舊的是同一個 不需要重新一個一個copy **直接拿來用**

5.不可變對象為其他對象提供了大量的構建(building blocks)

當你本身是一個不可變對象時 當你被一個複雜對象使用 成為一部分時 也比較好維持複雜對象的約束

比如說一個Map 可以很放心的把一個不可變對象拿來當作Key 一個Set也可以很放心的把不可變對象拿來當作element 因為他們被丟進去之後就不會變化了
 
## 缺點

對於不同的值 需要一個不同的對象 而如果創建新的對象代價很高就會影響性能

比如你有一個100位數的BigInteger 你想把個位數從1改成2 可變對象就是直接一個assignment 
但不可變對象就是需要一個新的BigInteger 把前99位複製 再assign第100位

這是O(n)跟O(1)的時間差距

### [靜態工廠](/2017/09/20/static-factory-method/)

剛剛在五個原則裡面有提到 我們必須讓類別不能被繼承 除了宣告final之外 比較好的方法就是讓所有構造器是私有或是包級私有 然後用靜態工廠來取代構造器

上例子

{% highlight java %}
// Immutable class with static factories instead of constructors
public class Complex {
  private final double re;
  private final double im;
  private Complex(double re, double im) {
    this.re = re;
    this.im = im;
  }
  public static Complex valueOf(double re, double im) {
    return new Complex(re, im);
  }
  ... // Remainder unchanged
}
{% endhighlight %}

這是最靈活也通常是最好的替代方案 比如說你今天想支援極座標 只需要再加一個工廠

{% highlight java %}
public static Complex valueOfPolar(double r, double theta) { 
  return new Complex(r * Math.cos(theta), r * Math.sin(theta)); 
}
{% endhighlight %}

### 放輕鬆

一開始講的五個規則 事實上 太過於嚴格 我們為了性能的要求 可以稍微放鬆一點點

事實上不可變類別的定義是**沒有一個方法能夠對對象的狀態產生外部可見**的改變

舉個例子PhoneNumber這個類別的物件 有一個非final的成員 專門用來存電話號碼的hash 可是這個hash如果沒有人想知道 它就沒有計算的必要
當第一次有人想知道這個實例的hash值 它會把它算好 存在這個hash裡(此時這個hash被變動了 可是這個值並不是外部可見) 之後有人再次想知道你的hash就直接拿這個值來用 因為我們已經保證了被拿來計算hash的其他成員是不可變的 


### 不可變類的序列化

如果你想讓自己的不可變類實現實現Serializable接口 並且包含一個或多個指向可變對象的引用 你就必須提供一個顯式的readResolve或是readObject 即使預設的序列化可以接受也是一樣
否則攻擊者可以從不可變的類中創建可變的物件 更多細節請見[Item88](/2017/10/19/write-readobject-method-defensively/)


### 總結

堅決不要為每個get方法都要寫一個set 除非你有很好的理由讓一個類是可變的類 否則就應該是不可變的 不可變類的優點很多 缺點就只有某些特定情況下的性能問提

如果對於某些類別 不可變性是不實際的 那即使你讓一個類可變 也應該要盡量限制它的可變性 降低對象可以存在的狀態數讓你更容易分析行為 除非你有很好理由 不然**每個變量都應該是預設final**


