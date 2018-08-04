---
layout: post
title: Effective Java Item24 - 優先考慮靜態成員類 
comments: True 
subtitle: effective java - 優先考慮靜態成員類
tags: effectiveJava
author: jyt0532
excerpt: 本篇文章介紹嵌套類的用法和使用時機
---

這篇是Effective Java - Favor static member classes over nonstatic章節的讀書筆記 本篇的程式碼來自於原書內容

## Item24: 優先考慮靜態成員類

### 嵌套類(nested class)

嵌套類是在某一個類 的內部 的類

嵌套類存在的目的 是為了他的外圍類(enclosing class)提供服務

嵌套類有四種: 靜態成員類 非靜態成員類 匿名類和局部類

除了第一種以外 剩下的三種被稱為內部類(inner class)

這個Item會告訴你什麼時候使用哪種類型的嵌套類

### 靜態成員類

#### 介紹

最常見 最普通的嵌套類 你可以把它想成一個普通的類剛好被定義在外圍類裡面 而他可以訪問所有外圍類的靜態成員(包含private成員)

靜態成員類是外圍類的一個靜態成員 所以他的可訪問規則也跟其他靜態成員一樣 

如果一個靜態成員類是private 
那就只有外圍類的其他成員可以存取

如果一個靜態成員類是public 那就任何人都可以存取

#### 用法

常見的用法是 作為公有類的輔助類 只有和他的外部類一起使用才有意義

舉個例子 
一個計算機類別Calculator裡面有一個靜態成員類enum Operation

這樣客戶就可以用Calculator.Operation.PLUS 或 Calculator.Operation.MINUS 來引用操作

### 非靜態成員類

#### 介紹

語法上很接近 非靜態成員類跟靜態成員類的差別就是有沒有static

但結果卻差很多 

1.**每一個非靜態成員類的實例都和一個外圍類的實例相關聯** 

關聯建立的時機 是外圍類的某個實例方法的內部調用非靜態成員類的構造器的時候

當然在非靜態成員類的實例方法中 可以調用外圍實例上的方法，或者使用this獲得對外圍實例的引用

非靜態成員類還有一個特性 就是他可以訪問所有外圍類的**所有**成員(包含靜態成員)


#### 用法

非靜態成員類的常見用法是[Adaptor](/2017/07/14/adaptor/) 這可以讓你的外部類的實例呼叫一個方法 就變成另外一個不相關的類別

比如說Map的介面的實作 通常會用一個non-static的類別來實作collection view(像是keySet, entrySet等等)

來看一下HashMap怎麼實作keySet

![Alt text]({{ site.url }}/public/item24.png)

看出端倪了嗎 因為非靜態成員類(keySet)可以存取外部類的所有成員 所以你可以把外部類**轉化**成你想要它長的樣子

所以HashMap也利用了非靜態成員類來把Map變成Set

再看細一點

![Alt text]({{ site.url }}/public/item24-1.png)

非靜態成員類KeyIterator定義在這
![Alt text]({{ site.url }}/public/item24-2.png)

又是一樣的把戲 KeySet利用了非靜態成員類來把Set變成Iterator

### 靜態成員類 vs 非靜態成員類

**如果一個成員類不需要訪問外圍的實例 那就應該要宣告成靜態成員類**

否則的話 每一個非靜態成員類的實例都包含了指到外圍實例的引用 

保存這份引用不但要花時間跟空間 而且還會害外圍實例無法被垃圾回收器收集

下面的程式碼小小的總結了一下靜態成員類跟非靜態成員類如何宣告 使用 以及實例化
{% highlight java %}
public class Item24 {
  private static int outerPrivateStatic = 5;
  int outer = 3;
  static class StaticNestedClass {
        //int c = outer+5; => 不合法 靜態嵌套類不能存取外圍類的非靜態變數
        static int nestedStatic = outerPrivateStatic + 3;
        static int get(){
          return nestedStatic;
        }
  }
  class NonstaticInnerClass {
    //static int innerStatic = 5; => 不合法 非靜態嵌套類不能有靜態變數
    int d = 4;
    int y = outerPrivateStatic + 1;
    int z = outer + 3;
  }

  public static void main(String... args) {
    //Initialize static nested class
    Item24.StaticNestedClass snc = new Item24.StaticNestedClass();
    //Initialize non-static inner class
    Item24.NonstaticInnerClass innerObject = new Item24().new NonstaticInnerClass();
  }
}
{% endhighlight %}

### 匿名類

匿名類的使用有許多限制 

1.沒有名字

2.不是外圍類的member

3.除了他們被聲明的當下 其他時候他們無法生出實例

4.不能用instanceof

5.不能用一個匿名類實現多個接口

6.不能同時拓展類跟實作接口

最常見的用法就是用來創建函數對象(只定義一個函數的介面或是抽象類別 的實例)

{% highlight java %}
// Anonymous class instance as a function object - obsolete!
Collections.sort(words, new Comparator<String>() {
  public int compare(String s1, String s2) {
    return Integer.compare(s1.length(), s2.length());
  }
});
{% endhighlight %}


### 局部類

在任何 可以聲明局部變量的地方 都可以聲明局部類


#### 匿名類跟局部類

來看看[Oracle](https://docs.oracle.com/javase/tutorial/java/javaOO/anonymousclasses.html)的例子

{% highlight java %}
public class HelloWorldAnonymousClasses {

  interface HelloWorld {
    public void greet();
    public void greetSomeone(String someone);
  }

  public void sayHello() {
    class EnglishGreeting implements HelloWorld {
      String name = "world";
      public void greet() {
        greetSomeone("world");
      }
      public void greetSomeone(String someone) {
        name = someone;
        System.out.println("Hello " + name);
      }
    }

    HelloWorld englishGreeting = new EnglishGreeting();

    HelloWorld frenchGreeting = new HelloWorld() {
      String name = "tout le monde";
      public void greet() {
        greetSomeone("tout le monde");
      }
      public void greetSomeone(String someone) {
        name = someone;
        System.out.println("Salut " + name);
      }
    };

    englishGreeting.greet();
    frenchGreeting.greetSomeone("Fred");
  }

  public static void main(String... args) {
    HelloWorldAnonymousClasses myApp =
        new HelloWorldAnonymousClasses();
    myApp.sayHello();
  }
}
{% endhighlight %}

我們看到有一個接口HelloWorld 裡面有兩個函數需要被實作

然後EnglishGreeting就實作了這個接口 並且直接創建實例englishGreeting

這裡的EnglishGreeting就是局部類

再往下看

frenchGreeting是一個實作了HelloWorld這個接口的類別的實例 但這個類別連名字都沒有 就是在實作的當下 可以馬上用 用完後就煙消雲散 這就是匿名類


### 總結

如果你在一個類別裡面 需要另一個類別來幫你做事 而這個類別只需要存在某個方法以內 那就使用匿名類或局部類 但如果這個方法需要在方法之外也可見 或是這個類別比較大 那就必須要用成員類(member class)

一個成員類應該要盡可能地讓它是靜態類 除非你每一個成員類的實例都需要指到外圍類的實例 你才定義成非靜態
