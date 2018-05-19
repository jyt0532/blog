---
layout: post
title: Effective Java Item51 - 謹慎的設計方法簽名
comments: True 
subtitle: effective java - 謹慎的設計方法簽名
tags: effectiveJava
author: jyt0532
excerpt: 本篇文章說明幾個設計方法簽名的注意事項
---

這篇是Effective Java - Design method signatures carefully章節的讀書筆記 本篇的程式碼來自於原書內容

這個項目列出了許多設計API時的概念 每個都重要但又小到不值得開一個新的項目 所以集中在item51


### Item51: 謹慎的設計方法簽名

### 謹慎的選擇方法的名稱

方法的名稱應該遵循[標準的命名習慣](/2018/01/28/adhere-to-generally-accepted-naming-conventions/) 首要目標是選擇易於理解的 並且與同一個package的其他名稱風格一致
次要目標是選擇大眾認可的名稱

### 不要提供太多不必要的方法

一個接口或是類別如果有太多方法 會大大增加了實作的負擔 當你確定某一項操作常被用到的時候 再提供[便利方法](https://stackoverflow.com/questions/19063652/what-is-a-convenience-method-in-java) 當你不確定的話就不需要 類別越簡單越好

### 避免過長的參數列表

沒有人會記得超過四個以上的參數 特別是有相同的類型的參數列表時更慘 可能用的人給錯順序可是還是跑得起來 很難debug

要避免有許多方法

1.把一個大方法拆解成若干小方法 但要注意方法越多越難maintain

2.建一些輔助類helper class 

這些輔助類通常是static member class 如果一個頻繁出現的參數序列可以被看作是某個獨特的實體 則建議可以用這種方法

比如說你現在在寫一個樸克牌遊戲 每次丟進一個方法時都要給花色Suit跟數字Number 
你就可以在你的樸克牌遊戲類裡面新增一個輔助類來表示一張牌Card 那你原本的方法就可以從兩個參數變成一個參數

3.用[Builder模式](/2017/06/29/builder/)

原本一個方法吃多個參數 且其中某些是選擇性的時候 builder模式最好

### 對於參數類型 優先使用接口而不是類
你沒有必要讓你的方法的輸入參數的類型是HashMap 如果你用的是Map 則使用者可以輸入HashMap, TreeMap等等 更加彈性

### 對於boolean參數 優先考慮兩個元素的枚舉

這使你的程式更容易了解 特別是你用IDE的話 使用者用起來非常方便

比如說你有一個溫度計 裡面有一個靜態工廠 工廠吃一個參數決定攝氏還是華氏 你可以先定義一個枚舉
{% highlight java %}
public enum TemperatureScale { FAHRENHEIT, CELSIUS }
{% endhighlight %}
然後你呼叫工廠 就好懂的多
{% highlight java %}
Thermometer.newInstance(TemperatureScale.CELSIUS)
{% endhighlight %}

看一下另一種作法
{% highlight java %}
Thermometer.newInstance(true) 
{% endhighlight %}

則是還需要看文件才知道哪個是華氏哪個是攝氏














