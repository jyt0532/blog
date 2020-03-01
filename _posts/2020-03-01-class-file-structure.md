---
layout: post
title: 每個程序員都該瞭解的JVM - 類文件結構
comments: True 
subtitle: 類文件結構
tags: jvm
author: jyt0532
---


# 類文件結構

## 我們在哪

![Alt text]({{ site.url }}/public/jvm/jvm-2-1.png)

## 介紹

JAVA語言主打”Compile once, run everywhere” 虛擬機的input是類文件(.class檔案) 只要你能夠給我一個符合規範的類文件 我就可以跑結果給你看 不論你的來源是哪一台電腦的哪一個編譯器 你要是夠厲害的話 你也可以自己手寫一個類文件 

這個意思就是說 所有的類文件都有一個共同的規範必須遵守 這一章節就是要帶你深入解析類文件的結構

這一張屬於理解JVM路上的必經之路 屬於練馬步的部分 

事不宜遲 直接來個最簡單的


{% highlight java %}
import java.io.Serialization;

public class DemoClass implements Serializable {
 int x  = 1;
 void test(){}
}
{% endhighlight %}

寫好這個類別之後 存在檔案`DemoClass.java` 然後把它編譯一下

> javac DemoClass.java

就會生出可愛的`DemoClass.class`檔案 用一個文字編輯器去打開它 會看到

![Alt text]({{ site.url }}/public/jvm/jvm-2-2.png)

這個byte stream就是虛擬機的輸入 別懷疑 現在看覺得這是天書 但看完這個章節之後 你就會覺得byte stream非常赤裸

Are you ready?

## 架構

一個類文件分成十五個部分 我已經幫你切好了

![Alt text]({{ site.url }}/public/jvm/jvm-2-3.png)

1.Magic Number 魔數

2.Version 版本號

3.Constant Pool Count 常量池數量

4.Constant Pool 常量池

5.Access Flags 訪問標誌

6.Class Name 類別名稱

7.Super Class Name 父類別名稱

8.Interfaces Count 接口的數量

9.Interfaces 接口

10.Fields Count 字段的數量

11.Fields 字段

12.Methods Count 方法的數量

13.Methods 方法

14.Attributes Count 屬性的數量

15.Attributes 屬性


### 1.Magic Number 魔數

清楚地看到前四個字節就是Cafe Babe 這樣任何人只要看到前四個字節 就知道這是一個類文件

JAVA之父James Gosling決定要用這個詞的原因大概跟JAVA的標誌大有關係

![Alt text]({{ site.url }}/public/jvm/jvm-2-4.png)

是不是更加好記了呢

Q:那為什麼是Babe不是Baby 

A:拜託來點有深度的問題 因為十六進位的英文字母只有ABCDEF

Q:那為什麼需要浪費4個byte讓大家知道是類文件 從檔案的副檔名(.class)就可以知道了不是嗎 

A:因為副檔名可以隨意修改

### 2.Version 版本號

前面的兩個byte是次版本號 後面的兩個byte是主版本號 十六進位的34就是十進位的52 52對應的是JAVA SE8

以下是版本號的配對

| 45 | JDK1.1 | 
| 46 | JDK1.2 | 
| 47 | JDK1.3 | 
| 48 | JDK1.4 | 
| 49 | Java SE 5.0 | 
| 50 | Java SE 6.0 | 
| 51 | Java SE 7.0 | 
| 52 | Java SE 8.0 | 
| 53 | Java SE 9.0 | 
| 54 | Java SE 10.0 | 
| 55 | Java SE 11.0 | 
| 56 | Java SE 12.0 | 
| 57 | Java SE 13.0 | 

### 3.Constant Pool Count 常量池數量

00 14

十六進位的14代表十進位的20 代表說接下來的常量有19個 照順序排列(因為之後會需要用索引值來找常量) 第0個值先做保留 表示不引用任何常量 

兩個byte 代表一個類的常量池數量最多65536個常量

### 4.Constant Pool 常量池

重頭戲來了 常量池指的是一個類文件中的資源倉庫 你可以想成是一個數組 記錄著很多之後會用到的東西 包含了字面量 跟 符號引用

