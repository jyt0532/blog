---
layout: post
title: 每個程序員都該瞭解的JVM - 反射
comments: True 
subtitle: 反射
tags: jvm
author: jyt0532
---


## 介紹

> 反射 指的是在程序運行期間動態的去操作類物件的原數據(meta-data) 包括類別名稱 方法名稱等等 

讀書要會抓重點 上面那句話的重點是 **動態** 知道重點不稀奇 解釋得好懂才難 待會我就來解釋解釋為什麼需要動態的去操作一個類別

反射的靈魂是類物件 要搞懂反射之前 要先問自己四個問題

How What Why Where

### How: 如何得到類物件

使用反射有一個前提 就是你的類物件需要已經被類加載器創建好並且存在heap裡 

大前提成立的情況下 有四種方法可以使用得到類物件

1.由物件的引用得到 `objectReference.getClass()`

直接上代碼


{% highlight java %}
Class clazz = "abc".getClass();
System.out.println(clazz.getName()); // java.lang.String

int[] intArray = {1, 2, 3};
clazz = intArray.getClass();
System.out.println(clazz.getName()); // [I
{% endhighlight %}

在Java裡面表達一個class的名稱的方式跟存在常量池的方法一樣 `[`代表陣列 `I`代表int (請參閱[類文件結構](/2020/03/01/class-file-structure/))


2.`Class.forName(classname)`

{% highlight java %}

try {
 Class clazz = Class.forName("java.lang.String");
 System.out.println(clazz.getName()); // java.lang.String
} catch(ClassNotFoundException e) {
 System.out.println("Can't find class");
}
{% endhighlight %}

注意要丟進forName方法的參數要是類別的完全限定名 
第二種方式不能對primitive type使用(int, byte, long...) 但可以對Array of primitive使用 用法如下


{% highlight java %}

try {
 Class clazz = Class.forName("java.lang.String");
 System.out.println(clazz.getName()); // java.lang.String
 clazz = Class.forName("[Ljava.lang.String;");
 System.out.println(clazz.getName()); // [Ljava.lang.String;
 clazz = Class.forName("[D");
 System.out.println(clazz.getName()); // [D
 clazz = Class.forName("[I");
 System.out.println(clazz.getName()); // [I
} catch(ClassNotFoundException e) {
 System.out.println("Can't find class");
}

{% endhighlight %}

3.`type.class`

{% highlight java %}
Class clazz = int.class;
System.out.println(clazz.getName()); // int
clazz = String.class;
System.out.println(clazz.getName()); // java.lang.String
clazz = double[][].class;
System.out.println(clazz.getName()); // [[D
clazz = void.class;
System.out.println(clazz.getName()); //void
{% endhighlight %}

天阿 void.class 是什麼鬼東西? 別懷疑 void也是primitive type的一種 當然可以操作類物件 待會再來講用途

4.`BP.TYPE`
第四種方法只有Boxed Primitive可以用 Boxed Primitive就是把一個primitive包裝成物件 比如說Integer類別或是Long類別 對於這種類別直接call TYPE就可以操作類物件

LONG.TYPE = long.class




### What: 怎麼使用反射

拿到clazz了 那我們可以怎麼用它呢?

{% highlight java %}
String getName()
Class<? super T> getSuperclass()
boolean isInterface()
boolean isPrimitive()
T newInstance()
ClassLoader getClassLoader()
{% endhighlight %}

這些只是最基本的 讓我們可以知道一個類別的mata-data 我們還可以直接把類別的方法當作一個變數來呼叫

直上例子 最快了解 先來隨便來一個類別Person

{% highlight java %}
public class Person {
 private int id;
 private String name;
 private String address;

 Person() {
 }

 Person(int id, String name, String address) {
   this.id = id;
   this.name = name;
   this.address = address;
 }

 public boolean func1() {
   return true;
 }

 static final protected void func2(int arg1, long arg2, String arg3) {
   System.out.println(arg3);
 }
}
{% endhighlight %}

主程式如下


