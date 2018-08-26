---
layout: default
title: JVM
---

# 每個程序員都該瞭解的JVM - Reference Queue

建議先看完[Java中的各種引用](/toc/jvm/jvm_4/)之後 再繼續閱讀本文

## Reference Queue

開宗明義 Reference Queue就是一個放引用的地方 只要這個引用指到的人(referent)已經被垃圾回收器判斷要被回收 這個引用就會被丟進reference queue

看例子

{% highlight java %}
public class GarbageCollectionExample {
  public static void main(String[] args) throws Exception{
    JYTClass strongReference = new JYTClass("Demo");
    ReferenceQueue<JYTClass> referenceQueue = new ReferenceQueue<JYTClass>();
    WeakReference<JYTClass> weakReference = new WeakReference<JYTClass>(strongReference, referenceQueue);

    System.out.println("Now to call gc...");
    Runtime.getRuntime().gc();
    Thread.sleep(1000);
    System.out.println("Any weak references in Queue? " + (referenceQueue.poll() != null));

    strongReference = null;
    System.out.println("Now to call gc...");
    Runtime.getRuntime().gc();
    Thread.sleep(1000);

    System.out.println("Is the weak reference added to the ReferenceQ  ? " + (weakReference.isEnqueued()));
    System.out.println("Does the weak reference still hold the heap object ? " + (weakReference.get() != null));
    Reference<? extends JYTClass> weakRefInQueue = referenceQueue.remove();
    System.out.println("Is this same as original weak reference ? " + (weakRefInQueue == weakReference));
    System.out.println("Is the weak reference added to the ReferenceQ  ? " + (weakReference.isEnqueued()));
  }
}
{% endhighlight %}
來解釋一下這段有趣的程式碼

{% highlight java %}
JYTClass strongReference = new JYTClass("Demo");
ReferenceQueue<JYTClass> referenceQueue = new ReferenceQueue<JYTClass>();
WeakReference<JYTClass> weakReference = new WeakReference<JYTClass>(strongReference, referenceQueue);
{% endhighlight %}

第一段 先宣告一個強引用跟弱引用 和一個reference queue
當然reference queue裡面要放什麼型態的引用都已經定義好 就是只能放 指到JYTClass的 軟引用弱引用跟虛引用

{% highlight java %}
System.out.println("Now to call gc...");
Runtime.getRuntime().gc();
Thread.sleep(1000);
System.out.println("Any weak references in Queue? " + (referenceQueue.poll() != null));
{% endhighlight %}

第一次清理 不會有人進queue 因為還有個強引用指著

接下來把強引用拿掉 呼叫垃圾收集器

{% highlight java %}
strongReference = null;
System.out.println("Now to call gc...");
Runtime.getRuntime().gc();
Thread.sleep(1000);
{% endhighlight %}

接下來就是介紹reference queue怎麼用了

{% highlight java %}
System.out.println("Is the weak reference added to the ReferenceQ? " + (weakReference.isEnqueued()));
{% endhighlight %}

weakReference.isEnqueued() 回傳true 因為weakReference在reference queue裡 
我們可以用這個方法 三不五時地去看看這傢伙指到的人被回收了沒

{% highlight java %}
System.out.println("Does the weak reference still hold the heap object? " + (weakReference.get() != null));
{% endhighlight %}

weakReference.get() 因為東西已經被回收 弱引用已經指不到東西了

{% highlight java %}
Reference<? extends JYTClass> weakRefInQueue = referenceQueue.remove();
{% endhighlight %}

remove會把referenceQueue裡面的東西移除並回傳 你就可以拿到在referenceQueue裡的引用

注意 remove是blocking call 如果你沒有先確認queue裡面有東西就呼叫remove 你的thread會等到queue裡面有東西為止

{% highlight java %}
System.out.println("Is this same as original weak reference ? " + (weakRefInQueue == weakReference));
{% endhighlight %}

weakRefInQueue == weakReference 回傳true 代表說我們宣告的弱引用就是被丟進referenceQueue的那個

{% highlight java %}
System.out.println("Is the weak reference added to the ReferenceQ  ? " + (weakReference.isEnqueued()));
{% endhighlight %}

weakReference.isEnqueued() 現在回傳false 因為我們把它從queue裡拿掉了

跑出來的結果是這樣
{% highlight txt %}
Now to call gc...
Any weak references in Queue? false
Now to call gc...
Is the weak reference added to the ReferenceQ  ? true
Does the weak reference still hold the heap object ? false
Is this same as original weak reference ? true
Is the weak reference added to the ReferenceQ  ? false
{% endhighlight %}

### 救活！

問題來了 那如果我又再finalize()裡面[胡搞 救人](/toc/jvm/jvm_3/) weakReference還存取的到物件嗎


{% highlight java %}
public class GarbageCollectionExample1 {
  private static JYTClass j;
  public static void main(String[] args) throws Exception{
    JYTClass strongReference = new JYTClass("Demo");
    System.out.println("original reference?" + strongReference);
    ReferenceQueue<JYTClass> referenceQueue = new ReferenceQueue<JYTClass>();
    WeakReference<JYTClass> weakReference = new WeakReference<JYTClass>(strongReference, referenceQueue);

    strongReference = null;
    System.out.println("Now to call gc...");
    Runtime.getRuntime().gc();
    Thread.sleep(1000);

    System.out.println("reattached? " + j);
    System.out.println("Does the weak reference still hold the heap object ? " + (weakReference.get() != null));

  }

  static class JYTClass {
    String a;
    JYTClass(String input) {
        a = input;
    }
    @Override
    public void finalize() {
        System.out.println("Finalizer of JYTClass");
        j = this;
    }
  }

}
{% endhighlight %}

輸出結果
{% highlight txt %}
original reference?com.jvm0532.reference.GarbageCollectionExample1$JYTClass@5305068a
Now to call gc...
Finalizer of JYTClass
reattached? com.jvm0532.reference.GarbageCollectionExample1$JYTClass@5305068a
Does the weak reference still hold the heap object ? false
{% endhighlight %}

{% highlight java %}
{% endhighlight %}

完美 基本上只要Reference進了queue之後 就再也無法從reference存取那個物件了 你要救人我阻止不了你 畢竟finalizer的問題很多大家都知道 但你用弱引用或是虛引用跟ReferenceQ來清理資源 就不會有這種問題

### 總結

不要用finalize()方法清理釋放你的資源 會有太多無法預期的行為 你可以使用PhantomReference + ReferenceQueue 這樣程式不但沒有任何搞怪(隨便救人)的機會 你還可以決定你想在什麼時候釋放資源(畢竟finalize的優先度很低 永遠不知道JVM什麼時候有空幫你跑)

常見的做法是再開一個thread 跑一個while迴圈去呼叫referenceQueue.poll 一有東西就可以開始釋放資源
