---
layout: post
title: Design Pattern(5) - Abstract factory
comments: True 
subtitle: design pattern - abstract factory
tags: designPattern
author: jyt0532
---
{% include design_pattern.html %}

### 前言

從工廠方法接到抽象工廠花了我不少時間去理解 
這一塊一開始我覺得深入淺出寫的不是很好 它把工廠方法到抽象方法的銜接講的太理所當然 
這驅使我去找了很多相關的資料 發現每個人講解抽象工廠的出發點都不太一樣 

有點像是每個人對於同樣一件事 有不同的角度去解讀 
而寫文章的人最希望下筆的角度 是他**最不熟悉的**角度 畢竟沒有人會想筆記你原本就會的東西

學習抽象方法的過程有點像是拼一塊拼圖 每個連結都給了我不同部分的拼圖
當我全部拼完 知道這是怎麼回事了之後 再回頭看每個連結 都講得有它的道理
現在我就覺得深入淺出講得不差 只是某些他覺得理所當然的知識我並不清楚
這才是我卡住的主因

其中最重要的一塊拼圖是[圖說設計模式](http://design-patterns.readthedocs.io/zh_CN/latest/creational_patterns/abstract_factory.html) 這是個講Design pattern非常好的連結 

拼圖拼完後 我想過為什麼我會搞混抽象工廠跟工廠方法
仔細想過之後 我認為簡單工廠跟工廠方法要先用例子解釋 但抽象方法要從使用時機下筆

大家就試試從我的角度去理解抽象工廠吧!

### 工廠方法的限制

工廠方法的factoryMethod 只能創建一個對象 比如說Pizza 

但如果我們想要更加細分想創建的東西 比如說Pizza的所需原料(麵團 醬料 起司 蛤蠣) 
如果我們用工廠方法的話 我們需要為每一個原料都創一個工廠

在這個例子就是NY麵團工廠 NY醬料工廠 NY起司工廠 NY蛤蠣工廠 Chicago麵團工廠 Chicago醬料工廠 Chicago起司工廠 Chicago蛤蠣工廠
**因為每個工廠方法只能生產一個產品** 

{% highlight java %}
public class NYPizzaStore extends PizzaStore {
    Pizza createPizza(String item) {
	DoughFactory doughFactory = new NYDoughFactory();
	SauceFactory sauceFactory = new NYSauceFactory();
	CheeseFactory cheeseFactory = new NYCheeseFactory();
	ClamFactory clamFactory = new NYClamFactory();
	if (item.equals("cheese")) {
    	    return new NYStyleCheesePizza(doughFactory, 
		sauceFactory, cheeseFactory);
	} else if (item.equals("veggie")) {
	    return new NYStyleVeggiePizza(doughFactory, 
		sauceFactory, cheeseFactory);
	} else if (item.equals("clam")) {
	    return new NYStyleClamPizza(doughFactory, 
		sauceFactory, cheeseFactory, clamFactory);
	} else if (item.equals("pepperoni")) {
	    return new NYStylePepperoniPizza(doughFactory, 
		sauceFactory, cheeseFactory);
	} else return null;
    }
}
{% endhighlight %}

這樣實在是太難maintain了 **所以我們把相關的產品(NY麵團工廠 NY醬料工廠 NY起司工廠 NY蛤蠣工廠)組成一個產品族 交給同一個工廠來生產** 鼎鼎大名的抽象工廠就誕生了

![Alt text]({{ site.url }}/public/abstractfactory1.png)

抽象的Pizza原料工廠定義了每個原料工廠要創建的東西的介面 
每個繼承了Pizza原料工廠的具體原料工廠乖乖implement**所有**需要創建的東西

再來些例子 如果工廠方法生產的是房子 抽象工廠可以生產一個房子的所有家具(沙發電視電風扇) 
如果工廠方法生產的是武士 抽象工廠可以生產一個武士的所有配備(盔甲 鞋子 手套)

第一眼看起來很可怕 但其實只是把工廠的責任從生產一個產品 變成生產一個產品族

### 產品族和產品等級結構

先解釋兩個名詞

產品等級結構: 產品的繼承結構 比如一個抽象類是麵糰 子類別有Chicago麵團跟NY麵團 這三個形成了一個產品等級結構

產品族: 同一個工廠生產的所有產品 其中的每個產品都是座落在不同的產品等級結構中的其中一個產品

一圖勝過千言萬語
![Alt text]({{ site.url }}/public/abstractfactory2.png)

### 套用抽象工廠後
{% highlight java %}
public class CheesePizza extends Pizza {
    PizzaIngredientFactory = ingredientFactory;
    public CheesePizza(PizzaIngredientFactory ingredientFactory){
	this.ingredientFactory = ingredientFactory;
    }
    void prepare(){
	dough = ingredientFactory.createDough();
	sauce = ingredientFactory.createSauce();
	cheese = ingredientFactory.createCheese();
    }
}
{% endhighlight %}
至於PizzaStore跟他的子類 概念跟工廠方法一樣 由子類決定要實例化的類別為何
{% highlight java %}
public class NYPizzaStore extends PizzaStore {
    protected Pizza createPizza(String item) {
	Pizza pizza = null;
	PizzaIngredientFactory ingredientFactory = new NYPizzaIngredientFactory();
	if (item.equals("cheese")) { 
  	    pizza = new CheesePizza(ingredientFactory);
	} else if (item.equals("veggie")) {
  	    pizza = new VeggiePizza(ingredientFactory);
	} else if (item.equals("clam")) {
	    pizza = new ClamPizza(ingredientFactory);
	} else if (item.equals("pepperoni")) {
	    pizza = new PepperoniPizza(ingredientFactory);			
	} 
	return pizza;
    }
}
{% endhighlight %}
點pizza的方法就跟工廠方法一樣
{% highlight java %}
PizzaStore nyStore = new NYPizzaStore();
Pizza pizza = nyStore.orderPizza("cheese");
{% endhighlight %}

對client來說 用法一樣 client並不知道裡面的產品已經變成產品族 

### 抽象工廠
**用一個抽象工廠來定義一個創建** _**產品族**_ **的介面 產品族裡面每個產品的具體類別由繼承抽象工廠的實體工廠決定**

上面那句話請務必讀懂他

### 結構

![Alt text]({{ site.url }}/public/abstractfactory3.png)

* AbstractFactory(PizzaIngredientFactory): 宣告出各個創建同一產品族產品的介面

* ConcreteFactory(NYPizzaIngredientFactory): 實作AbstractFactory

* AbstractProduct(Dough): 宣告產品等級結構的物品介面

* Product(ThickCrustDough): ConcreteFactory所建構的成品 需要實作AbstractProduct

### 優缺點

1.一樣區隔了每個產品的生成和使用 client被隔離在產品的class之外 client甚至不知道什麼產品被創建了 他只要知道那個產品有哪些函數可以給他call 這個特點使得抽換具體工廠這件事變得非常容易

2.性質類似的產品集中管理(所有NY的原料一起管理 或是所有日式的傢俱一起管理) 
今天有新的工廠要進來 他需要implement的method非常明確 照著AbstractFactory定義的介面實作就對了 某種程度而言也算是給client方便 我保證他只會用到同一個產品族的對象

3.缺點非常致命 就是當**我想在產品族加一個產品**非常困難 因為我所有子工廠要跟著改 這被稱為開閉原則的傾斜性: **新增產品族容易 但新增產品結構困難**

### 使用時機
1.一個系統必須和產品的生成/組合 保持獨立

2.許多類似的產品可以組成產品族 方便集中管理 **而且多於一個的產品族**

3.只想公開產品interface不想公開實作細節

### 細說抽象工廠

準備好豐收融會貫通的果實了嗎? Go!

1.抽象工廠定義了需要創建的產品族 由concreteFactory去實作抽象工廠 所以通常抽象工廠裡的每一個創建product的抽象方法 都是用**工廠方法** 由繼承的具體工廠來實現 這也是這兩個名詞常被搞混的原因
 
2 . 

![Alt text]({{ site.url }}/public/abstractfactory7.png)

由此圖可以看得出來 如果我們用抽象工廠來實作pizza店 我們只需要實作2個工廠(ChicagoPizzaIngredientFactory, NYPizzaIngredientFactory) 
但如果我們用工廠方法來實作 我們需要實作8個工廠

所以當你發現你的工廠方法們 遵循著一個產品族的pattern 試著把這產品族分離出來寫稱抽象工廠的interface 然後用具體工廠實現 
這就是**Design Pattern rule #2: Program to an interface, not an implementation**

3.退化成工廠模式

如果你的抽象工廠裡定義的創建方法只有一個(只有一個產品等級結構) 那你的抽象工廠就退化成工廠方法
![Alt text]({{ site.url }}/public/abstractfactory5.png)
你把上圖的Dough改成Pizza 就是我們上一篇[工廠方法]({{ site.url }}/2017/04/28/factory-method/)在說的東西

4.退化成簡單工廠模式

如果你只有一個具體工廠
![Alt text]({{ site.url }}/public/abstractfactory9.png)
因為每個配料只有一個實作 所以沒有使用interface的必要 你把上圖的各個配料改成各個pizza 就是上一篇的簡單工廠模式的例子

### 總結

只有簡單工廠跟抽象工廠 真的需要實作工廠

工廠方法指的是一個方法 這個方法負責創造東西 且交由子類別負責繼承


> 生命中所殘缺的部分 是一本自傳裡 不可或缺的內容 --席慕容

> 工廠方法所殘缺的部分 是抽象工廠裡 不可或缺的內容 --姜柏宇

