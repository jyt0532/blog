---
layout: post
title: Effective Java Item4 - 透過私有構造器來禁止實例化
comments: True 
subtitle: effective java - 透過私有構造器來禁止實例化
tags: effectiveJava
author: jyt0532
excerpt: 當你要編寫工具類別的時候 使用私有構造器是防止使用者實例化的最好方式
---

這篇是Effective Java - Enforce noninstantiability with a private constructor章節的讀書筆記 本篇的程式碼來自於原書內容

### Item4: 透過私有構造器來禁止實例化

有時候 你需要寫一些類別 這些類別裡面只有static的方法跟變量 這種類別在物件導向的語言比較不被推崇 因為這有點像是開了個後門 讓你可以不用**從物件出發**去設計程式

雖然如此 但還是有許多有名的這種類別 比如java.lang.Math或是java.util.Arrays

像這種像是工具類的類別 我們不希望有人去實例化 但若你在編寫一個類似的類別而不提供顯式的構造器的話 他會生一個預設的構造器給你 這個預設的構造器不吃任何參數

要如何避免呢？

提案ㄧ： 是不是把這個類別宣告成abstract 就不會有人實例化它了呢

不對 因為這類別還是可以被繼承 繼承之後就可以被實例化 甚至會誤導你的使用者 讓使用者以為這個類別就是想要被繼承

提案二: 自己先寫個構造器 但是宣告成private

{% highlight java %}
// Noninstantiable utility class
public class UtilityClass {
// Suppress default constructor for noninstantiability
private UtilityClass() {
  throw new AssertionError();
}
{% endhighlight %}


可行 但缺點就是 這個類別就不能被繼承
