---
layout: post
title: 測試替身(1) - 何謂測試替身
comments: True 
subtitle: test doubles(1)
tags: unitTest
author: jyt0532
---

寫過測試的人都知道什麼是Mock
但其實Mock是很多東西的統稱 包含了
Dummy Mock Fake Stub Spy

而不同語言或不同framework有時候會把類似的概念合在一起

本系列的目的是讓你寫單元測試的時候 對應不同情況 知道應該用哪一種替身 在學習任何一個unit test的framework之前 先弄清楚哪些被併在一起了 哪些是你手上可以用的武器

![Alt text]({{ site.url }}/public/the_prestige.jpg)

### 專業術語

SUT: System under test 就是需要被測試的東西

DOC: Depended On Component 就是SUT需要依賴的東西

DOC非常常見 幾乎無可避免

比如說SUT是web server 那DOC就是database

比如說SUT是web fronted 那DOC就是web server

你不太可能每個函數都自己玩自己的 你通常都會需要呼叫別人的函式 但這其實對測試帶來了負擔 比如說你每次想測試你的webserver可不可以新增使用者的時候 你都需要真的去database叫他加一個給你 

這實在開銷太大 也非常不實際

## Test Doubles的目的

1.第一個也是最重要的一個 是隔離你的SUT 不被任何DOC干擾

我不想要我測試新增使用者的時候 還要保證database是正常的 我任何時候都想跑測試 不依賴任何人

2.加速執行時間 避免不必要開銷

不依靠他人之後 所以你需要的DOC的回傳值都先定義好  當然加快了執行速度

3.讓你的測試deterministic

我不想要在不同時間或不同空間裡 會得到不一樣的測試結果 
比如說 尖峰時刻 database load太大 回傳了不預期的Http status 429
這是我不想在我預期要是happy case的情況看到的case

4.模擬特殊狀況(special case) 

比無法測試happy case的情況還要慘的事情 是無法測試bad case

這點也很重要 如果真的遇到了不預期的狀況(比如剛剛說的429) 最慘就是等一陣子 database正常後 就可以過了

但有時候我們就是想要知道 當回傳429的時候 我們處理的方法是不是正確
如果沒有test doubles 根本無法保證這種狀況一定發生 
也不可能去DDOS自己的database製造這種情況

5.可以讓你測試到你不想公開的資訊

這點就厲害了 來個例子:

SUT是WebServer DOC是database
{% highlight java %}
public class WebServer {
  private Database database;
  public void create(){
    database.insert()
  }  
}
{% endhighlight %}
比如說這樣 我想知道我webser.create 的時候 database.insert的確被呼叫 要怎麼測試？
我不想要開放一個public function getDatabase 供大家存取database的**狀態**僅僅只是為了測試用途

該怎麼辦呢 這時候來一個漂亮的替身
{% highlight java %}
public class WebServerTest{
  @Test
  public void databaseInsertedWhenServerCreate(){
    TestDatabase testDatabase = new TestDatabase();
    WebServer webserver = WebServer(testDatabase);
    webserver.create();
    assertTrue(testDatabase.isInserted)
  }
}
public class TestDatabase extends Database{
        private boolean isInserted
        public void insert(){
                isInserted = true;
         }
        public void isInsert(){
                return isInserted;
         }
}
{% endhighlight %}

注意:

1. TestDatabase 需要extends Database 不然丟不進Webserver的constructor

2. 原本的Database這個Class沒有isInsert這個函式 是我自己加的

搞定 prod上的Database完全不用動 我就可以知道當webserver.create被call的時候 我的的確確呼叫了database.insert

## 總結

看完了為什麼要test double之後 之後會一一介紹每個測試替身的使用時機跟用法

