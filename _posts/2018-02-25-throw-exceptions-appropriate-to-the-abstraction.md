---
layout: post
title: Effective Java Item73 - 拋出與抽象相對應的異常
comments: True 
subtitle: effective java - 拋出與抽象相對應的異常
tags: effectiveJava
author: jyt0532
---

這篇是Effective Java - Throw exceptions appropriate to the abstraction章節的讀書筆記 本篇的程式碼來自於原書內容

### Item73: 拋出與抽象相對應的異常

如果方法拋出的異常 和這個方法執行的任務沒有太大的關係  這種情形會讓人困惑 這很容易發生在 當異常是由好幾層之後的方法丟出來的情況下 最上層的可能根本看不懂 

比如A call B, B call C 但在C的地方有異常噴出 但在A那邊丟給client的時候client根本看不懂 這個異常是怎麼回事

為了避免這個問題 每一個方法都應該捕捉低一階的異常 然後轉化為這一層看得懂的異常 再丟出去 這個方法就做異常轉譯(Exception translation)

{% highlight java %}
//Exception translation
try {
  //call lower level function
} catch (LowerLevelException e){
  throw new HigherLevelException()
}
{% endhighlight %}

來個現實生活的例子
{% highlight java %}
/**
* Returns the element at the specified position in this list.
* @throws IndexOutOfBoundsException if the index is out of range
* ({@code index < 0 || index >= size()}).
*/
public E get(int index) {
  ListIterator<E> i = listIterator(index);
  try {
    return i.next();
  } catch(NoSuchElementException e) {
    throw new IndexOutOfBoundsException("Index: " + index);
  }
}
{% endhighlight %}

上面的例子 client就不會莫名其妙接收到一個NoSuchElementException 而是我們有好好的轉譯過的IndexOutOfBoundsException

### 異常鍊 

有些時候我們並不想像上面的例子一樣 直接把下層的異常消化掉 我們希望把下層的異常一路往上丟 可是直接不加修飾的丟 又會發生之前的client看不懂的情況

這時候可以用異常鍊Exception Chaining 讓高層的異常可以訪問低層的異常

看到這一part 我完全看不懂effective java在講什麼 經過多番的研究 終於在[Baeldung](http://www.baeldung.com/java-chained-exceptions)看到了比較完整的解釋

端上例子之前 先來看一下什麼是initCause跟getCause

{% highlight java %}
public class ExceptionDemo {
  public static void main(String[] args) {
    try {
      func1();
    } catch(IOException ioe) {
      System.out.println("Caught : " + ioe);
      System.out.println("Actual cause: "+ ioe.getCause());
    }
  }
  static void func1() throws IOException {
    IOException ioe = new IOException();
    ioe.initCause(new ArithmeticException("divide by zero"));
    throw ioe;
  }
}
{% endhighlight %}
很好 func1拋出了一個IOException 但是在拋出之前 我可以再多給一點info 告訴別人說 我的initial Cause其實是ArithmeticException 只是被我包裝過後 才丟出IOException的

跑完Output是這樣

![Alt text]({{ site.url }}/public/exceptiondemo1.png)

這樣即使上層的人看到IOException 也不會霧煞煞 因為他可以getCause看到到底原因是什麼
甚至可以一路getCause下去

有個基本概念之後呢 我們來看一下Exception的constructor

{% highlight java %}
public Exception(String message) {super(message);}
public Exception(String message, Throwable cause) {super(message, cause);}
public Exception(Throwable cause) {super(cause);}
{% endhighlight %}

![Alt text]({{ site.url }}/public/exception_bullet.jpg)

完全瞭然 你要自定義Exception的時候 可以決定要用哪一個constructor 
在等一下要端上的例子中 我們會比較chained exception跟unchained exception的差別 

所以HighLevelException我們overwrite兩個constructor


{% highlight java %}
class HighLevelException extends Exception {
  public HighLevelException(String message, Throwable cause) {
    super(message, cause);
  }
  public HighLevelException(String message) {
    super(message);
  }
}
class LowLevelException extends Exception {
  public LowLevelException(String message) {
    super(message);
  }
}
{% endhighlight %}

上主菜
{% highlight java %}
public class ExceptionChain {
  public static void main(String[] args) throws Exception {
    demo();
  }
  private static void demo() throws HighLevelException {
    try {
      compute();
    } catch (LowLevelException e) {
      throw new HighLevelException("translate to high level");
    }
  }
  private static void compute() throws LowLevelException {
    throw new LowLevelException("divide by zero");
  }
}
{% endhighlight %}

這個是unchained的例子 在demo這個方法裡面自己消化了LowLevelException 轉化成HighLevelException

Output如下

![Alt text]({{ site.url }}/public/exceptiondemo2.png)

現在來看看chained exception

![Alt text]({{ site.url }}/public/exceptiondemo4.png)

跟剛剛唯一的差別就是我們創建HighLevelException時用了不同的constructor


Output如下
![Alt text]({{ site.url }}/public/exceptiondemo3.png)

這就是effective java想表達的概念

### 總結

如果有辦法避免來自低層的異常 就在呼叫低層方法之前先確認要給進去的參數

如果不能處理或是避免來自低層的異常 就用異常轉譯 

但如果低層的異常可以給高層很多info 那就可以用異常鍊 把異常傳到上層 這不但讓你成功包裝了下層的異常 還讓接受到異常的人可以捕獲到低層的異常進而分析


