---
layout: post
title: 每個程序員都該瞭解的JVM - 運行時數據區
comments: True 
subtitle: 運行時數據區
tags: jvm
author: jyt0532
---

## 我們在哪

![Alt text]({{ site.url }}/public/jvm/jvm-5-1.png)

## 介紹

JVM會把內存分配成不同的區域 每個區域都有各自的用途 創建及銷毀時間 是否內存共享

JVM對於內存的管理 造就了JAVA跟C/C++最大的差異 Java工程師在虛擬機內存自動管理的機制之下 不需要去擔心每一個創建的物件是否有銷毀 這些都由虛擬機幫你處理 
所以了解這一塊至關重要 如果是寫C/C++ 你必須在你的程式裡負責好內存分配 所以程式執行出了什麼問題你可以debug找到 
但因為JAVA自動幫你管理內存 所以要是你不知道JAVA是怎麼管理的 發生問題你將debug無門

其中 程序計數器(PC) 棧(Stack) 跟 本地方法棧(Native method stack) 是每個線程私有 其他則是所有線程共享 

本篇會深入講解在運行時數據區的所有模型

## 程序計數器Program Counter

存放一個線程當下在執行的指令地址 藉由改變PC的值來確定下一條要執行的指令位址 因為一個程式可能會有循環 分支等等  所以下一條指令的地址並不一定是依序增加

每一個CPU或CPU core在任一時間只能執行一個線程中的指令 線程之間會彼此搶奪CPU資源 所以要是CPU從線程A跑去線程B跑一跑後想要回來 必須要知道剛剛線程A 跑到哪一行指令 這就是PC的目的

PC屬於線程私有 當線程正在執行JAVA方法的時候 PC指到方法區中字結碼指令的地址(別忘了方法區存了所有方法的byte code) 但若執行的是Native方法 PC的值是null

既然這只是一個pointer 就不會有內存溢出的問題

## 虛擬機棧 JVM Stack

線程私有且生命週期跟線程一樣 這個Stack跟你在其他語言中看到的Stack一樣 當一個線程要執行一個方法的時候 都會創建一個棧楨(Stack frame) 丟進虛擬機棧 一個方法跑完之後 再把棧楨丟出虛擬機棧 如同我們熟悉的LIFO結構

如果方法A被執行了 關於方法A的棧楨會被push到虛擬機棧 跑著跑著如果方法A呼叫方法B 那關於方法B的棧楨會被push到虛擬機棧裡 也就是方法A的棧楨的上面 依此類推 這樣子在方法B跑完之後 方法B的棧楨pop出來後 再繼續執行方法A

注意 方法跑完並不是讓棧楨pop出Stack的唯一方式 一個方法如果丟出Error且沒人處理 也同樣會讓棧楨pop出Stack

分配給每個線程的虛擬機棧大小可以藉由參數給定-Xss 如果虛擬機棧的大小給定 而你這個方法還是用超過允許的大小 則會拋出`StackOverflowError` 但其實大部分的虛擬機棧大小可以動態擴展 如果擴展到太大 超過允許的內存空間 就會拋出`OutOfMemoryError`



## 棧楨 Stack frame

每當**一個方法**被執行 關於這個方法的資訊就會被存在棧楨裡 並且push到虛擬機棧的最上面
棧楨裡面記錄的關於一個方法的資訊 包含了

1.局部變量

2.操作數棧

3.動態連接方法

4.返回位置

以下會一一講解

### 局部變量 Local Variables

在棧楨裡面的局部變量 由數組來儲存 每一個Slot是4個byte 

這個Array存著在編譯時期就已經知道會有的基本數據類型(boolean, byte, char, short, int ,float, long, double)和對象引用(reference) 除了long跟double會佔據兩個slot之外 其他的每個數據都是一個slot 如果是基本數據類型 那就簡單 這個slot就是存這個數據類型的值
但若如果是引用數據類型 那這個slot存的就是他指到的物件在heap裡面的位置

如果這個方法是個instance method 那麼這個數組的第一個slot(索引值0)放的就會是這個instance本身(“this” reference) 然後才開始放方法的input參數 如果這個方法是static 那第一個就從方法的input參數開始放起 

這個設計也是非常的直觀 每一個局部變量array的第一個都放instance的reference 就可以很輕易的在執行方法的時候去改變這個instance的值