字面量就像字串 整數 小數等等的常數 

符號引用包含類型的完全限定名 接口的完全限定名 或是字段型態 字段描述符 方法型態 方法描述符等等

很多書上講到常量池都很輕鬆的帶過 但其實常量池的目的很簡單 **就是節省空間**

既然常量池存了那麼多種類的東西 每個東西的長度都不同 這時後就要來好好規範一下

常量池裡面所有可能包含的項目如下

| 類型                               | 項目                  | 類型 | 描述                              |
|----------------------------------|---------------------|----|---------------------------------|
| CONSTANT_Utf8_info<br>(UTF8的字串)  | tag<br>length<br>bytes                 | u1<br>u2<br>u1 | tag值為1<br>UTF8的字串長度<br>UTF8的字串|
| CONSTANT_Integer_info<br>(整數)     | tag<br>bytes | u1<br>u4 | tag值為3<br> 整數值 |
| CONSTANT_Float_info<br>(浮點數)  | tag<br>bytes                 | u1<br>u4 | tag值為4<br>浮點數值|
| CONSTANT_Long_info<br>(長整數)  | tag<br>bytes                 | u1<br>u8 | tag值為5<br>長整數值|
| CONSTANT_Double_info<br>(雙精度浮點數)  | tag<br>bytes                 | u1<br>u8 | tag值為6<br>雙精度浮點數值|
| CONSTANT_Class_info<br>(類或接口 符號引用)  | tag<br>name_index                 | u1<br>u2 | tag值為7<br>名稱存在哪個索引|
| CONSTANT_String_info<br>(字串 字面量)  | tag<br>string_index                 | u1<br>u2 | tag值為8<br>字串存在哪個索引|
| CONSTANT_Fieldref_info<br>(字段 符號引用)  | tag<br>class_index<br>name_and_type_index  | u1<br>u2<br>u2 | tag值為9<br>CONSTANT_Class_info存在哪個索引<br>CONSTANT_NameAndType_info存在哪個索引|
| CONSTANT_Methodref_info<br>(類的方法 符號引用)  | tag<br>class_index<br>name_and_type_index | u1<br>u2<br>u2 | tag值為10<br>CONSTANT_Class_info存在哪個索引<br>CONSTANT_NameAndType_info存在哪個索引|
| CONSTANT_InterfaceMethodref_info<br>(接口中方法 符號引用)  | tag<br>class_index<br>name_and_type_index | u1<br>u2<br>u2 | tag值為11<br>CONSTANT_Class_info存在哪個索引<br>CONSTANT_NameAndType_info存在哪個索引|
| CONSTANT_NameAndType_info<br>(字段或方法的名稱跟型態 符號引用)  | tag<br>name_index<br>descriptor_index | u1<br>u2<br>u2 | tag值為12<br>指向名稱的字串<br>指向型態的字串|


事實上 在Java 7之後 又增加了三個常量結構 但你可以先不用理它 因為那些非常少會用到 把時間花在更有意義的事情上

這個表不需要背他 但下面解析常量池的每一步希望你能跟我一起走一遍 
走完一遍之後 你就知道怎麼用上面的表 而且這步驟之後類加載器也會走 會對你之後篇章的理解有幫助 

這個表的使用方式也很簡單 首先u1指的就是一個byte u2指的就是兩個byte 依此類推 

第一個byte說明這個常量的類型 確定類型之後查表 表中就會告訴你這個類型需要用幾個byte來描述其相對應的資訊

現在就來各個擊破這個可愛的常量池

![Alt text]({{ site.url }}/public/jvm/jvm-2-5.png)

索引1:

0A: 對應到上表的CONSTANT_Methodref_info 我們知道了常量池的第一個索引是MethodRef 再由上表得知MethodRef需要兩個u2

00 04: 對應到索引4 也就是java/lang/Object

00 0F: 對應到索引15 也就是 "<init>":()V,  <init>指的就是constructor, ()V 指的就是不吃參數 回傳void

索引1講的是這個類別的建構函示

![Alt text]({{ site.url }}/public/jvm/jvm-2-6.png)

索引2:

