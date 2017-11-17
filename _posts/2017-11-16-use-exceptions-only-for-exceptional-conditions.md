---
layout: post
title: Effective Java Item57 - 只針對異常的情況才使用異常
comments: True 
subtitle: effective java - 只針對異常的情況才使用異常
tags: effectiveJava
author: jyt0532
---

這篇是Effective Java - Use exceptionas only for exceptional condiiton章節的讀書筆記 本篇的程式碼來自於原書內容

### Item57: 只針對異常的時候才使用異常

上面那句話乍看是廢話 但你可以看看以下的例子

{% highlight java %}
// Horrible abuse of exceptions. Don't ever do this!
try {
  int i = 0;
  while(true)
    range[i++].climb();
} catch (ArrayIndexOutOfBoundsException e) {
}
{% endhighlight %}

當這個循環出界 就會拋出ArrayIndexOutOfBoundException 然後在catch裡面忽略

那為什麼不直接寫這樣就好呢

{% highlight java %}
for (Mountain m : range)
  m.climb();
{% endhighlight %}

寫出第一段code的人的理由也是很精彩 他認為 其實在access一個list的每一個元素的時候 JVM都會去檢查有沒有出界 既然這個檢查無法避免
那麼 i < list.length()的檢查是非常多餘的 

既然每次都會檢查有沒有出界 那我不如善用這個無法避免的檢查 當你噴exception的時候我再跳出迴圈


這個代碼有幾個大問題

1.因為exception機制的設計是用來處理非正常的情況 所以JVM不會對他進行優化

2.放在try catch裡面的東西本身就很難優化

3.對array iterate的模式不會進行多餘的檢查 在很多JVM上都已經簡化掉了


所以上面那段code 不但模糊了代碼的意圖 降低性能 而且還可能出錯 因為要是有你非預期的exception 比如說clime方法可能也會噴ArrayIndexOutOfBoundsException的話 這個異常就蒸發了

所以結論 不要用異常來處理正常的狀況 異常就指該用在異常情況














