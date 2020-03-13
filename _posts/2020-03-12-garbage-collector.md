---
layout: post
title: 每個程序員都該瞭解的JVM - 垃圾收集器
comments: True 
subtitle: 垃圾收集器
tags: jvm
author: jyt0532
---


## 垃圾收集器


垃圾回收機制 是講到JAVA不能不提到的特性 這個機制讓開發者不用去關心空間的創建跟回收 大大增加了產能 可惜的是 大部分的JAVA開發者並不清楚到底虛擬機為你做了什麼事 

在這一章我會讓你完全通透 原來 有個進程一直在默默地守護著你的記憶體

當你了解了虛擬機為你做的事情之後 中間的很多地方都可以讓使用者給定參數 當你為了你的應用程式給定了最合適的參數 你的程式會比別人有效率很多 這個地方就可以展現出你的專業度 所以這個章節絕對值得一讀

## 我們在哪

![Alt text]({{ site.url }}/public/jvm/jvm-7-1.png)

## 介紹

Java堆是在虛擬機啟動時自動創建 運行過程中創建的對象和數組都放在這個內存空間裡 如果沒有適時地清理 則會造成內存溢出 所以我們需要在一個物件不會再被用到的時候 把這個物件從堆中清掉

在垃圾回收的時候 通常所有的應用程式都被強迫暫停 整個世界因此中止(stop-the-world pause) 在垃圾回收演算法中會再作解釋

要徹底了解垃圾回收機制 首先要先問自己五個問題

Where Why What When How

各個擊破 

1.Where: Heap

2.Why 為什麼需要垃圾收集器

3.What 哪些東西需要被回收: 以後不會再用到的物件

4.When 什麼時候回收

5.How 怎麼回收

## 為什麼需要垃圾回收(Why)

因為Heap裡面會存放所有的instance 而在程式執行完之後 並不需要把分配你程式的空間釋出 所以要是不清理的話 很快的其他程式就沒有空間可以用了 很不方便

來個簡單的例子 順便來複習一下

{% highlight java %}
void a(){
  Object obj = new Object();
}

{% endhighlight %}

執行這個方法的時候 一個棧楨被push到JVM Stack裡面 obj是一個引用 存在棧楨的局部變量表裡 obj的值就是這個Object物件在Heap裡面的位置 那關於這個物件的其他詳細訊息(類型 父類 實現的接口 有哪些方法)等等 則是存在方法區裡面

這個方法跑完了以後 棧楨被pop出JVM Stack 但凡走過必留下痕跡 在heap裡的那個物件怎麼辦 沒有人會用到它也沒有人指到它 這就是我們需要垃圾回收的對象

看完這個解釋之後 你會發現垃圾回收變得很直覺 因為我們不像C語言 每次分配給你的空間你都要負責釋出 在Java我們就把它留在那邊 交給垃圾收集器處理


## 哪些東西需要被回收(What)

堆裡面的東西那麼多 我要怎麼知道哪些應該要被清掉哪些應該要留著呢

### 引用計數法

早期判定一個物件是不是沒有人在使用 我們創建一個計數器 就像一個表格一樣 原理很簡單 只要有一個人指到你 你的引用次數就加一

舉個例子

{% highlight java %}
Person p1 = new Person();
Person p2 = p1;
{% endhighlight %}

這個例子這個剛被建立的物件的計數就是2

如果p1又指到別人或是指到null 那就變回1

但如果p1 跟 p2 都指到別人或是null

{% highlight java %}
p1 = null
p2 = null
{% endhighlight %}

那這個剛剛創建的物件就不可能再被用到了 因為再也沒有人指得到他 那麼他就應該被回收 這就是引用計數法

聽起來不錯 這個方法也很好實作 那為什麼當今主流的虛擬機不用這個方法呢 給你三秒鐘

一秒鐘

&nbsp;

&nbsp;

&nbsp;

&nbsp;


兩秒鐘

&nbsp;

&nbsp;

&nbsp;

&nbsp;

三秒鐘

答對了 這個方法無法檢測到循環引用 就是兩個垃圾互相指著對方

來點code吧

{% highlight java %}

public class ReferenceCount {
    public Object reference = null;
} 

public static void main(String[] args) {
    ReferenceCount rc1 = new ReferenceCount();
    ReferenceCount rc2 = new ReferenceCount();
    rc1.reference = rc2;
    rc2.reference = rc1;
}
{% endhighlight %}

就這麼簡單 

main程式跑完之後  堆裡面的rc1和rc2的引用計數都是1 但卻再也沒有人指得到 垃圾收集器也不能清掉


