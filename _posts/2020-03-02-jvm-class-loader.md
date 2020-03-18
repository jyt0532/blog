---
layout: post
title: 每個程序員都該瞭解的JVM - 虛擬機類加載機制
comments: True 
subtitle: 類加載機制
tags: jvm
author: jyt0532
---

## 楔子

重頭戲來了 記得我們在[類加載機制導讀](/toc/jvm/jvm_1/)中提出的問題 

{% highlight java %}
class Superclass {
  static { System.out.println("9");}
  Superclass () { System.out.println("10");}
  {
    System.out.println("11");
  }
}
class Subclass extends Superclass {
  static final int var1 = 123;
  static final int var2 = (int) (Math.random() * 10);

  static { System.out.println("6");}
  Subclass () { System.out.println("7");}
  {
    System.out.println("8");
  }
}

public class JVMDemo {
  public static void main(String[] args) throws Exception {
    System.out.println("1");
    System.out.println("2: " + Subclass.var1);
    System.out.println("3");
    System.out.println("4: " + Subclass.var2);
    System.out.println("5");
    new Subclass();
  }
}
{% endhighlight %}

看完這章後 我們可以回答甚至更難的問題 

{% highlight java %}

interface SuperInterface {
 int STATIC_1 = printRandom();
 int STATIC_2 = printRandom();
 static void staticMethod() {
   System.out.println("16");
 }
 static int printRandom() {
   System.out.println((int)(Math.random() * 10));
   return 1;
 }
}


interface SubInterface extends SuperInterface {
 static void staticMethod() {
   System.out.println("17");
 }
}
class Superclass {
 {
   System.out.println("12");
 }
 static {
   System.out.println("13");
 }
 Superclass () {
   System.out.println("14");
 }
 {
   System.out.println("15");
 }
}
class Subclass extends Superclass implements SubInterface {
 static final int var1 = 123;
 static final int var2 = (int) (Math.random() * 10);
 {
   System.out.println("8");
 }
 static {
   System.out.println("9");
 }
 Subclass () {
   System.out.println("10");
 }
 {
   System.out.println("11");
 }
}
class Hello {
 {
   System.out.println("18");
 }
 static {
   System.out.println("19");
 }
 Hello (){
   Superclass superclass = new Subclass();
 }
 {
   System.out.println("20");
 }
}

public class ClassInitialization {
 {
   System.out.println("1");
 }
 static {
   System.out.println("2");
 }
 public static void main(String[] args) throws Exception {
   System.out.println("3");
   System.out.println("4: " + Subclass.var1);
   System.out.println("5");
   System.out.println("6: " + Subclass.var2);
   System.out.println("7");
   new Subclass();
   hello();
   SuperInterface.staticMethod();
 }
 static void hello(){
   new Hello();
 }
}

{% endhighlight %}

讀者可以先拿個紙筆試試 看你的答案跟正確答案一不一樣
輸出如下
{% highlight txt %}
2
3
4: 123
5
13
9
6: 9
7
12
15
14
8
11
10
19
18
20
12
15
14
8
11
10
4
2
16
{% endhighlight %}

要是你不用看這篇文章就直接答對 那你可以直接忽略這篇文章

要是不一樣 沒關係 看完這篇文章你就完全通透 一目了然 這也是這篇文章存在的意義和目的

Ready? Go!

## 我們在哪

![Alt text]({{ site.url }}/public/jvm/jvm-3-1.png)

## 為什麼需要類加載器

[上一篇](/2020/03/01/class-file-structure/)我們學習了一個類文件的規範 包含一個類文件如何存儲一個類別所會擁有的字段 方法等等

虛擬機需要在執行期間 動態的把描述類別的文件加載到虛擬機的內存 之後才可以被程序使用

類加載器的輸入是類文件(Hello.class) 輸出有兩個

第一個是是類物件(Hello Class Object) 存在堆裡

第二個是類別的各種資訊 比如說字段名稱描述符 方法名稱描述符等等 存在方法區

上面兩句話是本書的精華 

比如王維《過香積寺》

> 泉聲咽危石，日色冷青松

詩眼就是 咽 跟 冷 這兩個字把泉聲 危石 日色 青松結合在一起 側寫了香積寺的幽靜

回過頭來看 這本書的書眼就是這兩句

> 第一個是是類物件(Hello Class Object) 存在堆裡
>
> 第二個是類別的各種資訊 比如說字段名稱描述符 方法名稱描述符等等 存在方法區
>

