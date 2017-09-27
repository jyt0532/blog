---
layout: post
title: Effective Java Item1 - Consider static factory methods instead of constructors
comments: True 
subtitle: effective java - 靜態工廠方法
tags: effectiveJava
author: jyt0532
---
這篇是Effective Java - Consider static factory methods instead of constructors章節的讀書筆記

## Item1: 使用static factory method

注意這裡跟Design pattern的[Factory method](/2017/04/28/factory-method/)沒有關連 這點說的是 與其開放你的constructor讓你的使用者想new就new 不如用另外一個靜態工廠包裝起來 當你需要這個物件的時後 跟這個static factory method要人 這個method裡面在自己決定怎麼給你人

直接開放constructor給你用是很差的

{% highlight java %}
Foo x = new Foo()
{% endhighlight %}

不如這樣

{% highlight java %}
Foo x = Foo.create()
{% endhighlight %}

當然constructor是private  
而且工廠方法是static 代表說即使不需要依賴這個類別的物件 就可以call這個method

### 好處

1.工廠方法可以取名字: client看名字就知道應該call哪個工廠方法 而不是去看constructor的參數順序

2.控制物件數: 不一定每次都要創造新的物件給別人 你可以自己決定要給client舊的instance還是new新的 如果創造物件很貴的話 這點就很實用

3.除了回傳class的instance之外 還可以選擇回傳這個class的subclass(3-1) 或者是所有implement同一個interface的class(3-2) 這點讓我們的接口更加靈活

這是最難懂的一個好處 我把他分成兩個方向來細說

3-1:可以選擇回傳這個class的subclass 

{% highlight java %}
public class ParentClass{ 
  public static ParentClass getParentObject() { //靜態工廠方法 1
    return ParentClass.newInstance(); 
  } 
  public static ParentClass getChildObject() { //靜態工廠方法 2
    return ChildClass.newInstance(); 
  } 
} 
private class ChildClass extends ParentClass{ 
  public static ChildClass newInstance() { 
    return new ChildClass(); 
  } 
}
{% endhighlight %}
ㄜ 為什麼要這樣 為什麼我們要跟parentClass要一個childClass呢 直接跟ChildClass要不是很好嗎
這樣的好處是我們可以隱藏childClass的實作細節

3-2:可以選擇回傳所有implement同一個interface的class

interface-based-framework: **代表著只讓使用者/Client 存取到interface 但事實上我們給他們的卻是那些implement這個interface的class**

給個例子 

{% highlight java %}
public interface Runnable {
          void run();
}
{% endhighlight %}
{% highlight java %}
public class RunAnimal { 
  private static final Dog dog = new Dog(); 
  private static final Cat cat = new Cat(); 
  private RunAnimal(){} 

  public static Runnable getDog() {//static factory method 
    return dog; 
  } 
  public static Runnable getCat(){//static factory method  
    return cat; 
  } 
  private static class Dog implements Runnable{ 
    @Override public void run() { 
      System.out("Running fast");
    } 
  } 
  private static class Cat implements Runnable{ 
    @Override public void run() { 
      System.out("Running slow");
    } 
  } 
}
{% endhighlight %}
要用的時候 用interface接他
{% highlight java %}
Runnable myDog = RunAnimal.getDog()
Runnable myCat = RunAnimal.getCat()
myDog.run();
{% endhighlight %}

這就是所謂的接口更加靈活

4.代碼更加簡潔

原本長這樣
{% highlight java %}
Map<String, List<String>> m =
    new HashMap<String, List<String>>();
{% endhighlight %}
重複的東西必須一直寫 現在你可以這樣
{% highlight java %}
Map<String, List<String>> m = HashMap.newInstance();
{% endhighlight %}
為什麼可以這樣呢 因為HashMap有generic static factory
{% highlight java %}
public static <K, V> HashMap<K, V> newInstance(){
  return new HashMap<K, V>();
}
{% endhighlight %}
所以你給他任何的K, V他都可以直接帶進去

### 壞處

1.如果你的class沒有public或是protected的constructor 就不能有subclass

2.靜態工廠方法跟其他的static方法沒有太好的方法區隔 甚至javaDoc的constructor裡看不到你那些精美的靜態工廠方法


### 總結

靜態工廠方法跟constructor都有好處 寫程式的時候先考慮靜態工廠方法 如果他帶來的缺點無法接受 再公開你的constructor
