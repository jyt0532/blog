---
layout: post
title: Effective Java Item29 - 優先考慮泛型
comments: True 
subtitle: effective java - 優先考慮generic types
tags: effectiveJava
author: jyt0532
excerpt: 本篇文章介紹泛型化的好處 並實際操作泛型化
---

這篇是Effective Java - Favor generic type章節的讀書筆記 本篇的程式碼來自於原書內容

## Item29: 優先考慮泛型

一般來說 將集合聲明參數化 或是使用library提供的泛型方法 不會太困難 但要自己編寫泛型就需要多加練習

本篇會實際走過一個完整的泛型化的步驟 讓你知道怎麼讓一個類別實現泛型

### Stack實現

看一下我們在[Item7](/2018/07/22/eliminate-obsolete-object-references/)看到的Stack
{% highlight java %}
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public Object pop() {
        if (size == 0)
            throw new EmptyStackException();
        Object result = elements[--size];
        elements[size] = null; // Eliminate obsolete reference
        return result;
    }

    public boolean isEmpty() {
        return size == 0;
    }

    private void ensureCapacity() {
        if (elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size + 1);
    }
}
{% endhighlight %}

你看到 elements是個物件數組 這就是個練習泛型的好目標

#### 第一步: 給聲明添加類型參數

先在`Stack` 後面加上`<E>` 然後把所有類別裡面的`Object`換成`E`

{% highlight java %}
public class Stack<E> {
    private E[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        elements = new E[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(E e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public E pop() {
        if (size == 0)
            throw new EmptyStackException();
        E result = elements[--size];
        elements[size] = null; // Eliminate obsolete reference
        return result;
    }
    ... // no changes in isEmpty or ensureCapacity
}
{% endhighlight %}

通常第一部做完 會看到不少錯誤 幸運的是這次只有一個

![Alt text]({{ site.url }}/public/item29-1.png)

#### 第二步: 消除錯誤

原因是你不能創建一個**不能具體化**的Array 編譯器根本不知道E是什麼

對於這種問題 有兩種主要的解決方法

##### 解法1
創一個Object Array 後再強制轉型
{% highlight java %}
public Stack() {
  elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];
}
{% endhighlight %}

寫完後 從錯誤變成警告

![Alt text]({{ site.url }}/public/item29-2.png)

編譯器無法證明這個轉換是typesafe 但是我們可以 原因如下

1.可能會產生問題的`elements`是private 永遠不會傳給客戶

2.會寫進`elements`的唯一方法是`push` 而`push`的傳入參數是`E`

所以我們可以確定 這個強制Cast很安全 所以我們可以加上註解來抑制警告
別忘了當你要抑制警告時 [盡可能縮小範圍](/2018/12/02/eliminate-unchecked-warnings/)

{% highlight java %}
// The elements array will contain only E instances from push(E).
// This is sufficient to ensure type safety, but the runtime
// type of the array won't be E[]; it will always be Object[]!
@SuppressWarnings("unchecked")
public Stack() {
    elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];
}
{% endhighlight %}

##### 解法2

把`elements`從`E[]` 變成`Object[]`

既然elements是`Object[]` 那constructor裡就沒問題了 問題在`pop`

一樣我們強制轉型`(E)` 看到了警告

![Alt text]({{ site.url }}/public/item29-3.png)

抑制它

{% highlight java %}
// Appropriate suppression of unchecked warning
public E pop() {
    if (size == 0)
        throw new EmptyStackException();

    // push requires elements to be of type E, so cast is correct
    @SuppressWarnings("unchecked") E result =
        (E) elements[--size];

    elements[size] = null; // Eliminate obsolete reference
    return result;
}
{% endhighlight %}

#### 兩種解法比較

兩種解法都有追隨者 第一種明確的定義`elements`是`E[]` 而且只需要在constructor中處理好就可以

但第二種 因為`elements`是`Object[]` 所以在每次讀取的時候 你都必須要Cast 

下面的程式展示了如何使用我們的泛型Stack
{% highlight java %}
public static void main(String[] args) {
    Stack<String> stack = new Stack<>();
    for (String arg : args)
        stack.push(arg);
    while (!stack.isEmpty())
        System.out.println(stack.pop().toUpperCase());
}
{% endhighlight %}

`Stack<E>`裡面的`E`可以是任何東西 `Stack <Object>` `Stack <int []>` `Stack <List <String >>` 

但不能是primitive type 比如說`Stack<int>` `Stack<long>` 你只能用`Stack<Integer>`或`Stack<Long>`代替

### 結論

使用泛型 比使用強制轉換更安全 當你手上有一些現有的類型應該要被泛型化 像是本文一開始的Stack 就試著把它泛型化 
這會讓這類型的新用戶覺得易於使用 而且不會破壞現有的客戶端