![Alt text]({{ site.url }}/public/jvm/jvm-2-28.png)

好啦不鬧了 我們繼續

從一個類文件變成一個 **類物件+類別資訊** 要經歷五個階段

加載 

驗證

準備

解析

初始化

等等會一一談論這五個步驟

## 類加載的時機

Java 提供了動態加載功能 這句話的意思呢 是指Java不在編譯時期就把可能用到的類別都載好 而是等到需要用到的時候再加載 

就像斯斯有兩種 **用到也有兩種** 一種是主動使用 一種是被動使用

主動使用一個類別 代表需要初始化這個類別 要初始化的話自然要經歷加載的步驟

被動使用一個類別的話 那個類別會被加載 但不會被初始化

這個章節最重要的概念就是 何謂主動使用 何謂被動使用

主動使用一個類別的時機如下 如果有人:

1.用了new 關鍵字 代表新物件被創建

2.static method被呼叫

3.static variable被存取(但compile-time 常數例外)

4.如果此類別的子類被初始化時 此類別還沒被初始化 則會先初始化此類別

5.從命令列被執行時 用戶指定的主類(就是public static void main(String[] args) 所在的類)

6.任何的反射調用

再說一次 只有以上的情況發生時 稱為主動使用一個類別 而主動使用一個類別會觸發類別初始化 而要觸發類別的初始化 當然必須經歷初始化之前的其他步驟(包含加載) 

其他使用 都叫做被動引用

而主動使用一個接口的時機如下 如果你的:

1.static method被呼叫

2.static variable被存取(但compile-time 常數例外)

3.從命令列被執行時 用戶指定的主類(就是public static void main() 所在的類)

4.任何的反射調用

原本的第一點當然拿掉 沒有人在new 接口 
所以主動使用類別跟主動使用接口最主要的區別是第四點 當一個接口的子類被初始化時 並不會強制要初始化接口的父類 只有等真正需要被使用的時候才會初始化父類接口


如果一個類被主動使用 那就會經歷類加載器的所有週期直至初始化 如果是被動使用 那JVM並沒有明確的規範什麼時候要執行類加載週期的第一階段(加載) 你可以先加載 也可以晚一點加載 當你要加載 你可以只先經歷第一個階段(加載) 或是第一加第二個階段(加載和驗證) 

## 類加載週期介紹

再深入研究每一個階段之前 我們先提一下各個階段的大概念 讓你有個完整的架構

*Loading(加載):*

由類加載器完成 當一個類別或是一個介面被存取的時候 類加載器會去搜尋這個類的classpath 找到了之後 去載這個類別的字節流(byte code) 載完後變成類物件(class object)

類物件是instance of java.lang.Class, 類物件會擁有這個class的所有meta information 像是class name, super class name, methods等等

注意: 類加載總共包含了五個階段 Loading(加載)只是類加載的第一個階段 但翻譯起來容易讓人混淆

*Verification(驗證):* 

這個步驟負責查看class的byte stream是不是合法的 而不是個惡意的字串流 會用一個叫做bytecode verifier的東西檢查 怎麼檢查呢? 在上一章節中已經描述了類文件結構 所以只要不符合上一章節講的結構 就驗證不過 比如說MAGIC Number變成CAFE BEBE 那就會驗證不過

也許你會說 當我在把一個java檔編譯成class檔的時候 他就會幫我編譯了不是嗎? 那時候就已經保證不會亂存取不屬於你的記憶體位置了吧？

別忘了編譯過後的class檔是任何人都有可能加以修改的 JVM為了確保不會為了加載一個類而系統崩潰 所以必須在載入的時候仔細檢查這個類的字節流合不合法

**因為我們追求了跨平台執行 所以這就是必要的支出**

這個階段會驗證的東西包含 

類文件格式驗證 

元數據驗證

字節碼驗證

符號引用驗證
 
如果驗證不合法 就會拋出`java.lang.VerifyError`異常或是它的子類異常  
 
*Preparation(準備):*

這步很簡單 就是要先為static variable分配空間 並且給予預設初始值
 
但如果是static final variable 那麼這個時候就不會再給預設值 而是直接賦予給定的final值

*Resolution(解析):*

把類文件中的常量池裡面的符號引用轉化為直接引用的過程

*Initialize(初始化):*

剛剛Preparation我們有初始成預設值 現在我們把不是final的靜態變數賦予真正的初始值

### 類加載週期深入探討之前

在深入探討五個步驟之前 有兩件事情必須先提及

