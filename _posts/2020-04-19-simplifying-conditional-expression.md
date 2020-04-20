---
layout: post
title: 重構 - 改善既有程式的設計 - Simplifying Conditional Expressions
comments: True
subtitle: 簡化條件式
tags: refacforing
author: jyt0532
excerp: 本文介紹如何簡化條件式
---

這篇文章討論《重構 - 改善既有程式的設計》裡的第九章 - Simplifying Conditional Expressions

主要會討論[程式碼的壞味道](/toc/refactoring/)中 沒有被提及的重構方法裡面有關於 簡化函式呼叫 的內容

圖片以及程式碼來源自[重構 - 改善既有程式的設計](https://www.tenlong.com.tw/products/9789861547534)

## 簡化條件式

### [Decompose Conditional](/2020/04/09/large-method/#decompose-conditional-分解條件式)

你有一個複雜的條件語句 就把那複雜的條件語句提煉出獨立函式

### [Consolidate Conditional Expression](/2020/04/15/duplicate-code/#consolidate-conditional-expression)

你有一系列條件測試都得到相同結果 就可以把那些條件式提煉成獨立函式

### [Consolidate Duplicate Conditional Fragments](/2020/04/15/duplicate-code/#consolidate-duplicate-conditional-fragments)

合併重複的條件片段

### Remove Control Flag

控制旗標(control flag) 就是你常看到的 **用以判斷何時停止條件檢查** 的變數

比如說下面程式中的`found`
{% highlight java %}
void checkSecurity(String[] people) {
  boolean found = false;
  for (int i = 0; i < people.length; i++) {
    if (! found) {
      if (people[i].equals ("Don")){
        sendAlert();
        found = true; 
      }
      if (people[i].equals ("John")){ 
        sendAlert();
        found = true;
      } 
    }
  } 
}
{% endhighlight %}

這樣的控制旗標帶來的麻煩超過了帶來的便利 我們應該用 break 或是 continue

{% highlight java %}
void checkSecurity(String[] people) {
  for (int i = 0; i < people.length; i++) {
    if (people[i].equals ("Don")){
      sendAlert();       
      break;
    }
    if (people[i].equals ("John")){
      sendAlert();
      break; 
    }
  } 
}
{% endhighlight %}


### Replace Nested Conditional with Guard Clauses

當你函式中的條件邏輯複雜 使人難以看出正常情況的執行路徑 就可以使用guard clause來表現所有特殊狀況

來看下面這個程式

{% highlight java %}
public double getPayAmount() {
  double result;
  if (isDead){
    result = deadAmount();
  }
  else {
    if (isSeparated){
      result = separatedAmount();
    }
    else {
      if (isRetired){
        result = retiredAmount();
      }
      else{
        result = normalPayAmount();
      }
    }
  }
  return result;
}
{% endhighlight %}

很難看得出 正常情況應該是第二個條件的第二個條件的第二個條件 我們應該重構成這樣

{% highlight java %} 
public double getPayAmount() {
  if (isDead){
    return deadAmount();
  }
  if (isSeparated){
    return separatedAmount();
  }
  if (isRetired){
    return retiredAmount();
  }
  return normalPayAmount();
}
{% endhighlight %}

這樣一目了然 

大方向就是 if-else的條件是 給人的感覺是 if的情況跟else的情況同等的重要 

但guard clause不同 就是用來專門處理比較特別的情況

### [Replace Conditional with Polymorphism](/2020/04/11/switch-statement/#replace-conditional-with-polymorphism)

當你發現switch裡面的判斷式彼此是subclass的關係 那就可以用多型

### [Introduce Null Object](/2020/04/11/switch-statement/#introduce-null-object)

如果其中一個條件選項是null 那你可以考慮用Null Object


