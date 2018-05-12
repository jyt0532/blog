---
layout: post
title: Effective Java Item44 - 為所有導出的API元素編寫文檔註釋
comments: True 
subtitle: effective java - 為所有導出的API元素編寫文檔註釋
tags: effectiveJava
author: jyt0532
excerpt: 本篇文章說明幾個為什麼要寫文檔以及如何寫文檔
---

這篇是Effective Java - Write doc comments for all exposed API elements章節的讀書筆記 本篇的程式碼來自於原書內容


### Item44: 為所有導出的API元素編寫文檔註釋

如果你想讓一個API可以用 你就必須要寫文檔

Java提供了Javadoc可以讓你更方便的寫文檔 如果你對於文檔註釋的規範還不熟悉 你應該要去了解

### 哪裡寫

如果你的目的是想寫出一個大家想用 好用的API 則每個會被導出的類 接口 構造器  方法 域 的聲明之前 都應該加上文檔

如果你的目的還包含了在未來繼續維護你的類 那就連沒導出的類 接口 構造器  方法 域 都要有文檔

### 寫什麼

1.寫出這個方法與客戶端之間的約定 包含了使用這個方法的前提(必須先完成什麼事) 跟後置條件(調用方法後 什麼條件會被滿足)

一般情況下 前提條件可以藉由@throws 一個非受檢異常來描述

2.每個參數都要有@param標籤 一小句話描述這個參數

3.要有@return標籤(除了return void的方法之外) 一小句話描述回傳值

4.對於可能拋出的異常 用@throws描述那個異常跟什麼時候會拋出這個異常

上個例子


{% highlight java %}
/**
 * Returns the element at the specified position in this list. *
 * 
 * This method is <i>not</i> guaranteed to run in constant
 * time. In some implementations it may run in time proportional * to the element position.
 *
 * @param index index of element to return; must be
 * non-negative and less than the size of this list
 * @return the element at the specified position in this list
 * @throws IndexOutOfBoundsException if the index is out of range
 *         ({@code index < 0 || index >= this.size()})
 */
E get(int index);
{% endhighlight %}

注意 這份文檔包含了一些html標籤 Javadoc會把你的文檔出現的html標籤適當的轉換到html檔案裡 你喜歡你也可以用html拼一個表格放在你文檔裡

注意 @throws子句裡面有@code標籤 這是Javadoc的標籤 他會讓你的代碼片段以代碼形式呈現


### 概要描述

每個文檔的第一句話 成個該元素的概要描述 同一個類或接口的兩個成員 不應該有相同的概要描述 特別是overload的情況

對於方法或構造器 概要描述是個簡短的動詞短語 形容該方法所執行的動作

看個例子 

ArrayList(int initialCapacity) — Constructs an empty list with the spec- ified initial capacity.

Collection.size() — Returns the number of elements in this collection.

對於類 接口和域 概要描述是一個名詞短語 描述了類別的實例 或是域本身所代表的事物

上例子
 
TimerTask — A task that can be scheduled for one-time or repeated execution by a Timer.

Math.PI — The double value that is closer than any other to pi, the ratio of the circumference of a circle to its diameter.


### 泛型 枚舉 註解(Annotation)

在為泛型寫文檔時 確保要在文檔中說明所有的類型參數

{% highlight java %}
/**
 * An object that maps keys to values. A map cannot contain * duplicate keys; each key can map to at most one value.
 *
 * (Remainder omitted)
 *
 * @param <K> the type of keys maintained by this map
 * @param <V> the type of mapped values
 */
public interface Map<K, V> { ... // Remainder omitted
}
{% endhighlight %} 

為枚舉編寫文檔 要確保在文檔中說明常量 以及類型 和任何公有的方法
{% highlight java %}
/**
 * An instrument section of a symphony orchestra.
 */
public enum OrchestraSection {
   /** Woodwinds, such as flute, clarinet, and oboe. */
   WOODWIND,
   /** Brass instruments, such as french horn and trumpet. */
   BRASS,
   /** Percussion instruments, such as timpani and cymbals */
   PERCUSSION,
   /** Stringed instruments, such as violin and cello. */
   STRING;
}
{% endhighlight %} 

為註解(Annotation)類型編寫文檔 描述當程序元素有該註解時 代表什麼含義

{% highlight java %}
/**
 * Indicates that the annotated method is a test method that
 * must throw the designated exception to succeed.
 */
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
   /**
    * The exception that the annotated test method must throw
    * in order to pass. (The test is permitted to throw any
    * subtype of the type described by this class object.)
    */
   Class<? extends Exception> value();
}
{% endhighlight %} 

想更了解註解(Annotation) 請參考[秒懂，Java 註解 （Annotation）你可以這樣學](https://blog.csdn.net/briblue/article/details/73824058) 把註解想成標籤的觀念簡單易懂

### 總結

雖然為所有導出的API元素提供文檔很重要 但不代表這樣就足夠 

如果不同類的很多個方法需要交互合作 那你還必須在各個類別方法的文檔提供其他類別的方法的文檔 這樣其他人單獨閱讀某一個的文檔 還可以知道整個大架構