1.驗證+準備+解析 這三個階段又統稱連接(Linking) 然後因為Java是在運行時動態連接 所以又稱Dynamic Linking

2.解析這個步驟也可以放在初始化之後

有個這五個步驟的大概念後 現在我們來一一討論各個步驟吧

## 加載: Loading

### 類加載器

類加載器就是用來動態加載類別的東西 我們在需要使用一個類之前 需要把這個類 從網路或從硬碟中加載到內存裡面 並且讀取byte stream之後生成類物件

除了自定義的類加載器之外 JVM裡面有啟動類加載器,擴展類加載器和應用程序類加載器

在這個階段 有三件事情必須要完成

1.用其中一個加載器 透過一個類別的完全限定名找到.class檔案(字節流)

2.將這個字節流中的類別資訊存進方法區

3.在Heap中 生成一個這個類別的類物件

### 加載的步驟

![Alt text]({{ site.url }}/public/jvm/jvm-3-2.png)

1.類加載器先去看這個類物件是不是已經在heap裡了 

2.如果已經在heap裡了 就從heap回傳class object 

3.沒有在heap的話 代表這個type第一次被存取 那類加載器就會去尋找這個class 

4.如果找不到 就拋出ClassNotFoundException 

5.找到的話 就生成類物件 並且存進heap


### 細談所有的類加載器

啟動類加載器：啟動類加載器負責加載放在 `$JAVA_HOME/lib` 裡面能被JVM識別的類庫(最有名的就是rt.jar 裡面有所有runtime會用到的庫) 還有加載被`-Xbootclasspath`參數指定的路徑的類 

Note: 啟動類加載器無法直接被Java程序引用 意思就是你無法在你的程序中直接取用它

擴展類加載器：擴展類加載器負責加載放在 `$JAVA_HOME/lib/ext/*.jar` 的庫或是系統變量 java.ext.dirs 所指定的目錄中的文件 一個Java程序可以直接引用擴展類加載器

應用程序類加載器：應用程序類加載器負責加載用戶類路徑($classpath)中的文件 一個Java程序可以直接引用應用程序類加載器 如果使用者沒有自定義類加載器的話 應用程序類加載器就是我們預設的類加載器 所以又稱系統類加載器

自定義類加載器: 你只要繼承`java.lang.Classloader`就可以自己定義類加載器 可以用來加載加過密的檔案 或是特殊來源比如數據庫或網路
 
當然 擴展類加載器跟應用程序類加載器和自定義加載器也必須要被啟動類加載器加載

### 父親委派模型(Parent Delegation Model)

(你在其他地方看到的翻譯都會是雙親委任模型 個人覺得這個翻譯會容易讓人誤解 所以在本書一律以父親委派模型代替)

其實這個模型也很簡單 因為我們有很多的類加載器 層次關係如下

![Alt text]({{ site.url }}/public/jvm/jvm-3-3.png)

(Note:類加載器的父子關係並不是以繼承關係來實現 而是以組合(Coposition)關係來重複使用父加載器的程式)

如果一個類加載器收到了一個類加載的請求 它會先請他的上面一層類加載器來加載 只有當你的上面一層跟你說它無法加載(負責的範圍裡面 找不到需要加載的類) 才輪到你加載 

這麼做的理由有二
 
第一個理由: 因為越接近啟動類加載器的加載器越值得信賴 所以同時有很多類加載器都可以加載的話 就由越靠近啟動類加載器的來加載 這某種程度可以加速JVM的運行 因為如果一個Class可以從啟動類加載器來加載的話 那下一步驟的驗證基本上都可以跳過 但如果是由應用程序類加載器或以下的類加載器來加載 那麼驗證階段就要把所有的驗證都跑過 來確保這個不是惡意的Class

第二個理由: 如果一個類 即使他們的路徑一樣檔案都一模一樣 但若是被不同的加載器加載 加載完的類就不一樣 所以如果同一個類 但我們並不是每次都給同一個類加載器加載的話 那同樣名稱的類可能會有不同的行為 所以統一給能加載這個類的最上面那層的加載器加載

講那麼多四書五經 來看個例子吧



{% highlight java %}
public class TestBootstrap {
 public static void main(String[] args) {

   ClassLoader cl = new CustomClassLoader();
   while (cl!=null){
     System.out.println(cl);
     cl = cl.getParent();
   }
 }
 private static class CustomClassLoader extends ClassLoader {}
}

{% endhighlight %}

就這麼簡單 我就已經自己定義了一個類加載器 只要extends ClassLoader就可以

