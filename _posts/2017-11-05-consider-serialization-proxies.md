---
layout: post
title: Effective Java Item78 - Consider serialization proxies instead of serialized instances
comments: True 
subtitle: effective java - 考慮用序列化代理代替序列化實例
tags: effectiveJava proxy
author: jyt0532
---
這篇是Effective Java - Consider serialization proxies instead of serialized instances章節的讀書筆記 本篇的程式碼來自於原書內容

在看這篇文章之前 建議先看過[Proxy](/2017/10/06/proxy/)

## Item78: 考慮用序列化代理代替序列化實例

[Item74](/2017/09/29/implement-serializable-judiciously/)說到 當你決定要實現Serializable接口的時候 就很容易出錯或是有安全性的問題 有一個方法可以減少出錯的風險 就是序列化Proxy

要怎麼實現呢 首先 我們要先nested class 找一個替身的概念 這個class才是我們要拿來序列化的東西

![Alt text]({{ site.url }}/public/item78-2.png) 

拿我們最愛的Period舉例

所以這個內部class SerializationProxy要有一個constructor 參數就是外部class的物件 Period

{% highlight java %}
public final class Period implements Serializable {
  private final Date start;
  private final Date end;
  public Period(Date start, Date end) {
    this.start = new Date(start.getTime());
    this.end = new Date(end.getTime());
    if (this.start.compareTo(this.end) > 0)
      throw new IllegalArgumentException(start + " after " + end);
  }

  private static class SerializationProxy implements Serializable {
    private final Date start;
    private final Date end;

    SerializationProxy(Period p) {
      this.start = p.start;
      this.end = p.end;
    }

  }
}
{% endhighlight %}

內部的constructor只複製數據 不再做規範檢查或是defensive copy **因為你給的進來的object都應該已遵守規範**

別忘了內部類跟外部類都應該聲明實作Serializable interface

如果你想要自己寫serialVersionUID的話 只需要加在內部類而不需要加在外部 因為內部類才是真正會被序列化的類

那要怎麼樣才能讓JVM序列化內部類呢

### writeReplace

接下來 外部類需要實作writeReplace writeReplace會在序列化之前被call 它回傳的東西就是實際會被序列化的東西

{% highlight java %}
private Object writeReplace() {
  return new SerializationProxy(this);
}
{% endhighlight %}

意思就是 嘿嘿 不要序列化我 序列化SerializationProxy 這個物件

這麼一來我就保證JVM不會序列化Period這個Class 

但是還是可能有壞人從byte stream竄改 產生一個可以反序列化成Period的byte stream 所以我們需要在外圍類裡面加上

{% highlight java %}
private void readObject(ObjectInputStream stream)
    throws InvalidObjectException {
  throw new InvalidObjectException("Proxy required");
}
{% endhighlight %}

這樣要是有人想把一個byte stream反序列化成Period就會直接噴錯

![Alt text]({{ site.url }}/public/item78-3.jpg) 

好 我同意現在你只會序列化SerializationProxy 而且中途有人亂改 我們也不准JVM隨便反序列化成外部class(Period) 

但我要怎麼樣才能把SerializationProxy的byte stream反序列化成Period呢？

### readResolve

答對了 就是用我們可愛的readResolve

{% highlight java %}
private Object readResolve() {
  return new Period(start, end); // Uses public constructor
}
{% endhighlight %}

精彩 不像之前的例子 我們反序列化老半天的東西用都不用 這次我反序列化完之後呢 我只拿有用的資訊 也就是start跟end

ProxyPattern厲害的地方就在 我不用像之前一樣在readObject裡面還要寫跟constructor裡面一樣的確認規範的code 我根本就**重複利用了constructor裡面的規範檢查**

### 總結

用一個序列化代理 增加了很多安全性 但有幾個缺點

1.性能變差 因為要換來換去 作者實驗序列化代理的性能比defensive copy差14%

2.內部類的readResove方法不能call外部類的方法 因為在你call外部類的constructor之前 還沒有外部類的存在

3.如果Period可以被客戶繼承的話 繼承過後的subclass就不能用同一個序列化代理(不能相容)