{% highlight java %}
public class Reflection {
 public static void main(String[] args) throws Exception {
   System.out.println("Before loading Person class");
   Class clazz = Person.class;
   Object object = null;
   try {
     object = clazz.newInstance();
   } catch(InstantiationException e) {
     System.out.println("Can not instantiate");
   } catch (IllegalAccessException e) {
     System.out.println("Can not access the constructor");
   }
   System.out.println(object.getClass().getName());// Person

   for (Method m : clazz.getDeclaredMethods()) {
     System.out.println("Method name: " + m.getName());
     System.out.println("Method modifier: " + Modifier.toString(m.getModifiers()));
     if (m.getReturnType() == void.class) {
       System.out.println("Method's return type is void");
     }
   }
   for (Constructor c : clazz.getDeclaredConstructors()) {
     System.out.println("Constructor: " + c.getName() + ", # of parameters: " + c.getParameterTypes().length);
   }

   Constructor<Person> constructor = clazz.getDeclaredConstructor(int.class, String.class, String.class);
   Person person = constructor.newInstance(123, "jyt0532", "mountain view");
   System.out.println("Result of calling func1: " + person.func1());
   Method m1 = clazz.getDeclaredMethod("func1");
   Object ret1 = m1.invoke(person);
   System.out.println("Result of invoking func1: " + ((Boolean)ret1).booleanValue());

   Method m2 = clazz.getDeclaredMethod("func2", int.class, long.class, String.class);
   Object ret2 = m2.invoke(person, 1, 2L, "abc");
 }
}
{% endhighlight %}

執行Reflection 給定JVM參數 `-verbose:class` 執行結果如下

{% highlight txt %}

Before loading Person class
[Loaded Person from file:/Users/bchiang/Downloads/javaWeakRefernceExample/out/production/main/]
Person
Method name: func1
Method modifier: public
Method name: func2
Method modifier: protected static final
Method's return type is void
Constructor: Person, # of parameters: 0
Constructor: Person, # of parameters: 3
Result of calling func1: true
Result of invoking func1: true
[Loaded java.lang.Long$LongCache from /Library/Java/JavaVirtualMachines/jdk1.8.0_121.jdk/Contents/Home/jre/lib/rt.jar]
abc

{% endhighlight %}

範例程式應該是簡單易懂 相信聰明的你能夠讀得懂 但有幾點特別值得提出來說明

1.只有在需要初始化的時候 才會去加載一個類別 在這個程式裡面的

{% highlight java %}
Class clazz = Person.class;
{% endhighlight %}
就是符合類別主動使用的第六個時機 所以JVM去加載了Person類別 把Person類別存在堆中後 再回傳給clazz

2.newInstance()這個方法

{% highlight java %}
object = clazz.newInstance();
{% endhighlight %}

可能拋出兩種異常 

`InstantiationException` 指的是不能生出實例 可能是找不到相對應的建構子 newInstance() 沒給任何參數就是呼叫沒參數的建構子 如你所知 
在**編譯時期**你可是無法知道有沒有你要的建構子的 所以你必須處理這種異常

`IllegalAccessException` 指的就是你沒有權限碰到這個建構子 比如說你把`Person(){}`前面加上個private 就會拋出這個異常

3.這個迴圈就是我們跑遍每個方法的方式

{% highlight java %}
for (Method m : clazz.getDeclaredMethods()) 
{% endhighlight %}

這讓我們可以去看每個方法的回傳值 描述符等等 

4.這裡我們用的是第二個建構子

{% highlight java %}
Person person = constructor.newInstance(123, "jyt0532", "mountain view");
{% endhighlight %}

5.這裡你還看到了兩種方式來呼叫一個方法
{% highlight java %}
person.func1()
m1.invoke(person);
{% endhighlight %}

兩種方式呼叫同一個方法fun1 回傳結果都一樣

至於要呼叫一個帶有參數的方法 用法如下:


{% highlight java %}
m2.invoke(person, 1, 2L, "abc");
{% endhighlight %}

看懂這段程式 反射就學的差不多了


### Why:為什麼我們需要反射

學完了用法 再來一個重要的問題就是為什麼需要這麼做

現在我們有了兩種方式來生一個字串
{% highlight java %}
new String()
{% endhighlight %}
或是
{% highlight java %}
Class clazz = String.class;
clazz.newInstance()
{% endhighlight %}

其實做到的事是一樣的 用反射的好處當然有一些

1.在運行時期判斷一個對象所屬的類別

2.在運行時創建一個對象 這個對象的類別在編譯時期還不需要確定

3.在運行時存取或調用一個對象的所有成員以及方法


但是如果**可以不要用反射 就不要用反射** 理由如下

1.因為是動態才決定要做哪些事情 加載哪些類別 所以很多虛擬機的最佳化都無法執行

2.會透露很多你原本封裝好的資訊 因為你可以存取所有私有的方法跟變數

