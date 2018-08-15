---
layout: default
title: JVM
---

# 每個程序員都該瞭解的JVM - 3

## 垃圾收集器

垃圾回收的機制 讓開發者不用去關心空間的創建跟回收 大大增加了產能
可惜的是 大部分的JAVA開發者並不清楚到底虛擬機為你做了什麼事

在[每個程序員都該瞭解的JVM]()的垃圾回收器一章中 我們會了解所有垃圾回收器的演算法

為何垃圾回收 

何時垃圾回收

垃圾回收的對象

怎麼回收

全部讓你一看就通 一看就透

![Alt text]({{ site.url }}/public/garbage.gif)

但今天要來跟各位聊聊的 是一個我朋友死裡逃生的故事


### finalize方法

把書裡的觀念用一句話講完 就是每到需要垃圾回收的時候 有一個垃圾回收器會跑可達性分析 跑過所有在**堆**裡的物件 列出一堆目前該被清掉的垃圾 

對於每一個垃圾 都會執行這個物件(的class)的finalize方法 你可以在finalize方法釋放一些你之前跟系統要的記憶體 或是把你打開的檔案關掉 等等

執行完之後 物件就被回收 不再存在在堆裡 堆的空間於是可以再利用

簡單明瞭 看個例子
{% highlight java %}
public class Revive {
  private static B b;
  public static void main(String[] args) throws Exception {
    b = new B();
    cleanB();
  }

  static void cleanB() throws Exception {
    b = null;
    Runtime.getRuntime().gc();
    Thread.sleep(3000);

    if(b != null) {
      System.out.println("Survived: " + b);
    } else {
      System.out.println("Died");
    }
  }

  static class B {
    @Override
    public void finalize() {
      System.out.println("Finalizer of B");
    }
  }
}
{% endhighlight %}

{% highlight txt %}
Finalizer of B
Died
{% endhighlight %}

在呼叫垃圾回收時睡三秒 是因為垃圾回收是另一個thread在跑 而且跑得很慢 優先度又低

所以我們把b設定成null之後 原本的new B()沒有人指到 執行玩finalize()後就死去

### 虎口脫險

但你看到了 再回收之前 我們必須先跑finalize()方法 所以我們有一個機會可以在finalize()方法裡面救人!

{% highlight java %}
public class Revive {
  private static B b;
  public static void main(String[] args) throws Exception {
    b = new B();
    cleanB();
  }

  static void cleanB() throws Exception {
    b = null;
    Runtime.getRuntime().gc();
    Thread.sleep(3000);

    if(b != null) {
      System.out.println("Survived: " + b);
    } else {
      System.out.println("Died");
    }
  }

  static class B {
    @Override
    public void finalize() {
      System.out.println("Finalizer of B");
      b = this;
    }
  }
}
{% endhighlight %}

看到那精美的b = this;

{% highlight txt %}
Finalizer of B
Survived: com.jyt0532.jvm.Revive$B@60e53b93
{% endhighlight %}


跑完finalize之後 垃圾回收器發現 哇靠 你怎麼又有人指到了
好吧 只好繼續讓你活著

這篇文章的重點來了 **每一個物件的finalize()方法只會被執行一次**(不然怎麼叫finalize?)

所以要是你再死第二次 就沒人救得了你 因為不會再跑finalize()方法了
{% highlight java %}
public class Revive {

  private static B b;

  public static void main(String[] args) throws Exception {

    b = new B();

    cleanB();
    cleanB();
  }

  static void cleanB() throws Exception {
    b = null;
    Runtime.getRuntime().gc();
    Thread.sleep(3000);

    if(b != null) {
      System.out.println("Survived: " + b);
    } else {
      System.out.println("Died");
    }
  }

  static class B {

    @Override
    public void finalize() {
      System.out.println("Finalizer of B");
      b = this;
    }
  }
}
{% endhighlight %}
{% highlight txt %}
Finalizer of B
Survived: com.jyt0532.jvm.Revive$B@60e53b93
Died
{% endhighlight %}

這也是為什麼所有人都叫你不要用finalize方法來釋放你的記憶體等等 因為

1.跑finalize()的thread的優先度很低 你不知道什麼時候你的finalize才會被跑

2.你不知道一個物件是第一次跑finalize() 還是之前跑過了 

同樣的程式 不同的行為 最容易造成困惑

### 總結

當跑完可達性分析之後 所有沒被標記到的人 都會被丟進處刑台 此時會一個一個詢問 

1.你有沒有finalize方法
2.你的finalize方法有沒有被執行過

只有你有finalize方法 而且還沒有被執行過的時候 才會把你丟到一個緩刑區 等等再來處理你 

執行緩刑區的優先度非常低 要等到其他人都被清理處刑完 才會輪到緩刑區 這就是為什麼我們需要讓程式睡一下 

等到JVM有空的時候 會去執行每個在緩刑區的物件的finalize方法 你可以在這時候把你之前分配到的記憶體釋放 或是把像上面的例子 把自己救活

所有緩刑區的都跑完之後 會在跑一次小的可達性分析 來看到底哪些要做最後處決
 哪些存活 存活的就不會被回收

當然在finalize裡自救不是個好的寫法 太容易讓人困惑而且性能非常差 這裡舉這個例子是告訴你有這個方法 以及finalize的調用方式

更多精彩內容 敬請關注[每個程序員都該瞭解的JVM]()

{% include jvm_intro.html %}
