---
layout: post
title: Design Pattern(6) - Singleton
comments: True 
subtitle: design pattern - singleton
tags: designPattern
author: jyt0532
---
{% include design_pattern.html %}

### 徐志摩 《致梁啟超》

> 我將於茫茫人海中訪我**唯一**靈魂之伴侶,得之,我幸;不得,我命,如此而已。或得則吾生，不或則吾滅。
>
>
>
>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;徐志摩 《致梁啟超》

有的時候我們必須確保靈魂伴侶只能有一個 該怎麼做呢 這篇文章帶你慢慢解析

最多只有一個instance 代表說我不能讓任何人想建instance就建instance
唯一解法就是需要private constructor 這個constructor只有這個class本身的function才可以call

{% highlight java %}
public class SoulMate{
    private SoulMate() {}
}
{% endhighlight %}

那你還是要給我一個public function讓我生instance

{% highlight java %}
public class SoulMate{
    private SoulMate() {}
    public SoulMate getInstance(){
	return new SoulMate();
    }
}
{% endhighlight %}

不對 你要call getInstance()才會得到物件 可是你沒有物件你怎麼call getInstance()呢
這不是雞生蛋蛋生雞的問題嗎

答對了 用static

{% highlight java %}
public class SoulMate{
    private SoulMate() {}
    public static SoulMate getInstance(){
	return new SoulMate();
    }
}
{% endhighlight %}

static這個字代表這個function是class level的 不用物件也可以call

但這樣每call一次getInstance() 就會多一個物件 怎麼辦

我們用一個變數紀錄一下這個物件到底被創了沒 沒被創建過就創它 有被創建過就不要創建它

{% highlight java %}
public class SoulMate{
    private SoulMate uniqueInstance;//Initialized to null
    private SoulMate() {}

    public static SoulMate getInstance(){
	if(uniqueInstance == null){
            uniqueInstance = new SoulMate();
	}
	return uniqueInstance;
    }
}
{% endhighlight %}

嗯皆大歡喜 可是這樣的code不會compile過 因為static function不可以access non-static variable

{% highlight java %}
public class SoulMate{
    private static SoulMate uniqueInstance;
    private SoulMate() {}

    public static SoulMate getInstance(){
	if(uniqueInstance == null){
            uniqueInstance = new SoulMate();
	}
	return uniqueInstance;
    }
}
{% endhighlight %}

實在囉唆 這樣總可以了吧 而且這是lazy loading 只有在getInstance被call至少一次 才會去創建uniqueInstance
如果從頭到尾沒有人call過 那uniqueInstance就不會被創建 省時間空間 

這份code在大多數情況可以 但如果是multi-thread的話 
有可能thread1在if condition過了以後在創建實體的同時有另一個thread2進來 
這樣就爆了 生出兩個instance

那話不多說 直上synchronized

{% highlight java %}
public class SoulMate{
    private static SoulMate uniqueInstance = new SoulMate();
    private SoulMate() {}

    public static synchronized SoulMate getInstance(){
        if(uniqueInstance == null){
            uniqueInstance = new SoulMate();
        }
	return uniqueInstance;
    }
}
{% endhighlight %}

這代表說一次只有一個thread可以進getInstance 但這樣效率極低 synchronized是非常吃資源的關鍵字
就有點像是function的開始lock一個mutex function的結束unlock一樣 

有一個比較簡單的方法 

{% highlight java %}
public class SoulMate{
    private static SoulMate uniqueInstance = new SoulMate();
    private SoulMate() {}

    public static SoulMate getInstance(){
	return uniqueInstance;
    }
}
{% endhighlight %}

JVM會在任何thread有機會進來之前就先建好這個instance 這個getInstance就是always return 那個事先生好的instance

但人生沒有在全拿的 這樣就不是lazy-loading
這樣是eager-loading 就是我先把物件給生好 但這樣要是從頭到尾沒有人call getInstance的話
就浪費時間跟空間 這就是你的application要取捨的地方了

但如果你要同時解決race condition又要lazy loading又不希望用synchronized 降低效率
還是有辦法