如果你沒有自定義的類加載器 預設就是用應用程序類加載器 然後應用程序類加載器的父親就是擴展類加載器 在往上就變null了 這也是剛剛所說 啟動類加載器無法直接被Java程序引用

輸出如下
{% highlight txt %}
TestBootstrap$CustomClassLoader@511d50c0
sun.misc.Launcher$AppClassLoader@18b4aac2
sun.misc.Launcher$ExtClassLoader@60e53b93
{% endhighlight %}

### 類物件Class Object

現在我們知道類加載器的其中一個輸出結果是類物件  那麼類物件到底是什麼呢 類物件可以用來生成那個類的物件(instance) 不只類別跟介面有類物件 所有的type 都有類物件 包含

Class

Interface

Primitives(byte, short, int, long, float, double, boolean, char)

Void

Arrays 


注意 Array其實代表多數個type 比如說int[]也是一個type, short[] 也是一個type 但如果ClassA裡面有int[100] ClassB裡面有int[50] 算是同一個type, int[] 跟長度無關

一個類物件有一些函數可以call 讓我們可以了解這個類的meta-data 這樣我們就可以在執行期間知道現在傳進來的是什麼類

{% highlight java %}
void printClassName(Object obj) {
  System.out.println("The class of " + obj +" is " + obj.getClass().getName());
}
{% endhighlight %}

而類物件 就是類加載器的輸出 且這個輸出被存放在heap 之後如果需要用的話 不用再重新載一次

之後的反射一章中 會再細談類物件

### 類加載總結

再看一次類加載的流程圖

![Alt text]({{ site.url }}/public/jvm/jvm-3-2.png)

是不是更有感覺了？

再重申一次 兩個輸出 一個是類物件存在堆裡 一個是類別訊息存在方法區

## 驗證: Verification

驗證階段的目的是要確保被加載的類別的正確性

確認加載進來的類是不是符合java的規範 如先前所說 一個類別的二進制字節流可能是從網路上下載下來 非常有可能被任何人任意修改 JVM不能執行任何惡意的類別 這可能會讓自己崩潰 

驗證這一步 並非絕對必要 但一個虛擬機的驗證階段是否嚴謹 決定了這個虛擬機能承受惡意攻擊的程度 而且就性能而言 驗證這一步佔了類加載中相當大的部分

大部分的虛擬機都有下列四大驗證

### 類文件格式驗證

在類文件結構一章中有說明了類文件的格式規範 所以第一個驗證就是驗證一個byte stream是不是合法 一個類文件必須符合不少要求

1.是不是以magic number CAFE BABE開頭

2.版本號是否可以被這台JVM處理

… 等等

讀完上一章的類文件結構 相信你也可以人工驗證一個類文件

這個階段解析完這個類別的byte stream之後 就代表這個類別可以正確地被存放在方法區內 存放在方法區以後 剩下的三個驗證都是驗證存在方法區裡面的資料(如類型訊息 字段訊息等等) 不再操作byte stream

### 元數據驗證(metadata verifier)

比如說

1.final class沒有被繼承

2.final method沒有被覆寫

3.沒有不合法的overloading 比如說signature一樣可是返回值不一樣

4.如果這個類別不是抽象類 則這個類別是否實現了所有接口方法以及父類的抽象方法

如果說類文件驗證是驗證syntax error 那麼元數據驗證就是驗證semantic error

### 字節碼驗證(Byte Code Verifier)

大部分是針對一個類別的方法內容去驗證

1.Bytecode integrity: if condition不會跳到同一函式之外的行數

2.類型轉換沒有違反規則 比如說可以把子對象賦予給父類型 但反過來則不行 或是把對象賦予給毫無繼承關係的類型 都違法

### 符號引用驗證

對於常量池中的符號引用進行驗證

1.符號引用中的全限定名是否能找到對應的類

2.符號引用中的字段 方法等等能否被當前的類別訪問

符號引用驗證的目的是確保解析階段能被順利的進行

### 重要但非必要的步驟

驗證階段 是個非常重要但並非必要的步驟 但就跟所有的網站或系統一樣 絕對是越安全越謹慎越好 純粹是為了安全性考量 但如果你信任所有的你會用到的class 你也可以在你執行的時候給jvm 一個參數 -Xverifiy:none 把大部分的驗證都關掉 加速jvm運行

而且如果這個類別是由啟動類加載器加載的話 可能根本就不需要驗證 因為大家已經都是自己人了 那如果是自定義類加載器加載的類別 就要跑所有的驗證


