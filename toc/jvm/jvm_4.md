---
layout: default
title: JVM
---

# 每個程序員都該瞭解的JVM - Java中的各種引用

建議先看完[finalize方法](/toc/jvm/jvm_3/)之後 再繼續閱讀本文

## Java中的各種引用

從Java1.2發行以來 弱引用一直就是一個很重要的概念 但大多數的Java開發者都不是很清楚什麼是弱引用 更不用說其他類型的引用 

這篇文章會從垃圾回收的角度談各種引用 使用時機以及如何判斷一個物件會不會被回收

這篇文章相關內容是從我未來的書[每個程序員都該瞭解的JVM]()節錄而來 有興趣的人可以密切關注 期待一下

### 簡介

在[每個程序員都該瞭解的JVM]()的垃圾回收器一章中 我們會了解所有垃圾回收器的演算法 
跑完演算法之後 就會遍歷所有堆中的物件 判定每個物件還有沒有被引用
 
在Java1.2版本之前 對於引用的定義很簡單 有被指到就是有被引用 沒被指到就是沒被引用

這樣遍歷完一次之後 所有在堆的物件只會分兩種 有被引用(非垃圾)跟沒被引用(垃圾) 

但這樣比較不彈性 如果我們跑完一次遍歷之後可以把所有物件分成更多等級 比如說 完全垃圾 垃圾 中等垃圾 等等 

好處是在平常收垃圾的時候 只刪除完全垃圾 
在你的內存非常缺的時候再把中等色垃圾刪掉 
逼不得已的時候 再把輕微垃圾刪掉
不需要每次都全部刪光光 增加了內存管理的彈性

Java有的引用強度由強到弱分別是強引用 軟引用 弱引用以及虛引用
 

### 強引用(Strong reference)
{% highlight java %}
Object obj = new Object();
{% endhighlight %}

這種用法大家都很熟了 不再贅述 只要obj不指到別人 new Object()這個內存就不會被回收

### 軟引用(Soft reference)

先看一下用法

{% highlight java %}
Object obj = new Object();
SoftReference<Object> sobj = new SoftReference<Object>(obj);
obj = null;
{% endhighlight %}

先有個強引用 把強引用丟進軟引用的建構子 這樣sobj就是個指到new Object()的軟引用

從sobj存取這個物件的方式是sobj.get() 很好理解

如果一個物件被一個軟引用指到 那麼只有在**將要發生內存溢出的時候** 才會被垃圾回收器清掉 也就是說 你有事沒事呼叫個 
{% highlight java %}
System.gc();
{% endhighlight %}
這個物件是不會被回收的

### 弱引用(Weak reference)

看一下用法

{% highlight java %}
Object obj = new Object();
WeakReference<Object> wobj = new WeakReference<Object>(obj);
Obj = null;
{% endhighlight %}

用法一模一樣 

弱引用指到的內存在**下一次**垃圾回收的時候就會被回收

弱引用跟軟引用 都可以讓你在程式碼的任意時候 給你一個關心他的機會

{% highlight java %}
Object referent = wobj.get();
if (referent != null) {
    // GC hasn't removed the instance yet
} else {
    // GC has cleared the instance
}
{% endhighlight %}

你用弱引用跟軟引用指到某個對象後 他如果被垃圾回收了 你也沒有要阻止他的意思(不然你就會用強引用指他) 但還給了你一個隨時關心他的機會

當然你可以輕易的清掉你的弱引用

{% highlight java %}
wobj.clear()
{% endhighlight %}

clear之後 new Object()就真的再也沒有人關心了

### 虛引用(Phantom reference)

用法也完全一樣
{% highlight java %}
Object obj = new Object();
PhantomReference<Object> pobj = new PhantomReference<Object>(obj, null);
Obj = null;
{% endhighlight %}

pobj.get() always null
當一個內存被虛引用給引用 那對於垃圾回收器來說 跟沒有被引用是一樣的意思 唯一的差別是當這個內存被回收的時候 你有機會可以得到一個系統通知 你可以想成是一個callback 他會丟一個通知到Reference Queue

更多Reference Queue的講解 請看[Reference Queue]((/toc/jvm/jvm_5/))

### 同一引用路徑上的不同引用

做學問要求甚解 如果一條引用上有各種不同強度的引用 怎麼辦

![Alt text]({{ site.url }}/public/jvm4-1.png)

黑色箭頭是強引用 紅色箭頭是弱引用

光說不練假把戲 把code寫出來

{% highlight java %}
public class WeakOnePath {
  public static void main(String[] args) throws Exception {
    B b = new B();
    WeakReference<B> bRef = new WeakReference<B>(b);
    A a = new A(bRef);
    b = null;

    System.out.println("Run gc");
    Runtime.getRuntime().gc();
    Thread.sleep(1000);

    System.out.println("bRef's referent:" + bRef.get());
    System.out.println("bRef's referent thru a->bRef->B:" + a.getRefB().get());
  }
}

class A {
  private WeakReference<B> bRef;

  public A(WeakReference<B>bRef) {
    this.bRef = bRef;
  }

  public WeakReference<B> getRefB() {
    return bRef;
  }
  @Override
  public void finalize() {
    System.out.println("A cleaned");
  }
}

class B {

  @Override
  public void finalize() {
    System.out.println("B cleaned");
  }
}
{% endhighlight %}

new B() 被垃圾回收器清掉
{% highlight txt %}
Run gc
B cleaned
bRef's referent:null
bRef's referent thru a->bRef->B:null
{% endhighlight %}

可是如果我把所有的WeakReference改成SoftReference
{% highlight txt %}
Run gc
bRef's referent:com.jyt0532.jvm.B@60e53b93
bRef's referent thru a->bRef->B:com.jyt0532.jvm.B@60e53b93
{% endhighlight %}

就不會被清掉

所以 一條引用上 如果有多種不同的引用 就以最弱的那個為準

所以這個例子
![Alt text]({{ site.url }}/public/jvm4-1.png)
new B()就是弱引用

### 不同引用路徑上的引用

![Alt text]({{ site.url }}/public/jvm4-2.png)

黑色箭頭是強引用 紅色箭頭是弱引用 綠色箭頭是虛引用 藍色箭頭是軟引用

如果一個記憶體被多條引用指到 那這記憶體最終引用就是每條引用中最強的那個引用

就是個Max(min())的概念

這個例子中 記憶體B就是弱引用

### 使用時機 

講了那麼多理論部分的知識 現在來講一下各種引用的使用時機


軟引用: cache
因為他只有在記憶體真的不夠了 才會去做垃圾收集 所以很適合拿來做快取

弱引用: 最常見的例子是WeakHashMap 他的Key是WeakReference, Value是跟這個reference相關的資訊 當你map裡的的Key指到的東西被回收 value也跟著不見

虛引用: 用來通知你某個物件已經被回收了 可以讓你清理某些當初分配給他的資源 詳見Reference Queue

完整內容 敬請關注[每個程序員都該瞭解的JVM]()

{% include jvm_intro.html %}
