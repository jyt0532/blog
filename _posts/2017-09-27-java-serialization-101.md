---
layout: post
title: Effective Java - 序列化
comments: True 
subtitle: effective java - 序列化基本知識
tags: effectiveJava
author: jyt0532
---
這篇文章是閱讀Effective Java 第11章 - Serialization之前需要會的知識 
本篇的程式碼來源是[Tutorial Point](https://www.tutorialspoint.com/java/java_serialization.htm)

## 何謂序列化

序列化就是把一個java object轉變成一個byte stream 這個byte stream紀錄著所有關於這個object的所有訊息 比如說這個object是什麼型態 包含了什麼variable等等

變成byte stream後就可以拿來存儲和傳輸 你可以放在Disk裡面 等到需要的時候再把它拿出來反序列化 反序列化的程式會把byte stream建造回object

這些過程都是JVM independent 代表說可以發生在不同的JVM上面

## ObjectInputStream和ObjectOutputStream 

ObjectOutputStream可以把一個object序列化成byte stream

ObjectInputStream可以把一個byte stream反序列化成object

其中最重要的method分別是writeObject和readObject

{% highlight java %}
public final void writeObject(Object x) throws IOException
{% endhighlight %}

很好懂 input一個object 

{% highlight java %}
public final Object readObject() throws IOException, ClassNotFoundException
{% endhighlight %}
也很好懂 回傳一個object

## 例子
{% highlight java %}
public class Employee implements Serializable {
  public String name;
  public String address;
  public transient int SSN;
  public int number;

  public void mailCheck() {
    System.out.println("Mailing a check to " + name + " " + address);
  }
}
{% endhighlight %}

一個Class要能被序列化 需要滿足兩個條件

1.有實作Serializable

2.每個instance variable 要碼是可以被序列化 要碼是transient

trasient代表說這個東西不需要被序列化 代表說這個variable不是這個物件的persistent state的一部分

{% highlight java %}
public class Se {
  public static void main(String [] args) {
    Employee e = new Employee();
    e.name = "Reyan Ali";
    e.address = "Phokka Kuan, Ambehta Peer";
    e.SSN = 11122333;
    e.number = 101;

    try {
      FileOutputStream fileOut =
  	new FileOutputStream("tmp.txt");
      ObjectOutputStream out = new ObjectOutputStream(fileOut);
      out.writeObject(e);
      out.close();
      fileOut.close();
      System.out.printf("Serialized data is saved in tmp.txt");
    }catch(IOException i) {
      i.printStackTrace();
    }
  }
}
{% endhighlight %}

e這個物件被我們序列化 而且存到了tmp.txt

你可以把這個tmp.txt夾帶在email裡面寄給別人 只要別人的電腦裡有Employee 他就可以反序列化
{% highlight java %}
public class De {
  public static void main(String [] args) {
    Employee e = null;
    try {
      FileInputStream fileIn = new FileInputStream("tmp.txt");
      ObjectInputStream in = new ObjectInputStream(fileIn);
      e = (Employee) in.readObject();
      in.close();
      fileIn.close();
    }catch(IOException i) {
      i.printStackTrace();
      return;
    }catch(ClassNotFoundException c) {
      System.out.println("Employee class not found");
      c.printStackTrace();
      return;
    }

    System.out.println("Deserialized Employee...");
    System.out.println("Name: " + e.name);
    System.out.println("Address: " + e.address);
    System.out.println("SSN: " + e.SSN);
    System.out.println("Number: " + e.number);
  }
}
{% endhighlight %}

Standard Output長這樣
{% highlight shell %}
Deserialized Employee...
Name: Reyan Ali
Address: Phokka Kuan, Ambehta Peer
SSN: 0
Number: 101
{% endhighlight %}

注意兩個地方

1. readObject回傳一個object 必須cast成Employee

2. IOException發生在讀檔案有問題的時候 ClassNotFoundException發生在找不到Empoloyee的時候

3. SSN是0 因為我們說他是transient **所以SSN並沒有被記錄在byte stream裡** 所以在readObject的時候就assign了整數的default value

那如果我想要有自己的default value 該怎麼辦呢

只要在Employee 裡面寫一個readObject method
{% highlight java %}
private void readObject(ObjectInputStream in) throws IOException, ClassNotFoundException {
  in.defaultReadObject();

  SSN = 5;
}
{% endhighlight %}

很好懂 他做完該做的反序列化後 再把SSN設成你想要的值


以上就是你在讀Effective Java第十一章之前 需要懂的東西

