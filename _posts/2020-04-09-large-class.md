---
layout: post
title: 重構 - 改善既有程式的設計 - Large Class
comments: True
subtitle: 如何重構過大的類別
tags: refacforing
author: jyt0532
excerp: 本文介紹重構Large Class
---

這篇文章討論《重構 - 改善既有程式的設計》裡的3.3 - Large Clas

圖片以及程式碼來源自[重構 - 改善既有程式的設計](https://www.tenlong.com.tw/products/9789861547534)


## 過大的類別

一個類別不應該太大 太大的類別很難維護

## 起因 

大家習慣把新需求加到現有的類別裡 所以類別越變越大 味道越變越怪

## 解法

來聊聊常見的解法

### Extract Class

你可以用[Single Responsibility Principle](/2020/03/18/srp/)簡單的判斷一個類別的責任是不是太多 是的話就拆分成兩個類別

我們有個`Person`
{% highlight java %}
class Person{
  public String getName() {
    return _name; 
  }
  public String getTelephoneNumber() {
    return ("(" + _officeAreaCode + ") " + _officeNumber);
  }
  String getOfficeAreaCode() {
    return _officeAreaCode; 
  }
  void setOfficeAreaCode(String arg) { 
    _officeAreaCode = arg;
  }
  String getOfficeNumber() {
    return _officeNumber; 
  }
  void setOfficeNumber(String arg) { 
    _officeNumber = arg;
  }
  private String _name;
  private String _officeAreaCode; 
  private String _officeNumber;
}
{% endhighlight %}

恩...這個`Person`的責任好像有點太多 看起來要把電話號碼相關的資料分離出來


{% highlight java %}
class TelephoneNumber { 
  String getAreaCode() {
    return _areaCode; 
  }
  void setAreaCode(String arg) { 
    _areaCode = arg;
  }
  private String _areaCode; 
}
{% endhighlight %}

簡單 然後只要把TelephoneNumber丟進Person裡
{% highlight java %}
class Person{
  public String getName() {
    return _name; 
  }
  public String getTelephoneNumber(){
    return _officeTelephone.getTelephoneNumber();
  }
  TelephoneNumber getOfficeTelephone() {
    return _officeTelephone; 
  }
  private String _name;
  private TelephoneNumber _officeTelephone = new TelephoneNumber();
}
{% endhighlight %}

### Extract Subclass

當你發現你這個類別的某些feature只被部分的instance使用 那就可以考慮一個SubClass

![Alt text]({{ site.url }}/public/large-class-1.png)

把所有物件都會用到的放在Parent Class 把少部分物件才會用到的放在Derived Class

Extract Subclass 跟 [Extract Class](#extract-class) 概念很接近 差別就差在你要選擇[復合還是繼承](/2018/05/05/favor-composition-over-inheritance/)

### Extract Interface

很多個客戶都只用一個類別的某些方法們 或是兩個類別有某些方法相同 就可以把這某些方法給提煉出來

![Alt text]({{ site.url }}/public/large-class-2.png)

今天我要對 Employee 收費 我需要呼叫的是`getRate()`跟`hasSpecialSkill()` 其他的我不在乎

{% highlight java %}
double charge(Employee emp, int days) { 
  int base = emp.getRate() * days;
  if (emp.hasSpecialSkill())
    return base * 1.05; 
  else return base;
}
{% endhighlight %}

那就可以把這兩個函式分離出來
{% highlight java %}
interface Billable {
  public int getRate();
  public boolean hasSpecialSkill();
}
{% endhighlight %}

變成這樣

{% highlight java %}
double charge(Billable bil, int days) { 
  int base = bil.getRate() * days; 
  if (bil.hasSpecialSkill())
    return base * 1.05; 
  else return base;
}
{% endhighlight %}

這樣之後即使`charge()`要對Employee以外的類別收費 只要請那個類別實作`Billable`就可以

