---
layout: post
title: Effective Java Item39 - Make defensive copies when needed
comments: True 
subtitle: effective java - 必要時進行保護型拷貝
tags: effectiveJava
author: jyt0532
---
這篇是Effective Java - Make defensive copies when needed章節的讀書筆記

## Item39: 必要時進行保護型拷貝

JAVA相對於C/C++來說 已經是個很**安全**的語言 
你可以說C基本上就是把所有memory當作一個巨型Array 要非常小心處理memory的問題 但在java你要處理的問題已經少很多了 比如說buffer overflow, array overflow, wild pointer等等 但我們還是得永遠把client想成無惡不赦的壞蛋

來個例子

{% highlight java %}
public final class Period {
   private final Date start;
   private final Date end;

   /**
    * @param start the beginning of the period
    * @param end the end of the period; must not precede start * @throws IllegalArgumentException if start is after end
    * @throws NullPointerException if start or end is null
    */
   public Period(Date start, Date end) {
      if (start.compareTo(end) > 0)
         throw new IllegalArgumentException(start + " after " + end);
      this.start = start;
      this.end   = end;
   }

   public Date start() { return start; }
   public Date end() { return end; }
}
{% endhighlight %}

簡單易懂 我希望Period不能被改變 所以Class本身是final 所有variable是final 而且只有getter沒有setter 這樣不管client多邪惡都不能做壞事了吧?

上面的class簡單易破 只要這樣
{% highlight java %}
Date start = new Date();
Date end = new Date();
Period p = new Period(start, end);
end.setYear(78); // Modifies internals of p!
{% endhighlight %}

崩潰 因為Date這個field本身是mutable 所以其實任何人都可以改變Date的instance 

等等 final不是代表他不可變嗎 為什麼可以改

final代表的是start這個reference只能指到這個傢伙 不能指到其他人 那這個傢伙改變容貌了你也無可奈何

那怎麼辦呢 簡單 改constructor

{% highlight java %}
public Period(Date start, Date end) {
   this.start = new Date(start.getTime());
   this.end   = new Date(end.getTime());
   if (this.start.compareTo(this.end) > 0)
      throw new IllegalArgumentException(start +" after "+ end);
}
{% endhighlight %}
這跟剛剛的差在哪裡呢 這就是本篇的主題 Defensive copy

他並不是直接把input argument 直接assign給instance variable 而是去call start.getTime之後 再丟進Date的constructor

這樣的話剛剛的
{% highlight java %}
end.setYear(78);
{% endhighlight %}
就沒有用了 因為這個end跟Period裡的end不是同一個

注意 **Defensive copy發生在確認input的合理性之前 而且是針對被copy後的對象檢查而不是原本對象**

為什麼要多此一舉呢 要是copy完才發現input不合理 不是很浪費時間嗎

原因很簡單 在multithread的程式中 很常出錯的地方就是確認input沒問題 到copy參數的過程中(這段期間稱為window of vulnerability) 在這段時間內class的state被其他thread改變了是很常見的race condition 所以先copy再檢查是比較好的做法 

但其實改變constructor只解決了一半的問題
{% highlight java %}
p.end().setYear(78); //still fail
{% endhighlight %}

因為Date這個class是mutable 所以如果你直接回傳你的instance variable出去 別人還是可以做壞事

要防禦這種攻擊 要修改我們的兩個getter
{% highlight java %}
public Date start() {
   return new Date(start.getTime());
}
public Date end() {
   return new Date(end.getTime());
}
{% endhighlight %}

一樣 Denfensive copy

做完這些防護措施後 無論使用者多麼卑劣 都不可能違反Period的end一定在start之後的保證

因為除了Period之外 沒有其他人可以碰到這兩個instance variable **真正達到了private的封裝**

### 總結
每當你要寫一個方法或是constructor 確認以下幾點

1.Client給你的input是不是會直接進入到內部的資料結構 如果是

2.看看他給的東西是不是Mutable 如果是

3.你的Class能否容忍這個資料結構被改變 如果不能

4.你能不能保證你的Client不會亂改你的東西 如果不能

那你就該對你的對象進行defensive copy
