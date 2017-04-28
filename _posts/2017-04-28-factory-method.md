---
layout: post
title: Design Pattern(4) - Factory Method
comments: True 
subtitle: design pattern - factory method
tags: designPattern
author: jyt0532
---
{% include design_pattern.html %}

### 來點個Pizza吧

今天想設計一個PizzaStore 裡面可以點Pizza

{% highlight java %}
public class PizzaStore {
    Pizza orderPizza(String type){
    	Pizza pizza;
    	if(type.equals("cheese")){
	    pizza = new CheesePizza();
    	}else if(type.equals("greek")){
	    pizza = new GreekPizza();
    	}else if(type.equals("pepperoni")){
	    pizza = new PepperoniPizza();
    	}
    	pizza.prepare();
    	pizza.cook();
    	return pizza
    }
}
{% endhighlight %}

嗯 看起來不差 compile-time的時候pizza.prepare()跟pizza.cook()是哪一種pizza我不用知道 
我只要保證各個Pizza的subclass有實作prepare和cook就可以 

Polymorphism Rocks!

但只要有新的Pizza推出或是有舊的要拿掉 需要改這裡的if else 有點麻煩

### 簡單工廠模式

別忘了**Design Pattern rule #1: Encapsulate what varies**

把剛剛orderPizza裡面的所有if-else拉出來到一個工廠裡 這個工廠專門製作pizza

{% highlight java %}
public class SimplePizzaFactory {
    public Pizza createPizza(String type) {
	Pizza pizza = null;
	if(type.equals("cheese")){
	    pizza = new CheesePizza();
    	}else if(type.equals("greek")){
	    pizza = new GreekPizza();
    	}else if(type.equals("pepperoni")){
	    pizza = new PepperoniPizza();
    	}
    	return pizza;
    }
}
{% endhighlight %}

就這麼簡單 這就是簡單工廠 

**簡單工廠管理物件的創造 如果client要取得物件 只要給簡單工廠正確的參數就可以**

然後pizza店的constructor要丟一個工廠進去
{% highlight java %}
public class PizzaStore {
    SimplePizzaFactory factory;
    public PizzaStore(SimplePizzaFactory factory) { 
	this.factory = factory;
    }
    public Pizza orderPizza(String type) {
	Pizza pizza;
	pizza = factory.createPizza(type);
	pizza.prepare();
	pizza.cook();
	return pizza;
    }
}
{% endhighlight %}

簡單工廠模式 讓我們把**pizza的創造和pizza的使用分開**了 減少了client對於實作的依賴

我們成功的判斷出orderPizza這個函數裡面會變動的部分 分離出一個工廠 去處理他 
如果你今天要改變處理方式 你去改那個工廠或是給我一個新工廠就可以 
我不管你怎麼創造的 我只在乎你回傳給我的object的class是Pizza的subclass

![Alt text]({{ site.url }}/public/factory1.png)

簡單工廠模式的優點就是分離了物件的使用和創造 
client不管你怎麼生成的 但缺點也很明確 
每當有新的class出來工廠就要改 複雜度上升得很快 適用情況是需要創建的種類比較少 而且客戶對於怎麼創建對象的方法不關心
 
### 生意不錯 開個分店

實作了簡單工廠模式之後 同樣是cheese pizza, 紐約跟芝加哥的做法就完全不一樣
我們可以修改SimplePizzaFactory讓createPizza多吃一個參數style
{% highlight java %}
public class SimplePizzaFactory {
    public Pizza createPizza(String style, String type) {
	Pizza pizza = null;
	if(style.equals("NY")){
	    if(type.equals("cheese")){
	    	pizza = new NYStyleCheesePizza();
    	    }else...
	    ...
	}else if(style.equals("chicago")){
	    if(type.equals("cheese")){
                pizza = new ChicagoStyleCheesePizza();
            }else...    
            ...
	}
    	return pizza;
    }
}
{% endhighlight %}
![Alt text]({{ site.url }}/public/conan4.gif)
好啦我們都那麼熟了 再演就不像了 我們在這裡應該用繼承

ChicagoPizzaFactory跟NYPizzaFactory都繼承自SimplePizzaFactory
現在我們需要為不同style的PizzaStore建立不同的工廠 

