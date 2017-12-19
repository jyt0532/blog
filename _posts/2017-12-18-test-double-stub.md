---
layout: post
title: 測試替身(3) - Stub
comments: True 
subtitle: test doubles - stub
tags: unitTest
author: jyt0532
---

接下來進入正題 也是第一個可以讓我們獨立測試SUT的測試替身 -- STUB

## 使用時機

當我們需要測試一個SUT 但我們卻不想要依賴真實的DOC 我們可以用STUB去取代我們的DOC

STUB並不需要真的表現的跟DOC一樣 他只要api長得一樣(也就是輸入輸出長得一樣) 讓SUT以為是真正的DOC就可以

來個例子 今天我們想測試getSecretNumber是不是能正確回傳

{% highlight java %}
public class WebServer {
  private Database database;
  int secretNumber;
  public WebServer(Database database){
    this.database = database;
    secretNumber = 42;
  }
  public int getSecretNumber(String username,String password){
    return database.authorize(username, password) ? 
	secretNumber : -1;
  }
}
public class Database {
  public Database(){
  }
  public boolean authorize(String username, String password){
    //Connect to database
    //Query database
    //etc
  } 
}
{% endhighlight %}
在code裡面 我們需要去authorize 這一步會花費很多時間
這也不是現在這個測試的重點
這時候就來個Stub
{% highlight java %}
public class DatabaseStub extends Database {
  public boolean authorize(String username, String password) {
    return true;
  }
}
public class TestWithStub{
  @Test
  public void testGetSecretNumber(){
    WebServer webserver = new WebServer(new DatabaseStub());
    assertEquals(42, webserver.getSecretNumber());
  }
{% endhighlight %}

搞定 這樣就不用真的去query database

## Mockito

有Mockito的話 並不需要真的寫一個新的DatabaseStub

{% highlight java %}
Database databaseStub = mock(Database.class);
when(databaseStub.authorize(anyString(), anyString()))
    .thenReturn(true);
WebServer webserver = new WebServer(databaseStub);
assertEquals(42, webserver.getSecretNumber());
{% endhighlight %}

你把一個class Mock了之後 他的每一個function都只會回傳null
**你需要去指定你會用到的method的行為** 輸入值是什麼回傳什麼

## 測試特殊情況

如[介紹文](/2017/12/15/test-doubles/)所說 stub還能模擬特殊狀況

在這個例子裡 你想知道如果authorize不過 是不是回傳-1
就把剛剛stub的回傳值改成false就可以 
{% highlight java %}
Database databaseStub = mock(Database.class);
when(databaseStub.authorize(anyString(), anyString()))
    .thenReturn(false);
WebServer webserver = new WebServer(databaseStub);
assertEquals(-1, webserver.getSecretNumber());
{% endhighlight %}

還可以讓你的Stub throw exception
{% highlight java %}
when(databaseStub.authorize(anyString(), anyString()))
  .thenThrow(new NullPointerException());
{% endhighlight %}

隨便你愛怎麼玩就怎麼玩

## 總結

當你的SUT有一些indirect input(並不是在你測試的程式提供的input 而是DOC提供的input) 需要事先定義好DOC的回傳值 就是用STUB

