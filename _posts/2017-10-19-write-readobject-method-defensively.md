---
layout: post
title: Effective Java Item76 - Write readobject method defensively
comments: True 
subtitle: effective java - 保護性的編寫readObject方法
tags: effectiveJava
author: jyt0532
---
這篇是Effective Java - Write readobject method defensively章節的讀書筆記 本篇的程式碼來自於原書內容

在看這篇文章之前 強烈建議先看過[序列化基本知識](/2017/09/27/java-serialization-101/)和[必要時進行保護型拷貝](/2017/09/26/make-defensive-copies-when-needed/)和[深入解析序列化byte stream](/2017/10/12/decrypting-serialized-java-object/)

## Item76: 保護性的編寫readObject方法

在[Item39](/2017/09/26/make-defensive-copies-when-needed/)中介紹了一個不可變的Period Class 因為我們有做Defensive copy 所以我們給了保證 end的時間一定在start之後

以下就是不可變的Period Class

{% highlight java %}
public final class Period implements Serializable {
  private Date start;
  private Date end;
  public Period(Date start, Date end) {
    this.start = new Date(start.getTime());
    this.end = new Date(end.getTime());
    if (this.start.compareTo(this.end) > 0)
      throw new IllegalArgumentException(start + " after " + end);
  }
  public Date start() {
    return new Date(start.getTime());
  }
  public Date end() {
    return new Date(end.getTime());
  }
}
{% endhighlight %}

因為Period的物理表示法跟邏輯內容相同 所以可以用default的序列化 要使用預設的序列化我們只需要加上implements Serializable就可以

但你要是真的這麼做了 Period的物件就有可能違反了當初說好的 end的時間一定在start之後 的約束

為什麼呢 因為readObject可以想成是另一種public constructor 原本的constructor input argument是start跟end 但readObject的input是byte stream

這代表說 如果我們什麼都沒做 直接一五一十的反序列化別人給我們的byte stream 那是很有可能反序列化完後違反了約束 **因為當初我們constructor有的限制在反序列化的時候沒有apply**

如果你[深入的了解Java怎麼序列化](/2017/10/12/decrypting-serialized-java-object/)你就知道要在byte stream裡改變一個物件的instance variable的值是很簡單的

那怎麼辦呢 所以我們還是不能就放任它用預設的反序列化 我們還是得自己寫readObject 
{% highlight java %}
private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
  s.defaultReadObject();   
  if (start.compareTo(end) > 0)
    throw new InvalidObjectException(start +" after "+ end);
}
{% endhighlight %}

一樣 首先先call defaultReadObject 等預設的反序列化跑完之後 再去檢查我們的約束
如果發現這個物件違反了我們當初說好的約束 就噴錯

## 道高一尺

剛剛的解法 讓反序列化後的物件會符合約束 但還有一種可能的攻擊方法
來看下面這個例子
{% highlight java %}
public MutablePeriod() {
    try {
      ByteArrayOutputStream bos = new ByteArrayOutputStream();
      ObjectOutputStream out = new ObjectOutputStream(bos);

      // Serialize a valid Period instance
      out.writeObject(new Period(new Date(), new Date()));

      byte[] ref = { 0x71, 0, 0x7e, 0, 5 }; // Ref #5
      bos.write(ref); // The start field
      ref[4] = 4; // Ref # 4
      bos.write(ref); // The end field

      // Deserialize Period and "stolen" Date references
      ObjectInputStream in = new ObjectInputStream(
          new ByteArrayInputStream(bos.toByteArray()));
      period = (Period) in.readObject();
      start = (Date) in.readObject();
      end = (Date) in.readObject();
    } catch (Exception e) {
      throw new AssertionError(e);
    }
  }
{% endhighlight %}

這個class在序列化之後 再寫了兩次長度是5的byte stream 一次是

71007e0005

一次是

71007e0004

讀的時候呢 讀完Period這個object之後 再接著讀兩個Date object
讀完之後呢 就可以很輕鬆的access到private的值了

