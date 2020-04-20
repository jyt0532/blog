---
layout: post
title: 重構 - 改善既有程式的設計 - Organizing Data
comments: True
subtitle: 重新組織資料
tags: refacforing
author: jyt0532
excerp: 本文介紹如何重新組織資料
---

這篇文章討論《重構 - 改善既有程式的設計》裡的第八章 - Organizing Data

主要會討論[程式碼的壞味道](/toc/refactoring/)中 沒有被提及的重構方法裡面有關於 重新組織資料 的內容

圖片以及程式碼來源自[重構 - 改善既有程式的設計](https://www.tenlong.com.tw/products/9789861547534)

## 重新組織資料

### [Replace Data Value with Object](/2020/04/10/primitive-obsession/#replace-data-value-with-object)

當你有一筆資料項(data item) 需要額外的資料和行為 就把那個資料項變成一個物件

### Change Value To Reference

將實質物件改為引用物件

![Alt text]({{ site.url }}/public/change-value-to-reference.png)

上圖的意思 就是把一個value object轉成reference object

![Alt text]({{ site.url }}/public/know-rule.jpg)

別激動別激動 我一開始看也是看不懂 我來說說我的理解吧

我們繼續使用[Replace Data Value with Object](#replace-data-value-with-object)的例子 在使用[Replace Data Value with Object](#replace-data-value-with-object)重構之後 
每張`Order`上都有一個`Customer`物件而不是String

{% highlight java %}
class Order{
  private Customer customer;
  public String getCustomerName() {
    return customer.getName();
  }
  public void setCustomer(String customerName) {
    customer = new Customer(customerName);
  }
  public Order(String customerName) {
    customer = new Customer(customerName);
  }
}
{% endhighlight %}

但當`Order`多了之後 發現很多Customer都是同一個人(同一個名字) 但同一個人在不同的`Order`物件裡面卻是不同的`Customer`物件 
之後管理起來會麻煩 比如顧客換名字或是換地址 就得改所有的`Order`物件


你說這還不簡單 建構子跟setter裡面 傳進來`Customer`物件不就得了?

{% highlight java %}
class Order{
  private Customer customer;
  public String getCustomerName() {
    return customer.getName();
  }
  public void setCustomer(Customer c) {
    customer = c;
  }
  public Order(Customer c) {
    customer = c;
  }
}
{% endhighlight %}

這的確是把實質物件改為引用物件 但你的使用者使用起來還是很麻煩 他在創建一個新Order的時候 手上拿著customerName 
你要先找出有著這個名字的`Customer`物件 才能把這個物件丟進`Order`的constructor 

好麻煩 怎麼辦呢

我們使用[Replace Constructor with Factory Method](/2020/04/19/making-method-calls-simpler/#replace-constructor-with-factory-method) 我們就可以控制Customer的創建過程
{% highlight java %}
class Customer {
  //用一個hash table紀錄目前有的顧客
  private static HashMap<String, Customer> map = new HashMap<>();
  
  public static Customer create(String name) {
    if(!map.containsKey(name)) {
      map.put(name, new Customer(name));
    }
    return map.get(name);
  }

  private final String name;
}
{% endhighlight %}

有了`create`這個靜態工廠之後 Order只要有customerName 就很好辦了

{% highlight java %}
class Order {
  //…
  private Customer customer;
  public String getCustomerName() {
      return customer.getName();
  }
  public void setCustomer(String customerName) {
    customer = Customer.create(customerName);
  }
  public Order(String customerName) {
    customer = Customer.create(customerName);
  }
}
{% endhighlight %}

### Change Reference To Value

[Change Value To Reference]的反向 如果你的reference object很小而且**不可變** 那就不如把它換成value object

### [Replace Array with Object](/2020/04/10/primitive-obsession/#replace-array-with-object)

當你有一個陣列 其中的元素代表不同的東西 那就用物件來替換

### [Change Bidirectional Association to Unidirectional](/2020/04/16/inappropriate-intimacy/#change-bidirectional-association-to-unidirectional)

如果兩個類別A,B 只有類別A需要知道類別B的特性 那就把依賴關係變成單向

### Change Unidirectional Association to Bidirectional

[Change Bidirectional Association to Unidirectional](#change-bidirectional-association-to-unidirectional)的反向

當兩個類別彼此需要知道對方特性 但目前只有一個方向的依賴 就把依賴變成雙向

![Alt text]({{ site.url }}/public/change-unidirectional-association-to-bidirectional.png)

雖然比較複雜 但卻是比較常見的use case 你會想知道每個`Order`是由哪個`Customer`下訂的 你當然也會想知道每個`Customer`有下了哪些`Order`

### Replace Magic Number with Symbolic Constant

為一個有意義的數值命名

{% highlight java %}
double potentialEnergy(double mass, double height) {
  return mass * height * 9.81;
}
{% endhighlight %}

把重力常數命名一下

{% highlight java %}
static final double GRAVITATIONAL_CONSTANT = 9.81;

double potentialEnergy(double mass, double height) {
  return mass * height * GRAVITATIONAL_CONSTANT;
}
{% endhighlight %}

### [Encapsulate Field](/2020/04/16/data-class/#encapsulate-field)

封裝欄位 把public的欄位變成private 並提供存取函式

### [Encapsulate Collection](/2020/04/16/data-class/#encapsulate-collection)

檢查是不是對於Collections也都做好了封裝

### [Replace Type Code with Class](/2020/04/10/primitive-obsession/#replace-type-code-with-class)

Type Code三兄弟之老大: 類別之中有一個數值型別代碼(numeric type code) 但他不影響類別的行為

### [Replace Type Code with Subclasses](/2020/04/10/primitive-obsession/#replace-type-code-with-subclasses)

Type Code三兄弟之老二: 當類別之中的數值型別代碼會影響類別行為的時候 就用這個方法

### [Replace Type Code with State/Strategy](/2020/04/10/primitive-obsession/#replace-type-code-with-statestrategy)

Type Code三兄弟之老三: 當你想動態的改變類別的行為的時候 就用這個方法

### Replace Subclass with Fields

你的各個subclass的差別 只差在**回傳常數資料**的函式 你就可以把這個函式提煉到superClass 然後移除所有subClass

![Alt text]({{ site.url }}/public/replace-subclass-with-fields.png)