## 準備: Preparation

為static variable分配堆的空間 並給予預設值 如果方法區沒有足夠空間 則拋出`OutOfMemoryError`

沒有被修飾成final的static變數 都會在這個階段被賦予預設值 比如說:


{% highlight java %}
public static int value = 111;
{% endhighlight %}

此時value會被設為0 在初始化階段才會再設成111

下表是java裡面各個形態的初始值

| Data Type              | Default Value (for fields) |
|------------------------|----------------------------|
| byte                   | 0                          |
| short                  | 0                          |
| int                    | 0                          |
| long                   | 0L                         |
| float                  | 0.0f                       |
| double                 | 0.0d                       |
| char                   | ‘u0000’                    |
| String (or any object) | null                       |
| boolean                | FALSE                      |

但若是被修飾成final的static變數 都會在這個階段被賦予最終值 例如:

{% highlight java %}
public static final int value = 111;
{% endhighlight %}

這時value就會在這個階段就被設定為111


注意:
記不記得我們說過類別加載有很多種trigger的方法 比如說 有人用new關鍵字創建新物件 或是static method被呼叫等等

如果一個類別是因為有實例被創建而被加載 進而進到準備階段的話 那個實例變數也會在這個階段被分配空間並初始化成預設值 而這些賦予預設值的動作 又會發生在所有的靜態變數(static fields)都初始化完後 而初始化靜態變數又是從父類先開始 一路初始化到子類

這是正確回答本篇文章一開始的問題中 最重要的關鍵

## 解析: Resolution

上一章類文件結構中提到 每一個類別都把這個類別所有的符號引用存在常量池中 而且每個被加載完的類別都會有個別的運行時常量池在方法區中 可是我們在類文件結構看到 類文件裡面有許多CONSTANT_Class_info, CONSTANT_Fieldref_Info, CONSTANT_Methodref_Info等等的符號引用 

但這只能在編譯時期 確定每個符號引用都合理 你可以在未來需要的時候 定位的到你的目標 但你在執行期的時候 並不知道你想用某個符號引用的時候該去內存的哪裡存取 因為這個你想定位的目標實際的位置 取決於虛擬機的內存佈局

故事是這麼進行的 加載這個步驟會把類別訊息放在方法區 包含了把類別的常量池放進運行時常量池 但此時運行時常量池裡面的那個索引的值還是個符號引用 **解析就是把這個符號引用取代成直接引用** 來個例子 看一下我們可愛的常量池

這一段程式碼


{% highlight java %}
public class Resolution {
 public void foo() {
   new Person();
 }
}
class Person{}
{% endhighlight %}

編譯過後(javac Resolution.java) 看一下類文件結構(javap -v Resolution)


{% highlight txt %}
Constant pool:
   #1 = Methodref          #5.#13         // java/lang/Object."<init>":()V
   #2 = Class              #14            // Person
   #3 = Methodref          #2.#13         // Person."<init>":()V
   #4 = Class              #15            // Resolution
   #5 = Class              #16            // java/lang/Object
   #6 = Utf8               <init>
   #7 = Utf8               ()V
   #8 = Utf8               Code
   #9 = Utf8               LineNumberTable
  #10 = Utf8               foo
  #11 = Utf8               SourceFile
  #12 = Utf8               Resolution.java
  #13 = NameAndType        #6:#7          // "<init>":()V
  #14 = Utf8               Person
  #15 = Utf8               Resolution
  #16 = Utf8               java/lang/Object

{% endhighlight %}

長得真可愛  再來看一下我們foo方法的byte code

{% highlight txt %}
    public void foo();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=2, args_size=1
         0: new           #2                  // class Person
         3: dup
         4: invokespecial #3                  // Method Person."<init>":()V
         7: astore_1
         8: return
      LineNumberTable:
        line 3: 0
        line 4: 8
{% endhighlight %}

(這註解不是我寫的 javap -v自動會幫你寫好註解)

在foo的Code裡面 invoke了一個方法 什麼方法呢 

{% highlight txt %}
4: invokespecial #3 //自己去看常量池的第3個
{% endhighlight %}

常量池的第3個是什麼呢

{% highlight txt %}
#3 = Methodref          #2.#13         // Person."<init>":()V
{% endhighlight %}

#2跟#13都幫你找好了 總之呢 就是要呼叫Person的constructor

這些 就是符號引用 

之前的驗證階段讓你知道 ok的確有Person這麼個類別 這個方法是合理的等等 處於紙上談兵的階段 

