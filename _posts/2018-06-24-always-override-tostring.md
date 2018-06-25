---
layout: post
title: Effective Java Item12 - 始終要覆蓋toString
comments: True 
subtitle: effective java - 始終要覆蓋toString
tags: effectiveJava
author: jyt0532
excerpt: 本篇文章介紹什麼時候要覆蓋toString  以及如何覆蓋toString
---

這篇是Effective Java - Always override toString章節的讀書筆記 本篇的程式碼來自於原書內容

## Item12: 始終要覆蓋toString

### 為何要複寫toString

雖然java.lang.Object有提供toString方法的實現 但他返回的字符串長得像這樣

PhoneNumber@163b91 

就是類的名稱 + @ + 一個hash 這也很明顯就是要你自己複寫 最好是能寫成一個 簡潔 信息豐富 易於閱讀 的形式

雖然覆蓋toString的約定不像遵守equals和hashCode的約定那麼重要 但是提供好的toString可以讓類的使用更加舒適 比如說傳進println,字串連結符(+), assert 或是debuger

比如說你今天要debug

System.out.println(“Failed to call “ + phoneNumber)

你想看到的是

PhoneNumber@163b91 

還是

(408) 867-5309

### 要怎麼複寫

實際應用中 toString應該返回對象中包含的值得關注的信息 最少要包含equals裡比較的每一個關鍵域

你總不希望在unit test看到

Assertion failure: expected {abc, 123}, but was {abc, 123}

吧

### 是否在文檔指定格式

在實現toString的時候 必須要做個重要決定 就是要不要在文檔中指定返回值的格式

好處:
1.用作一種標準的 明確的 適合人閱讀的對象表示法 可以用於輸入輸出 也可以存儲在適合人類閱讀的檔案文件中(比如xml)

2.最好再搭配一個靜態工廠或構造器 這樣使用者就可以輕鬆地在對象以及表示法之間來回轉換 java裡面已經有很多類別都這麼做了 比如說BigInteger BigDecimal和boxed primitive type

壞處:
一但指定格式 生生世世都必須遵守 因為你的使用者會寫程式去解析字串表示法 或是用來存儲

不論你要不要在文檔指定 都要在文檔表明意圖以及為什麼

直接看電話號碼的例子

{% highlight java %}
/**
 * Returns the string representation of this phone number. 
 * The string consists of fourteen characters whose format 
 * is "(XXX) YYY-ZZZZ", where XXX is the area code, YYY is 
 * the prefix, and ZZZZ is the line number. (Each of the 
 * capital letters represents a single decimal digit.)
 * 
 * If any of the three parts of this phone number is too 
 * small to fill up its field, the field is padded with 
 * leading zeros. For example, if the value of the line 
 * number is 123, the last four characters of the string 
 * representation will be "0123".
 * 
 * Note that there is a single space separating the closing 
 * parenthesis after the area code from the first digit of the prefix.
 */  
  @Override
  public String toString() {
    return String.format("(%03d) %03d-%04d", areaCode, prefix, lineNumber);
  }
{% endhighlight %}


如果你不想指定格式 你可以再加一段
{% highlight java %}
/**
* Returns a brief description of this phone number. The exact 
* details of the representation are unspecified and subject 
* to change, but the following may be regarded as typical:
* 
* "(707) 867-5309"
*
*/
{% endhighlight %}

這樣如果有程序員去過度使用你的格式 那在格式改變之後 就只能自行承擔後果

這也代表說當你在用某個類別時 你不該過度依賴一個類別的格式 比如說太過依賴格式的細節 進行存儲或是parsing


### 提供api

別忘了所有你在toString有用到的域 都要提供getter 比如說你一定要有getPrefix, getAreaCode和getLineNumber

否則的話你也是在強迫你的使用者去parse你的字串格式

### 不需要複寫toString的情況

static utility class

enum type


### 總結

除非你的superclass已經寫好了toString 不然你應該在每個可以被實例化的類別寫toString
toString方法應該回傳一個簡潔好用的物件描述 
這會讓你的類別更好用 更好開發和除錯