![Alt text]({{ site.url }}/public/jvm/jvm-7-1-1.jpeg)


引用計數法防不了這種循環引用 所以當今的虛擬機都不用這種方法

### 可達性分析法

這個方法是目前最主流的判斷存活的方法 當今其他需要支援垃圾回收的語言都是選用這個方法 方法也很簡單 虛擬機存著一些GC Root 他就是物件的頭 由GC Root開始跑一些很常見的樹的遍歷演算法 所有會被遍歷跑到的 就是存活者 跑不到的 就是垃圾

也是很直觀 舉個例子

![Alt text]({{ site.url }}/public/jvm/jvm-7-2.png)

在這個例子 在座的Object6跟Object7 都是垃圾

#### GC Root

那什麼是GC Root呢 下列的人選都是我們的GC Root

1.JVM棧楨裡的局部變量 指到的對象: 理所當然 棧楨還在 那局部變量指到的都不應該被回收

2.方法區裡的static變數 指到的對象: 理所當然 類別等級的變量 即使沒有任何的實例 只要方法區的類別沒有被回收 那方法區中跟這個類別相關的靜態變數都不該被清理 因為那些靜態變數已經被加載了 代表隨時可能會用到

3.系統類別(System Class): 也就是被啟動類加載器加載的類別 比如說`rt.jar`(Runtime enviroment) 或是`java.util.*`等類別

4.本地方法棧中的本地方法所引用的對象: 也是理所當然 人家棧裡面的東西都不該清掉 包含了本地方法的局部變數跟全局變量

5.線程(Thread): 所有正在跑的線程以及這些線程所指到的物件

6.在finalizer queue裡面 還沒跑finalizer方法的物件

被這些指到的 都是菁英中心裡面的人 沒被指到的就是垃圾

### 不同等級的引用

在JDK1.2版本以前 對於引用的定義很簡單 有被指到就是有被引用 沒被指到就是沒被引用
這樣遍歷完一次之後 所有在堆的物件只會分兩種 有被引用(非垃圾)跟沒被引用(垃圾) 這樣比較不彈性 如果我們跑完一次遍歷之後可以把所有物件分成更多等級 比如說 完全垃圾 有點垃圾 中等垃圾  等等 好處是你可以在你的內存非常缺的時候再把中等色垃圾刪掉 不需要每次都全部刪光光 由引用強度由強到弱分別是強引用 軟引用 弱引用以及虛引用

1.強引用(Strong reference): 像是


{% highlight java %}
Object obj = new Object();
{% endhighlight %}

這種的用法 只要obj不指到別人 這個內存就不會被回收

2.軟引用(Soft reference): 這是一個特別的類別 你可以宣告你的變數成為軟引用藉由

{% highlight java %}
Object obj = new Object();
SoftReference<Object> wobj = new SoftReference<Object>(obj);
Obj = null;
{% endhighlight %}

這樣子的話 這個new Object()就變成了軟引用 存取這個內存的方式是wobj.get()

軟引用的內存只有在**將要發生**內存溢出的時候 才會被清掉

比較常見的應用是Cache


3.弱引用(Weak reference): 這是一個特別的類別 你可以宣告你的變數成為弱引用藉由

{% highlight java %}
Object obj = new Object();
WeakReference<Object> wobj = new WeakReference<Object>(obj);
Obj = null;
{% endhighlight %}

這樣子的話 這個new Object()就變成了弱引用 弱引用的內存會在下一次的垃圾清除中被清掉 不管內存是否不夠

4.虛引用(Phantom reference): 這是一個特別的類別 你可以宣告你的變數成為虛引用藉由

{% highlight java %}
Object obj = new Object();
PhantomReference<Object> wobj = new PhantomReference<Object>(obj);
Obj = null;
{% endhighlight %}

這樣子的話 這個new Object()就變成了虛引用 當一個內存被一個虛引用給引用時 基本上對於垃圾回收器的判斷跟沒有被引用沒有差別 虛引用的目的是當這個東西被回收的時候 你可以得到一個系統通知 他會丟一個通知到ReferenceQueue

有關於弱引用的進階問題跟ReferenceQueue的用法 都在本部落格裡

[Java中的各種引用](/toc/jvm/jvm_4/)

[Reference Queue](/toc/jvm/jvm_5/)

### Stop the World

介紹完可達性分析法之後 你就知道為什麼垃圾回收的時候需要stop the world 

