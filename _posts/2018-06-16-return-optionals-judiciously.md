---
layout: post
title: Effective Java Item55 - 謹慎的回傳Optionals
comments: True 
subtitle: effective java - 謹慎的回傳Optionals
tags: effectiveJava
author: jyt0532
excerpt: 本篇文章介紹何XXXX
---

這篇是Effective Java - Return optionals judiciously章節的讀書筆記 本篇的程式碼來自於原書內容

### Item55: 謹慎的回傳Optionals

在Java8之前 如果你要寫一個方法 這個方法並不是永遠可以回傳你要的值 你有兩種方式處理特殊情況 第一個是拋出錯誤 第二個是回傳null























### 本地方法(Native Method)

什麼是native method 呢 你可能曾經在某些方法的signature中有**native**這個關鍵字 這代表的是這個方法是用其他語言寫的

來看一下[Stackoverflow](https://stackoverflow.com/questions/18900736/what-are-native-methods-in-java-and-where-should-they-be-used)的例子


{% highlight java %}
public class Main {
    public native int intMethod(int i);
    public static void main(String[] args) {
        System.loadLibrary("Main");
        System.out.println(new Main().intMethod(2));
    }
}
{% endhighlight %}
{% highlight cpp %}
#include <jni.h>
#include "Main.h"

JNIEXPORT jint JNICALL Java_Main_intMethod(
    JNIEnv *env, jobject obj, jint i) {
  return i * i;
}
{% endhighlight %}

輸出就會是
{% highlight cpp %}
4
{% endhighlight %}

用C語言寫的方法 回傳的結果可以直接拿來java用 而這個功能是由JVM的Java Native Interface 支持的

為什麼要提供這個東西呢 在之前java還不夠快的時候 某些比較吃性能的方法 都是用其他比較快的語言寫(比如說C) 或者是你想做系統呼叫(System call)等等 java做不到的事 都可以使用本地方法


### Item66: 謹慎的使用本地方法

知道有這個東西 以後別人寫你也可以理解 但是effective java的作者並不提倡這種寫法 因為

1.現在JVM已經越來越快 現在用java直接寫 性能也不輸native method 

2.不安全 如果你的C程式產生了什麼記憶體問題 JVM無法補救 垃圾回收器也無法控管記憶體的使用

3.native method是 platform-dependent 你在不同的平台可能要不同的開發方式甚至執行方式

4.使用native code的context switch也是需要一些性能損耗 要是你的native code很短 那整體性能還會降


### 結論

除非必要 不要用Native Method

如果你真的需要提高性能或是需要訪問OS的資源 千萬要進行全面的測試
