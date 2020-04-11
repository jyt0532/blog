---
layout: post
title: 重構 - 改善既有程式的設計 - Primitive Obsession
comments: True
subtitle: 如何重構基本型別偏執
tags: refacforing
author: jyt0532
excerp: 本文介紹重構Primitive Obsessions
---

這篇文章討論《重構 - 改善既有程式的設計》裡的3.9 - Primitive Obsession

圖片以及程式碼來源自[重構 - 改善既有程式的設計](https://www.tenlong.com.tw/products/9789861547534)


## 基本型別偏執

不要執著於使用一堆基本型別(Primitive) 在物件的世界裡該用物件的時候就用物件

比如 Money 類別 就是一個int number跟string currency合在一起

Range 類別 就是兩個int

諸如此類 用物件管理方便得多


## 起因 

因為懶

"我這裡就兩個int搞定 一個表示start 一個表示end 就好了啊 幹嘛還要寫新的類別"

然後一樣 時間久了 一個一個基本型別一直加進去 就越來越難維護

## 解法

來聊聊常見的解法

### Replace Data Value with Object

當你有一個欄位需要支持更多的行為


我們有個`Order`
{% highlight java %}
class Order{
  private String customer;
}
{% endhighlight %}

當你需要紀錄customer的生日或是email 那就可以把這個欄位變成一個物件


{% highlight java %}
class Order{
  private Customer customer;
}
class Customer{
  private String name;
  private Date birthday;
  private String email;
}
{% endhighlight %}

### Get Rid of Type Codes

什麼是Type Code呢 就是一個Class裡面列出了某個欄位的所有可能的值 有點像是enum的概念 比如說血型就是A或B或O或AB

### Replace Type Code with Class

來個`Person`類別 裡面有著血型的type code

{% highlight java %}
class Person {
  public static final int O = 0; 
  public static final int A = 1; 
  public static final int B = 2; 
  public static final int AB = 3;
  private int _bloodGroup;
  public Person (int bloodGroup) { 
    _bloodGroup = bloodGroup;
  }
  public void setBloodGroup(int arg) { 
    _bloodGroup = arg;
  }
  public int getBloodGroup() { 
    return _bloodGroup;
  } 
}
{% endhighlight %}

恩... 這時候應該把血型本身當作一個類別分離出來

{% highlight java %}
class BloodGroup {
  public static final BloodGroup O = new BloodGroup(0); 
  public static final BloodGroup A = new BloodGroup(1); 
  public static final BloodGroup B = new BloodGroup(2); 
  public static final BloodGroup AB = new BloodGroup(3); 
  private static final BloodGroup[] _values = {O, A, B, AB};
  private final int _code;
  private BloodGroup (int code ) { 
    _code = code;
  }
  public int getCode() { 
    return _code;
  }
  public static BloodGroup code(int arg) {
    return _values[arg]; 
  }
}
{% endhighlight %}

搞定 這樣`Person`就清爽多了

### Replace Type Code with SubClasses

剛剛的情況 是`BloodGroup`類別的選擇並不會影響到`Person`類別的行為 那就可以直接分離出一個新類別

但如果現在的type code會決定宿主類別的行為的話 那我們就依賴多型吧

![Alt text]({{ site.url }}/public/primitive-obsession-1.png)

{% highlight java %}
class Employee{
  private int _type;
  static final int ENGINEER = 0; 
  static final int SALESMAN = 1; 
  Employee (int type) { 
    _type = type;
  }
  int getType() { 
    return _type;
  }
}
{% endhighlight %}

不要懷疑 就是多寫`Engineer`類別和`Salesman`類別 然後都去繼承Employee就可以

### Replace Type Code with [State](/2017/05/30/state/)/[Strategy](/2017/04/07/strategy/)

剛剛的情況 是一個type code決定之後 這個物件就一直是那個type的物件 終身不會再改的情況 那就可以用繼承

但你知道 設計模式裡面 有兩個支持動態改變行為的模式 就是 Strategy 跟 State 大家都很熟了就不再多說

重構的結構變這樣

![Alt text]({{ site.url }}/public/primitive-obsession-2.png)

### Replace Array with Object

當你有一個陣列 其中的元素代表不同的東西 那就用物件來替換

{% highlight java %}
String[] row = new String[2];
row[0] = "Liverpool";
row[1] = "15";
{% endhighlight %}

變成這樣

{% highlight java %}
Performance row = new Performance();
row.setName("Liverpool");
row.setWins("15");
{% endhighlight %}