當你上戰場了 當你真的需要Person了 你就需要去解析他 這可能包含了需要加載Person等等 其中還包含了存取權限的確認 看我現在的類別有沒有權限存取Person 
如果沒有的話會拋出`java.lang.IllegalAccessError` 加載完並且初始化完後會有一個Person物件在堆中 
然後呢 **就可以把運行時常量池裡面的符號引用改寫成直接引用**也就是這個person物件在heap裡的內存位址

這也是JAVA的一個重要的特性 **動態連接** 我在第一次使用的時候才去連接

所以解析步驟 並不強制一定要在類加載時期的初始化之前 在類加載時期的初始化之前就解析的叫做Eager Loading 有的虛擬機實現甚至是在類加載之後 一個符號真正要被使用之前 再去解析就來得及了 這叫做Lazy Loading

舉個例子 父子雖然沒有住一起 但是常常一起到處搬家 分別是爸爸F和兒子S 每個人想當然爾都有他自己的名字跟特性 每搬到一個不同到國家 他們會有各自的地址 

每當台灣戶政事務所需要登記兒子S的資料時 比較嚴格 需要明確的知道爸爸住哪裡 才可以登記兒子的戶籍資料 所以這時需要去幫爸爸F登記 並且要知道爸爸住哪裡 登記完爸爸後 再來登記兒子的資料跟住哪裡 以後如果戶政事務所要靠兒子找爸爸就直接看戶政事務所的資料

但美國就比較不一樣 因為DMV效率實在慢 所以在幫兒子S登記時 我只要知道你爸爸的名字 確定你找得到人就好 我並不是一定要知道你爸住哪 那如果真的需要找人的話 就去認真地找一次 找到後登記在兒子的資料裡 這樣下次需要的時候就可以直接找

![Alt text]({{ site.url }}/public/jvm/jvm-3-6.jpeg)

扯了這麼多 其實要完全懂解析這個部分 需要了解oop-klass模型 而我認為這個主題太過艱澀違背了本書的主旨 所以你只要知道這是個把類文件裡面的符號引用轉化為可以直接指到目標在內存的位置的指針 的步驟就可以

## 初始化: Initialization

執行所有static區塊的程式 按照順序執行 比如說


{% highlight java %}

public class Initialization {
 static int i;
 static {
   i = 122;
   System.out.println(i);
 }
 static {
   i = 123;
   System.out.println(i);
 }
}

{% endhighlight %}

會印出
122
123

其實就只是把所有static區塊照順序合併

要注意的是 如果靜態變數是宣告在靜態區塊之後 那就只能賦值 不能存取
比如說


{% highlight java %}
public class Initialization {
 static {
   i = 122;
   System.out.println(i);//illegal forward reference
 }
 static int i;
}
{% endhighlight %}

這個類別無法編譯

但下面這個就可以


{% highlight java %}
public class Initialization {
 static {
   i = 122;
 }
 static int i;
 static {
   System.out.println(i);
 }
}
{% endhighlight %}

## 決鬥

滿口的四書五經 該上戰場了

把本文一開始的程式碼用Intellij跑起來 但這裡我要加一個jvm的option `-verbose:class`

![Alt text]({{ site.url }}/public/jvm/jvm-3-4.png)

加完之後執行 你可以清楚地看到所有類別被加載的順序

在解釋之前 有些小觀念要先講清楚

1.所有的靜態區塊 會照順序被合併在一起(包含靜態變數宣告)

2.所有的Instance Initializer 會照順序被合併在一起

3.Instance Initializer 會被移到constructor之前執行

比如說下例


{% highlight java %}
public class StaticOrder {
   static final int var1 = 123;
   static final int var2 = (int) (Math.random() * 10);
   {
     System.out.println("1");
   }
   static {
     System.out.println("2");
   }
   StaticOrder () {
     System.out.println("3");
   }
   {
     System.out.println("4");
   }
   static {
     System.out.println("5");
   }
   static {
     System.out.println("6");
   }
   static int a(){
     System.out.println("7");
     return 1;
   }
   static final int var3 = a();

 public static void main(String[] args) throws Exception {
   new StaticOrder();
 }
}
{% endhighlight %}

輸出會是

{% highlight txt %}
2
5
6
7
1
4
3
{% endhighlight %}

簡單來說 就是跑完所有static的程式碼(類別初始化) 然後跑完所有Instance Initializer(物件初始化) 然後才跑constructor

回到正題 話不多說 答案揭曉




{% highlight txt %}