{% highlight java %}
MutablePeriod mp = new MutablePeriod();
    Period p = mp.period;
    Date pEnd = mp.end;

    // Let's turn back the clock
    pEnd.setYear(78);
{% endhighlight %}
為什麼可以這樣直接access呢 因為pEnd這個variable存的是p這個物件的end instance的reference

那為什麼會發生這種事呢 因為在序列化的時候 
每寫完一個**物件**或是**Class descriptor** 可愛的JVM都會偷偷的把這個東西的reference記下來 如果接下來要寫的是已經寫過的**物件**或是**Class descriptor** JVM就不會重寫一次 而是直接寫reference serial number

而reference serial number的格式就是71 00 7e 00 [number]

## Period的其他reference分別是什麼

請讀者試著人工deserialize一下 我把bytestream貼在這

![Alt text]({{ site.url }}/public/java76.png)
ref serial #0: **org.effectivejava.examples.chapter11.item76.Period**: Class

ref serial #1: **Ljava/util/Date**: String

ref serial #2: **Period object**: Object reference

ref serial #3: **java.util.Date**: Class

ref serial #4: **end variable**: Object reference

ref serial #5: **start variable**: Object reference

因為在邊反序列化的時候 JVM就邊把已經反序列化的物件或是class descriptor存到memory裡 當然JVM還要maintain一個mapping 之後再遇到一樣東西 就直接從相對應的ref serial number找那個reference 

所以在反序列化完Period之後 memory裡面的第四個人就是end的reference 第五個人就是start的reference 所以我們再用兩個Date 變數去接的話 就可以隨意修改Period裡面的變數

## 魔高一丈
為了避免這樣的問題 readObject裡面也需要defensive copy

{% highlight java %}
private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
  s.defaultReadObject();
  start = new Date(start.getTime());
  end = new Date(end.getTime());
  if(start.compareTo(end) > 0){
    throw new InvalidObjectException(start + " after " + end);
  }
}
{% endhighlight %}

再次提醒 我們是先拷貝 再檢查是否有效 而且用clone不安全 詳細理由請看[Item39](/2017/09/26/make-defensive-copies-when-needed/)

但如果你想要做defensive copy的話 start跟end就不能是final 是final的話在反序列化之後就不能再改了

## 總結

所以什麼時候要自己寫readObject?

如果你的Class可以有一個public constructor 裡面的參數是所有transient的instance variable 且在這個constructor裡不需要任何檢查 可以直接assign的話 那你就不用自己寫readObject 

另一種方法是使用[Serialization Proxy](/501.html)

所以寫reaedObject有什麼訣竅?

1.對於private的變數 要用defensive copy 比如說immutable class裡面的mutable component

2.檢查約束條件的時機是在defensive copy之後 檢查copy到的那個對象

3.剛剛講的約束檢查都是在反序列化的**過程中**發生 但如果你的檢查需要再整個物件或是圖都反序列完後才能做的話(比如說看有沒有cycle) 就要implement ObjectInputValidation這個interface

{% highlight java %}
public interface ObjectInputValidation
{
    public void validateObject()
        throws InvalidObjectException;
}
{% endhighlight %}

把validateObject實作一下 這個function會在你全部反序列化完之後call

4.如果你的Class不是final(代表說可以被繼承)的話 那你的readObject裡面不可以去call可以被overwrite的方法 不論是直接還是間接都不行 理由很簡單 因為parent的constructor會先跑 才跑subclass的constructor 如果在parent的constructor裡去call了subclass的函式 那可能會fail(細節請參考[Item17](/501.html))  因為根本還沒deserialize到subclass

## 後記

為了把這篇看懂 我還讀了非常多書上沒講的序列化的東西 我敢說所有讀過這個章節的人大概有三成不懂序列化原理 五成不懂序列化的reference serial number 八成的人沒有看懂書上的example 恭喜你看完這篇文章 你成為了完全通透這個章節的那兩成java developer
