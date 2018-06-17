---
layout: post
title: Effective Java Item10 - 覆蓋equals請遵守通用規範
comments: True 
subtitle: effective java - 覆蓋equals請遵守通用規範
tags: effectiveJava
author: jyt0532
excerpt: 本篇文章介紹編寫新的類別的 覆蓋equals的時機以及如何覆蓋equals
---

這篇是Effective Java - Obey the general contract when overriding equals章節的讀書筆記 本篇的程式碼來自於原書內容

## Item10: 覆蓋equals請遵守通用規定

覆蓋equals方法看似簡單 但有許多覆蓋方式會導致錯誤 最簡單的避免方式 就是不要覆蓋equals 那就代表說每個實例只會和自己一樣 不會和別人一樣(即使所有內部變量的值都一樣)

## 什麼時候不應該覆蓋equals

只要滿足以下任一條件

1.類別的每個實例 本質上都是唯一的

比如說Thread 代表的就是一個實體而不是一個**值**

2.類別不需要提供 **邏輯相等** 的測試

比如說java.util.regex.Pattern這個類別 這個類別也許可以覆蓋equals 讓客戶比較兩個regex expression是不是相等 但是並沒有客戶會這麼做 所以Pattern類別就沒有覆蓋equals

3.超類已經覆蓋了equals 以超類繼承過來的行為對於子類也是合適的

比如說 Set從AbstractSet繼承了equals,
List從AbstractList繼承了equals,
Map從AbstractMap繼承了equals

以上任一條件滿足 你就不應該要覆蓋equals

當然你也可以極端一點 你的類別不想讓任何人呼叫equals
你也可以直接拋出例外

{% highlight java %}
@Override public boolean equals(Object o) {
  throw new AssertionError(); // Method is never called
}
{% endhighlight %}


## 什麼時候應該覆蓋equals

通常是值類(value class) 比如說Integer或是Date 

當程序員在呼叫equals的時候 他們希望知道他們在邏輯上是否相等 而不是想知道**他們是不是指到同一個對象** 當你的客戶有這種需求時 你就必須覆蓋equals 

如果你不這麼做 你甚至無法把你的類別放進Map的key或是Set的key

不過有一種值類不需要覆蓋equals 就是有實例控制的類別(比如說最多只能存在一個實例) 比如說[Enum](/2017/10/20/enforce-the-singleton-property-with-a-private-constructor-or-enum/) 對於這種類來說 邏輯相同 跟對象等同 是同一件事

## 覆蓋的約束

當你決定要覆蓋equals的時候 你必須要遵守通用約定如下

**equals方法必須實現等價關係(equivalence relation)**

等價關係代表

1.自反性(reflexive): 對於非null的值x x.equals(x) 回傳true

2.對稱性(symmetric): 對於非null的值x和y 如果x.equals(y) 回傳true 那 y.equals(x) 回傳true

3.傳遞性(transitive): 對於非null的值x和y和z 如果x.equals(y) 回傳true 且 y.equals(z) 回傳true 則x.equals(z) 回傳true

4.一致性(consistent): 對於非null的值x和y 多次呼叫x.equals(y)應該一直回傳同樣的值

5.非空性(Non-nullity): 對於非null的值x x.equals(null) 返回false

只要違反了任一一個 你的程序就會表現不正常

## 違反的下場和例子

### 自反性

只要有任何一個x違反這個要求 則你把x放進任意的collection 第二次的時候 他的contains(x)會回傳false 你就可以再放進去一次 因為collection以為這東西沒出現過

### 對稱性

來看個可愛的例子 我們有一個不管大小寫的字串

{% highlight java %}
public final class CaseInsensitiveString {
  private final String s;
  public CaseInsensitiveString(String s) {
    if (s == null)
      throw new NullPointerException();
    this.s = s;
  }
}
{% endhighlight %}

它覆蓋了equals
{% highlight java %}
@Override 
public boolean equals(Object o) {
  if (o instanceof CaseInsensitiveString)
    return s.equalsIgnoreCase(((CaseInsensitiveString) o).s);
  if (o instanceof String) // One-way interoperability!
    return s.equalsIgnoreCase((String) o);
  return false;
}
{% endhighlight %}