因為要是我們跑可達性分析到一半 比如說已經遍歷完GC Root1的所有子結點了 這時有新的物件被創建且被GC Root1指到 那這個新創建的物件很快就會被我們不小心清掉 
因為垃圾回收器沒有給他標記 會被認為是dead object

## 什麼時候回收(When)

正確答案是 當JVM覺得應該應該要回收的時候

![Alt text]({{ site.url }}/public/jvm/jvm-7-3.png)

冷靜冷靜 這個問題難回答的原因是因為垃圾收集的這個進程的優先度非常低 而且會隨著內存的使用狀況調整這個進程的優先度 這也是為什麼垃圾收集的時間那麼不確定的原因

除了JVM會自動清垃圾之外 你也可以手動清理垃圾

{% highlight java %}
System.gc();
{% endhighlight %}

上面那行程式會對堆執行一次Major GC

## 怎麼回收(How)

有幾種比較常見的方法 都非常直覺 以下一一說明並舉例

### 標記並清除(Mark-Sweep)

最簡單的方法 標記就是上一節所講的方法 判斷每一個空間是不是垃圾 是垃圾就標記起來 最後再把所有標記起來的記憶題空間清掉 之後就可以給別人用


跑之前是這樣

![Alt text]({{ site.url }}/public/jvm/jvm-7-4.png)

使用標記並清除後 會變這樣

![Alt text]({{ site.url }}/public/jvm/jvm-7-5.png)

簡單易懂 但缺點也很明顯 

第一個是效率問題 標記跟清除都非常耗時 

第二個是空間問題 這個方法會產生太多內存碎片 比如說我現在想要一個連續8單位的空間 你生不出來 雖然你明明有11單位的空間

### 標記並整理(Mark-Compact)

標記步驟做的跟剛剛一樣 標記之後 把所有要用的擠在一起 這樣就沒有內存碎片了

跑之前是這樣

![Alt text]({{ site.url }}/public/jvm/jvm-7-4.png)

使用標記並整理後 會變這樣

![Alt text]({{ site.url }}/public/jvm/jvm-7-6.png)

這兩種簡單的方法 適合用在 **垃圾少** 的情況 要是垃圾佔了總共的記憶體的大多數空間 那麼標記並整理會花很多時間在Compact上 

如果垃圾只有一兩個 清理之後 只需要找最後面的一兩個存活者往剛清出的空間塞 就會快很多

### 標記並複製(Mark-Copy)

於是有個好點子出現了 我們把所有的內存空間分成兩份 一半用完了之後 把存活的移到另外一半 繼續用

跑之前是這樣

![Alt text]({{ site.url }}/public/jvm/jvm-7-7.png)

使用標記並整理後 開始複製到另外一半區域 跑完變這樣

![Alt text]({{ site.url }}/public/jvm/jvm-7-8.png)

這個做法好處是容易實作 而且不會有內存碎片 而且**非常快** 因為我們可以邊mark邊copy 就是你可以在標記的過程中 把已經標記好的直接開始複製 缺點顯而易見 我們就只剩一半的空間可以用 需要三不五時就垃圾回收一下

這個方法則是適合 **垃圾多 存活對象少** 的情況 因為我們是複製存活的到另外一區 所以如果只有一兩個存活 我們就只要複製一兩個到另外一區就可以

聰明的你一定想到了 既然存活對象少 所以並不需要1:1的空間分配

### 分代回收算法

因為兩種不同類型的做法(複製跟標記整理) 的優點不同 一個適合垃圾多 一個適合垃圾少 所以我們又可以來做可愛的撒尿牛丸了

![Alt text]({{ site.url }}/public/jvm/jvm-7-9.jpeg)

我們就把所有內存的空間分成兩個部分 分別用不同的算法


![Alt text]({{ site.url }}/public/jvm/jvm-7-10.png)

在新生代 因為存活對象少 1:1的分兩半是不切實際的 因為這樣新生代的內存只剩一半 所以改成9:1分配

![Alt text]({{ site.url }}/public/jvm/jvm-7-11.png)

但仔細想一想 我們在新大區的記憶體要用完的時候 把存活的複製到新小 然後新大清空 那新大區第二次滿的時候 我們要放去哪？別忘了新小區也需要垃圾回收 這們一來根本沒地方放

所以我們只好再切一塊小區 兩個小區輪流放


![Alt text]({{ site.url }}/public/jvm/jvm-7-12.png)


故事變這樣 第一次新大區記憶體滿了以後 對新大作一次複製算法 把存活的複製到新小1區 然後繼續在新大區分配記憶體 等新大區又快滿的時候 把(新大 + 新小1)做一次複製算法 把存活的複製到新小2區 繼續在新大區分配記憶體 等新大區又快滿的時候 把(新大 + 新小2)做一次複製算法 把存活的複製到新小1區 然後就一直交互蹲跳 

