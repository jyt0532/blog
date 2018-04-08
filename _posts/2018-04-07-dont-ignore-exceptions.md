---
layout: post
title: Effective Java Item65 - 不要忽略異常
comments: True 
subtitle: effective java - 不要忽略異常
tags: effectiveJava
author: jyt0532
---

這篇是Effective Java - Don’t ignore exceptions章節的讀書筆記 本篇的程式碼來自於原書內容

### Item65: 不要忽略異常

雖然這一條看起來非常直覺 但卻驚人的常常被違反 而需要另外獨立一條Item

當一個API的設計者告訴你一個方法可能拋出某個異常 要把它當一回事

要忽略一個異常非常容易

{% highlight cpp %}
try{
  something();
}catch(WhateverException e){
  //do nothing
}
{% endhighlight %}

空的catch會讓你的異常達成不了目的 本書認為上面的程式碼就猶如把火災警報關掉一樣 沒火災的時候沒事 火災發生時就沒人知道

每次看到catch裡面是空的 要非常警覺 至少要在catch裡面說明為什麼可以忽略

忽略非受檢異常比忽略受檢異常更加危險 這會讓你在真正發生失敗的瞬間放他一馬 等到後面再度失敗時你根本找不到真正的原因是什麼

Fail fast 正確地將異常傳播給外界 才能夠保留有助於調試失敗條件的訊息
