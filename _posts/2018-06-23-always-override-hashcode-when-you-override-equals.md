---
layout: post
title: Effective Java Item11 - 覆蓋equals時總要覆蓋hashCode
comments: True 
subtitle: effective java - 覆蓋equals時總要覆蓋hashCode
tags: effectiveJava
author: jyt0532
excerpt: 本篇文章介紹為何覆蓋equals時總要覆蓋hashCode 以及如何寫一個好的hashCode
---

這篇是Effective Java - Always override hashcode when you override equals章節的讀書筆記 本篇的程式碼來自於原書內容

## Item11: 覆蓋equals時總要覆蓋hashCode

tldr 這是規定

### Object的規範及約定

1.只要物件的equals方法的比較操作所用到的信息沒被修改 那麼對一個對象調用多次hashCode 要始終如一的返回同一個整數

2.如果兩個對象根據equals的比較是相等的 那麼對這兩個對象呼叫hashCode的整數也要相等

3.反之則否 如果對兩個對象呼叫hashCode的整數相等 並不要求兩個對象根據equals的比較要相等 但相等的話 hash table的性能會比較好

沒有複寫hashCode的話 很可能會違反第二點

### PhoneNumber

{% highlight java %}
public final class PhoneNumber {
  private final short areaCode;
  private final short prefix;
  private final short lineNumber;

  public PhoneNumber(int areaCode, int prefix, int lineNumber) {
    this.areaCode = (short) areaCode;
    this.prefix = (short) prefix;
    this.lineNumber = (short) lineNumber;
  }
  @Override
  public boolean equals(Object o) {
    if (o == this)
      return true;
    if (!(o instanceof PhoneNumber))
      return false;
    PhoneNumber pn = (PhoneNumber) o;
    return pn.lineNumber == lineNumber && pn.prefix == prefix
        && pn.areaCode == areaCode;
  }
}
{% endhighlight %}

現在用個hashmap來記一下電話
{% highlight java %}
Map<PhoneNumber, String> m = new HashMap<PhoneNumber, String>();
m.put(new PhoneNumber(707, 867, 5309), "Jenny");
System.out.println(m.get(new PhoneNumber(707, 867, 5309)));//return null
{% endhighlight %}

很抱歉 HashMap使用hashCode來計算一個物件的hash值 而如果你沒有複寫的話 兩個不同的物件回傳的值就會不一樣(對大部分的JVM來說 預設的hashCode是用記憶體位置來計算) 即使這兩個的物件裡的每一個域的值都一樣

### hash collision

那雖然說要複寫 但也不要說就直接寫死

{% highlight java %}
@Override
public int hashCode(){return 42;}
{% endhighlight %}

雖然合法是合法 但是每次hashmap要計算hash值的時候 都會跑到同一個bucket 

一個Map硬生生比你拉成linked-list O(1)的操作全部變成O(n)

所以你要為你的類別寫一個好的hash function 讓不同的的物件返回不同的hash值(且越平均的分配越好)

### 設計hash

那麼要如何設計hash呢 作者給了一個通用的公式:

1.給一個非0的常數 比如說17 存進result變量

2.對於每個equals用到的變數f 計算c:

2-1.如果f是primitive type, c = Type.hashCode(f)
Type就是f的boxed primitive type 

比如說f是整數 c = Integer.hashCode(f)

2-2.如果f是對象引用 recursively呼叫這個物件的hashCode()

2-3.如果是個Array 那就對每一個元素做一樣的處理

算完c之後 result = 31*result + c

3.result就是hash值

4.確認是否相同的實例會有相同的hash值

來看看電話號碼的例子 我們只要加上hashCode

{% highlight java %}
@Override public int hashCode() {
  int result = 17;
  result = 31 * result + Short.hashCode(areaCode);
  result = 31 * result + Short.hashCode(prefix);
  result = 31 * result + Short.hashCode(lineNumber);
  return result;
}

{% endhighlight %}

剛剛的程式就會正確地返回Jenny

為什麼乘數選31 應該不難理解選奇數比選偶數好 如果選的是偶數則hash的結果則不夠平均分佈 經驗法則是選質數最好 那選31的理由 是因為31是個對於電腦來說很好記算乘法的質數

31 * i == (i << 5) - i

### 如果不在乎performance

那你就還有另一個選項

{% highlight java %}
@Override public int hashCode() {
  return Objects.hash(areaCode, prefix, lineNumber)
}
{% endhighlight %}
就把所有你有的東西丟進去 他就會幫你算了 但是這個函式的性能很低 要小心使用

### 不可變類

如果類別是不可變類 那你可以在第一次hashCode被呼叫的時候 把hash值算好時 存在一個變數裡 這樣以後就不用重新再算 這是lazy initialize


{% highlight java %}
  private int hashCode;

  @Override public int hashCode() {
    int result = hashCode;
    if (result == 0) {
      result = 17;
      result = 31 * result + Long.hashCode(areaCode);
      result = 31 * result + prefix;
      result = 31 * result + lineNumber;
      hashCode = result;
  }
  return result;
}
{% endhighlight %}

注意如果執行環境不是thread-safe 則hashCode需要加上volatile

### 最後兩點

1.不用為了性能 而忽略一個對象的關鍵域(significant field)的計算: 雖然這看起來是省下了計算的時間 但之後處理hash碰撞的時間會加倍奉還給你

2.不要在你的spec中跟你的client保證hashCode的實作方式: 當你的客戶依賴著你的實作 你之後就很難再修改你的程式

### 結論

你每次複寫equals 都必須要複寫hashCode 這個hashCode不但要遵守規範 而且hash值散佈的越平均越好