[Loaded demo.ClassInitialization from file:/Users/bchiang/code/jyt0532JVM/out/production/jyt0532JVM/]
2
3
4: 123
5
[Loaded demo.SuperInterface from file:/Users/bchiang/code/jyt0532JVM/out/production/jyt0532JVM/]
[Loaded demo.SubInterface from file:/Users/bchiang/code/jyt0532JVM/out/production/jyt0532JVM/]
[Loaded demo.Superclass from file:/Users/bchiang/code/jyt0532JVM/out/production/jyt0532JVM/]
[Loaded demo.Subclass from file:/Users/bchiang/code/jyt0532JVM/out/production/jyt0532JVM/]
13
[Loaded java.lang.Math$RandomNumberGeneratorHolder from /Library/Java/JavaVirtualMachines/jdk1.8.0_72.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.Random from /Library/Java/JavaVirtualMachines/jdk1.8.0_72.jdk/Contents/Home/jre/lib/rt.jar]
9
6: 9
7
12
15
14
8
11
10
[Loaded demo.Hello from file:/Users/bchiang/code/jyt0532JVM/out/production/jyt0532JVM/]
19
18
20
12
15
14
8
11
10
4
2
16

{% endhighlight %}

驗收時間 我們來一一解釋一下

{% highlight txt %}
[Loaded demo.ClassInitialization from file:/Users/bchiang/code/jyt0532JVM/out/production/jyt0532JVM/]
2
{% endhighlight %}

因為我們執行了包含這個類別 也就是包含public static void main(String[] args)的ClassInitialization類 
符合主動使用的第五個時機 所以ClassInitialization被初始化 當然初始化包含了類加載跟執行靜態block

{% highlight txt %}
3
4: 123
{% endhighlight %}

因為Subclass.var1 是compile-time的常數 所以為Subclass不需要加載(看看我們主動使用類別的第三種情況的例外)

{% highlight txt %}
5
[Loaded demo.SuperInterface from file:/Users/bchiang/code/jyt0532JVM/out/production/jyt0532JVM/]
[Loaded demo.SubInterface from file:/Users/bchiang/code/jyt0532JVM/out/production/jyt0532JVM/]
[Loaded demo.Superclass from file:/Users/bchiang/code/jyt0532JVM/out/production/jyt0532JVM/]
[Loaded demo.Subclass from file:/Users/bchiang/code/jyt0532JVM/out/production/jyt0532JVM/]
13
[Loaded java.lang.Math$RandomNumberGeneratorHolder from /Library/Java/JavaVirtualMachines/jdk1.8.0_72.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.Random from /Library/Java/JavaVirtualMachines/jdk1.8.0_72.jdk/Contents/Home/jre/lib/rt.jar]
9
6: 9
{% endhighlight %}

但Subclass.var2 就不一樣了 var2是運行期的變數 符合主動使用的第三個時機 所以我們必須初始化Subclass

初始化的第一步 是加載Subclass 但要加載SubClass之前 必須先加載好SuperClass跟SubInterface 要加載SubInterface之前 又必須先加載好SuperInterface 所以才會造成你看到的加載順序

(注意這個時候SuperClass和SubInterface和SuperInterface都已經加載好了 但都還沒初始化)

加載完SubClass了之後 驗證 -> 準備 -> 解析 -> 初始化

當要初始化SubClass 發現SuperClass沒初始化 符合主動使用的第四個時機 所以我們必須初始化Superclass

