---
layout: post
title: Effective Java Item64 - 通過接口引用對象
comments: True 
subtitle: effective java - 通過接口引用對象
tags: effectiveJava
author: jyt0532
excerpt: 本篇文章介紹為何要夠過接口引用對象的好處
---

這篇是Effective Java - Refer to objects by their interfaces章節的讀書筆記 本篇的程式碼來自於原書內容


## Item64: 通過接口引用對象

如果有合適的接口類型存在 那麼對於參數 返回值 變量和域來說 就應該使用接口來**進行聲明** 當你真正使用constructor來**構建某對象**的時候 才真正需要引用這個對象的類

端上例子 Vector是接口List的一個實現 你在聲明變量的時候就要習慣這樣:

{% highlight java %}
// Good - uses interface as type
List<Subscriber> subscribers = new Vector<Subscriber>();
{% endhighlight %}

而不是這樣:

{% highlight java %}
// Bad - uses class as type!
Vector<Subscriber> subscribers = new Vector<Subscriber>();
{% endhighlight %}

養成這個習慣之後 你的程式會更加靈活 當你未來想改變實現 需要做的就是改變構造器的名稱 或是改成另一個靜態工廠

比如說你可以輕鬆改成其他實現

{% highlight java %}
List<Subscriber> subscribers = new ArrayList<Subscriber>();
{% endhighlight %}

而不需要去改變其他地方

要稍微小心的是 如果原來的實現有某些特殊功能並不是接口所要求的 但之後的程序有依賴這個功能 那新的實現也必須要有這個特殊功能

比如說 第一個實現的之後的程序依賴了Vector的同步策略 那就不能輕易地改成ArrayList  這種依賴都要好好寫文檔

### 用類別來引用對象的例外

1.沒有合適的接口

好的例子就是value class 比如說BigInteger或是String 它們通常不會有太多實現 通常是final也不太會有對應的接口 就可以直接使用這個類別來當作參數 返回值 變量和域

2.對象本身是屬於某個框架 而這個框架的基本類型就是類別而非接口

往往這種對象的基類(base class)會是抽象類 比如說java.util.TimerTask

3.A類實作了B接口 除了實現B接口中的所有方法之外 還有其他的方法

如果你需要使用那些除了B接口定義的方法之外的其他方法 那就可以直接引用類別A

### 總結

基本上一個對象有沒有相對應的接口是很明顯的 如果有 那你用接口來引用對象就會讓你的程式靈活很多 如果沒有 那就只能用相關的基類來引用對象

