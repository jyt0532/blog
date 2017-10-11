---
layout: post
title: Effective Java - 深入解析序列化byte stream
comments: True 
subtitle: effective java - 序列化深入了解
tags: effectiveJava
author: jyt0532
---
這篇文章是閱讀Effective Java 第11章 - Item76之前需要會的知識 

### 深談序列化

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

我們來仔細看看這個文件

![Alt text]({{ site.url }}/public/serializedObject.png)

現在看他是天書 五分鐘後這個byte stream在你眼前就會變得非常赤裸

Are you Ready?

### 庖丁解牛

![Alt text]({{ site.url }}/public/serializedObject2.png)

<span style="color:red">ACED</span>: MAGIC NUMBER

<span style="color:rgb(54, 241, 73)">0005</span>: STREAM_VERSION STREAM的版本

<span style="color:blue">73</span>: 新的對象

<span style="color:orange">72</span> 新的Class descriptor

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

### 殺進物件內部

![Alt text]({{ site.url }}/public/serializedObject3.png)

<span style="color:red">0002</span>: 這個物件有多少個instance (name, age 兩個)

<span style="color:rgb(54, 241, 73)">49</span>: 這個variable型態 "I" 代表integer

<span style="color:blue">0003</span>: 這個variable長度 3

<span style="color:orange">616765</span> 

0x61 = 97 = a

0x67 = 103 = g

0x65 = 101 = e

<span style="color:rgb(54, 241, 73)">4C</span>: 這個variable型態 "L" 代表object

<span style="color:blue">0004</span>: 這個variable長度 4

<span style="color:orange">6E616D65</span> 

0x6E = 110 = n

0x61 = 97 = a

0x6D = 109 = m

0x65 = 101 = e

![Alt text]({{ site.url }}/public/serializedObject4.png)

<span style="color:red">74</span>: 接下來是個String (TC_STRING = (byte)0x74;)

<span style="color:rgb(59, 247, 247)">0012</span>: 接下來這個String長度是18

<span style="color:rgb(52, 237, 67)">4C6A6176612F6C616E672F537472696E673B</span>: 

師爺給我翻譯翻譯

"Ljava/lang/String;"

<span style="color:rgb(169, 123, 86)">78</span> 結束對象標誌(TC_ENDBLOCKDATA = (byte)0x78)

<span style="color:rgb(1, 1, 11)">70</span> 這個class沒有服類(TC_NULL = (byte)0x70;)

<span style="color:rgb(167, 93, 172)">00000001</span> 第一個變數的value

<span style="color:rgb(242, 149, 36)">74</span> 接下來是個String (TC_STRING = (byte)0x74;)

<span style="color:rgb(18, 44, 237)">0004</span>: 接下來這個String長度是4

<span style="color:rgb(138, 26, 128)">4A6F686E</span>: 

"John"
