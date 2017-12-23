---
layout: post
title: 測試替身(4) - Mock
comments: True 
subtitle: test doubles - mock
tags: unitTest
author: jyt0532
---

我們學會了用STUB來獨立測試SUT 可是我們用STUB並無法確定我們預期要被call的function是不是真的被call了 或是被call了幾次 這時候就需要一個專業一點的mock來幫忙

## 範例

用[STUB](/2017/12/18/test-double-stub/)的例子
{% highlight java %}
Database databaseStub = mock(Database.class);
when(databaseStub.authorize(anyString(), anyString()))
    .thenReturn(true);
WebServer webserver = new WebServer(databaseStub);
assertEquals(42, webserver.getSecretNumber("BoYu", "jyt"));
{% endhighlight %}

很好 我們stub了databaseStub.authorize 我們告訴我們的程式說 Hey 如果有人這麼呼叫你 你就回傳true

那要是今天有個新手手滑改到了code

{% highlight java %}
public class WebServer {
  private Database database;
  int secretNumber;
  public WebServer(Database database){
    this.database = database;
    secretNumber = 42;
  }
  public int getSecretNumber(String username,String password){
    return secretNumber;
  }
}
{% endhighlight %}

不好意思 unit test會過 因為你的unit test並沒有保證一定要有人**必須要**去呼叫databaseStub.authorize

## 該怎麼用mock

我們定義一個跟DOC一樣的介面 然後 把這個DOC裡面預期會被呼叫的函式都先定義好input argument跟return value
然後把這個Mock替身丟給SUT去取代真正的DOC

在unit test**正在跑**的時候
我們可以用mock確認 如果某個非預期的方法被呼叫 就噴錯 
或是這個mock某個預期會被call的方法收到了跟預期不一樣的參數就噴錯


Mock注重的是對於行為的測試 所以你再跑一個unit test之前 你就已經要很清楚這個SUT會call到DOC的**每一個方法的每一個參數** 

## 注意

1.這是唯一一個測試替身裡面有assertion的替身 因為只有mocked object知道他收到的東西正不正確

2.在單元測試的最後 你需要去verify你預期要被call的方法真的被呼叫了 就像本篇的第一個例子

## 所以本篇的範例該怎麼寫

{% highlight java %}
Database databaseStub = mock(Database.class);
when(databaseStub.authorize(anyString(), anyString()))
    .thenReturn(true);
WebServer webserver = new WebServer(databaseStub);
assertEquals(42, webserver.getSecretNumber("BoYu", "jyt"));

verify(databaseStub).authorize("BoYu", "jyt");

{% endhighlight %}

所以那個新手改的code在這個測試程式的最後 verify那一行就會噴錯

## 總結

當你的SUT有一些indirect output(間接的output) 你在測試之前就知道你的DOC應該要接受到什麼值
那就用mock
