---
layout: post
title: Effective Java Item43 - 方法引用優於lambda表達式 
comments: True 
subtitle: effective java - 方法引用優於lambda表達式
tags: effectiveJava
author: jyt0532
excerpt: 本篇文章介何謂方法引用 以及方法引用的使用時機
---

這篇是Effective Java - Prefer method reference over lambdas章節的讀書筆記 本篇的程式碼來自於原書內容

## Item43: 方法引用優於lambda表達式

[Item42](/2018/08/05/prefer-lambdas-to-anonymous-classes/)說明lamdba很簡潔 但有一種寫法比lambda更簡潔 那就是方法引用

來看一個例子 map.merge可以讓你改變任意給定的key的value

{% highlight java %}
map.merge(key, 1, (count, incr) -> count + incr);
{% endhighlight %}

稍微講解一下merge這個方法

{% highlight java %}
default V merge(K key, V value, BiFunction<? super V,? super V,? extends V> remappingFunction)
{% endhighlight %}

第一個參數就是你要改變的key 第二個跟第三個參數就是你要怎麼改

其中第三個參數是個函數介面 你要給一個實作這個介面的實例 這個介面有一個方法 就是給兩個整數回傳一個整數 

所以第二個參數就是這個函式的第二個輸入

{% highlight java %}
//map[3] = x
map.merge(3, 5, (old, new) -> old + new); //map[3] = x + 5
{% endhighlight %}

但注意如果old value是null 或是map裡沒有那個key 那就直接變成new value

{% highlight java %}
//map[3] = null or !map.containsKey(3)
map.merge(3, 5, (old, new) -> old + new); //map[3] = 5
{% endhighlight %}

了解了之後再回頭看我們的範例程式

{% highlight java %}
map.merge(key, 1, (count, incr) -> count + incr);
{% endhighlight %}

簡單 就是把map[key]++

這樣的寫法還是挺冗長的 而且這個lambda功能非常常見(其實就是把兩個整數相加) 所以就有方法引用可以直接用

{% highlight java %}
map.merge(key, 1, Integer::sum);
{% endhighlight %}

輕鬆 簡潔

#### 冗長的方法引用

但也並不是每次都是方法引用獲勝 來看一下例子

{% highlight java %}
service.execute(GoshThisClassNameIsHumongous::action);
{% endhighlight %}

如果這個方法是出現在ClassA內部 那不如這樣

{% highlight java %}
service.execute(() -> action());
{% endhighlight %}

lambda又短又簡單易懂

所以你應該在有機會用方法引用的時候考慮使用它 除非它讓你的程式變更長而且沒有比較好懂

### 方法引用的種類

#### 靜態方法引用(Static)

lambda: str -> Integer.parseInt(str)

方法引用: Integer::parseInt

#### 特定對象方法引用(Bound)

lambda: Instant then = Instant.now(); t -> then.isAfter(t)

方法引用: Instant.now()::isAfter

#### 任意對象方法引用(Unbound)

lambda: str -> str.toLowerCase()

方法引用: String::toLowerCase

#### 類別建構子(Class Constructor)

lambda: () -> new TreeMap<K,V>

方法引用: TreeMap<K,V>::new

#### 數組建構子(Array Constructor)

lambda: len -> new int[len]

方法引用: int[]::new

### 結論

只要能用方法引用就試用看看 方法引用通常比lambda更簡潔

但如果真的沒有比較好的話就還是堅持lambda