看起來挺正常的 現在來一個正常的字串跟一個不管大小寫的字串
{% highlight java %}
CaseInsensitiveString cis = new CaseInsensitiveString("Polish");
String s = "polish";
cis.equals(s) // return ture
s.equals(cis) // return false
{% endhighlight %}

因為我們知道CaseInsensitiveString的equals裡面有特別處理String的case

但String哪看得懂什麼是CaseInsensitiveString? 

所以正確的複寫方法應該是

{% highlight java %}
@Override public boolean equals(Object o) {
  return o instanceof CaseInsensitiveString &&
    ((CaseInsensitiveString) o).s.equalsIgnoreCase(s);
}
{% endhighlight %}


### 傳遞性

有趣的來了 先來一個Point類別

{% highlight java %}
public class Point {
  private final int x;
  private final int y;

  public Point(int x, int y) {
    this.x = x;
    this.y = y;
  }

  @Override
  public boolean equals(Object o) {
    if (!(o instanceof Point))
      return false;
    Point p = (Point) o;
    return p.x == x && p.y == y;
  }
}
{% endhighlight %}

現在我想繼承Point之後 加一個變量 代表這個點的顏色
{% highlight java %}
public class ColorPoint extends Point {
  private final Color color;

  public ColorPoint(int x, int y, Color color) {
    super(x, y);
    this.color = color;
  }
}
{% endhighlight %}

端上最直觀的equals

{% highlight java %}
@Override
public boolean equals(Object o) {
  if (!(o instanceof ColorPoint))
    return false;
  return super.equals(o) && ((ColorPoint) o).color == color;
}
{% endhighlight %}

剛剛說過了 這個解法會違反對稱性

可不可以我們在equals裡面判斷 如果傳進來的是ColorPoint 我們就比較x, y 跟color

除果傳進來的是Point 我們就只比較x, y
{% highlight java %}
@Override public boolean equals(Object o) {
  if (!(o instanceof Point))
    return false;

  // If o is a normal Point, do a color-blind comparison
  if (!(o instanceof ColorPoint))
    return o.equals(this);

  // o is a ColorPoint; do a full comparison
  return super.equals(o) && ((ColorPoint)o).color == color;
}
{% endhighlight %}

這個解法的確是解決了對稱性的問題 但是傳遞性卻沒有遵守

{% highlight java %}
Point p = new Point(1, 2);
ColorPoint cp = new ColorPoint(1, 2, Color.RED);
p.equals(cp)//true
cp.equals(p)//true 遵守對稱性

ColorPoint p1 = new ColorPoint(1, 2, Color.RED);
Point p2 = new Point(1, 2);
ColorPoint p3 = new ColorPoint(1, 2, Color.BLUE);
p1.equals(p2)//true
p2.equals(p3)//true
p1.equals(p3)//false 違反傳遞性

{% endhighlight %}

因為p1和p2只比較座標 p2和p3也只比較座標

好啦我知道了啦 我用你剛剛在對稱性教的方法寫equals就是了嘛
{% highlight java %}
@Override public boolean equals(Object o) {
  return (o instanceof ColorPoint) 
    && super.equals(o) 
    && ((ColorPoint)o).color == color;
}
{% endhighlight %}

不好意思 這樣寫 還是違反對稱性 

因為這種情況下
p.equals(cp) return true

這是什麼道理 為什麼同樣寫法(父類.equals(子類)) 前一個例子
卻s.equals(cis) return false呢?

在effective java的這個章節中 前後的段落很不連貫 一下子把所有的觀念一次丟出來 可是前因後果卻沒有解釋清楚 看了這篇文章你才會知道為什麼

希望讀者看到這裡可以想一下 如果你能自己想到為什麼的話 請按左下角的linkedin按鈕與我聯繫 我很願意幫你refer linkedin

給你十分鐘

<br><br><br><br><br><br><br>
<br><br><br><br><br><br><br>
#### 再現高壇

答案就是 繼承做不到

因為我們ColorPoint繼承了Point

**我們無法在擴展可實例化的類的同時 既增加新的值組件 同時又保留equals約定**

那該怎麼辦呢 記不記得有一個東西一直比繼承好用有彈性 

那個東西就是 [復合](/2018/05/05/favor-composition-over-inheritance/)

