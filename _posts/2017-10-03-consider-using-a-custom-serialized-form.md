---
layout: post
title: Effective Java Item75 - Consider using a custom serialized form
comments: True 
subtitle: effective java - 考慮使用自己定義的序列化
tags: effectiveJava
author: jyt0532
---
這篇是Effective Java - Consider using a custom serialized form章節的讀書筆記
本篇的程式碼來自於原書內容

## Item75: 考慮使用自己定義的序列化形式

[Item74](/2017/09/29/implement-serializable-judiciously/)講到 當你宣告一個class有實作序列化 他就必須永遠支援序列化
所以非經過謹慎地思考之前 不要輕易地使用預設的序列化方式 你需要從靈活性 性能 正確性下手比較

且當你自己寫出來的跟default的一樣 你才可以用default的序列化

通常什麼情況下需要用default的序列化呢 就是一個class的**物理表示法跟邏輯內容相同**

比如說下面這個例子 

{% highlight java %}
// Good candidate for default serialized form
public class Name implements Serializable {
  /**
  * Last name. Must be non-null.
  * @serial
  */
  private final String lastName;
  /**
  * First name. Must be non-null.
  * @serial
  */
  private final String firstName;
  /**
  * Middle name, or null if there is none.
  * @serial
  */
  private final String middleName;
  ... // Remainder omitted
}
{% endhighlight %}
邏輯上來說 一個名字包含三個String 物理上來說 就是三個String 這種沒什麼爭議的東西就可以用default的序列化

但如果是下面這個例子
{% highlight java %}
public final class StringList implements Serializable {
  private int size = 0;
  private Entry head = null;
  private static class Entry implements Serializable {
    String data;
    Entry next;
    Entry previous;
  }
  ... // Remainder omitted
}
{% endhighlight %}

這是一個String的List但他用double-linked list實作 那他的物理表示就跟邏輯內容不同

當物理表示跟邏輯內容不同時而你卻還是想用預設的序列化時 有下列四個缺點

1.這個class的API永遠被當前的內部表示法束縛

已剛剛的例子來說 private的Entry變成了API的一部分 即使之後的版本不再用linkedlist實踐 你的input還是永遠是linkedinlist

2.消耗過多空間

以剛剛的例子來說 序列化除了記錄每個element之外 還會序列化所有實作的細節比如說linkedlist 這些都非必要 會讓序列化完的byte stream過大 傳輸浪費


3.消耗過多時間

序列化對於原本的圖形沒有概念 通常需要經過昂貴的traversal

4.導致stack overflow

預設的序列化會跑一個recursive traversal消耗很多空間 在序列化的過程中可能就會把你的stack用完 

所以比較好的物理表達 就是先來一個 string數量 再接其他的string 這樣物理表示跟邏輯內容就一樣
所以當你自己實作序列化時 就該這麼做
{% highlight java %}
public final class StringList implements Serializable {
  private transient int size = 0;
  private transient Entry head = null;

  // No longer Serializable!
  private static class Entry {
    String data;
    Entry next;
    Entry previous;
  }

  // Appends the specified string to the list
  public final void add(String s) {
    // Implementation omitted
  }
  private void writeObject(ObjectOutputStream s) throws IOException {
    s.defaultWriteObject();
    s.writeInt(size);

    for (Entry e = head; e != null; e = e.next)
    	s.writeObject(e.data);
  }
  private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
    s.defaultReadObject();
    int numElements = s.readInt();

    for (int i = 0; i < numElements; i++)
    	add((String) s.readObject());
  }
}
{% endhighlight %}

注意 當我們反序列化這個StringList的時候 我們並不需要size跟head 因為我們都是從0開始建這個StringList 所以這兩個變數都是trasient
代表說這兩個值不需要被儲存起來

那當我所有的instance variable都是trasient的時候 理論上我的readObject跟writeObject是不需要call defaultReadObject跟defaultWriteObject的
但作者給的建議 還是call一下比較好 原因有點複雜 借我30秒

#### 倒數計時

如果你因為所有的instance variable都是transient就不call defaultReadObject/defaultWriteObject的話 如果你下一個版本增加了一個non-transient的變數 那麼可能發生一種情況

你的object在一個拿著新版的JVM中被序列化 但在一個拿著舊版的JVM中被反序列化 那你的那個新的non-transient變數會被忽略 因為你舊版的那個class的readObject中沒有defaultReadObject

看不懂的跳過也沒關係 你就記得always要呼叫defaultReadObject/defaultWriteObject就對了

#### 計時結束

對於StringList的例子 用預設的序列化只是不適合而已 你真的要用也可以 但在某些時候你用預設的就真的會爆

比如說一個Hash table 每一個item應該放在哪一個bucket是透過一個hash function 不妙的是這個hash function在不同的JVM可能不一樣 即使用同一個JVM也不能保證每次都會用同一個函數 所以你用預設的序列化和反序列化可能會還原出完全不一樣的東西

### 盡可能地讓你的變數是transient

每一個你的instance variable都應該仔細想想能不能是transient 代表說你需要序列化的東西越少越好 把所有能transient的都transient之後呢 要注意反序列化之後 那些變數都會是那些資料型態的預設值 無法接受預設值的話 你就必須有個readObject 裡面先defaultReadOject之後再來assign你的變數初始值


{% highlight java %}
private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
  s.defaultReadObject();
  //assign initial value for transient variables  
}

{% endhighlight %}
### 總結
當你決定要實作序列化之後 盡可能的自己定義如何序列化 你也應該花足夠多的時間來決定你怎麼序列化才能合理的描述物件狀態 因為一個錯誤的序列化對於一個class的複雜性和性能會有永久的負面影響
