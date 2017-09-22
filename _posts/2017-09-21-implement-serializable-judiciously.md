---
layout: post
title: Effective Java(74) - Implement Serializable judiciously
comments: True 
subtitle: effective java - 謹慎實現Serializable介面
tags: effectiveJava
author: jyt0532
---
這篇是Effective Java - Implement Serializable judiciously章節的讀書筆記

## Item74: 實現Serializable介面要謹慎

要讓一個class變成Serializable很簡單 原本是

{% highlight java %}
public class Employee{
    public String name;
    public String email;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }
}
{% endhighlight %}

現在只要加上implements Serializable就可以

{% highlight java %}
public class Employee implements Serializable {
    public String name;
    public String email;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }
}
{% endhighlight %}
用了有什麼好處呢

1.可以把他的狀態存進Disk裡面 需要再把它讀出來

2.可以傳輸 serialize後 傳輸給另外一台機器或另外一個process deserialize後繼續用

好處很多 做法簡單 所以很常被濫用 這點要教你的就是要謹慎地使用它 為什麼呢

1.當你宣稱你實現了Serializable而且release之後 你就不能隨便改變你的實現方式了(因為你可能後面的版本的Serialize的方式跟前面的不相容) 彈性大幅降低

當你一宣告你支援serializable之後 你就一輩子需要支援 如果你用的是default的Serialization 你的private variable也不能變動 成為了API的一部分

比較好的方式 就是把你序列化的版本當作序列化的一部分

![Alt text]({{ site.url }}/public/java74.png)


那一行非常重要 如果沒有那一行 這個你承諾說有實現序列化的class會自動生成一個serialVersionUUID 這個變數隨著class名稱 變數名稱等等的都會改變 

然後JVM會拿著這個生成的變數 確認說我現在準備要反序列化的這個class 跟它當初被序列化的時候版本一樣 確認完後才開始反序列化

2.實現序列化 增加了bug出現的機會 還有安全性的漏洞

為什麼這麼說呢 正常來說物件的創造是由constructor 把一個class從bit的資料串decode成物件 可以說是包含了一個隱藏的constructor 那**既然你沒有透過constructor就把物件建立了** 某些你在跑constructor會跑的確認或是檢查他都沒做 更慘的是如果你用的是default的反序列化 那hacker可以反推的出來你的物件裡面有什麼等等 太好猜了

3.隨著發行的版本號 測試負擔增加

很好理解 當你有個新的版本出來 你必須確認你可以序列化新版本 然後用舊的版本反序列化 或是你可以序列化舊版本 然後用新的版本反序列化 版本越多 你要寫的測試線性成長


說了這麼多的缺點 就是要你謹慎考慮到底要不要實作序列化 但還是有一些大原則可以依循

1.Value class比如說Date或是BigInteger 就很可以實作序列化

2.需要被繼承的class不太需要實作序列化

比較有名的特例 就是Throwable, Component和HttpServlet 不難想像這些東西都是需要傳輸的 但大多數情況需要被繼承的class不太需要實作序列化

以下是關於這點比較深的討論 我花了大概兩小時才把它讀通 由於書上寫的太難懂了我甚至懷疑有多少人真正的懂它

如果你真的必須要有一個class是必須要被繼承又必需要實作序列化的 那你必須要去確認你有沒有哪些變數型態的default值 讓你的物件不合理

來個例子
{% highlight java %}
class Person implements Serializable{
  protected String name;
  protected int age;

  Person() {
    this("John",1);
  }
  Person(String name, int age) {
    this.name = name;
    this.age = age;
  }
}
{% endhighlight %}

{% highlight java %}
class Employee extends Person  {
  protected String address ;
  public Employee()
  {
    super();
    address ="N/A";
  }

  public Employee(String name , int age, String address)
  {
    super(name,age);
    this.address = address;
  }
}
{% endhighlight %}

有趣的來的 Person的預設年齡是1歲 因為一個人0歲不合理 但非常不巧 0剛好是int的預設值

而且好死不死 你在這個繼承Person的版本之前 你Employee自己是一個獨立的class

{% highlight java %}
class Employee{//Old version
  protected String address ;
  protected String name;
  protected int age;

  public Employee()
  {
    name= "John";
    age = 1;   
    address ="N/A";
  }

  public Employee(String name , int age, String address)
  {
    this.name = name;
    this.age = age;
    this.address = address;
  }
}

{% endhighlight %}
如果你收集了以上的所有條件 那麼就有可能發生一種情況: 當你序列化的時候用的是舊版本Employee 但你返序列化的時候用的是新的版本Employee
那他在還原Employee的時候因為不認識Person 不知道怎麼還原name跟age 就會把age設成0 

那怎麼辦呢 所以在你新版本的Employee裡面你就要加個

{% highlight java %}
class Employee extends Person  {       
  String address ;  
  public Employee()
  {   
    super();
    this.address ="N/A";
  }      

  public Employee(String name , int age, String address)
  {
    super(name,age); 
    this.address = address;
  }
  private void readObjectNoData() throws ObjectStreamException{
    name= "John";
    age = 1;     
  } 
}
{% endhighlight %}

搞定 

3.interface應該儘少extends序列化

4.當你繼承了一個沒有實作序列化的class 而你想要序列化的話 你必須確保那個你繼承的class**要有一個沒有參數的constructor**
換句話說 當你想寫一個給別人繼承的class但又不想實作序列化 你必須要有一個沒參數的constructor讓未來繼承你又需要序列化的人可以restore你的狀態

{% highlight java %}
public Bicycle() {
    gear = 1;
    cadence = 10;
    speed = 0;
}
{% endhighlight %}
這就是一個不吃參數的constructor

5.Inner class不應該實作序列化

## 總結

實作序列化看起來很簡單 但除非你這個class只用一下子 不然當你跟別人承諾你實作序列化之後 之後要考慮的事可是相當複雜
特別是可以被繼承的class 你必須要提供一個沒變數的constructor 這樣你的subclass繼承你之後就可以彈性的選擇要不要實作序列化