{% highlight java %}
public class ColorPointComposition {
  private final Point point;
  private final Color color;

  public ColorPointComposition(int x, int y, Color color) {
    if (color == null) {
      throw new NullPointerException();
    }
    this.point = new Point(x, y);
    this.color = color;
  }
}
{% endhighlight %}

Point變成了一個新類別的一個變量 這樣事情就非常好辦

{% highlight java %}
@Override public boolean equals(Object o) {
  if (!(o instanceof ColorPointComposition)) {
    return false;
  }
  ColorPointComposition cp = (ColorPointComposition) o;
  return cp.point.equals(point) && cp.color.equals(color);
}
{% endhighlight %}

這就是復合厲害的地方



### 一致性

如果兩個對象相等 那這兩個對象就要一直相等(除了有人被改了) 意思就是一個可變對象 在不同時候可以跟不同對象相等 所以如果你的對象是[不可變對象](/2018/04/28/minimize-mutability/) 那兩個不相等的對象就應該永遠不相等

### 非空性

所有對象都必須不等於null

基本上很少情況會讓你的x.equals(null) 回傳true 但是如果你沒寫好 是有可能拋出NullPointerException的 照慣例 equals方法不允許拋出NullPointerException

也許不少人會這麼寫
{% highlight java %}
@Override 
public boolean equals(Object o) {
  if (o == null)
    return false;
}
{% endhighlight %}

但這是沒有必要的 因為equals的輸入參數是Object 所以你幾乎會被迫檢查物件的型態
{% highlight java %}
@Override public boolean equals(Object o) {
  if (!(o instanceof MyType))
    return false;
  MyType mt = (MyType) o;
  ...
}
{% endhighlight %}

而instanceof會在第一個型態給null的時候直接return false 

所以單獨的null檢查是沒有必要的


## 所以要怎麼寫equals

1.在一個equals的一開始直接用==判斷參數是否是這個對象的引用 如果是 就直接傳true 少了後面很多昂貴的計算

2.用instanceof檢查參數類型是否正確

3.把參數轉成正確類型

4.一一檢查類別中的每個**關鍵域**(Significant field)

如果所有關鍵域的測試都成功 就回傳true
其中: 

4-1.對於非float跟非double的基本類型 可以使用==

4-2.對於引用對象 可以使用equals(剩下的交給遞迴處理)

4-3.對於float 可以用Float.compare

4-4.對於double 可以用Double.compare

4-5.對於數組 則每個元素都要套用以上的原則

對於有些對象引用域來說 null值是合法的 為了避免NPE 你可以用Objects.equals(Object, Object) 來比

5.域的比較順序也會影響到效能 把最有可能不一致的或是開銷最低的 拿來先比較 

道理就跟 A且B且C 的時候 最有可能錯的擺最前面 但是A或B或C 的時候 最有可能對的擺最前面一樣道理

6.當你寫完equals方法 問自己三個問題 他是否是對稱的 傳遞的 一致的 自反性以及非空性 並且為這些特性都寫unit test

## 其他注意事項

1.[覆蓋equals時 總要覆蓋hashCode](/2018/06/17/always-override-hashcode-when-you-override-equals/)

2.不要企圖讓equals方法太過複雜

只要遵循這篇文章講的規範就可以 如果你除了這些規範之外 還想去測試其他的等價關係 可能不會是個好主意 比如說有個類別裡面有網址 如果你去比較不同實例的網址是不是指到同一個網站(比如說其中一個是縮網址) 會太過複雜

3.不要把equals的輸入參數改成其他類型

{% highlight java %}
public boolean equals(MyClass o) {
...
}
{% endhighlight %}

你很可能這樣寫完之後 會花好幾個小時debug

為什麼呢 因為這個並沒有複寫(override)到equals(Object o) 而是**重載(overload)**了Object.equals(詳見[Item52 - 慎用重載](/2018/06/02/use-overloading-judiciously/))

4.用Override annotation

這樣第三點就不會犯 因為方法簽名不一樣

## 總結

如果不需要就不要覆寫equals 大多數情況 從Object繼承而來的equals就做得到你想做的事

當你真的要覆寫 保證你有比較每個significant fields 並且保證每個約束都有被遵守
