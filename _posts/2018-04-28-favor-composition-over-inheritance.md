---
layout: post
title: Effective Java Item16 - 復合優先於繼承
comments: True 
subtitle: effective java - 復合優先於繼承
tags: effectiveJava
author: jyt0532
excerpt: XXXXXXXXXXXXXXXXXXXXXXXXX
---

這篇是Effective Java - Favor composition over inheritance章節的讀書筆記 本篇的程式碼來自於原書內容

類似的主題 在[設計模式](/toc/design_pattern/) 中也是再三強調 非常重要


### Item16: 復合優先於繼承

繼承(Inheritance) 是實現代碼重用的常見方法 但他並非是唯一的工具 用的不好的話 會使你的類別非常脆弱

繼承的一個最大的缺點 就是打破了**封裝性** 也就是說 子類依賴於父類的實作細節的話 如果某個新版本中父類的實作細節變了 子類也會被破壞 所以當你使用繼承的時候 你就要一直注意你的父類的更新 並且以此決定你需不需要跟著更新

![Alt text]({{ site.url }}/public/item77-3.jpg)

上例子 假設我們想要紀錄一個HashSet總共自從被創建以來 被添加過幾次 我們用一個InstrumentedHashSet去繼承它

因為HashSet有兩個添加函數 add 和 addAll 所以這兩個都要複寫

{% highlight java %}
// Broken - Inappropriate use of inheritance!
public class InstrumentedHashSet<E> extends HashSet<E> {
  // The number of attempted element insertions
  private int addCount = 0;
  public InstrumentedHashSet() {
  }
  public InstrumentedHashSet(int initCap, float loadFactor) {
    super(initCap, loadFactor);
  }
  @Override public boolean add(E e) {
    addCount++;
    return super.add(e);
  }
  @Override public boolean addAll(Collection<? extends E> c) {
    addCount += c.size();
    return super.addAll(c);
  }
  public int getAddCount() {
    return addCount;
  }
}
{% endhighlight %}


這個類別最大的問題 在於你的正確性依賴於父類的實作 如果舊的版本的HashSet裡面 addAll就是自己把某個collection的東西加到HashSet裡 那你的子類就會有正確的行為 但若新的版本裡面 
他們覺得 addAll其實可以loop這個collection 對於每個元素都呼叫add就好 那你的子類就爆錯了

看一下為什麼 假設你InstrumentedHashSet加了三個元素

{% highlight java %}
InstrumentedHashSet<String> s =
  new InstrumentedHashSet<String>();
s.addAll(Arrays.asList("Snap", "Crackle", "Pop"));
s.getAddCount() // return 6
{% endhighlight %}

InstrumentedHashSet.addAll加了3 之後 呼叫super.addAll 然而父類改變實作 super.addAll呼叫三次super.add super.add呼叫InstrumentedHashSet.add 三次 所以總共是6

父類改變實作是非常常見的事 他也不需要在文檔寫得很清楚他的實作細節 因為你繼承了它 你就要有責任搞清楚他的實作會不會影響到你

解法一: 你可以說 簡單 我就不要複寫addAll就好了 客戶直接呼叫HashSet的addAll這樣就會得到正確的結果了

那也是因為你知道了HashSet的addAll實作依賴了HashSet.add 才會有正確結果 如果他明天又改回去 你的結果又不正確了

解法二: 那你會說 真麻煩 那我的InstrumentedHashSet.addAll就自己遍歷就好 不管HashSet是怎麼實作 我都會對

這個方法有兩個問題 

1.你等於是重新實現一次addAll方法 就本末導致 沒利用到繼承的優勢 code reuse

2.你不能存取父類的私有域 有些方法無法實現

解法三: 既然override問題那麼多 我自己加一個新的方法addAllElements總可以吧 

世事難料 如果父類的下一個版本有新增一個一樣名字的方法 那就回到剛剛的兩個問題


高譚淪陷 英雄登場

### 復合(Composition)

我們不去擴展原有的類 而是在新類中加上一個成員(私有域) 這個成員指到舊成員的一個實例

因為舊類變成了新類的一個成員 新類的每一個方法都可以呼叫舊類的任何方法 稱為轉發(forwarding) 新類的方法稱為轉發方法(forwarding method)

用剛剛的用一個新的CountingSet當例子
{% highlight java %}
public class CountingSet<E> {
  private int addCount = 0;
  private final Set<E> s;

  public CountingSet(Set<E> s) {
    this.s = s;
  }	
  public boolean add(E e) {
    addCount++;
    return s.add(e);
  }
  public boolean addAll(Collection<? extends E> c) {
    addCount += c.size();
    return s.addAll(c);
  }
  public int getAddCount() {
    return addCount;
  }

  public static void main(String[] args) {
    CountingSet<String> s = new CountingSet<String>(new TreeSet<String>());
    s.addAll(Arrays.asList("Snap", "Crackle", "Pop"));
    System.out.println(s.getAddCount());//3
  }
}
{% endhighlight %}

就是這麼直觀 Effective Java書上的例子過早最佳化 讓讀者難以理解這個主題想要表達的東西 
看我的例子一目瞭然

再看一眼我們怎麼宣告這個Class

{% highlight java %}
CountingSet<String> s = new CountingSet<String>(new TreeSet<String>());
{% endhighlight %}

(這裡當然也可以丟HashSet進去 任何一種Set都可以)

注意 CountingSet其實就是把另一個Set給包裝起來了 你任意給我一個Set 
我就回傳另一個**多了個新功能**的Set 有沒有覺得看起來很眼熟 本部落格的忠實讀者應該已經看出來
其實這個例子就是一個[裝飾模式](/2017/04/18/decorator/)

## 總結

當你想讓B繼承A的時候 先問自己每個B都確實是A嗎 如果這個問題是不確定的 那就不應該用繼承 應該用復合

即使每個B都確實是A 再問最後一個問題 A是不是沒有缺陷的一個類 如果A有缺陷的話 你願不願意讓你的B有同樣的缺陷 因為繼承會把所有的缺陷都傳播到子類上 但復合可以允許設計新的API來隱藏這些缺陷


建議同時把這兩個設計模式讀懂:[策略模式](/2017/04/07/strategy/)跟[裝飾模式](/2017/04/18/decorator/)