在新生代發生的垃圾回收 我們稱為Minor GC 就是把(新大+新小x) 複製到新小y的過程

然後 只要一個同樣的記憶體空間 重複在新小1跟新小2被搬超過15次 就有資格進入老年代 這個數字可以在執行JAVA的時候給定 `XX:+MaxTenuringThreshold`

![Alt text]({{ site.url }}/public/jvm/jvm-7-13.jpeg)

當老年代快滿的時候 就執行一次Major GC 也就是執行標記並整理算法 因為進到老年區的人 我們合理認為他應該不太會被清除(都是已經經過15次認證的佼佼者) 所以我們可假設存活數量多 垃圾少 那就是我們標記並整理算法的登場時間

故事講完了 有沒有覺得heap的記憶體空間分配非常的直覺 

![Alt text]({{ site.url }}/public/jvm/jvm-7-14.jpeg)

再來就剩下一些細節

1.新大區又稱為伊甸園區(Eden) 新小1稱為Survivor1 新小2稱為Survivor2

2.伊甸區跟Survivor區的比例可以在跑程式的時候自己給定 `-Xx:SurvivorRatio`

3.在Survivor區存活幾次才進老年代的次數也可以自己給定 `-XX:MaxTenuringThreshold`

4.如果有個對象太大根本無法在伊甸區分配空間 那就直接在老年代分配

5.如果有對象太大 無法在Minor GC之後放進Survivor區 那也是直接進老年代