SuperClass之前已經加載過了 直接[初始化](#初始化-initialization) 印出13

前置作業完畢 終於SubClass可以開始初始化了 執行`(int) (Math.random() * 10);` 
random是Math類別的靜態函式 符合主動使用的第二個時機 所以加載Random相關的類別 加載完之後 初始化Subclass.var2

最後一個SubClass的靜態區塊 印出9
回到主程式碼 印出靜態變數Subclass.var2


{% highlight txt %}
7
12
15
14
8
11
10
{% endhighlight %}

再來 我們new一個SubClass 符合主動使用的第一個時機 
但因為SubClass已經加載完畢 SubClass的類物件已經在堆裡 
**所以這裡就只會跑SubClass的constructor** 跑SubClass的constructor之前要先跑SuperClass的constructor 跑constructor之前又要先跑Instance Initializer 於是造成了如下的順序

父Instance Initializer -> 父constructor -> 子Instance Initializer -> 子constructor

{% highlight txt %}
[Loaded demo.Hello from file:/Users/bchiang/code/jyt0532JVM/out/production/jyt0532JVM/]
19
18
20
12
15
14
8
11
10

{% endhighlight %}

再來 我們new一個Hello 符合主動使用的第一個時機 於是我們加載Hello 然後初始化(印出19) 然後跑Hello的instance initializer 再來是Hello的建構子 建構子中new了一個SubClass 後面都一樣

父Instance Initializer -> 父constructor -> 子Instance Initializer -> 子constructor


{% highlight txt %}
4
2
16
{% endhighlight %}

最後 我們存取了接口的靜態函式`staticMethod` 符合主動使用接口的第一個時機 需要執行初始化 但因為剛剛已經加載過了 所以直接初始化 跑出了兩個random number就是初始化的證明

要證明一個接口被初始化比較困難 因為接口不能有靜態區塊 所以我只能宣告一個靜態變數去呼叫靜態函式 看起來比較醜 但相信你懂作者的用心良苦

如果沒有最後一行

{% highlight java %}
SuperInterface.staticMethod();
{% endhighlight %}

的話 我們的SuperInterface就只有被加載 沒有被初始化 就像我們的SubInterface一樣

初始化的最後印出16

那要是最後一行改成

{% highlight java %}
SubInterface.staticMethod();
{% endhighlight %}

那就只有SubInterface被初始化 Superinterface不會初始化 這也是類跟接口最大的差別 在初始化類的時候會強迫初始化所有父類 接口則不然


## 意猶未盡

![Alt text]({{ site.url }}/public/jvm/jvm-3-5.jpeg)

我可以隔著螢幕感覺到你有點意猶未盡 最後再來補充一個概念 也是面試常見陷阱題 搞懂這題 就能徹底搞懂什麼是主動使用 什麼是被動引用

{% highlight java %}

class SuperClass {
 static int STATIC_1 = (int) (Math.random() * 3.0);
 static {
   System.out.println("SuperClass was initialized.");
 }
}
class SubClass extends SuperClass {
 static int STATIC_2 = 6 + (int) (Math.random() * 2.0);
 static {
   System.out.println("SubClass was initialized.");
 }
}
class PassiveDemo {
 public static void main(String[] args) {
   int x = SubClass.STATIC_1;
 }
 static {
   System.out.println("PassiveDemo was initialized.");
 }
}

{% endhighlight %}


輸出如下


{% highlight txt %}

[Loaded demo2.PassiveDemo from file:/Users/bchiang/code/jyt0532JVM/out/production/jyt0532JVM/]
PassiveDemo was initialized.
[Loaded demo2.SuperClass from file:/Users/bchiang/code/jyt0532JVM/out/production/jyt0532JVM/]
[Loaded demo2.SubClass from file:/Users/bchiang/code/jyt0532JVM/out/production/jyt0532JVM/]
[Loaded java.lang.Math$RandomNumberGeneratorHolder from /Library/Java/JavaVirtualMachines/jdk1.8.0_72.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.Random from /Library/Java/JavaVirtualMachines/jdk1.8.0_72.jdk/Contents/Home/jre/lib/rt.jar]
SuperClass was initialized.
{% endhighlight %}

哇靠 這什麼巫術 為什麼只有初始化SuperClass呢?

因為這裡我們**利用**了SubClass來存取STATIC_1 在這個情況下 SubClass並沒有符合我們主動使用的條件 SubClass只是來過過水的!


那如果我們直接藉由SuperClass來存取STATIC_1呢



{% highlight java %}
int x = SuperClass.STATIC_1;
{% endhighlight %}

那麼 SubClass連加載都不會加載


## 收工

恭喜恭喜 現在的你幾乎可以回答所有類似的問題 知其然也知其所以然

要是你從頭到尾認真讀完 你會發現市面上找不到講得比這篇文章更清楚的了 為什麼呢?

因為我給了讀者一個**想了解類加載步驟的理由** 也就是本篇文章開頭 楔子中的程式碼

你捫心自問 如果不是因為你想徹底了解輸出的順序 有誰會在乎類加載的步驟中 是先加載還是先初始化呢? 

所以我除了把前因後果在這篇文章說明白之外 還使用了楔子程式碼 讓你不得不對類加載的步驟感興趣

有沒有感受到作者的用心良苦? 有沒有感覺這樣引進門之後 JVM學習的門檻降低了不少呢?

本篇文章還埋了不少坑 比如類文件的厲害之處 以及反射 這都會在未來的章節中提到 建議讀者先把這篇通透之後 
再繼續接下來的學習


