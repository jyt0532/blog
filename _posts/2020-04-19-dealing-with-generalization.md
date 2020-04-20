---
layout: post
title: 重構 - 改善既有程式的設計 - Deal with Generalization
comments: True
subtitle: 處理繼承關係
tags: refacforing
author: jyt0532
excerp: 本文介紹如何處理繼承關係
---

這篇文章討論《重構 - 改善既有程式的設計》裡的第十一章 - Deal with Generalization

主要會討論[程式碼的壞味道](/toc/refactoring/)中 沒有被提及的重構方法裡面有關於 如何處理繼承關係 的內容

圖片以及程式碼來源自[重構 - 改善既有程式的設計](https://www.tenlong.com.tw/products/9789861547534)

## 如何處理繼承關係

### [Pull Up Field](/2020/04/15/duplicate-code/#pull-up-field)

把欄位提到superClass

### [Pull Up Method](/2020/04/15/duplicate-code/#pull-up-method)

把方法提到superClass

### [Pull Up Constructor](/2020/04/15/duplicate-code/#pull-up-constructor)

把會重複使用的建構子放到superClass


### [Extract Class](/2020/04/10/large-class/#extract-class)

你可以用[Single Responsibility Principle](/2020/03/18/srp/)簡單的判斷一個類別的責任是不是太多 是的話就拆分成兩個類別

### [Extract Subclass](/2020/04/10/large-class/#extract-subclass)

當你發現你這個類別的某些feature只被部分的instance使用 那就可以考慮一個SubClass

### [Extract Superclass](/2020/04/12/refused-bequest/#extract-superclass)

把subClass不需要的函式跟變數丟回給SuperClass

### [Extract Interface](/2020/04/10/large-class/#extract-interface)

很多個客戶都只用一個類別的某些方法們 或是兩個類別有某些方法相同 就可以把這某些方法給提煉出來

### [Collapse Hierarchy](/2020/04/15/lazy-class/#collapse-hierarchy)

把用不到的抽象類別移除

### [Form Template Method](/2020/04/15/duplicate-code/#form-template-method)

兩個subClass有相似的演算法結構 就用[Template method](/2017/09/12/template/)

### [Replace Inheritance with Delegation](/2020/04/12/refused-bequest/#replace-inheritance-with-delegation)

繼承往往是造成過度親密的原因 因為subClass對superClass的瞭解總是超出預期 你可以用delegation來減少兩者的依賴

### Replace Delegation with Inheritance

[Replace Inheritance with Delegation](#replace-inheritance-with-delegation)的反向

![Alt text]({{ site.url }}/public/replace-delegation-with-inheritance.png)

奇怪了 我們不是一直在鼓吹[復合優先於繼承](/2018/05/05/favor-composition-over-inheritance/)嗎 什麼情況才應該用繼承重構呢

來看個例子

我們有一個可愛的`Person`

{% highlight java %}
class Person { 
  String _name;
  public String getName() { 
    return _name;
  }
  public void setName(String arg) {
    _name = arg; 
  }
  public String getLastName() {
    return _name.substring(_name.lastIndexOf(' ')+1);
  } 
}
{% endhighlight %}

還有一個`Employee` 使用了復合來delegate `Person`

{% highlight java %}
class Employee {
  Person _person = new Person();
  public String getName() { 
    return _person.getName();
  }
  public void setName(String arg) {
    _person.setName(arg); 
  }
  public String toString () {
    return "Emp: " + _person.getLastName();
  } 
}
{% endhighlight %}

既然你幾乎每個函式都是直接去問`Person` 那不如直接繼承吧

{% highlight java %}
class Employee extends Person {
  @Override public String toString() {
    return "Emp: " + getLastName();
  }
}
{% endhighlight %}



