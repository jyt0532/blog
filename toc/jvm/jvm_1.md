---
layout: default
title: JVM
---

# 每個程序員都該瞭解的JVM - 1

聽說你Java很熟？考考你。

{% highlight java %}
class Superclass {
  static { System.out.println("9");}
  Superclass () { System.out.println("10");}
  { 
    System.out.println("11");
  }
}
class Subclass extends Superclass {
  static final int var1 = 123;
  static final int var2 = (int) (Math.random() * 10);

  static { System.out.println("6");}
  Subclass () { System.out.println("7");}
  {
    System.out.println("8");
  }
}

public class JVMDemo {
  public static void main(String[] args) throws Exception {
    System.out.println("1");
    System.out.println("2: " + Subclass.var1);
    System.out.println("3");
    System.out.println("4: " + Subclass.var2);
    System.out.println("5");
    new Subclass();
  }
}
{% endhighlight %}

請問一下跑起來output是什麼？

答案是：
{% highlight txt %}
1
2: 123
3
9
6
4: 9(這個數字是 0-9 的隨機數字)
5
11
10
8
7
{% endhighlight %}

## 如果你答對

不好意思浪費了你兩分鐘的時間，你根本是Java神，右上角關閉視窗，感恩。

## 如果你沒答對，但看著答案可以找出合理的解釋

那我相信你是個資深的Java工程師，你可以看這本書的小說部分，幫助你複習相關知識。

## 如果你完全不知道為什麼

恭喜你，2019年年初有一本改變世界的書籍即將上市。熟讀此書，讓你將來寫程式能神擋殺神佛檔殺佛。

{% include jvm_intro.html %}

