---
layout: default
title: JVM
---

# 每個程序員都該瞭解的JVM - 2

你可能很常看到這樣的JAVA面試題

{% highlight java %}
String s1 = "aaa";//string literal
String s2 = "aaa";
String s3 = "bbb";
String s4 = new String("aaa");//using new
String s5 = new String("aaa");
{% endhighlight %}

然後問你 

s1 == s2 return true or false

s2 == s4 return true or false

s4 == s5 return true or false

請問一下跑起來output是什麼？

要弄懂之前 要先知道

1.如果你用的是string literal來創建一個新的字串 那麼這個字串本身會被存在字串池 字串池的目的就是節省空間(重複的東西不要重複創建) 

2.如果你用的是new來創建一個新的字串 那麼這個字串會被存在heap

3.== 比較的不是兩邊東西的值 而是兩邊reference的記憶體位置


## Java最高點

Java最高點 心中有圖形 寫程式的最高境界就是程形結合

在別人眼中是程式碼 在JVM高手眼中 是圖形 是內存分配

![Alt text]({{ site.url }}/public/jvm2-1.png)


所以公佈答案 

s1 == s2 return true 因為在創建s2的時候 運行時常量池已經有”aaa”了 就直接用一個address給s2 不再重新創建	

s2 == s4 return false 因為s4在heap裡

s4 == s5 return false 因為s5在heap裡 但和s4不同位置

## intern

![Alt text]({{ site.url }}/public/jvm2-2.png)

另一個常見考題 什麼是intern()呢?

當你對一個字串呼叫intern()時 
他會先去看字串池有沒有這個常量 
有的話就回傳這個常量的地址 
沒有的話 就在字串池生一個字串常量 然後回傳這個常量的地址

{% highlight java %}
String s1 = new String("aaa");
String s2 = s1.intern();
String s3 = "aaa"
String s4 = "bbb";
String s5 = new String("bbb");
{% endhighlight %}

心中有圖形:

![Alt text]({{ site.url }}/public/jvm2-3.png)

事實上 對於任何一個String literal 

String s = "abc";

你都可以想成是 

String s = new String("abc").intern();



這樣子世界就容易許多 當然在堆裡面那個abc會被垃圾回收記回收(因為沒有任何人引用他)

intern()做的事情 就是把這個String拿去String pool找一下有沒有存在過 有的話 就回傳在String pool裡面的引用 沒有的話 就生一個 然後回傳這個新的引用

## 讓我們再深入一點點

{% highlight java %}
String py = "py";
String s1 = "Happy";
String s2 = "Hap" + "py";
String s3 = "Hap" + py;
String s4 = "Hap" + py.intern();
{% endhighlight %}

s1 == s2 => true

s1 == s3 => false

s1 == s4 => false

圖形:

![Alt text]({{ site.url }}/public/jvm2-4.png)

值得一提的有兩點

1.String literal + String literal之後 會再call一次intern() 就像s2一樣

2.在建造s3的時候 因為不知道py的值是什麼 所以根本不可能在compile-time 呼叫intern() 但如果py是final的話 s1 == s3

3.s4也一樣 compile-time根本不知道py是什麼東西

## 讓我們再深入一點點

{% highlight java %}
String s = new String("Happy!")
{% endhighlight %}

請問這行程式 創造了幾個object?

答案是兩個 

它先生了”Happy!”物件到string pool裡面 然後再生一個String object在heap裡

那麼請問
{% highlight java %}
String s1 = new String("Happy!")
String s2 = new String("Happy!")
{% endhighlight %}

請問這行程式 創造了幾個object?

答案是三個 s1跟上一個問題一樣 但跑到s2的時候 因為”Happy!”物件已經在字串池裡 所以不再重複 所以只需要再生一個在heap裡即可

### 內存分配

這篇文章的內容 只是簡略的示範當你了解字串池的概念後 你可以多輕易的把程式轉化成圖像

本書除了堆跟字串池之外 還會講到方法區 虛擬機棧 程序計數器 等等

當你讀完這本書 你會對於java程式更有體會 各種各樣的程式你都了解內部運作以及內存分配的原理
 
{% include jvm_intro.html %}
