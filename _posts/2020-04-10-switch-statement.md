---
layout: post
title: 重構 - 改善既有程式的設計 - Switch Statements
comments: True
subtitle: 如何重構switch statement
tags: refacforing
author: jyt0532
excerp: 本文介紹重構Switch Statements
---

這篇文章討論《重構 - 改善既有程式的設計》裡的3.10 - Switch Statements

圖片以及程式碼來源自[重構 - 改善既有程式的設計](https://www.tenlong.com.tw/products/9789861547534)


## Switch Statements

物件導向的一個很明顯的特徵就是少用switch

從本質上來說 switch代表的是**重複** 因為如果同樣的switch散佈在各地 當你想再加一個情況的時候 就所有的switch都得改

你需要仔細看清楚你的switch 是否可以用多型優雅的解決


## 例外 

1.當switch只是拿來取代多層if-else的簡單情況 多型就殺雞用牛刀

2.當switch只是用來實作[Factory Method](/2017/04/28/factory-method/)或是[Abstract factory](/2017/05/03/abstract-factory/)
## 解法

如果可以用多型的話 該怎麼做呢

### Replace Conditional with Polymorphism

當你發現switch裡面的判斷式彼此是subclass的關係 那就可以用多型

多型的好處 就是讓你不用寫明顯的條件式


{% highlight java %}
class Bird{
  double getSpeed() { 
    switch (_type) {
      case EUROPEAN:
        return getBaseSpeed();
      case AFRICAN:
        return getBaseSpeed() - getLoadFactor() * _numberOfCoconuts;
      case NORWEGIAN_BLUE:
        return (_isNailed) ? 0 : getBaseSpeed(_voltage);
    }
    throw new RuntimeException ("Should be unreachable"); 
  }
}
{% endhighlight %}

改成這樣

{% highlight java %}
abstract class Bird {
  abstract double getSpeed();
}

class European extends Bird {
  double getSpeed() {
    return getBaseSpeed();
  }
}
class African extends Bird {
  double getSpeed() {
    return getBaseSpeed() - getLoadFactor() * numberOfCoconuts;
  }
}
class NorwegianBlue extends Bird {
  double getSpeed() {
    return (isNailed) ? 0 : getBaseSpeed(voltage);
  }
}
{% endhighlight %}

優雅 

### Replace Parameter with Explicit Methods

如果你的switch裡面 每個條件是離散的 而且目標是呼叫函式 那就可以直接針對每個情況寫單獨的函式

### Introduce Null Object

如果其中一個條件選項是null 那你可以考慮用Null Object

畢竟多型帶給我們的好處 就是我們不用再去問說 "你是什麼類別" 你就是只管呼叫那個函式就對了

回到鳥的例子 原本我們應用多型了之後 我們可以不管你是什麼鳥 我就是呼叫`getSpeed()`就對了
{% highlight java %}
Bird b = getSomeBird();
speed = b.getSpeed();
{% endhighlight %}

可是當你的`getSomeBird()`有可能回傳null 你就又不得不處理它

{% highlight java %}
Bird b = getSomeBird();
if(b == null) {
  speed = 0;
} else {
  speed = b.getSpeed();
}
{% endhighlight %}

好醜 我們這時候就可以來個Null Object

{% highlight java %}
class NullBird extends Bird {
  boolean isNull() {
    return true;
  }
  int getSpeed() {
    return 0;
  }
  // Some other NULL functionality.
}
{% endhighlight %}

這樣你以後就儘管放心呼叫就對了

{% highlight java %}
Bird b = getSomeBird();
speed = b.getSpeed();
{% endhighlight %}
