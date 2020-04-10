---
layout: post
title: 重構 - 改善既有程式的設計 - Large Method
comments: True
subtitle: 如何重構過長的函式
tags: refacforing
author: jyt0532
excerp: 本文介紹重構Large Method
---

這篇文章討論《重構 - 改善既有程式的設計》裡的3.2 - Large Method

圖片以及程式碼來源自[重構 - 改善既有程式的設計](https://www.tenlong.com.tw/products/9789861547534)


## 過長函式

一個函式不應該太長 一般來說超過十行就很值得考慮重構

## 起因 

寫程式比讀程式簡單得多 所以通常一有新增需求 人們就直接把程式碼加在原本的函式裡 

所以函式越變越大 味道越變越怪

## 解法

你應該更積極的去分解函式 有一個原則可以遵循: 

當你感覺需要註釋來說明什麼的時候 你就把這區塊拉近另一個函式 並以**用途(非實作方法)**來命名新函式 即使只是一行的程式 只要函式名稱可以清楚的說明用途 就該這麼做

要不要分離出新的函式的判斷基準 不在於函式的長度夠不夠長 而是這個函式 **做什麼 和 如何做**的semantic distance(語意距離)

"做什麼" 是比 "如何做" 還要高一層的抽象 所以如果這兩者的語意距離很遠 那就應該分離出個新的函式

好 開始聊常見的解法

### Decompose Conditional 分解條件式

把你複雜的if-else 條件判斷分離成小函式

{% highlight java %}
if (date.before(SUMMER_START) || date.after(SUMMER_END)) {
  charge = quantity * winterRate + winterServiceCharge;
}
else {
  charge = quantity * summerRate;
}
{% endhighlight %}

變成這樣

{% highlight java %}
if (isSummer(date)) {
  charge = summerCharge(quantity);
}
else {
  charge = winterCharge(quantity);
}
{% endhighlight %}

### Extract Method

{% highlight java %}
void printOwing(double amount) {
  printBanner();
  //print details
  System.out.println ("name:" + _name);
  System.out.println ("amount" + amount);
}
{% endhighlight %}

當你需要特別寫註解 就很值得分離出來 把註解的內容當作函式名稱

{% highlight java %}
void printOwing(double amount) {
  printBanner();
  printDetails(amount);
}
void printDetails (double amount) {
  System.out.println ("name:" + _name);
  System.out.println ("amount" + amount);
}
{% endhighlight %}

但你仔細看會看到 當你需要Extract Method的時候 常常需要把local variable(amount)一起傳進那個新的函式 這樣子可讀性可能也很難提升 

所以我們可以用[Replace Temp with Query](#replace-temp-with-query), [Introduce Parameter Object](#introduce-parameter-object) 和 [Preserve Whole Object](#preserve-whole-object)來讓程式簡潔一點


### Replace Temp with Query

原本你需要一個區域變數 

{% highlight java %}
double calculateTotal() {
  double basePrice = quantity * itemPrice;
  if (basePrice > 1000) {
    return basePrice * 0.95;
  }
  else {
    return basePrice * 0.98;
  }
}
{% endhighlight %}

現在你用一個函式取代

{% highlight java %}
double calculateTotal() {
  if (basePrice() > 1000) {
    return basePrice() * 0.95;
  }
  else {
    return basePrice() * 0.98;
  }
}
double basePrice() {
  return quantity * itemPrice;
}
{% endhighlight %}

為什麼要這樣呢 因為暫時性的區域變數就只是出現一下 如果一個類別裡有很多地方要用 會導致你的類別有很多很長的函式 這麼做之後 類別裡的所有人都可以直接拿到這個值

所以在我們[Extract Method](#extract-method)的例子裡 我們就不用把`amount`傳來傳去 我們就可以在`printDetails()`裡呼叫`getAmount()`

### Introduce Parameter Object

![Alt text]({{ site.url }}/public/introduce-parameter-object.png)

如果參數間的關係很強 可以把參數們包成一個參數物件

### Preserve Whole Object

如果你把一個物件裡的若干資料拿出來傳給別人
{% highlight java %}
int low = daysTempRange.getLow();
int high = daysTempRange.getHigh();
boolean withinPlan = plan.withinRange(low, high);
{% endhighlight %}

那你不如整個物件傳進去
{% highlight java %}
boolean withinPlan = plan.withinRange(daysTempRange);
{% endhighlight %}

好處是參數列更穩定 如果你想要多傳一個物件裡的參數 你公開的interface甚至不用改

壞處就是增加了兩個函式之間的dependency

### 大招: Replace Method with Method Object

如果以上的方式都不行 代表說你有太多的區域變數需要傳來傳去 讓你很難[Extract Method](#extract-method) 也可能是你[Extract Method](#extract-method)之後程式碼的好讀性也沒上升


![Alt text]({{ site.url }}/public/replace-method-with-method-object.jpeg)

大招就是 把這個函式放進一個單獨物件之中 這樣的話**每個區域變數就變成物件之內的欄位(fields)** 那之後你愛怎麼用都行

來個例子
{% highlight java %}
class Order {
  // ...
  public double price() {
    double primaryBasePrice;
    double secondaryBasePrice;
    double tertiaryBasePrice;
    // A lot of computation.
    // Hard to refactor because the above three parameters are passed back and forth
  }
}
{% endhighlight %}

因為區域變數太多 傳來傳去很醜 我們就直接多加一個類別`PriceCalculator`
{% highlight java %}
class Order {
  // ...
  public double price() {
    return new PriceCalculator(this).compute();
  }
}

class PriceCalculator {
  private double primaryBasePrice;
  private double secondaryBasePrice;
  private double tertiaryBasePrice;
  
  public PriceCalculator(Order order) {
    // Initialize primaryBasePrice/secondaryBasePrice/tertiaryBasePrice
  }
  
  public double compute() {
    // Do whatever you want without passing parameters
  }
}
{% endhighlight %}

有梗吧

