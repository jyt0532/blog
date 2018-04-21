---
layout: post
title: Effective Java Item13 - 使類和成員的可訪問性最小化
comments: True 
subtitle: effective java - 使類和成員的可訪問性最小化
tags: effectiveJava
author: jyt0532
excerpt: 本篇講解Java裡面 各個訪問級別的使用時機 以及設計一個類別時所要注意的事項
---

這篇是Effective Java - Minimize the accessibility of classes and members章節的讀書筆記 本篇的程式碼來自於原書內容

### Item13: 使類和成員的可訪問性最小化

要區分寫得很好的module跟不好的module最好的方法 在於這塊module對外部而言 是否隱藏了所有實現的細節

設計良好的module會隱藏內部數據和實現細節 把API和實現清楚地分開 只透過API通信 module彼此之間不知道彼此內部的工作情況 這個概念稱為封裝(encapsulation)或是信息隱藏(information hiding)

追求封裝最重要的原因 就是使module之間沒有耦合關係 可以獨立的開發 測試 優化 

追求信息隱藏的第一個原則很簡單 **盡量使每個類或者成員不被外界訪問** 就是讓每個成員能開放的存取權限最小

對於頂層的類和接口 只有兩種可能的訪問類別 public或是package-protected 當你把它宣告成package-protected 你就還有機會在之後的版本改變 替換 刪除它 但若是已經宣告成public 你就有責任永遠維護支持它

對於所有成員 不論是變數或是方法 有四種存取的級別

1.私有(private): 只有同一個類別的成員才可以存取

2.包級私有(package-protected): 也是default的存取級別 跟這個成員同一個package的成員才可以存取

3.受保護的(protected): 除了以上的權限之外 這個成員的子類別也可以存取

4.公有的(public): 任何人都可以存取

所以當你設計一個類別的API之後 其他的方法跟成員都應該預設成私有 
除非你的成員會被同一個package的另一個類別存取 才把它改成包級私有 
如果你發現你的大多數成員都被改成包級私有的話 就代表你可能需要重新思考你的設計 也許你應該分離出某部分的成員到另外一個類別

私有成員和包級私有是這個類別**實作**的一部分 通常不會影響導出的API 但若這個類別有實現serializable接口的話 [這些域就有可能被洩露到導出的API裡面](/2017/09/29/implement-serializable-judiciously/)


當你把一個成員從包級私有升級成保護(protected) 會大大增加了這個成員的可訪問性 因為這個成員已經成為了API的一部分 你必須永遠支持他 一般來說保護是最少用到的


### 繼承

也許你聽過Java的某一個Rule: 

**如果你的方法覆蓋了父類的一個方法 那你的存取級別只能跟父類一樣或是更多的訪問權限** 比如說如果父類有個方法是proteced, 子類覆蓋的方法就只能是proteced或是public

這個原則明顯的跟這篇的主題相違背 我們想要越降低存取權限越好 可是這個限制只會讓類別越繼承權限開放越多 那你知道為什麼要有這個rule嗎 給你想三秒

3

2

1


因為我們要確保Polymorphism 我們希望父類的引用可以保證存取的到子類的方法或成員 為了達到Java的一大賣點 我們只好在這方面妥協

所以 所有實現自接口的方法都會是public 因為接口的每個方法都是public



### 測試

為了測試 把一個私有的變成包級私有的情況 是可以接受的 但超過包級私有就不應該

### 實例域

Instance variable如果不是final的話 絕對不能是公有的 如果是公有的 代表你放棄了管理這個成員的能力 況且這個類別就不再是線程安全

### 靜態域

當你的類別有一些常數 不會被變化的 你可以用public static final來讓這些常數被所有人使用 但要非常注意 這只能對於primitive type或是不可變的對象(immutable class)

如果是一個指到可變對象的引用 代表的是引用本身不變 但被引用的那個東西可變 那很容易會有災難性的後果 

比如說下面的這個就很危險 
{% highlight java %}
// Potential security hole!
public static final Thing[] VALUES = { ... };
{% endhighlight %}

因為數組可以被改變 這讓你的封裝失敗 有兩個解法 

第一個是讓你的數組是private 並用一個不能被更改的List包起來

{% highlight java %}
private static final Thing[] PRIVATE_VALUES = { ... };
public static final List<Thing> VALUES =
  Collections.unmodifiableList(Arrays.asList(PRIVATE_VALUES));
{% endhighlight %}

第二個方法 一樣讓你的數組是private 別人要存取的時候就給一個備份
{% highlight java %}
private static final Thing[] PRIVATE_VALUES = { ... };
public static final Thing[] values() {
  return PRIVATE_VALUES.clone();
}
{% endhighlight %}

當你要選擇兩個方法之一的時候 還要考慮客戶會怎麼用你的資料 返回哪種類型性能會更好

## 總結

你應該盡其所能降低你類別成員的可訪問性 先決條件當然就是了解每個訪問級別的用途以及限制

當你想要公開你的某成員成為public static final時 必須確認這個引用的對象是不可變的
