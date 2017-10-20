---
layout: post
title: Effective Java Item3 - Enforce the singleton property with an enum type
comments: True 
subtitle: effective java - 用Enum實作Singleton
tags: effectiveJava enum
author: jyt0532
---
這篇是Effective Java - Enforce the singleton property with a private constructor or an enum type章節的讀書筆記 本篇的程式碼來自於原書內容 教學內容來自[Geeksforgeeks](http://www.geeksforgeeks.org/enum-in-java/)

在看這篇文章之前 強烈建議先看過[Singleton](/2017/05/19/singleton/)

## Item3: 用Enum實作Singleton

在[Singleton](/2017/05/19/singleton/)中說明了很多實現單例的方法 大多數都是利用private constructor來實現

但之前說了那麼多實作方法 是要一步一步讓你知道單例怎麼實現 和實現上會遇到的困難 

比較常見的第一個方法 eager-loading 
{% highlight java %}
public class Elvis {
  public static final Elvis INSTANCE = new Elvis();

  private Elvis() {
  }

  public void leaveTheBuilding() {
    System.out.println("Whoa baby, I'm outta here!");
  }
}
public static void main(String[] args) {
  Elvis elvis = Elvis.INSTANCE;
  elvis.leaveTheBuilding();
}

{% endhighlight %}

比較常見的第二個方法 靜態工廠
{% highlight java %}
public class Elvis {
  private static final Elvis INSTANCE = new Elvis();

  private Elvis() {
  }

  public static Elvis getInstance() {
    return INSTANCE;
  }

  public void leaveTheBuilding() {
    System.out.println("Whoa baby, I'm outta here!");
  }
}
public static void main(String[] args) {
  Elvis elvis = Elvis.INSTANCE;
  elvis.leaveTheBuilding();
}
{% endhighlight %}
事實上 實現單例有一個最簡單的方法

{% highlight java %}
public enum Elvis {
  INSTANCE;

  public void leaveTheBuilding() {
    System.out.println("Whoa baby, I'm outta here!");
  }
}
public static void main(String[] args) {
  Elvis elvis = Elvis.INSTANCE;
  elvis.leaveTheBuilding();
}
{% endhighlight %}

就是這麼簡單 

## 什麼是Enum

Enumerations 枚舉 當你用在一些compile time就會知道的常數 比如說一個禮拜有幾天
{% highlight java %}
enum Week
{
    MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY;
}
{% endhighlight %}
或是樸克牌花色
{% highlight java %}
enum Suit
{
    CLUB, SPADE, HEART, DIAMOND;
}
{% endhighlight %}

用法呢 簡單
{% highlight java %}
Suit s1 = Suit.CLUB;
System.out.println(s1); // print "CLUB"
{% endhighlight %}

在java裡面的enum比在C++的強 enum裡面還可以加instance variable或是函數甚至是constructor 事實上用起來很像一個class

你可以把上面的enum Suit想成這樣
{% highlight java %}
Class Suit
{
    public static final Suit CLUB = new Suit();
    public static final Suit SPADE = new Suit();
    public static final Suit HEART = new Suit();
    public static final Suit DIAMOND = new Suit();
}
{% endhighlight %}

每一個enum constant都是一個enum type的物件

既然每個enum constant都是public static 我們可以直接從enum name取得而不需要創造物件

{% highlight java %}
Suit s1 = Suit.CLUB;
System.out.println(s1); // print "CLUB"
{% endhighlight %}

既然每個enum constant都是final 我們不能繼承enum 既然不能被繼承 那所有的method都必須是concrete method

注意enum的constructor不能是public或protected 所以枚舉對象無法再run-time的時候由call constructor來初始化

那enum的constructor什麼時候會被call呢 在load enum這個class的時候 每個enum constant會跑一次constructor 

{% highlight java %}
enum Suit
{
    CLUB, SPADE, HEART, DIAMOND;
 
    private Suit()
    {
        System.out.println("Constructor called for : " +
        this.toString());
    }
    public void suitInfo()
    {
        System.out.println("Universal Suit");
    }
}
 
public class Test
{    
    public static void main(String[] args)
    {
        Suit s1 = Suit.CLUB;
        System.out.println(s1);
        s1.suitInfo();
    }
}
{% endhighlight %}

{% highlight sh %}
Constructor called for : CLUB
Constructor called for : SPADE
Constructor called for : HEART
Constructor called for : DIAMOND
CLUB
Universal Suit
{% endhighlight %}

![Alt text]({{ site.url }}/public/item3.jpg)

## Enum是實現Singleton最好的方法

剛剛講了那麼多的特性 有沒有感覺到Enum perfectly fits our requirement?
{% highlight java %}
public enum Elvis {
  INSTANCE;

  public void leaveTheBuilding() {
    System.out.println("Whoa baby, I'm outta here!");
  }
}
{% endhighlight %}

INSTANCE是個public static final 而且在class loading的時候創建

簡直簡單完美 而且enum還支援序列化 簡直皆大歡喜 下次當你的程式需要實現Singleton 考慮一下enum吧！