局部變量的大小在編譯時期就已經確定 隨著方法運行並不會改變大小

### 操作數棧 Operand Stack

最簡單的解釋 就是把操作數棧想成是**暫存器**的棧 可以用這些暫存器來做方法指令的計算 

我發現這個東西用講得實在不好理解 筆者自身也是看了好幾次搭配例子才了解 我直接帶領你們走最快的道路


{% highlight java %}
int x = 1;
int y = 2;
int z = x + y;
{% endhighlight %}

要怎麼做到這件事情呢 首先你要把其中一個暫存器存1 然後存進局部變量array裡 再把另外一個暫存器存2 然後存進局部變量array裡 
然後 再把這兩個局部變量讀出來到暫存器裡 加在一起之後 再寫回局部變量裡

Instruction set如下圖:

![Alt text]({{ site.url }}/public/jvm/jvm-5-2.png)

詳細過程如下圖

![Alt text]({{ site.url }}/public/jvm/jvm-5-3.png)

注意

1.別忘了operand stack是個棧 所以執行iadd就是從stack拿最上面的兩個 加完之後再放回stack

2.今天的例子剛好這三個區域變數放在區域變數數組的前三個 如果這個方法是instance方法的話 第0個應該要放這個instance本身 如果這個方法有input參數的話 input參數在區域變數數組的順序 也會出現在x, y, z這三個區域變數之前


### 動態連接方法

每個棧楨都包含了指向**運行時常量池**中這個棧楨所屬方法的引用

文言文看不懂 我來翻成白話 
![Alt text]({{ site.url }}/public/jvm/jvm-chou-5-1.jpeg)

什麼意思呢 我們知道常量池裡面有非常多的符號引用 這會在類加載的 解析 階段被解析成直接引用 這些在類加載階段就解析的步驟 稱為靜態解析 

但也有些符號引用 會在運行時 在**這個符號引用第一次被使用的時候 進行動態連接** 這就是為什麼

棧楨 需要存有指向 運行時常量池 的 這個棧楨所屬方法 的 引用

(記不記得考GRE閱測的時候 一個句子都要看老半天才知道誰是主詞誰是受詞 這句話就考察閱測實力)

這樣才可以讓JVM幫你動態解析

### 方法返回地址

也很直觀 沒有這個你就不知道這個方法跑完時(或是遇到異常且沒人處理時) 也就是棧楨需要pop出棧的時候 你的程序計數器要回到哪裡去

### 心中有圖形

JVM最高點 心中有圖形

以上關於stack frame的介紹 在很多網路上的文章都找得到 但願意認真解釋到你懂的人還真不多 恰巧一個就是我 我在這裡直接Demo一個例子給你看

{% highlight java %}
void foo(){
  int x = 1;
  bar(x);
}
void bar(int a){
  long b = 2L;
  Hello h = new Hello();
} 
{% endhighlight %}

跑到`foo`方法的時候 一個棧楨被推進Stack

![Alt text]({{ site.url }}/public/jvm/jvm-5-4.png)

假設foo方法在bytecode位置0x6666的地方呼叫bar 一個全新的棧楨再被推進Stack

![Alt text]({{ site.url }}/public/jvm/jvm-5-5.png)

在heap創建了一個物件`new Hello()`

![Alt text]({{ site.url }}/public/jvm/jvm-5-6.png)

bar方法跑完  棧楨pop出stack 把程式計數器更新為0x6666 

![Alt text]({{ site.url }}/public/jvm/jvm-5-7.png)

Foo從0x6666開始往下跑 至於方法內部的運算在介紹棧楨的時候已經有例子了 就不再贅述

方法跑完  棧楨pop出stack 程式計數器更新為0x2222 也就是呼叫foo之前所在的地方


![Alt text]({{ site.url }}/public/jvm/jvm-5-8.png)

至於在Heap的Hello 因為已經沒有人指到了 所以他遲早會被垃圾回收器回收


各位客官 搭配了圖形之後 想必你已經很清楚Stack跟Stack Frame的關係 以及Stack Frame裡面記錄的資訊

這個圖細節到局部變量表的第一個擺`this`, long型態的佔了兩個slot 簡直用心良苦

![Alt text]({{ site.url }}/public/jvm/jvm-chou-5-2.jpg)

