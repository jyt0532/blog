---
layout: post
title: Effective Java Item77 - For instance control, prefer enum over readResolve
comments: True 
subtitle: effective java - 用Enum實現物件控制
tags: effectiveJava enum
author: jyt0532
---
這篇是Effective Java - For instance control, prefer enum over readResolve章節的讀書筆記 本篇的程式碼來自於原書內容

在看這篇文章之前 強烈建議先看過[Item3](/2017/10/20/enforce-the-singleton-property-with-a-private-constructor-or-enum/)

## Item77: 對於實例控制 枚舉優先於readResolve

在[Singleton](/2017/05/19/singleton/)中說明了很多實現單例的方法 大多數都是利用private constructor來實現

{% highlight java %}
public class Elvis {
  public static final Elvis INSTANCE = new Elvis();

  private Elvis() {
  }

  public void leaveTheBuilding() {
    System.out.println("Whoa baby, I'm outta here!");
  }
}
{% endhighlight %}

今天如果你想要序列化這個應該要是Singleton的類別 如果只是加上implements Serializable 那他就再也不是Singleton

就如我們所說 readObject其實也可以看做一個constructor 只是input argument是byte stream 不論readObject是預設的還是你自行定義的都一樣
你反序列化後拼出來的那個東西就是個新的instance

該怎麼辦呢

### readResolve

對於一個正在被反序列化的物件 如果他的class定義了readResolve 那麼在反序列化完之後 就會call readResolve

對於一個正在被反序列化的對象 如果他的類定義了一個readResolve 
那麼在反序列化之後 新建對象上的readResolve方法就會被調用 

然後該方法返回的對象引用將會被返回 取代新建的對象

{% highlight java %}
public class Elvis implements Serializable{
  public static final Elvis INSTANCE = new Elvis();

  private Elvis() {
  }

  private Object readResolve() {
    return INSTANCE;
  }

  public void leaveTheBuilding() {
    System.out.println("Whoa baby, I'm outta here!");
  }
}
{% endhighlight %}

所以你只要在readResolve裡面回傳當初那個唯一的instance就搞定 剛創出來的物件就那麼隨風而去 煙消雲散

![Alt text]({{ site.url }}/public/dreamleft.gif)

既然如此 那這個class就不應該有需要被傳輸的instance variable(反正到最後你反序列化完之後 也是return原本的singleton object) 所以**如果你依賴readResolve進行實例控制 所有instance variable應該要是transient**

不然的話可能會遭受到下列的攻擊

### 攻擊例子

如果一個你想支援序列化的Singleton包含了一個非transient的對象 favoriteSongs

{% highlight java %}
public class Elvis implements Serializable {
  public static final Elvis INSTANCE = new Elvis();

  private Elvis() {
  }

  private String[] favoriteSongs = { "Hound Dog", "Heartbreak Hotel" };

  public void printFavorites() {
    System.out.println(Arrays.toString(favoriteSongs));
  }

  private Object readResolve() throws ObjectStreamException {
    return INSTANCE;
  }
}
{% endhighlight %}

那麼就很有機會可以做一個壞事 我們可以偷偷寫一個壞Class
這個Class也有一個instance variable 也有一個readResolve
{% highlight java %}
public class ElvisStealer implements Serializable {
  static Elvis impersonator;
  private Elvis payload;

  private Object readResolve() {
    // Save a reference to the "unresolved" Elvis instance
    impersonator = payload;

    // Return an object of correct type for favorites field
    return new String[] { "A Fool Such as I" };
  }

  private static final long serialVersionUID = 0;
}
{% endhighlight %}

這樣子的話 我們如果偷偷改了一個傳輸中的byte stream 變成這樣
![Alt text]({{ site.url }}/public/item77-1.png)

的話 就會產生兩個不同的Elvis instance了

{% highlight java %}
public static void main(String[] args) {
  Elvis elvis = (Elvis) deserialize(serializedForm);
  Elvis impersonator = ElvisStealer.impersonator;

  elvis.printFavorites();
  impersonator.printFavorites();
}
{% endhighlight %}

{% highlight sh %}
[Hound Dog, Heartbreak Hotel]
[A Fool Such as I]
{% endhighlight %}

YAY! 皆大歡喜 又過了和平的一天 上面的code我相信所有唸過這本書的九成都沒看懂就這樣帶過 
總之知道結論就好 不要有non-transient的變數

