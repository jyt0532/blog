---
layout: post
title: Effective Java Item8 - 避免使用Finalizer和Cleaner機制
comments: True 
subtitle: effective java - 避免使用Finalizer和Cleaner機制
tags: effectiveJava
author: jyt0532
excerpt: 本篇文章介##
---

這篇是Effective Java - Avoid finalizers and cleaners章節的讀書筆記 本篇的程式碼來自於原書內容

## Item8: 避免使用Finalizer和Cleaner機制

### TLDR

Finalizer 通常是[危險 不可預知 而且沒必要的](/toc/jvm/jvm_3/)

使用終結方法會導致行為不穩定 降低性能 還會產生可移植性的問題

Java 9中使用了Cleaner機制代替了Finalizer機制 但還是不要用它

這篇文章可以看到這裡就好 下面會細講原因XXSSXSX

### finally例子

先來簡單看一下之前常用finally的情況

{% highlight java %}
// try-finally - No longer the best way to close resources!
static String firstLineOfFile(String path) throws IOException {
    BufferedReader br = new BufferedReader(new FileReader(path));
    try {
        return br.readLine();
    } finally {
        br.close();
    }
}
{% endhighlight %}
即使br.readLine()拋出異常 br.close()還是會執行


Java的libraries裡面 有很多需要調用close方法手動關閉的資源 比如說 InputStream, OutputStream 或 java.sql.Connection

至於常用到finalize()的情況 在垃圾回收器相關文章都有提過 如果讀者不清楚如何使用finalize 可以從[這裡](/toc/jvm/jvm_3/)去看

### 缺點

1.不能保證他們能夠及時執行

當你的程式拋出了什麼例外 或是有個物件要被回收 想利用finalize或finally幫你清資源時 你無法保證到底什麼時候會跑finalize跟finally 

比如說上面用finally來關閉文件的例子 你無法預期文件什麼時候會被關閉 那可能就會突然你打開的文件數量超過OS上限

2.不同JVM行為不同

每個JVM實作垃圾回收的演算法也都不同 可能你的程式寫完後在你的電腦完美執行 但在你重要客戶的主機上跑不起來

3.不能保證他們一定會被運行

某些時候 一個程式跑完了 但不可達物件的finalizer還沒跑 也是有可能的

所以要是你在finalizer釋放共享資源的鎖 那就很容易讓你所有程式死掉

4.執行finalizer的異常

當你執行finalizer時如果有拋出異常 那些異常會被忽略 finalizer機制會被終止 無法保證資源到底釋放完了沒


