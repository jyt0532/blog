---
layout: post
title: 到底 &lt;T extends Comparable&lt;? super T&gt;&gt;是什麼意思
comments: True 
subtitle: 泛型番外篇 - 2
tags: effectiveJava
author: jyt0532
excerpt: 本篇文章介紹
---

每次看到聲明中有

`<T extends Comparable<? super T>>` 都會想辦法研究一下 搞懂之後 過幾天又忘記了 為了減少下次複習的時間 也為了分享學習的心得 產生了這篇文章 而且我也很有信心這篇會是網路上講解這個聲明最清楚的文章

希望讀者可以先看過[Item31](/2018/12/01/generics/)之後 再來讀這篇

### 前情提要

以下的聲明 大家都看得懂

{% highlight java %}
<T extends Comparable<T>>
{% endhighlight %}

限制是說 T必須要實作Comparable<T>(只有這樣 T之間才能互相比大小) 比如具體類T是Student 那它必須 implements Comparable<Student>

{% highlight java %}
<T extends Comparable<? super T>>
{% endhighlight %}

限制是說 T必須要實作Comparable<T或是T的任意父類>(只有這樣 T的實例之間 **或是T和他的父類的實例之間** 才能互相比大小)

以上的定義取自[Item31](/2018/12/16/use-bounded-wildcards-to-increase-api-flexibility/)

但我相信各位看官根本有看沒有懂 這篇文章舉個實際例子 來讓你真正通透

### Welcome to the jungle

先來各自定義一下兩種不同的sort
{% highlight java %}
public static <T extends Comparable<T>> void mySort1(List<T> list)
{
  Collections.sort(list);
  for(T item: list) System.out.println(item);
}
public static <T extends Comparable<? super T>> void mySort2(List<T> list)
{
  Collections.sort(list);
  for(T item: list) System.out.println(item);
}
{% endhighlight %}

定義一下我們的類別 Person是父類 Student是子類 藉由年紀排序

{% highlight java %}
class Person implements Comparable<Person>
{
  protected int age;
  public Person(int age)
  {
    this.age = age;
  }
  @Override
  public int compareTo(Person other)
  {
    return this.age - other.age;
  }
}
class Student extends Person
{
  public Student(int age) {
    super(age);
  }

}
{% endhighlight %}

建立一下我們的側資

{% highlight java %}
List<Person> allPersons = new ArrayList<Person>();
allPersons.add(new Person(10));
allPersons.add(new Person(20));

List<Person> mixedPerson = new ArrayList<Person>();
mixedPerson.add(new Person(30));
mixedPerson.add(new Student(40));

List<Student> allStudent = new ArrayList<Student>();
allStudent.add(new Student(5));
allStudent.add(new Student(18));
{% endhighlight %}

### 好戲登場

對於這三組測資 猜猜哪些可以排序哪些不行
{% highlight java %}
mySort1(allPersons);  // 1
mySort1(mixedPerson); // 2
mySort1(allStudent);  // 3 
mySort2(allPersons);  // 4
mySort2(mixedPerson); // 5
mySort2(allStudent);  // 6
{% endhighlight %}

給你一分鐘 一分鐘後公佈答案 

相信我 請認真的看一下mySort1跟mySort2的聲明 認真的想一下 這樣等一下看解釋的時候會學到最多 我不會害你

### 揭曉答案

答案是 第三個無法執行 且聽我娓娓道來

先看mySort1 

{% highlight java %}
public static <T extends Comparable<T>> void mySort1(List<T> list)
{% endhighlight %}

如果輸入是**List of T** 那裡面的每個T都必須要有實作`Comparable<T>`

1.allPersons: T在這裡是Person 不用多說 Person有實作Comparable<Person> 當然可以跑

2.mixedPerson: T在這裡是Person 有些Person 有些Student 問題在於 Student有沒有實作Comparable<Person>呢 答案是有 因為繼承自Person

3.allStudent: **T在這裡是Student** Student有沒有實作Comparable<Student>呢 答案是沒有 Student只有實作Comparable<Person> 所以不能執行 compile-time爆錯

![Alt text]({{ site.url }}/public/item31-1-1.png)

再看mySort2

{% highlight java %}
public static <T extends Comparable<? super T>> void mySort2(List<T> list)
{% endhighlight %}

如果輸入是**List of T** 那裡面的每個T都必須要有實作`Comparable<T>` 或是`Comparable<T的父類>`

1.allPersons: T在這裡是Person 不用多說 Person有實作Comparable<Person> 當然可以跑

2.mixedPerson: T在這裡是Person 有些Person 有些Student 問題在於 Student有沒有實作Comparable<Person>呢 答案是有 因為繼承自Person

3.allStudent: **T在這裡是Student** Student有沒有實作Comparable<Student>或是Comparable<Person>呢 答案是有 Student有繼承Comparable<Person> 所以可以執行


![Alt text]({{ site.url }}/public/how_to_play.gif)

### 老師我有問題

Q1.老師你偷懶 你的Student在繼承Person的同時 也同時實作Comparable<Student>不就好了嗎

A1.答案是不行 一個類別不能同時實作兩個不同類型的同個介面

![Alt text]({{ site.url }}/public/item31-1-2.png)

你的Student不能同時實作Comparable<Student> 又同時有Comparable<Person>

Q2.不管啦 我就是要實作Comparable<Student>

A2.那你剩兩個選擇 

第一個是Person也有實作Comparable<Person>然後Student不繼承Person 相信這不構成選項 Student沒繼承Person就是兩個完全不相關的類別 mixedPerson也無法創建

第二個選項 Person不實作Comparable<Person>

{% highlight java %}
class Person
{
  protected int age;
  public Person(int age)
  {
    this.age = age;
  }
}
class Student extends Person implements Comparable<Student>
{
  public Student(int age) {
    super(age);
  }
  @Override
  public int compareTo(Student other)
  {
    return this.age - other.age;
  }
}
{% endhighlight %}

{% highlight java %}
mySort1(allPersons);  // 1
mySort1(mixedPerson); // 2
mySort1(allStudent);  // 3
mySort2(allPersons);  // 4
mySort2(mixedPerson); // 5
mySort2(allStudent);  // 6
{% endhighlight %}

那這六個只剩3跟6可以 因為根本沒有人實作Comparable<Person>

### 結論


現在知道為什麼`Comparable<? super T>`這麼重要了吧 因為**一個類別不能同時實作兩個不同類型的同個介面** 
所以最好一個List中的所有人都繼承自最父類的Comparable 

這麼一來`Comparable<? super T>` 這個宣告 就可以在類型安全的前提下 讓你的API最廣泛地被客戶使用

看看Oracle的官方文件[Collection.sort](https://docs.oracle.com/javase/6/docs/api/java/util/Collections.html#sort) 或是其他支援泛型的函式 都是這樣宣告的 相信看到這裡你已經知其然也知其所以然

