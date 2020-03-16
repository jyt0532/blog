---
layout: post
title: 無瑕的程式碼 - 物件及資料結構
comments: True 
subtitle: Objects and Data Structures
tags: jvm
author: jyt0532
---

《無瑕的程式碼》第六章 物件及資料結構 中 有個很有趣的話題 想跟大家討論討論

本篇的程式碼來自於原書內容


### 資料結構 vs 物件

物件(Object) 跟 資料結構(Data structure)差在哪裡呢

物件把data給隱藏在在抽象層之後 只提供給你操縱這些data的函式

資料結構呢 則是把data直接提供給你 那就不用提供操縱這些data的函式

我們來看個例子 首先我們先定義正方形長方形以及圓形

{% highlight java %}
public class Square { 
  public Point topLeft; 
  public double side;
}

public class Rectangle { 
  public Point topLeft; 
  public double height; 
  public double width;
}

public class Circle { 
  public Point center; 
  public double radius;
}
{% endhighlight %}

然後再來定義一個計算面積的類別

{% highlight java %}
public class Geometry {
  public final double PI = 3.141592653589793;
  public double area(Object shape) throws NoSuchShapeException {
    if (shape instanceof Square) { 
      Square s = (Square)shape; return s.side * s.side;
    }
    else if (shape instanceof Rectangle) { 
      Rectangle r = (Rectangle)shape; return r.height * r.width;
    }
    else if (shape instanceof Circle) {
      Circle c = (Circle)shape;
      return PI * c.radius * c.radius; 
    }
    throw new NoSuchShapeException(); }
}
{% endhighlight %}

好的我知道 我們物件導向玩這麼久了 這種Procedural(結構化)的程式設計我們一定看不習慣 

秀一下物件導向的玩法

{% highlight java %}
public class Square implements Shape { 
  private Point topLeft;
  private double side;
  public double area() { 
    return side*side;
  } 
}

public class Rectangle implements Shape { 
  private Point topLeft;
  private double height;
  private double width;
  public double area() { 
    return height * width;
  } 
}

public class Circle implements Shape { 
  private Point center;
  private double radius;
  public final double PI = 3.141592653589793;
  public double area() {
    return PI * radius * radius;
  } 
}
{% endhighlight %}

這不是輕鬆愉快嗎 程式就該這麼寫啊 這樣我今天要是加了一個新的形狀 我只要實作Shape裡的`area()`就好啦 我根本不用改`Square`或`Rectabgle`或`Circle` 難道不是嗎???

答案是... 

&nbsp;

&nbsp;

&nbsp;

&nbsp;

看情況

&nbsp;

&nbsp;

&nbsp;

&nbsp;

我們會先入為主的認為OOP的寫法好 是因為我們主觀的認為

**我們的程序要彈性的支持新增的形狀**

那如果今天我們的形狀 就算天塌下來了都是只有這三種 但是我們需要的操作函式很可能會變 比如說我們如果需要對每個形狀加上個周長`perimeter()`函式的話

結構化的程式設計 就是只要在`Geometry`裡面多寫個方法 我三個形狀都不用動

OOP的程式設計 我就是`Shape`需要多加個抽象方法 且**每個形狀都需要多實作這個抽象方法**

## 看情況

結論就是 看情況決定要怎麼寫 兩者都要會 然後在適當的時候寫出適當的程式

![Alt text]({{ site.url }}/public/object-and-data-structure.jpeg)


結構化的程式碼(使用資料結構的程式碼) 容易添加新的函式 但不容易新增新的類別

物件導向的程式碼 容易添加新的類別 但不容易添加新的函式