但若想要變強 就得要參加死亡行軍

![Alt text]({{ site.url }}/public/item77-2.png)

### 愛上地獄的男人們

要讀懂這個section要先完全通透[序列化基本知識](/2017/09/27/java-serialization-101/)和[深入解析序列化byte stream](/2017/10/12/decrypting-serialized-java-object/) 不然沒有讀下去的必要

要看懂這個攻擊 鐵定是要認真讀一下這個byte stream
![Alt text]({{ site.url }}/public/item77-1.png)
???
![Alt text]({{ site.url }}/public/item77-3.jpg)

![Alt text]({{ site.url }}/public/item77-6.png)

注意malicious的那一個block 原本那個block應該要開始寫favoriteSongs 也就是個長度是2的String Array 分別是"Hound Dog"跟"Heartbreak Hotel"

可是卻被竄改成了ElvisStealer的instance

現在你再重新回想一下序列化Elvis的順序 

1.寫下Elvis的Class descriptor

2.如果Elvis有parent class 就一路寫class descriptor上去(在這個例子 沒有parent class)

3.開始寫non-transient的variable(別忘了static variable不寫)

4.這個variable的Class是個我沒看過的Class(ElvisStealer) 那再開始寫ElvisStealer的class descriptor

5.寫完後 再來寫這個instance的value

### 好戲登場

記不記得我們說readResolve會在反序列化完之後被call 而且readResolve回傳的就是最終反序列化的結果

我們用一點變數來輔助說明 

1.假設我們想把剛剛那個byte stream反序列化成一個Elvis的object **A** 這個object的reference是**refA**

2.Elvis已存在一個singleton object **I**

然後我們再反序列化Elvis的時候(此時refA已經存起來了 71007e0002) 因為看到了新的Class 所以當**反序列化Elvis到一半的時候** 我們先反序列化ElvisStealer 然後當ElvisStealer的Class descriptor寫完時 我們assign 變數payload的值是**refA**

![Alt text]({{ site.url }}/public/item77-7.png)

值assign完了之後 ElvisStealer的readResolve會先被call 

{% highlight java %}
private Object readResolve() {
  // Save a reference to the "unresolved" Elvis instance
  impersonator = payload;

  // Return an object of correct type for favorites field
  return new String[] { "A Fool Such as I" };
}
{% endhighlight %}

他把他手上的payload變數assign給他的static variable impersonator 然後return了 一個Array of String {"A Fool Such as I"} 所以refA被偷偷地記在了ElvisStealer裡面 但是對於Elvis來說 覺得它的favoriteSongs是{"A Fool Such as I"} 因為這個Array of String就是那一整個malicious block反序列化的**結果**

然後所有byte stream讀完後 開始跑Elvis的readResolve 那就簡單了 剛剛辛苦了老半天的A沒有人要 對於Elvis來說煙消雲散了(至少Elvis以為煙消雲散) 直接回傳那個singleton object **I**

這下好了 看回主要程式

{% highlight java %}
public static void main(String[] args) {
  Elvis elvis = (Elvis) deserialize(serializedForm);
  Elvis impersonator = ElvisStealer.impersonator;

  elvis.printFavorites();//print I.favoriteSongs
  impersonator.printFavorites();//print refA.favoriteSongs
}
{% endhighlight %}

{% highlight sh %}
[Hound Dog, Heartbreak Hotel]
[A Fool Such as I]
{% endhighlight %}

這樣就打破了Singleton的規範 elvis變數和impersonator應該要是同一個物件 有著相同的行為才對

所以結論是 當你要序列化一個Singleton而且要用readResolve的話 不要有non-transient的變數

這樣 你看懂了嗎

### 所以

最好的實現Singleton方式 還是用enum

{% highlight java %}
public enum Elvis {
  INSTANCE;
  private String[] favoriteSongs = { "Hound Dog", "Heartbreak Hotel" };

  public void printFavorites() {
    System.out.println(Arrays.toString(favoriteSongs));
  }
}
{% endhighlight %}

### 總結

可以用enum實現Singleton就用enum 直接幫你處理好序列化的問題 但如果你無法用enum 但你又需要序列化和instance control 那就要提供一個readResolve並且所有instance variable都要是non-transient
