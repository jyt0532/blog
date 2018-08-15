---
layout: post
title: Effective Java Item7 - 消除過期的對象引用
comments: True 
subtitle: effective java - 消除過期的對象引用
tags: effectiveJava
author: jyt0532
excerpt: 本篇文章介紹為何要消除過期的對象引用來防止內存洩漏
---

這篇是Effective Java - Eliminate obsolete object references章節的讀書筆記 本篇的程式碼來自於原書內容

## Item7: 消除過期的對象引用

當我們從必需人工管理內存分配的語言(C/C++)轉換到Java 程序員的工作變得容易許多 因為有人會幫你把你不再用到的東西垃圾回收

雖然是這樣 但你也不可以過度鬆懈

來個例子 看你能不能看出這個棧的問題

{% highlight java %}
public class Stack {
  private Object[] elements;
  private int size = 0;
  private static final int DEFAULT_INITIAL_CAPACITY = 16;
  public Stack() {
    elements = new Object[DEFAULT_INITIAL_CAPACITY];
  }
  public void push(Object e) {
    ensureCapacity();
    elements[size++] = e;
  }
  public Object pop() {
    if (size == 0)
      throw new EmptyStackException();
      return elements[--size];
  }

  /**
  * Ensure space for at least one more element, roughly
  * doubling the capacity each time the array needs to grow.
  */
  private void ensureCapacity() {
    if (elements.length == size){
      elements = Arrays.copyOf(elements, 2 * size + 1);
    }
  }
}
{% endhighlight %}

在一般的情況下 這個棧會正確的運行 但你仔細看的話 
當你push一個東西再pop 那個東西會一直留在element數組的同一個index 
只要這個index沒有被其他東西覆蓋的話 
你push的那個東西會永遠在那裡 
一直有人指到它 所以就永遠不會被垃圾回收

在極端一點 假設你的棧一般情況下都十個左右個物件 突然某天request變多 同一時間存放了一萬個 然後再慢慢pop出去回到十幾個 你可能會發現你程式的性能越來越低 可是永遠找不到原因 因為有9900個物件從來沒用到 未來也不會用到 但卻沒有被垃圾回收器清掉


### 修復方法

修復方法也很簡單 就是除了把size-1之外 還把指到的東西換成null

{% highlight java %}
public Object pop() {
  if (size == 0)
    throw new EmptyStackException();
    Object result = elements[--size];
    elements[size] = null; // Eliminate obsolete reference
    return result;
}
{% endhighlight %}

### 好習慣

當程序員第一次被這個問題困擾時 可能會讓你寫程式過度小心 對於每一個對象引用 只要沒用到它 就指到null 其實沒有必要這樣

**清空對象引用 是一種例外 而不是一種規範** 

消除過期引用最好的方法 是讓包含該引用的變量結束它的生命週期 如果一直[將局部變量的作用域最小化](/2018/04/05/minimize-scope-of-local-variable/) 那就自然而然地會發生

### 何謂例外

既然都說了清空對象引用應該要是例外 那應該要在什麼情況下清空對象引用呢

我們來仔細研究一下這個Stack

這個Stack類別 一開始先分配好空間 然後棧的0~size-1的element是**有在使用**的空間 size~elements.length的是**沒在使用**的空間 

天知地知你知我知 但是垃圾回收器不知 對垃圾回收器來說 
所有elements的元素都是有在使用的 
所以如果你需要垃圾回收器幫你回收 你就要設成null

結論就是 **只要這個類別是自己管理內存 程序員就應該警惕內存泄露問題**

### 其他內存泄露的原因

內存泄露的第二個常見原因 是cache

一但你把對象引用放進緩存 他就很容易被遺忘 然後就一直存在緩存裡面很久

內存泄露的第三個常見原因 是監聽器(listener)跟回調(callback)

當你提供了一個API給你的客戶去註冊call back(也就是你的API結束之後去呼叫你的客戶) 客戶很常在已經不需要你回調的時候忘記取消註冊

對於這些情況 一個好的解決方式 是保存這些對象引用的[弱引用](/toc/jvm/jvm_4/)

### 總結

由於內存泄露通常不會表現成明顯的失敗 所以他們會一直存在一個系統多年 如果你第一時間沒有查覺 就可能在多年之後發現性能變慢了 才借助Heap剖析工具(Heap Profiler)除錯