你 看懂了嗎?

## 方法區 Method Area

所有線程共享 主要放著每一個被加載的類文件信息 注意這個區域也可能被垃圾收集器收集 

來複習一下 還記不記得我們講類加載是什麼呢 當虛擬機要加載一個類別 他使用了某一個加載器去找到class file 然後加載器去讀calss file 去拿到他所需要的資訊

類文件 存在堆中

所有關於這個類別的資訊 存在方法區

複習完畢 我們知道方法區存了每個被加載的類別的資訊 所以對於每一個類別 都有以下的八種meta-data 

![Alt text]({{ site.url }}/public/jvm/jvm-5-9.png)

![Alt text]({{ site.url }}/public/jvm/jvm-5-18.png)

### Runtime Constant Pool 運行時常量池

存放該類型所用到的常量的有序集合 包括直接常量和對其他類型或字段或方法的符號引用 跟數組一樣透過index訪問 
注意這裡的運行時常量池跟類別的常量池非常像 最大的差別是運行時常量池具備動態性 除了編譯時期把類別的常量池置入之外 運行時如果有新的常量 也可以動態加入運行時常量池 


### Type Information 類型信息

1.類型的完全限定名(Fully qualified name) : 完全限定名指的是包含類別的(packageName.typeName) 比如說`java.lang.Object`

2.類型是接口還是類別

3.類型的父類的完全限定名: 因為Class沒有多重繼承 所以如果是此Type是類別 就只有一個父類 但若此Type是接口 那就照順序存父接口 

4.類型的訪問修飾(public, final, abstract)


### Field Information 字段信息

對於一個類別的所有字段 都必須存在方法區

1.字段名稱

2.字段類別

3.字段修飾符(public, private, static, final, transient, volatile)

請容我稍微打岔一下 我們複習過很多次 instance variable的引用被存在棧 這個沒問題 但是static variable的引用被存在哪裡 有沒有人想過這個問題

答對了 就是方法區！ 因為這個資訊是對於所有這個類別的物件共用的

所以 列出所有排列組合給你

|                         | 原始類型(primitive type) | 引用類型(reference) |
|-------------------------|----------------------|-----------------|
| 靜態變量(static variable)   | 值:方法區                | 引用: 方法區<br>值: 堆          |
| 實例變量(instance variable) | 值:堆                  | 引用: 堆<br>值: 堆            |
| 局部變量(local variable)    | 值:棧                  | 引用: 棧<br>值: 堆            |


上範例程式

{% highlight java %}
public class Demo {
 public static void main(String args[]){
   int a = 1;
   Integer b = 2;
   Hello c = new Hello();
 }
}
public class Hello {
 int d  = 3;
 static int e = 4;
 Foo f1 = new Foo();
 static Foo f2 =  new Foo();
}
public class Foo {
}
{% endhighlight %}

![Alt text]({{ site.url }}/public/jvm/jvm-5-10.png)

聰明如你一定看得懂的 我就留給看官細細體會

### Method Information 方法信息

當然類別中的每個方法都得存下來

1.方法名稱

2.方法返回類型

3.方法的所有參數 包括類型跟順序

4.方法修飾符(public, private, static, final, synchronized, native, abstract)

5.方法的內容: 對於不是native也不是abstract的方法 我們必須要記下方法的bytecode 之後Program counter才知道要跑哪一行程式要做什麼

