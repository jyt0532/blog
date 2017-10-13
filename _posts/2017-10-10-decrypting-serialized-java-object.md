---
layout: post
title: Effective Java - 深入解析序列化byte stream
comments: True 
subtitle: effective java - 序列化深入了解
tags: effectiveJava
author: jyt0532
---
這篇文章是閱讀Effective Java 第11章 - Item76之前需要會的知識 

關於序列化的Protocol詳細文件[在這](https://docs.oracle.com/javase/7/docs/platform/serialization/spec/protocol.html) 不過這篇門檻太高 需要知道什麼是Context Free Grammar 有興趣的各位自動機大神可以來看一下
### 深談序列化

序列化的目的是為了傳輸或儲存 所以最重要的事情就是序列化的那端用的protocol跟反序列化的那端用的protocol一樣

既然如此 Java就定義了所有人都要共同遵尋的protocol

1.寫下你當下的class的metadata

2.Recursively寫parent class的metadata 寫到java.lang.Object為止

3.再來 寫下class的instance variable的值 **從最Parent class的instance開始寫**

### 直上例子

序列化的目標當然是我們最可愛的Person Class
{% highlight java %}
class Person implements Serializable {
  public String name;
  public int age;

  Person() {
    this("John",1);
  }

  Person(String name, int age) {
    this.name = name;
    this.age = age;
  }
}
{% endhighlight %}
跑以下的程式 就會把Person物件存進person.ser這個文件裡
{% highlight java %}
public class TestDrive{

  public static void main(String[] args) {
    String filename = "person.ser";
    Person p = new Person();

    // save the object to file
    FileOutputStream fos = null;
    ObjectOutputStream out = null;
    try {
      fos = new FileOutputStream(filename);
      out = new ObjectOutputStream(fos);
      out.writeObject(p);

      out.close();
    } catch (Exception ex) {
      ex.printStackTrace();
    }
  }
}
{% endhighlight %}

我們來仔細看看person.ser這個文件

![Alt text]({{ site.url }}/public/serializedObject.png)

現在看他是天書 五分鐘後這個byte stream在你眼前就會變得非常赤裸

Are you Ready?

### 庖丁解牛

![Alt text]({{ site.url }}/public/serializedObject2.png)

<span style="color:red">ACED</span>: MAGIC NUMBER 告訴你說這是段被序列化的byte stream

<span style="color:rgb(54, 241, 73)">0005</span>: STREAM_VERSION STREAM的版本

<span style="color:blue">73</span>: 新的Object

<span style="color:orange" id="72">72</span> 新的Class descriptor 接下來的是一個新的class

<span style="color:purple">0006</span> Class name長度 (Person: 6)

<span style="color:rgb(168, 125, 83)">506572736F6E</span> ClassName
把16進位的值轉成ASCII

0x50 = 80 = P

0x65 = 101 = e

0x72 = 114 = r

0x73 = 115 = s

0x6F = 111 = o

0x6E = 110 = n



<span style="color:rgb(252, 89, 250)">B983994E4F36A9BC</span> serialVersionUID 如果你在你的Person寫死serialVersionUID 那個值就會出現在這裡

<span style="color:rgb(57, 88, 38)">02</span> 表示這個object支援序列化

### 殺進物件內部 繼續描述這個class

![Alt text]({{ site.url }}/public/serializedObject9.png)

<span style="color:red">0002</span>: 這個物件有多少個instance (name, age 兩個)

<span style="color:rgb(54, 241, 73)">49</span>: 第一個variable型態 0x49 = "I" 代表integer

<span style="color:rgb(147, 73, 152)">0003</span>: 這個variable長度 3

<span style="color:orange">616765</span> 

0x61 = 97 = a

0x67 = 103 = g

0x65 = 101 = e

<span style="color:rgb(54, 241, 73)">4C</span>: 這個variable型態 "L" 代表object

<span style="color:rgb(147, 73, 152)">0004</span>: 這個variable長度 4

<span style="color:orange">6E616D65</span> 

0x6E = 110 = n

0x61 = 97 = a

0x6D = 109 = m

0x65 = 101 = e

<span style="color:red">74</span>: 接下來是個String (TC_STRING = (byte)0x74;)

<span style="color:rgb(59, 247, 247)">0012</span>: 接下來這個String長度是18

<span style="color:rgb(52, 237, 67)">4C6A6176612F6C616E672F537472696E673B</span>: 

"Ljava/lang/String;"

因為String並不是java的Primitive type 只能算是一個object 

那在序列化如何描述一個物件呢 來看一下CFG:

objectDesc:
  
&nbsp;&nbsp;&nbsp;&nbsp;obj_typecode fieldName className1

原來在byte stream裡面要描述一個物件 需要先說object的type 再說object的name 再說class的name


至於為什麼前面有一個L? JVM會用最簡潔的方式儲存class L[class]; 代表一個class

有興趣可以參考[這裡](https://stackoverflow.com/questions/9909228/what-does-v-mean-in-a-class-signature)

![Alt text]({{ site.url }}/public/serializedObject10.png)
<span style="color:rgb(169, 123, 86)">78</span> 結束對象標誌(TC_ENDBLOCKDATA = (byte)0x78)

這個class描述完了 如果這個class有parent class 那就會從[72(點我)](#72)開始 recursive繼續描述class

<span style="color:rgb(1, 1, 11)">70</span> Recursive完畢 因為Person沒有父類 所以寫完一個class description就結束(TC_NULL = (byte)0x70;)

看到70就知道 所有class都描述完了 再來 開始記錄object裡面的instance的值
<br>
<br>
![Alt text]({{ site.url }}/public/serializedObject8.png)

<span style="color:rgb(167, 93, 172)">00000001</span> 第一個變數的value

<span style="color:rgb(242, 149, 36)">74</span> 接下來是個String (TC_STRING = (byte)0x74;)

<span style="color:rgb(18, 44, 237)">0004</span>: 接下來這個String長度是4

<span style="color:rgb(138, 26, 128)">4A6F686E</span>: 

"John"

### 總結

因為這個序列化可能會很長 也會有很多class重複用到很多次 
java不會每次看到同樣的class還跟第一次看到一樣全部寫上去 
所以對於每一個已經寫過的**物件**或是已經寫過的**class descriptor** 它會用一個reference serial number記住他 下次再遇到一個一樣的class 我就直接寫那個出現過的class的reference number



