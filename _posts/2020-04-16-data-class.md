---
layout: post
title: 重構 - 改善既有程式的設計 - Data Class
comments: True
subtitle: 如何重構純稚的資料類別
tags: refacforing
author: jyt0532
excerp: 本文介紹重構Data Class
---

這篇文章討論《重構 - 改善既有程式的設計》裡的3.20 - Data Class

圖片以及程式碼來源自[重構 - 改善既有程式的設計](https://www.tenlong.com.tw/products/9789861547534)


## 純稚的資料類別

所謂的Data Class 就是一個類別只擁有一些欄位 以及用來存取讀寫這些欄位的函式(getter/setter) 除此之外什麼都沒有 那我們就稱這個類別為Data Class

這種類別呢 就只是一個不會說話的資料容器 被其他類別所操控 更慘的是如果Data Class還有一些public的欄位讓大家直接存取 那就更慘了


## 解法 - 有public欄位

### Encapsulate Field

把public的欄位變成private 並提供存取函式

{% highlight java %}
class Person {
  public String _name;
}
{% endhighlight %}

變成
{% highlight java %}
class Person {
  private String _name;

  public String getName() {
    return _name;
  }
  public void setName(String arg) {
    _name = arg;
  }
}
{% endhighlight %}

封裝 是物件導向的原則之一

### Encapsulate Collection

在檢查是不是對於Collections也都做好了封裝

![Alt text]({{ site.url }}/public/encapsulate-collection.png)

奇怪了 這個跟剛剛的不是一樣嗎? 就寫好getter/setter就好啦

{% highlight java %}
class Person{
    List<String> phone;
    void setPhone(List<String> p) {
        phone = p;
    }
    List<String> getPhone(){
        return phone;
    }
}
{% endhighlight %}

我的老天鵝啊 千萬不要寫出這樣的程式跟別人說你已經封裝好了

記不記得我們在[Effective Java Item50 - 必要時進行保護型拷貝](/2017/09/26/make-defensive-copies-when-needed/)提到 因為如果你getter回傳的是這個Collection本身的話 那使用者可以任意新增刪減你的Collection而你毫不知情

使用者在`getPhone`之後 可以任意刪減 然後甚至不需要set回去 你的`phone`就已經被改掉了

所以你的getter應該變成這樣 就是保護性拷貝

{% highlight java %}
List<String> getPhone(){
  return new ArrayList<>(phone);
}
{% endhighlight %}

你也可以激進一點 

{% highlight java %}
List<String> getPhone(){
  return Collections.unmodifiableList(phone);
}
{% endhighlight %}

完全不給改 一想改就會出現`UnsupportedOperationException`

getter搞定了不是就沒事了 我們也不希望setter可以隨便就把原本的全部替換掉 
我們希望一次只能加一個或減一個元素

{% highlight java %}
void addPhone(String s) {
  phone.add(s);
}
void removePhone(String s) {
  phone.remove(s);
}
{% endhighlight %}

## 解法 - 把重構之前的使用者呼叫改掉

讓所有使用者都必須使用剛剛封裝好的getter/setter