6.操作數棧跟局部變量數組的大小(見[棧楨](#棧楨-stack-frame))

7.異常表

### Class variable 類變量

類別裡面的static變量 所有對象共享 類變量分兩種 一種是final 一種是non-final
回想一下在類加載的準備階段 我們為所有non-final的類變量預留空間 並設為預設值 這裡就是在幫non-final的類變量分配空間的地方 
至於final變量 通常直接存在運行時常量池或是直接複寫方法的bytecode

### Method table 方法表

每一個類別的instance method都會在方法表裡面有一個entry 可以讓虛擬機快速地知道應該要執行哪一個方法(可能是繼承而來的父類方法 也可能是自己的方法)

### Reference to ClassLoader table指向加載區的指標

在類加載一章說道 我們有不同的類加載器 如果一個類別是由自定義類加載器加載 則在這個地方必須要存下指到那個類加載器的指標 目的很簡單 為了動態解析 我們要動態解析一個類別的方法 一定需要同一個類加載器 這就是為什麼你要把加載你的人記下來 以備不時之需

### Reference to Class object指向類物件的指標

對於每一個type 都會有一個指向heap裡面的類物件的指標 就可以用類物件來創建物件


回想一下 我們在類加載器在判斷要不要加載的時後 有這麼一張圖

![Alt text]({{ site.url }}/public/jvm/jvm-3-2.png)

第一個菱形 判斷類物件在不在heap 並不是去heap裡面從頭找到尾 而是到方法區看有沒有這個class 有的話代表類物件已經在heap了 就可以直接回傳類物件(然後程式就可以用類物件來生物件等等)的指標

## 堆Heap

看到這裡 你應該已經要對堆很熟悉了

堆的用途就是存放對象實例 也是虛擬機內存中最大的一塊 所有線程共享 在虛擬機啟動時創建 所有的對象實例和數組都存在這裡 包含類物件 也是垃圾收集器作用的地方 為了要達成更有效率的垃圾回收 還會再把這塊區域分配成新生代老年代等等 在垃圾回收一節會再詳談

如果堆中已經沒有空間可以給新建的實例 則會拋出OutOfMemoryError異常

調整你的heap size對於程式的效能有至關重要的影響(-Xmx, -Xms)

對於每一個在堆裡的物件 都會有相對應的Class data在方法區裡


### 字串池(String pool)

堆裡面有一個特別重要的區域 專門處理字串 這值得在這個小節特別一提

你可能很常看到這樣的JAVA面試題

{% highlight java %}
String s1 = "aaa";//string literal
String s2 = "aaa";
String s3 = "bbb";
String s4 = new String("aaa");//using new
String s5 = new String("aaa");
{% endhighlight %}

Q1: s1 == s2 return true or false

Q2: s2 == s4 return true or false

Q3: s4 == s5 return true or false

要弄懂之前 要先知道

1.如果你用的是string literal來創建一個新的字串 那麼這個字串本身會被存在字串池 字串池的目的就是節省空間(重複的東西不要重複創建) 

2.如果你用的是new來創建一個新的字串 那麼這個字串會被存在heap

3.== 比較的不是兩邊東西的值 而是兩邊reference的記憶體位置

所以公佈答案 

A1: s1 == s2 return true 因為在創建s2的時候 字串池已經有”aaa”了 就直接用同一個address給s2 不再重新創建	

A2: s2 == s4 return false 因為s4在heap裡

A3: s4 == s5 return false 因為s5在heap裡 但和s4不同位置

JVM最高點 心中有圖形

![Alt text]({{ site.url }}/public/jvm/jvm-5-11.png)

意猶未盡是吧 我們再來個例子

{% highlight java %}
String s1 = new String("aaa");
String s2 = s1.intern();
String s3 = "aaa"
String s4 = "bbb";
String s5 = new String("bbb");
{% endhighlight %}

那什麼是intern()呢 當你對一個字串call intern()時 他會先去看字串池有沒有這個常量 有的話就回傳這個常量的地址 沒有的話 就在字串池生一個字串常量 然後回傳這個常量的地址

心中有圖形

![Alt text]({{ site.url }}/public/jvm/jvm-5-12.png)

事實上 對於任何一個String literal 

{% highlight java %}
String s = "abc";
{% endhighlight %}

你都可以想成是


{% highlight java %}
String s = new String("abc").intern();
{% endhighlight %}

這樣子世界就容易許多 intern()做的事情 就是把這個String拿去String pool找一下有沒有存在過 有的話 就回傳在String pool裡面的引用 沒有的話 就生一個 然後回傳這個新的引用



讓我們再深入一點點 

{% highlight java %}
String py = "py";
String s1 = "Happy";
String s2 = "Hap" + "py";
String s3 = "Hap" + py;
String s4 = "Hap" + py.intern();
System.out.println((s1 == s2));
System.out.println((s1 == s3));
System.out.println((s1 == s4));
{% endhighlight %}

回傳結果是true, false, false 心中有圖形

![Alt text]({{ site.url }}/public/jvm/jvm-5-13.png)


值得一提的有兩點

1.String literal + String literal之後 會再call一次intern() 就像s2一樣

2.在建造s3的時候 因為不知道py的值是什麼 所以根本不可能在compile-time 呼叫intern() 但如果py是final的話 s1 == s3

3.s4也一樣 compile-time根本不知道py是什麼東西

讓我們再深入一點點 兩個常見的面試問題


{% highlight java %}
String s = new String("Happy!")
{% endhighlight %}

請問這行程式 創造了幾個object?

答案是兩個 它先生了”Happy!”物件到string pool裡面 然後再生一個String object在heap裡

{% highlight java %}
String s1 = new String("Happy!")
String s2 = new String("Happy!")
{% endhighlight %}
請問這行程式 創造了幾個object?

答案是三個 s1跟上一個問題一樣 但跑到s2的時候 因為”Happy!”物件已經在字串池裡 所以不再重複 所以只需要再生一個在heap裡即可

![Alt text]({{ site.url }}/public/jvm/jvm-5-19.jpeg)

## 棧跟堆跟方法區的互動

努力讀到這裡 真是辛苦你了 我信了你有著想徹底了解JVM的熱情了

![Alt text]({{ site.url }}/public/jvm/jvm-5-20.jpg)

為了犒賞你的認真用功 我決定要在這個章節讓你嘗到豐收的果實

天知地知你知我知 物件會被存放在Java堆中 引用放在棧中 其他跟類別有關的東西放在方法區 而我們從加載這個章節得知 Java堆不只存放類別實例 也存放著類物件

至於類別裡面的每個方法的名稱 描述符 內容等等 則是存放在方法區裡面 其實這也很直觀 對於同一個類別可能有數以百計的物件 我們也不可能每個在堆裡的物件都有每個方法或字段的資訊 因為其實也是重複的資訊 所以關於類別的資訊我們放在方法區 關於物件的資訊我們放在堆

直上例子 今天我的程式第一次需要用到Hello的物件

{% highlight java %}
Hello hello1 = new Hello();
{% endhighlight %}

這時 類加載器先去看堆裡面有沒有Hello的類物件 發現沒有 開始加載 加載完畢後 類物件在堆裡 類別資訊在方法區

![Alt text]({{ site.url }}/public/jvm/jvm-5-14.png)

注意這兩個東西彼此有引用可以指到對方

然後 JVM在堆中分配空間 生成一個Hello實例

![Alt text]({{ site.url }}/public/jvm/jvm-5-14-1.png)


當我的程式第二次需要用到Hello的物件

{% highlight java %}
Hello hello1 = new Hello();
{% endhighlight %}

類加載器發現 Hello的類物件已經在堆裡了 所以不需要加載了 直接生實例

![Alt text]({{ site.url }}/public/jvm/jvm-5-15.png)

當我的程式第一次需要用到World的物件

{% highlight java %}
World world = new World()
{% endhighlight %}

![Alt text]({{ site.url }}/public/jvm/jvm-5-16.png)

這樣是不是融會貫通了呢?

希望看完這一段的解釋之後 你理解為什麼類物件跟類別資料要分開放 而生出這兩個東西就是[類加載](/2020/03/02/jvm-class-loader/)階段的目的

## 本地方法棧Native Method stack

講解這個模型之前 一定要先知道什麼是本地方法 

### 本地方法(Native method)

本地方法代表這個方法是用其他語言寫的 在早期的時候Java還沒有那麼快 跑最快的還是當之無愧的C 對於某些要求性能的某些程式區塊 我們就可以呼叫其他語言的函示 來加速執行 

注意 [Effective Java](/toc/effective_java/)的作者並不提倡使用本地方法

### 本地方法棧

本地方法棧與虛擬機棧所發揮的作用非常相似 區別不過是虛擬機棧為虛擬機執行Java方法(也就是字節碼)服務 而本地方法棧則為虛擬機使用到的Native方法服務

值得一提的是 Java可以call C語言 C語言也可以再call Java

![Alt text]({{ site.url }}/public/jvm/jvm-5-17.png)

簡單來說 就是JVM很貼心的專門給本地方法留一個Stack讓它去玩

## 總結

我們已經把運行時數據區的所有內容模型給講解完畢 

![Alt text]({{ site.url }}/public/jvm/jvm-5-1.png)

相信你可以比其他Java開發者看得更深 更透徹
