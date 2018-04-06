---
layout: post
title: Effective Java Item45 - 將局部變量的作用域最小化
comments: True 
subtitle: effective java - 將局部變量的作用域最小化
tags: effectiveJava
author: jyt0532
---

這篇是Effective Java - Minimize the scope of local variables章節的讀書筆記 本篇的程式碼來自於原書內容

### Item45: 將局部變量的作用域最小化

比較早期的語言(比如C) 他要求局部變量要在每個方法的一開始聲明宣告 但這個習慣應該要改掉

不要讓一個讀你程式的人在看到一個變量之後 還需要往回滑去找初始值之類的 

**所以應該要在你第一次使用的時候 再宣告**

每個變量宣告的時候就應該被初始化了 如果你宣告的時候還不知道初始值 那就代表你可以晚點再宣告

有一個例外 是你一個變量的初始值是一個函式的返回值

{% highlight cpp %}
int x = getRandomInt();
{% endhighlight %}

但這個函式可能會拋出受檢異常 所以要用try-catch包住
{% highlight cpp %}
try{
  int x = getRandomInt();
}catch(WhateverException e){
  x = 0;
}
{% endhighlight %}

這樣的話 x會跑到範圍外 所以這種情況的話 先宣告而沒有初始值就可以接受
{% highlight cpp %}
int x;
try{
  x = getRandomInt();
}catch(WhateverException e){
  x = 0;
}
{% endhighlight %}

### for和while

如果一個變數只是拿來跑loop而之後不會再用到 那就用for迴圈
因為可以讓一個變數的作用域只在這個迴圈裡

### 函式小而集中

最後一個讓作用域最小化的實踐就是 讓你的每個方法小而集中 只要你每個方法都不超過10行 那你的變數的作用域也不會太大