關於所有JVM的Option參數 你都可以在[這裡](https://www.oracle.com/technetwork/java/javase/tech/vmoptions-jsp-140102.html)找到

### 再來點細節好不好呢

![Alt text]({{ site.url }}/public/jvm/jvm-7-15.jpg)

要表現出這篇文章跟網路上其他文章的不同 就必須在細節上下功夫

除了堆裡面的新生代跟老年代之外 還有一個地方也可能會被垃圾收集器清理 那個地方就是鼎鼎大名的 -- 

方法區(Java 8以前被稱為永久代PermGen Java 8之後稱為MetaSpace)


JAVA虛擬機規範裡面 並沒有規範垃圾回收一定要回收方法區 而且方法區裡面的東西大多數都不會是垃圾 唯一有可能是垃圾的只有兩種 一種是不會再用到的常量 另一種是不會再用到的類別

1.廢棄的常量: 這個比較直覺 雖然方法區是線程共享 但運行時常量池中 可能有些常量之後不會再被任何線程用到 那就可以回收 節省空間

2.廢棄的類別: 這個的判斷自然就比較嚴謹 要是不夠嚴謹有可能你之後明明就要用可是卻在方法區找不到人 

一個類要能夠被回收要滿足三個條件

條件一:該類別的所有實例都已經被回收(java堆中再也找不到任何實例)

條件二:加載該類別的類別加載器已經被回收

條件三:該類別的類物件沒有被引用過 沒有人透過反射訪問該類的方法

什麼?? 類別加載器也可以被回收?? 

乍聽之下很驚人 但其實使用者自定義的類別加載器其實就是個一般的變數(存在堆中) 只是這個變數可以加載Class

這個概念對一般人來說很抽象 沒關係 跑一段程式碼你就通透

在看這段程式之前 需要補充一下 當JVM要清除一個物件的時候 會呼叫這個物件的finalize方法 所以你可以在你的finalize方法裡寫一些釋放資源的指令 

{% highlight java %}

public class GarbageCollector {
 public static void main(String... args) throws InterruptedException {
   {
     CustomClassLoader cl;

     for (int i = 0; i < 2; i++) {
       cl = new CustomClassLoader();
       cl = new CustomClassLoader();
       triggerGC();
     }
   }
   triggerGC();
 }

 private static void triggerGC() throws InterruptedException {
   System.out.println("GC start");
   System.gc();
   Thread.sleep(1000);
   System.out.println("GC end");
 }

 private static class CustomClassLoader extends ClassLoader {
   public CustomClassLoader() {
     super();
     System.out.println(this.toString() + " - CL Constructor called.");
   }
   @Override
   protected void finalize() throws Throwable {
     super.finalize();
     System.out.println(this.toString() + " - CL Finalized.");
   }
 }
}

{% endhighlight %}

Output長這樣

![Alt text]({{ site.url }}/public/jvm/jvm-7-16.png)

首先宣告一個類加載器的變數cl

第一個for loop的第一次創建時 在堆中創建一個類加載器物件5e2de80c

第一個for loop的第二次創建時 在堆中創建一個類加載器物件1d44bcfa 第一個物件沒有人指到 也再也沒有人可以指到 

所以第一次垃圾回收時 回收了5e2de80c

第二個for loop的第一次創建時 在堆中創建一個類加載器物件266474c2 第二個物件沒有人指到 也再也沒有人可以指到

第二個for loop的第二次創建時 在堆中創建一個類加載器物件6f94fa3e 第三個物件沒有人指到 也再也沒有人可以指到

所以第二次垃圾回收時 回收了物件1d44bcfa跟物件266474c2

For loop結束後 變數cl不在範圍內了 第四個物件沒有人指到 也再也沒有人可以指到 所以會在最後一次回收時被回收

這段程式一舉兩得  

1.原來自定義的類加載器就像是其他的類一樣 生命週期差不多 回收方式也差不多

2.原來垃圾回收是這麼的直觀這麼的簡單

## 起死回生

現在來把[導讀](/toc/jvm/jvm_3/)的內容做個完整的介紹

剛剛講到 當一個物件要被回收前 需要執行finalize指令 你會發現 即使可達性分析沒有跑到這個物件 也不代表這個物件已經被判死刑 

**你還有最後一次起死回生的機會** 那就是你在finalize這個方法裡證明自己還是有用的 可是這個機會對於任何一個物件都只有一次

上例子 今天有個生還者想逃出鋼鐵墳墓

{% highlight java %}
public class EscapePlan {
 public static EscapePlan survivor;
 @Override
 protected void finalize() throws Throwable {
   super.finalize();
   System.out.println(this.toString() + " - Finalized.");
   survivor = this;
 }

 static void killSurvivor() throws Throwable {
   survivor = null;
   System.gc();
   Thread.sleep(1000);

   if(survivor != null) {
     System.out.println("Escaped!");
   } else {
     System.out.println("Died!");
   }
 }
 public static void main(String[] args) throws Throwable{
   survivor = new EscapePlan();
   killSurvivor();
   killSurvivor();
 }
}
{% endhighlight %}

執行結果如下:

{% highlight txt %}
garbageCollector.EscapePlan@635cb856 - Finalized.
Escaped!
Died!
{% endhighlight %}

為什麼第一次可以逃出 第二次才真的死掉呢 

![Alt text]({{ site.url }}/public/jvm/jvm-7-17.jpeg)

當跑完可達性分析之後 所有沒被標記到的人 都會被丟進處刑台 此時會一個一個詢問 

1.你有沒有finalize方法

2.你的finalize方法有沒有被執行過

只有你有finalize方法 而且還沒有被執行過的時候 就會把你丟到一個緩刑區 等等再來處理你 

執行緩刑區的優先度非常低 要等到其他人都被清理處刑完 才會輪到緩刑區 這就是為什麼我們需要讓程式睡一秒 

等到JVM有空的時候 會給所有在緩刑區的人最後一次起死回生的機會 也就是去執行你的finalize方法 你可以在這時候把你之前分配到的記憶體釋放 或是把像上面的例子 把自己救活

所有緩刑區的都跑完之後 會再跑一次小的可達性分析 來看到底哪些要做最後處決 哪些存活 存活的就不會被回收

當然在finalize裡自救不是個好的寫法 太容易讓人困惑而且性能非常差 這裡舉這個例子是告訴你有這個方法 以及finalize的調用方式

## Java物件的生命週期

讀懂垃圾收集器之後 你對於Java物件的理解已經比別人深了 我來幫你複習一下

一個物件的開始到結束 經歷了許多階段

1.創建階段: 當一個物件被創建 觸發類加載器加載 等到加載器把類物件寫進堆 把類別方法寫進方法區

2.使用階段:當一個物件在堆裡 至少被一個引用給指著 而且那個引用還在作用域內

3.不可見階段:當一個指到那個物件的引用已經超出作用域了(out of scope) 但還是有被某些GC Root指到

4.不可達階段: 已經沒有被任何人指到了 物件孤獨的被遺忘在堆裡

5.收集階段: 被收集到處刑場 如果是第二次到這個階段 或是類別沒有覆蓋finalize方法 則跳過階段6

6.復活一次: finalize救活對象 重回步驟2 

7.宣告死亡: 清出堆的空間給別人用

## 總結

看懂了這篇文章 你已經比大部分的人都還了解垃圾收集器了 恭喜恭喜

