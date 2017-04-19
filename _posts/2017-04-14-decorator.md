---
layout: post
title: Design Pattern(3) - Decorator
comments: True 
subtitle: design pattern - Decorator
tags: designPattern
author: jyt0532
---
{% include design_pattern.html %}

### 開咖啡店囉 

今天想設計所有的咖啡的class 目前主要就賣四種咖啡

![Alt text]({{ site.url }}/public/decorator1.png)

每個subclass各自overwrite cost() 

但是其實咖啡可以加很多料 牛奶 豆漿 摩卡 奶泡 每個都有不同的價格 要是所有排列組合都創一個subclass...
![Alt text]({{ site.url }}/public/decorator2.png)

崩潰 而且之後每加個新配料 class數目都是指數性的成長

![Alt text]({{ site.url }}/public/civilwar.gif)

### 重新設計 把所有責任丟給parent

![Alt text]({{ site.url }}/public/decorator3.png)

好 今天如果要Houseblend加豆漿跟奶泡
![Alt text]({{ site.url }}/public/decorator4.png)

算錢的時候一一確認
![Alt text]({{ site.url }}/public/decorator5.png)

這個設計有什麼問題呢 

1.某個配料價錢一改 Beverage code要改

2.有新的配料 Beverage code要改

3.不彈性 某些咖啡跟某些配料不可能配一起 可是setMocha, setSoy這些function是compile-time繼承自Beverage的 不能不要

4.如果今天想要雙倍摩卡 雙倍奶泡？ 只用hasMocha是無法算錢的 還要算MochaCount等等 

天啊怎麼辦啊啊啊啊啊

### 開放封閉守則

> Classes should be open for extension but closed for modification

**DP rule#5 類別應該開放以便擴充 應該封閉禁止修改**

在這裡 應該把Beverage這個code給封閉 我們不希望每次有新的配料或新的咖啡 就必須改動beverage

那要怎麼開放呢 簡單 只要把咖啡跟配料的super class用一樣就可以了 **咖啡跟配料都是Beverage**

![Alt text]({{ site.url }}/public/decorator6.png)

當然也不是說奶泡跟濃縮咖啡是同一個class 只是咖啡跟配料都繼承了Beverage
 
差別差在 配料多了一個指到Beverage的reference 畢竟咖啡可以單獨存在 但配料不行

為什麼這樣就解決了問題了呢

先來看一下Beverage 這是我們說好要封閉的東西

{% highlight java %}
public abstract class Beverage {
    String description = "Unknown Beverage";
    public String getDescription() {
	return description;
    }
    public abstract double cost();
}
{% endhighlight %}

再來看一下HouseBlend 就是某一種繼承了beverage的咖啡
{% highlight java %}
public class HouseBlend extends Beverage {
    public HouseBlend() {
	description = "HouseBlend";
    }
    public double cost() {
	return 1.99;
    }
}
{% endhighlight %}

主角登場
{% highlight java %}
public abstract class CondimentDecorator extends Beverage {
    public abstract String getDescription();
    Beverage beverage;
}
{% endhighlight %}

沒錯 就這樣 這裡可以是interface可以是abstract class

那來看看Soy(這裡的Soy是個配料)
{% highlight java %}
public class Soy extends CondimentDecorator {
    public Soy(Beverage beverage) {
	this.beverage = beverage;
    }
    public String getDescription() {
	return beverage.getDescription() + ", Soy";
    }
    public double cost() {
	return beverage.cost() + 0.10;
    }
}
{% endhighlight %}
每個配料需要有一個指向Beverage的reference 這也是配料跟主要咖啡的區別
那main function怎麼call呢
{% highlight java %}
public static void main(String args[]) {
    Beverage houseBlend = new HouseBlend();
    houseBlend_soy = new Soy(houseBlend);
    houseBlend_soy_mocha = new Mocha(houseBlend_soy);
    System.out.println(houseBlend_soy_mocha.getDescription()+
	" $"+beverage2.cost());
}
{% endhighlight %}
STDOUT:
{% highlight sh %}
HouseBlend, Soy, Mocha $2.19
{% endhighlight %}

這個有意思了 Soy需要一個beverage instance 這個是他裝飾(decorate)的對象 也是Soy的constructor需要丟進來的東西

這樣事情簡單多了 每一個配料的input都是Bevarage的instance 我把它加上配料後也是變成一個beverage
這樣就可以一路chain下去 我們跳脫了配料是咖啡的attribute的思維 

**一個未裝飾的東西(Beverage) 丟進裝飾的物件(Condiment)的constructor 回傳一個裝飾過的東西(Beverage)** 

簡直精彩

### Decorator Pattern

#### 時機

想動態(runtime)增加或取消責任到個別物件 而不是加到整個類別裡


#### 結構
![Alt text]({{ site.url }}/public/decorator7.png)

* Component(Beverage): 制定你想要動態增減責任的interface

* ConcreteComponent(HouseBlend): 定義可以被decorate的物件

* Decorator(CondimentDecorator): 繼承Component 但多了一個指向Component物件的reference

* ConcreteDecorator(Soy): 繼承Decorator 實作責任本身 你想怎麼裝飾ConcreteComponent

### 優缺點

1.比compile time的繼承更有彈性 可以在run-time把Decorator加上去 想要任何排列組合都行

2.避免了把太多功能加上class階層的頂端

3.因為新增Decorator實在方便 很容易到後面會有一堆功能非常類似的Decorator

### Decorator vs Strategy

這兩個Pattern都讓我們可以在run-time變動一個物件的特性 主要差別有二

1.Strategy是改變特性 Decorator是增加特性

2.Strategy pattern, 使用的人必須知道有什麼strategy option跟現在apply了哪一個strategy(就是你必須有一個strategy物件) 但Decorator Pattern不用 因為Decorator像是從外面改變了這個object(被包了一層)

大家常說Decorator像是換皮變臉 但Strategy像是換了骨頭 

一個畫皮 一個畫心
![Alt text]({{ site.url }}/public/paintedskin.jpg)

> 留住你一面 畫在我心間 誰也拿不走 初見的畫面
>
>
>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<畫情>

### Python 裝飾器

[理解 Python 裝飾器看這一篇就夠了](https://foofish.net/python-decorator.html)

這篇文章漂亮的詮釋了Python如何利用First class function的特性來實作Decorator 淺顯易懂 由這篇文章你可以看出Decorator有多強大 

First class function意思就是說function可以被傳入其他的function當做input argument 也可以被當return value傳回來

符合這個先決條件 就是使用Decorator的好時機 
因為你輸入輸出是一樣的東西 你可以動態無限包裝很多個Decorator