09: 對應到上表的CONSTANT_Fieldref_info 我們知道了常量池的第二個索引是Fieldref 再由上表得知Fieldref 之後有兩個u2

00 03: 對應到索引3 也就是DemoClass

00 10: 對應到索引16 也就是 x&I, x是字段名稱, I 是int

索引2講的是這個類別有一個字段 叫做x 型態是整數


![Alt text]({{ site.url }}/public/jvm/jvm-2-7.png)

索引3:

07: 對應到上表的CONSTANT_Class_info 我們知道了常量池的第三個索引是Class 再由上表得知 之後有一個u2

00 01: 對應到索引17 也就是DemoClass這個字串

索引3講的是有一個類別 叫做DemoClass

![Alt text]({{ site.url }}/public/jvm/jvm-2-8.png)

索引4:

07: 對應到上表的CONSTANT_Class_info 我們知道了常量池的第四個索引是Class 再由上表得知之後有一個u2

00 12: 對應到索引18 也就是java/lang/Object這個字串

索引4講的是有一個類別 叫做java/lang/Object

![Alt text]({{ site.url }}/public/jvm/jvm-2-9.png)

索引5:

07: 對應到上表的CONSTANT_Class_info 我們知道了常量池的第五個索引是Class 再由上表得知之後有一個u2

00 13: 對應到索引19 也就是java/io/Serializable這個字串

索引5講的是有一個類別 叫做java/io/Serializable

![Alt text]({{ site.url }}/public/jvm/jvm-2-10.png)

索引6:

01: 對應到上表的CONSTANT_Utf8_info 我們知道了常量池的第六個索引是字串 再由上表得知之後有一個u2 若干個u1

00 01: 長度為1

78: ascii 編碼後(見附錄)轉換為x

索引6講的是有一個字串 叫做x

![Alt text]({{ site.url }}/public/jvm/jvm-2-11.png)

索引7:

01: 對應到上表的CONSTANT_Utf8_info 我們知道了常量池的第七個索引是字串 再由上表得知之後有一個u2 若干個u1

00 01: 長度為1

49: ascii 編碼後轉換為 I

索引7講的是有一個字串 叫做I

![Alt text]({{ site.url }}/public/jvm/jvm-2-12.png)

索引8:

01: 對應到上表的CONSTANT_Utf8_info 我們知道了常量池的第八個索引是字串 再由上表得知之後有一個u2 若干個u1

00 06: 長度為6

3C 69 6E 69 74 3E: ascii 編碼後轉換為`<init>`

索引8講的是有一個字串 叫做`<init>`

![Alt text]({{ site.url }}/public/jvm/jvm-2-13.png)

索引9:

01: 對應到上表的CONSTANT_Utf8_info 我們知道了常量池的第九個索引是字串 再由上表得知之後有一個u2 若干個u1

00 03: 長度為3

28 29 56: ascii 編碼後轉換為()V

索引9講的是有一個字串 叫做()V

![Alt text]({{ site.url }}/public/jvm/jvm-2-14.png)

索引10:

01: 對應到上表的CONSTANT_Utf8_info 我們知道了常量池的第十個索引是字串 再由上表得知之後有一個u2 若干個u1

00 04: 長度為4

43 6F 64 65: ascii 編碼後轉換為Code

索引10講的是有一個字串 叫做Code


![Alt text]({{ site.url }}/public/jvm/jvm-2-15.png)

索引11:

01: 對應到上表的CONSTANT_Utf8_info 我們知道了常量池的第十一個索引是字串 再由上表得知之後有一個u2 若干個u1

00 0F: 長度為15

4C 69 6E 65 4E 75 6D 62 65 72 54 61 62 6C 65: ascii 編碼後轉換為LineNumberTable

索引11講的是有一個字串 叫做LineNumberTable


![Alt text]({{ site.url }}/public/jvm/jvm-2-16.png)

索引12:

01: 對應到上表的CONSTANT_Utf8_info 我們知道了常量池的第十二個索引是字串 再由上表得知之後有一個u2 若干個u1

00 04: 長度為4

74 65 73 74: ascii 編碼後轉換為test

索引12講的是有一個字串 叫做test


![Alt text]({{ site.url }}/public/jvm/jvm-2-17.png)