如果今天我要點NY風味的起司pizza

{% highlight java %}
NYPizzaFactory nyFactory = new NYPizzaFactory();
PizzaStore nyStore = new PizzaStore(nyFactory);
nyStore.orderPizza("cheese");
{% endhighlight %}
明天我想點芝加哥風味的起司pizza
{% highlight java %}
ChicagoPizzaFactory chicagoFactory = new ChicagoPizzaFactory()
PizzaStore chicagoStore = new PizzaStore(chicagoFactory);
chicagoStore.orderPizza("cheese");
{% endhighlight %}

看似沒啥問題 我們現在其實進入了見山不是山見水不是水的境界

![Alt text]({{ site.url }}/public/spin.gif)

為什麼這麼說呢 我們為了decouple**物件的創造**和**物件的使用** 製造了一個工廠
可是為了reuse工廠的code 我們使用了繼承
![Alt text]({{ site.url }}/public/factory3.png)

現在我的SimplePizzaFactory其實只是一個介面 我定義了所有繼承了我的class應該要做什麼事(返回customized defined pizza) 真正實際創造物件的地方是子類別的實體工廠

這兩種功能(decouple + hierarchy)同時需要的時候 我們就可以用上今天的主角

### 工廠方法模式

**工廠方法模式定義了一個建立物件的介面 但由子類決定要實例化的類別為何 工廠方法讓類別把** _**實例化**_ **的動作推遲到了子類**

我們現在把createPizza拉回來PizzaStore裡 **讓子類別來決定怎麼createPizza**
{% highlight java %}
public abstract class PizzaStore {
    abstract Pizza createPizza(String type);
    public Pizza orderPizza(String type) {
	Pizza pizza = createPizza(type);
	pizza.prepare();
	pizza.cook();
	return pizza;
    }
}
{% endhighlight %}
createPizza是抽象方法 留給子類別繼承

讓NYPizzaStore去繼承PizzaStore, 實作createPizza
{% highlight java %}
public class NYPizzaStore extends PizzaStore {
    Pizza createPizza(String item) {
	if (item.equals("cheese")) {
   	    return new NYStyleCheesePizza();
	} else if (item.equals("veggie")) {
	    return new NYStyleVeggiePizza();
	} else if (item.equals("clam")) {
	    return new NYStyleClamPizza();
	} else if (item.equals("pepperoni")) {
	    return new NYStylePepperoniPizza();
	} else return null;
    }
}
{% endhighlight %}

原本我物件的建立 交給一個外來的工廠處理 現在我把它交給我的子類別處理 
而且父類別還可以call子類別實作的函數
![Alt text]({{ site.url }}/public/factory2.png)
這種會互call的function通常依賴性都很高
但我們利用工廠模式讓父類別跟子類別的依賴鬆綁(decouple)了

套用了工廠方法模式之後 怎麼點pizza呢

{% highlight java %}
PizzaStore nyStore = new NYPizzaStore();
Pizza pizza = nyStore.orderPizza("cheese");
{% endhighlight %}

輕鬆

### 結構

![Alt text]({{ site.url }}/public/factory4.png)

* Product(Pizza): 定義factoryMethod(createPizza)所造物件的介面

* ConcreteProduct(NYStyleCheesePizza): 實作Product

* Creator(PizzaStore): 宣告factoryMethod(必須傳回Product) 和其他client可以call的API

* ConcreteCreator(NYPizzaStore): 實作factoryMethod 回傳ConcreteProduct的instance

有個小細節 其實工廠方法不一定是abstract 也可以Creator就先偷偷實作factoryMethod 回傳Product, subclass可以選擇要不要override工廠方法



### 優缺點

優點除了跟簡單工廠一樣 隱藏了創建物件的細節 最重要的是加入新產品不需要改動Creator 你直接繼承Creator就好了 
client的用法都是一樣不需要改 完全符合[開放封閉守則]({{ site.url }}/2017/04/18/decorator/#開放封閉守則)

缺點就是ConcreteCreator跟ConcreteProduct會成對的增加 比如你今天想做加州披薩 你在定義完加州pizza之後 還要再定義一個加州pizza工廠

![Alt text]({{ site.url }}/public/problemFactory.jpg)

下一篇進入Design Pattern重頭戲 抽象工廠