仔細分析一下 為什麼直接上synchronized效率會低 因為每次call getInstance我們都block住 但其實沒有必要 只有第一次在new的時候
我們才需要擔心race condition的問題 生完之後 他愛怎麼call我就怎麼直接return 所以

{% highlight java %}
public class SoulMate{
    private static SoulMate uniqueInstance;
    private SoulMate() {}

    public static SoulMate getInstance(){
	if(uniqueInstance == null){
	    synchronized(SoulMate.class){
            	uniqueInstance = new SoulMate();
	    }
	}
	return uniqueInstance;
    }
}
{% endhighlight %}

這個的意思是說 我並不想每次有人call getInstance我就synchronized 我先看uniqueInstance有沒有被生過
有被生過我根本不用block 我就直接return 沒被生過的話 我再把這個class block住 慢慢生

這樣還是會有race condition的問題 如果第一個thread正要new的時候 第二個thread到if 發現沒東西 進if condition
要synchronized之前 第一個thread因為還在生所以第二個thread被block住 第一個thread生完後release lock 
換第二個thread進場new 那這樣就會有兩個

解法就是當第二個thread拿到鎖之後(也就是synchronized裡) 再確認一次instance還是不是null


{% highlight java %}
public class SoulMate{
    private static SoulMate uniqueInstance;
    private SoulMate() {}

    public static SoulMate getInstance(){
	if(uniqueInstance == null){
	    synchronized(SoulMate.class){
		if(uniqueInstance == null){
            	    uniqueInstance = new SoulMate();
		}
	    }
	}
	return uniqueInstance;
    }
}
{% endhighlight %}


精彩的來了 這樣還是會有個問題 

![Alt text]({{ site.url }}/public/inceptionthirdpanel-lets-go-one-level-deeper.jpg)

就是當thread1正在new的時候 他**有可能**先跟記憶體allocate了一些空間後 
才開始執行constructor(建這個Singleton instance需要的東西) 所以有可能thread1還在跑constructor的時候thread2進到第一個if發現不是null 
就直接回傳給別人 別人就直接開始用 但是thread1根本就construct到一半而已

這時候只需要把這個instance加一個精美的關鍵字 volatile

{% highlight java %}
public class SoulMate{
    private volatile static SoulMate uniqueInstance;
    private SoulMate() {}

    public static SoulMate getInstance(){
	if(uniqueInstance == null){
	    synchronized(SoulMate.class){
		if(uniqueInstance == null){
            	    uniqueInstance = new SoulMate();
		}
	    }
	}
	return uniqueInstance;
    }
}
{% endhighlight %}

volatile通常出現在multi-threading的code裡面 代表著這個變數很不穩定 
加上這個變數之後給了我們兩個保證

1.對於這個變數的寫 會保證寫進memory(所以其他thread會看到最新的值) 對於這個變數的讀 會保證從memory讀

2.JVM 跑compiler optimization的時候 不可以隨便改變volatile變數的順序

其實本來打算再開一篇文寫volatile 可是在找資料的時候發現了這篇文[Java Volatile Keyword](http://tutorials.jenkov.com/java-concurrency/volatile.html) 這篇寫得實在清楚 我看著看著也覺得我寫的不會比他清楚 大家有興趣可以讀一下這篇

總之 volatile的變數 即使不在synchronized的block裡 我的讀會保證其他的thread寫完之後

這就是赫赫有名的Double checked locking 

### Singleton

**保證一個class只會有最多一個instance 同時提供一個存取方法**

### 結構

![Alt text]({{ site.url }}/public/singleton1.png)

* Singleton: 定義一個getInstance函式讓外界可以存取 外界也只有這個方法可以存取

### 優缺點

1.只有一個存取instance的方法 方便maintain管控 也可以嚴格控制client要怎麼access 或什麼時機可以access

2.**最多一個**instance的限制可以輕鬆地變成**最多n個**

3.很難擴展 很難被extend 因為沒有任何抽象層

4.違背**單一職責原則** singleton class不但包含了工廠 還包含了business logic 一個class的責任過多

### 後記

最近越來越多關於singleton的討論 甚至有不少人覺得這是anti-pattern 基本上是見仁見智 
不過要小心很多人會錯誤的利用它來包裝global variable