索引13:

01: 對應到上表的CONSTANT_Utf8_info 我們知道了常量池的第十三個索引是字串 再由上表得知之後有一個u2 若干個u1

00 0A: 長度為10

53 6F 75 72 63 65 46 69 6C 65: ascii 編碼後轉換為SourceFile

索引13講的是有一個字串 叫做SourceFile



![Alt text]({{ site.url }}/public/jvm/jvm-2-18.png)

索引14:


01: 對應到上表的CONSTANT_Utf8_info 我們知道了常量池的第十四個索引是字串 再由上表得知之後有一個u2 若干個u1

00 0E: 長度為14

44 65 6D 6F 43 6C 61 73 73 2E 6A 61 76 61: ascii 編碼後轉換為DemoClass.java

索引14講的是有一個字串 叫做DemoClass.java

![Alt text]({{ site.url }}/public/jvm/jvm-2-19.png)

索引15:

0C: 對應到上表的CONSTANT_NameAndType_info 我們知道了常量池的第十五個索引是NameAndType 再由上表得知之後有兩個u2 

00 08: 名稱指到常量池索引8 `<init>`

00 09: 名稱指到常量池索引9 ()V

索引15講的是有一個名稱跟型態 叫做`<init>` and ()V

而這也在索引1的時候被引用


![Alt text]({{ site.url }}/public/jvm/jvm-2-20.png)

索引16:

0C: 對應到上表的CONSTANT_NameAndType_info 我們知道了常量池的第十六個索引是NameAndType 再由上表得知之後有兩個u2 

00 06: 名稱指到常量池索引6 x

00 07: 名稱指到常量池索引7 I 也就是Integer

索引16講的是有一個名稱跟型態 叫做x and I

這也在索引2的時候被引用

![Alt text]({{ site.url }}/public/jvm/jvm-2-21.png)

索引17:

01: 對應到上表的CONSTANT_Utf8_info 我們知道了常量池的第十七個索引是字串 再由上表得知之後有一個u2 若干個u1

00 09: 長度為9

44 65 6D 6F 43 6C 61 73 73: ascii 編碼後轉換為DemoClass

索引17講的是有一個字串 叫做DemoClass



![Alt text]({{ site.url }}/public/jvm/jvm-2-22.png)

索引18:

01: 對應到上表的CONSTANT_Utf8_info 我們知道了常量池的第十八個索引是字串 再由上表得知之後有一個u2 若干個u1

00 10: 長度為16

6A 61 76 61 2F 6C 61 6E 67 2F 4F 62 6A 65 63 74: ascii 編碼後轉換為java/lang/Object

索引18講的是有一個字串 叫做java/lang/Object

![Alt text]({{ site.url }}/public/jvm/jvm-2-23.png)

索引19:

01: 對應到上表的CONSTANT_Utf8_info 我們知道了常量池的第十九個索引是字串 再由上表得知之後有一個u2 若干個u1

00 14: 長度為20

6A 61 76 61 2F 69 6F 2F 53 65 72 69 61 6C 69 7A 61 62 6C 65: ascii 編碼後轉換為java/io/Serializable

索引19講的是有一個字串 叫做java/io/Serializable

恭喜你 你已經人工解析完了所有的常量 你應該感受到了 再複習一次 常量池的目的就是節省空間 這樣同樣的字串只需要存在byte stream一次 有需要用相同字串的時候只要給我index 的number就可以

### 5.Access Flag 訪問標誌

之後的兩個字節 代表著這個類別的訪問標誌 訪問標誌的計算是利用bitmask 一個類文件的訪問標誌的字節 是由所有的標誌的值去or運算所得

先列出所有的標誌的值

| ACC_PUBLIC     | 0x0001 | 是否為public                                                                                                        |
| ACC_FINAL      | 0x0010 | 是否是final類別 |
| ACC_SUPER      | 0x0020 | 此類別是否包含了invokespecial指令(所有Java Class都有) |
| ACC_INTERFACE  | 0x0200 | 是否為接口 |
| ACC_ABSTRACT   | 0x0400 | 是否為抽象類別或接口 |
| ACC_SYNTHETIC  | 0x1000 | 這個類並非由用戶產生 |
| ACC_ANNOTATION | 0x2000 | 這是一個Annotation |
| ACC_ENUM       | 0x4000 | 這是一個枚舉 |

