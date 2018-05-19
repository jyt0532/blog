---
layout: post
title: Effective Java Item71 - 避免不必要的使用受檢異常
comments: True 
subtitle: effective java - 避免不必要的使用受檢異常
tags: effectiveJava
author: jyt0532
---

這篇是Effective Java - Avoid unnecessary use of checked exceptions章節的讀書筆記 本篇的程式碼來自於原書內容

### Item71: 避免不必要的使用受檢異常

受檢異常是java語言一個很好的特性 也許你會想說發生異常的話 我也可以丟出一個特殊的回傳值 這樣客戶也會知道發生異常了不是嗎？

不同於利用回傳值來代表一個方法的完成情況 拋出受檢異常**強迫**程序員出來面對 
增加了程序的可靠性 但也不可否認 當你過份使用的時候 會給你的使用者造成很多負擔

只有在下列兩個條件都達成的時候 用受檢異常是理想的

1.正確使用API不能阻止異常發生

2.一但發生異常 程序員知道如何處理

否則的話 非受檢異常比較合適

舉個例子 如果你處理異常的方式是這樣

{% highlight java %}
} catch(CheckedException e) {
	e.printStackTrace();
	System.exit(1);
}
{% endhighlight %}

那就不如用非受檢異常 反正程序員也無法做什麼

還有一個常見的作法 就是把你的方法分成兩個方法 第一個方法是看會不會拋出異常(validation) 第二個方法才是真正的方法

{% highlight java %}
//Invocation with state-testing method and unchecked exception
if (obj.actionPermitted(args)) {
  obj.action(args);
} else {
//Handle exceptional condition
}
{% endhighlight %}

這種用法會讓你的調用比較舒服 至少if-else 比try-catch好多了 

但有兩種情況不該這樣重構

1.對於多線程的程式 如果對象在呼叫actionPermitted跟action之間會改變狀態 
2.actionPermitted重複了action所做的事




