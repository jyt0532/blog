---
layout: post
title: Effective Java Item6 - 避免創建不必要的對象
comments: True 
subtitle: effective java - 避免創建不必要的對象
tags: effectiveJava
author: jyt0532
excerpt: 本篇文章介紹為何要避免創建不必要的對象
---

這篇是Effective Java - Avoid creating unnecessary objects章節的讀書筆記 本篇的程式碼來自於原書內容

## Item6: 避免創建不必要的對象

一般來說 最好可以重用對象 而不是每次需要的時候都創建一個相同功能的新對象 
如果對象是不可變的 那這個對象就始終可以被重用

來個例子
{% highlight java %}
String s = new String("stringette"); // DON'T DO THIS!
{% endhighlight %}

這樣的話 這句話每次執行的時候 都會創建一個新的String 實例

而且每次傳遞給String構造器的參數 也本身是一個String實例 
如果這行程式是在for-loop裡面 那很多不必要的物件會被創造

那應該怎麼寫? 就這樣

{% highlight java %}
String s = "stringette";
{% endhighlight %}

這個版本只用了一個String實例 而不是每次創建一個新的實例 他保證在同一個虛擬機中運行的代碼 即使是不同的程式 他們都還可以重用同一個對象!

書中就輕描淡寫地寫到這樣 但如果你有興趣的話 可以參考[字串池](/toc/jvm/jvm_2/)
所有可以重用的字串都被存在字串池裡面了

### 靜態工廠

事實上一個類別如果同時提供建構子和靜態工廠 你應該要用[靜態工廠](/2017/09/20/static-factory-method/)

因為建構子會一直創建新的物件 但靜態工廠可以有內部的邏輯決定要不要創建 或是要不要重用對象

### 避免昂貴的物件創建

有些物件創建是無可避免的 但是卻非常昂貴

比如我們想判斷一個字串是不是Roman numeral

{% highlight java %}
static boolean isRomanNumeral(String s){
  return s.matches("^(?=[MDCLXVI])M*(C[MD]|D?C{0,3})(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
}
{% endhighlight %}

String.matches是最簡單直觀的方式沒錯 但如果這段程式是出現在一個要求性能的地方 那可能就不適合這樣寫

因為每次確認的時候 這個方法都會創建了一個Pattern的物件(非常昂貴) 而且只用一次 用完就等著垃圾收集器回收

改進方式就是把Pattern給cache下來

{% highlight java %}
public class RomanNumerals {
  private static final Pattern ROMAN = Pattern.compile("^(?=[MDCLXVI])M*(C[MD]|D?C{0,3})(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
  static boolean isRomanNumeral(String s){
    return ROMAN.matcher(s).matches();
  }
}
{% endhighlight %}

改成這樣寫 程式的性能會大幅增加

### 思考你的程式的使用情況

之前討論的例子 因為物件在初始化之後就不會也不能再改變了 
所以你可以很輕易地感覺到 你應該可以重用物件 

但某些情況就不是這麼明顯

比如說你想實作一個Map.keySet() 功能是把所有的key丟進Set裡面回傳

每次呼叫 keySet()都去重新生一個Set然後一個一個把key丟進去是很花時間的 因為只要Map沒有變化的話 keySet()回傳的set就不會有變化

所以你可以觀察你的函式的使用狀況 如果get/set比例>100 你可以每次set的時候更新keySet 然後把keySet給cache起來
如果get/set比例<1/100 你就是每次get的時候重算

### 總結

本條目跟[Item50 - 必要時進行保護型拷貝](/2017/09/26/make-defensive-copies-when-needed/) 相互對應

本條目說 當你應該重用既有對象的時候 不要創建對象 

Item50說 當你應該創建新對象的時候 請不要重用現有對象

注意Item50比本條目嚴重 當你應該創建新對象而沒創建 造成的問題是潛在錯誤跟安全漏洞 而當你應該重用而沒有重用 就只是性能會受影響

建議讀者搭配[Item50](/2017/09/26/make-defensive-copies-when-needed/)服用 了解何時該重用 何時該創建


