---
layout: post
title: 重構 - 改善既有程式的設計 - Making Method Calls Simpler
comments: True
subtitle: 簡化函式呼叫
tags: refacforing
author: jyt0532
excerp: 本文介紹如何簡化函式呼叫
---

這篇文章討論《重構 - 改善既有程式的設計》裡的第八章 - Making Method Calls Simpler

主要會討論[程式碼的壞味道](/toc/refactoring/)中 沒有被提及的重構方法裡面有關於 簡化函式呼叫 的內容

圖片以及程式碼來源自[重構 - 改善既有程式的設計](https://www.tenlong.com.tw/products/9789861547534)

## 簡化函式呼叫

在物件導向的技術中 最重要的概念就是**介面** 容易被理解和使用的介面 是開發良好物件導向軟體的關鍵 本章會介紹所有讓介面變得更簡潔易用的重構方法

### [Rename Method](/2020/04/15/comments/#rename-method)

函式的名稱就應該顯示函式的用途

### Separate Query From Modifier

當你有個函式 技回傳物件狀態 也修改物件狀態 那就最好分開 一個負責查詢 一個負責修改

![Alt text]({{ site.url }}/public/separate-query-from-modifier.png)

一個常見的好習慣是區分出 有副作用的 和 無副作用的 函式

### Parameterize Method

你可能在重構的過程中發現很多類似的函式 只是差別差在一個數字 而且函式的實作中 **對於每個數字的處理方式都一樣** 那就把這個變量提出來 讓函式攜帶參數

![Alt text]({{ site.url }}/public/parameterize-method.png)

### [Replace Parameter with Explicit Methods](/2020/04/11/switch-statement/)

[Parameterize Method](parameterize-method)的反向 把參數取消 分離成不同的函式

什麼時候使用哪個呢 如果函式的實作中 對於每個數字的處理方式不一樣 而且還有一個if-else 那就很值得分離

來看個小例子

{% highlight java %}
void setHeightToTwo(){
  a.setCurrentHieght(2);
}
void setHeightToFive(){
  a.setCurrentHieght(5);
}
void setHeightToTen(){
  a.setCurrentHieght(10);
}
{% endhighlight %}

這個例子就很適合加個參數

{% highlight java %}
void setHeight(int h){
  a.setCurrentHieght(h)
}
{% endhighlight %}

但如果是下面的情況

{% highlight java %}
void setHeight(int h){
  if(h == 2) {
    methodA();
  }
  if(h == 5) {
    methodB();
  }
  if(h == 10) {
    methodC();
  }
}
{% endhighlight %}

那就應該把參數拿掉

{% highlight java %}
void setHeightToTwo(){
  methodA();
}
void setHeightToFive(){
  methodB();
}
void setHeightToTen(){
  methodC();
}
{% endhighlight %}

### Preserve Whole Object(/2020/04/09/large-method/#preserve-whole-object)

你從一個物件中取出某些值 去傳給其他函式 那你不如直接把那個物件丟進去

### [Replace Parameter with Method](/2020/04/10/long-parameter-list/#replace-parameter-with-method)

如果被呼叫端也可以自己拿到那個參數 就可以直接讓被呼叫端自己算

### [Introduce Parameter Object](/2020/04/09/large-method/#introduce-parameter-object)

如果參數間的關係很強 可以把參數們包成一個參數物件

### Remove Setting Method

如果你的物件只應該在初始時被設值 那你就應該把setter拿掉

### Hide Method

如果有一個函式 從來沒有被其他類別用到的話 就應該把它改成private

可以參考[Effective Java Item15 - 使類和成員的可訪問性最小化](/2018/04/21/minimize-the-accessibility-of-classes-and-members/)

### Replace Constructor with Factory Method

請參考[Effective Java Item1 - 靜態工廠方法](/2017/09/20/static-factory-method/) 好處非常多

### Replace Error Code with Exception

如果你有一個函式 使用回傳值來表示出了什麼錯 那你不如直接拋出例外

{% highlight java %}
int withdraw(int amount) {
  if (amount > _balance) {
    return -1;
  }
  else {
    balance -= amount;
    return 0;
  }
}
{% endhighlight %}

不如這樣

{% highlight java %}
void withdraw(int amount) throws BalanceException {
  if (amount > _balance) {
    throw new BalanceException();
  }
  balance -= amount;
}
{% endhighlight %}

### Replace Exception with Test

不要濫用異常 如果這個異常的程式碼可以被輕易的避免

比如你有以下的程式

{% highlight java %}
double getValueForPeriod(int periodNumber) {
  try {
    return values[periodNumber];
  } catch (ArrayIndexOutOfBoundsException e) {
    return 0;
  }
}
{% endhighlight %}

你只要確認periodNumber是不是出界就可以

{% highlight java %}
double getValueForPeriod(int periodNumber) {
  if (periodNumber >= values.length) {
    return 0;
  }
  return values[periodNumber];
}
{% endhighlight %}