其實目前只定義了8個訪問標誌 其實只要用一個byte就夠了 但為了未來擴充方便 預留兩個字節 

所以在這裡 就是ACC_SUPER跟ACC_PUBLIC取or

`0000000000100000 | 0000000000000001 = 0000000000100001 = 0x0021`

### 6.Class Name 類別名稱

00 03 意思就是要你去看常量池的索引3 也就是DemoClass

### 7.Super Class Name 父類別名稱

00 04 意思就是要你去看常量池的索引4 也就是java/lang/Object

### 8.Interfaces Count 接口的數量

00 01 代表這個類別有一個接口

兩個byte 代表一個類的接口數量最多65536個

### 9.Interfaces 接口

00 05 意思就是要你去看常量池的索引5 也就是java/io/Serializable

### 10.Fields Count 字段的數量

00 01 代表這個類別有一個字段

兩個byte 代表一個類的字段數量最多65536個

### 11.Fields 字段

接下來會依序列出每個字段的資訊 每個字段會有如下資訊

| access_flags     | 2 bytes | 訪問標誌                                                                                                       |
| name_index      | 2 bytes | 名稱存在哪個索引 |
| descriptor_index      | 2 bytes | 指向型態的字串 |
| attributes_count  | 2 bytes | 有幾個attribute |
| attributes   | Attributes_count * x bytes | |

注意這裡的x並不固定 因為每一個attribute的長度是不固定的)

先看access_flags 字段的訪問標誌跟類別的很像 都是利用bitmask

以下是所有的標誌的值

| ACC_PUBLIC    | 0x0001 | 字段是否為public    |
| ACC_PRIVATE   | 0x0002 | 字段是否為private   |
| ACC_PROTECTED | 0x0004 | 字段是否為protected |
| ACC_STATIC    | 0x0008 | 字段是否static     |
| ACC_FINAL     | 0x0010 | 字段是否final      |
| ACC_VOLATILE  | 0x0040 | 字段是否volatile   |
| ACC_TRANSIENT | 0x0080 | 字段是否transient  |
| ACC_SYNTHETIC | 0x1000 | 字段是否由編譯器產生     |
| ACC_ENUM      | 0x4000 | 字段是否是一個枚舉      |

在這個範例裡 access_flag是00 00 

但如果變數x是

{% highlight java %}
private final static int x  = 1;
{% endhighlight %}

那access_flag就是

`0x0002 | 0x0010 | 0x0008 = 0x001A`

Name_index就是這個字段名稱在常量池裡面的索引 0006 對應到的是x

Descriptor_index 就是字段的型態在常量池裡面的索引 0007對應到的是I

Descriptor_index在常量池裡的符號如下

| B | Primitive type: byte           |
| C | Primitive type: char           |
| D | Primitive type: double         |
| F | Primitive type: float          |
| I | Primitive type: int            |
| J | Primitive type: long           |
| S | Primitive type: short          |
| Z | Primitive type: boolean        |
| V | 特殊型態: void                     |
| L | 任何物件(第一個字母是L 接著是物件的全域名稱 以分號結尾) |

在這個例子 x變數的型態是int 所以是I
如果x的型態是Object 那麼在常量池裡面對應到的值就是Ljava/lang/Object;
如果x的型態String 那麼在常量池裡面對應到的值就是Ljava/lang/String;

