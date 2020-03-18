---
layout: post
title: Effective Java Item5 - 依賴注入優於硬連接資源
comments: True 
subtitle: effective java - Prefer dependency injection to hardwiring resource
tags: effectiveJava
author: jyt0532
excerpt: 本篇文章介紹為何依賴注入優於hardwiring resources
---

這篇是Effective Java - Prefer dependency injection to hardwiring resource章節的讀書筆記 本篇的程式碼來自於原書內容

## Item5: 依賴注入優於硬連接資源

有某些類依賴於一個或多個底層資源 比如說拼寫檢查器需要依賴字典 而我們常會把這種類別實現成靜態工具類

{% highlight java %}
// Inappropriate use of static utility - inflexible & untestable!
public class SpellChecker {
    private static final Lexicon dictionary = new Lexicon();

    private SpellChecker() {} // Noninstantiable

    public static boolean isValid(String word) { ... }
    public static List<String> suggestions(String typo) { ... }
}
{% endhighlight %}

注意 我們把Lexicon定為final 而且在這裡寫死(`new Lexicon()`)

當然你也可以把它變成Singleton

{% highlight java %}
// Inappropriate use of singleton - inflexible & untestable!
public class SpellChecker {
    private final Lexicon dictionary = new Lexicon();

    private SpellChecker(...) {}
    public static INSTANCE = new SpellChecker(...);

    public boolean isValid(String word) { ... }
    public List<String> suggestions(String typo) { ... }
}
{% endhighlight %}

但這兩種方法都不夠好 因為這代表SpellChecker基本上跟底層字典綁死 你想從英文字典換成中文字典做不到

所以**靜態工具類和單例類 不適合需要引用底層資源的類**


## Dependency Injection

我們在這裡需要的是一個可以support多個instance的類別 而且可以依照客戶的喜好來決定要提供什麼instance

**為了滿足這個需求 我們需要可以在創建新實例時將資源(resource)傳遞到構造器中**

這就是依賴注入 我們把對於字典的依賴 注入到拼寫檢查器中

{% highlight java %}
// Dependency injection provides flexibility and testability
public class SpellChecker {
    private final Lexicon dictionary;

    public SpellChecker(Lexicon dictionary) {
        this.dictionary = Objects.requireNonNull(dictionary);
    }

    public boolean isValid(String word) { ... }
    public List<String> suggestions(String typo) { ... }
}
{% endhighlight %}


## 依賴注入的好處

### 減少依賴 

在我們的例子 如果我們今天想把英文字典改成中文字典 我們不用再改SpellChecker類別 只要改變Client的使用方式就可以

### 程式碼重複使用

這也好懂 我們就不需要有一個英文的SpellChecker和一個中文的SpellChecker

### 更好測試

因為我們可以輕鬆Mock底層的Depended On Component 詳情可以參考[測試替身系列文](/2017/12/15/test-doubles/)