順便教你一下怎麼存取 好孩子不要學 
我們剛剛的例子如果func1是private 是無法invoke的(當然也不能直接call) 但你只要偷偷加上一行


{% highlight java %}

Method m1 = clazz.getDeclaredMethod("func1");
m1.setAccessible(true);//evil
Object ret1 = m1.invoke(person);
System.out.println("Result of invoking func1: " + ((Boolean)ret1).booleanValue());
{% endhighlight %}

就可以呼叫私有了 是不是覺得反射破壞了不少Java引以為傲的特性? 所以反射 能不用就不用

### Where: 哪裡我們用到反射

最常見的用途就是IDE的自動完成的提示

![Alt text]({{ site.url }}/public/jvm/jvm-3-7.png)

實作方式就是運行期間去得到這個類別所有的方法 然後把signature列出來 非常實用

第二常見的用途是Annotation 你可以在運行時期去看每一個方法的annotation來決定不同的行為 

比如說JUnit test framework 它就使用Reflection去看每個有@Test的方法 並且是以test開頭的方法 並只執行那些符合條件的方法


### 哪些方法可以用

#### 存取類別方法
{% highlight java %}
Method[] getDeclaredMethods()
Method getDeclaredMethod(String name, Class[] params)
Method[] getMethods()
Method getMethod(String name, Class[] params)
{% endhighlight %}

Declared代表說這個類別自己宣告的方法 反之呢就是所有方法 包含繼承而來的

Class[] params是什麼呢 用法也很有趣 要把Array of class給進getMethod

{% highlight java %}
Class[] cArg = new Class[3];
cArg[0] = Integer.TYPE;
cArg[1] = Long.TYPE;
cArg[2] = String.class;
Method m = clazz.getMethod("func2", cArg);
System.out.println("method = " + m.toString());
{% endhighlight %}

Output如下

{% highlight txt %}
method = public void Person.func2(int,long,java.lang.String)
{% endhighlight %}

#### 存取類別字段

{% highlight java %}
Field[] getFields()
Field getField(String name)
Field getDeclaredField(String name)
Field[] getDeclaredFields()
{% endhighlight %}

#### 存取構造器
{% highlight java %}
Constructor<T> getConstructor(Class<?>... parameterTypes)
Constructor<?>[] getConstructors()
Constructor<?>[] getDeclaredConstructors()
Constructor getDeclaredConstructor(Class[] params)
{% endhighlight %}
用法在剛剛的範例程式都有

#### 存取類加載器

{% highlight java %}
ClassLoader getClassLoader()
{% endhighlight %}

如果是bootstrap加載器 那會回傳null 小心不要直接存取

{% highlight java %}
System.out.println(Integer.TYPE.getClassLoader().toString());
{% endhighlight %}

會拋出NullPointerException

### 反射 vs 類別自識

要特別注意的是 反射跟類別自識(Type introspection)不同 類別自識指的是可以在執行期間查看一個物件是屬於什麼類別 你就可以知道這個物件做得到什麼事 但反射指的是除了知道以外 還可以動態修改

所以reflection比introspection強上太多

### 為什麼反射叫做反射

反射快到講到尾聲 你們有沒有想過為什麼反射要叫做反射 我來稍微說一下我的想法

一般我們在寫程式的時候 是先有類別 然後藉由創建物件來取得實例化的對象 這是比較為人知的用法 反射就是相反 先得到一個物件之後 從這個物件存取類物件 然後再從類物件進而得知這個類別的訊息

所以用法跟一般的用法相反 一般用法叫入射(incidence) 相反用法叫反射(reflection)

## 總結

再看一次反射的定義 

> 反射 指的是在程序運行期間動態的去操作類物件的原數據(meta-data) 包括類別名稱 方法名稱等等

經過本篇文章的教學 你知道你可以寫一個函數 這個函數給入一個物件 你可以在運行期間分析這個物件 來決定程式要做什麼

比如說 如果這個傳入的物件的類別有兩個constructor 就 a++

如果這個傳入的物件的類別有兩個static variable 就 b += 2

等等用途 主要是可以讓我們

1.在運行時判斷任意一個對象所屬的類

2.在運行時構造任意一個類的對象(即使這個類別在編譯時期仍未給定)

3.在運行時判斷任意一個類所具有的成員變量和方法(透過反射甚至可以呼教private方法)

4.在運行時調用任意一個對象的方法