那如果型態是數組 那麼每個維度都會多一個 [ 來表示 

比如說

{% highlight java %}
String[][] x  = {};
{% endhighlight %}

那麼在常量池裡面對應到的值就會是[[Ljava/lang/String; 

接下來還有attribute_count和attributes 可以在這個地方幫字段加上一些屬性 這個例子x就是個int 如果這裡的x是個

{% highlight java %}
final static int x  = 5;
{% endhighlight %}

那x這個字段就還會有一個屬性(attribute_count = 00 01) 這個屬性會是Constant Value


### 12.Methods Count 方法的數量

00 02 代表說這個類別有兩個方法 除了test方法之外 另外一個是構造函數

### 13.Methods 方法

在類文件裡面 描述字段跟描述方法幾乎一樣 會依序列出每個方法的資訊 每個方法會有如下資訊

| access_flags     | 2 bytes | 訪問標誌 |
| name_index      | 2 bytes | 名稱存在哪個索引 |
| descriptor_index      | 2 bytes | 指向型態的字串 |
| attributes_count  | 2 bytes | 有幾個attribute |
| attributes   | Attributes_count * x bytes | |

(注意這裡的x並不固定 因為每一個attribute的長度是不固定的)

先看access_flags 00 01 代表是public

以下是所有的標誌的值

| ACC_PUBLIC       | 0x0001 | 方法是否為public           |
| ACC_PRIVATE      | 0x0002 | 方法是否為private          |
| ACC_PROTECTED    | 0x0004 | 方法是否為protected        |
| ACC_STATIC       | 0x0008 | 方法是否static            |
| ACC_FINAL        | 0x0010 | 方法是否final             |
| ACC_SYNCHRONIZED | 0x0020 | 方法是否synchronized      |
| ACC_BRIDGE       | 0x0040 | 方法是否由編譯器產生的橋接方法(見附錄1) |
| ACC_VARARGS      | 0x0080 | 方法是否接受不定參數            |
| ACC_NATIVE       | 0x0100 | 方法是否native            |
| ACC_ABSTRACT     | 0x0400 | 方法是否abstract          |
| ACC_STRICTFP     | 0x0800 | 方法是否strictfp(見附錄1)    |
| ACC_SYNTHETIC    | 0x1000 | 方法是否由編譯器自動產生          |

Name_index 是#8 對照常量池第八個是 `<init>`

這個其實就是建構函數 每個類別的建構函數都是叫這個名字 這個函式只有虛擬機可以呼叫 你不能在你的JAVA程式裡這麼寫

{% highlight java %}
Object.class.getDeclaredMethod("<init>");
{% endhighlight %}

你會得到`java.lang.NoSuchMethodException`


Descriptor_index 是#9 對照常量池第九個是 ()V 

這裡要來好好講一下一個函式的Signature是怎麼存儲在常量池裡 一個變數在常量池的符號跟字段的一樣 所以遵循著這個規則

(參數型態1參數型態2參數型態3...)回傳型態

比如說如果test的signature是這樣

{% highlight java %}    
boolean[][] test(int a, double b, String[][] c, int y) {
 return null;
}
{% endhighlight %} 

那麼在常量池裡面對應到的值就會是`(ID[[Ljava/lang/String;I)[[Z`

所以回到我們的類別 `()V`就是不帶參數 回傳void

第一個方法說完了 再來是第二個方法

access_flags 00 00

Name_index 00 0C(#12) 對照常量池第12個是 test

Descriptor_index 是#9 對照常量池第九個是 ()V 

Attributes_count 是00 01  有一個attribute

Attribute有一個 是Code屬性 會在接下來的屬性章節一併講


### 14.Attributes Count 屬性的數量

代表這個類別有幾個屬性


### 15.Attributes 屬性

這裡要把之前欠很久的屬性交代一下 我們在每個字段的最後 每個方法的最後 跟每個類別的最後 都會看到可以添加屬性來描述這個主角 


以下是幾個常見的屬性

| 屬性名稱               | 描述對象      | 解釋                    |
| Code               | 方法        | Byte code instruction |
| ConstantValue      | 字段        | final描述符所描述的字段        |
| Exceptions         | 方法        | 方法拋出的異常               |
| SourceFile         | 類文件       | 生成這個類文件的源碼文件名稱        |
| LineNumberTable    | Code屬性    | 程式碼的行號和bytecode的對應    |
| Deprecated         | 類 方法表 字段表 | 被聲明為deprecated的東西     |
| InnerClasses       | 類文件       | 內部類列表                 |
| LocalVariableTable | Code屬性    | 方法的局部變量描述             |

#### Code屬性

我們挑最常見的Code屬性來細講:

| attribute_name_index   | u2             | 指向CONSTANT_Utf8_Info的索引                                                         |
| attribute_length       | u4             | 描述屬性值的長度 = 總長度 - 6bytes(其中attribute_name_index佔2bytes attribute_length佔4 bytes) |
| max_stack              | u2             | Operand stacks的最大值                                                              |
| max_locals             | u2             | 局部變量表需要的存儲空間                                                                    |
| code length            | u4             | 儲存byte code instruction                                                         |
| code                   | u1*code length |                                                                                 |
| exception table length | u2             | 儲存如何處理各個例外 包含處理例外的                                                              |
| exception table        | exception_info | instruction跟catch的例外的型態等等資訊                                                     |
| Attribute count        | u2             | Code有多少attribute                                                                |
| attributes             | attribute_info | 描述Code的屬性                                                                       |

我們實際來看一下 剛剛在Method章節的時候 我們跳過了屬性沒講 現在拿它來當個例子

![Alt text]({{ site.url }}/public/jvm/jvm-2-24.png)

照著上表分析:

Attribute_name_index: 0A: 對應常量池的第十個索引 Code

Attribute_length: 26 = 38

接下來的38個byte描述這個attribute

max_stack: 02

Max_locals:01

code length: 1A = 10 接下來的10個描述這個code

Code(見附錄 [虛擬機字節碼指令表](/2020/02/29/jvm-byte-instruction/)):
	
2A: aload_0: 把本地變量推到棧頂
	
B7: invokespecial 調用超類構造方法
	
00 01: 要被調用的是哪個超類哪個方法 java/lang/Object."<init>":()V

2A: aload_0: 把本地變量推到棧頂

04: iconst_1: assign成1

B5: 為指定的類的實例域賦值

00 02: 要被賦值的是哪個類哪個實例域 DemoClass.x

B1: return


如果你學過組合語言 這其實也很好理解

我了解只看一個建構方法太沒有感覺了 那我們再來看一個簡單的方法



{% highlight java %}
public class Add {
 int x  = 1;
 void add(){
   x = x +1;
 }
}
{% endhighlight %}

跑出來的Code長這樣

![Alt text]({{ site.url }}/public/jvm/jvm-2-25.png)

這樣看懂bytecode怎麼玩了吧

建議讀者可以玩玩看 你自己跑過兩三個byte code後你會更有感覺

#### ConstantValue屬性

雖然在虛擬機規範中並沒有說一定要用final修飾 但對於當今運行的JVM來說 只有一個字段被宣告為static final 才會有這個屬性

#### Exceptions屬性

列出可能拋出的受檢異常 也就是throws後面的類型

#### SourceFile屬性

生成這個類文件的源碼文件名稱 在這個範例裡 就是DemoClass.java

#### LineNumberTable屬性

程式碼的行號和bytecode的對應

#### Deprecated屬性

如果某個方法或或類或字段已經讓作者標記為不建議使用 那就會有這個屬性

#### InnerClasses屬性

如果這個類別是內部類 就會有這個屬性

#### LocalVariableTable屬性

方法的局部變量和源碼的變量之間的對應關係描述

還有其他更多的屬性 但每個屬性都有他的屬性結構和相對應的意思 這本書就不再多談 有興趣的可以自行研究


## 分析類文件結構的命令

如果你有一個.class檔案 你可以跑一個指令直接看到byte code

> javap -verbose DemoClass

你會看到以下輸出

![Alt text]({{ site.url }}/public/jvm/jvm-2-26.png)

一個指令就全部幫你解析完畢 我們剛剛解析的老半天的東西 全部都有 就連常量池都一目瞭然

![Alt text]({{ site.url }}/public/jvm/jvm-2-27.png)

不好意思 剛剛真是辛苦你了 希望你不要覺得很無聊很浪費時間 這些努力是值得的 你要相信你現在為了學習所做的每一件事 會在未來串在一起 


這個命令把一個類別的所有資訊全部列出來 所以對於任何一台JVM來說 只要你給他一個合乎規範的類文件 他都可以幫你跑出你預期的結果 

## 總結

看完本章之後 給你一個類文件 你也能夠自己解析這個類文件是不是合法 並且可以知道這個類文件的所有資訊 而且了解類文件結構 對於未來了解JVM有著舉足輕重的關係

