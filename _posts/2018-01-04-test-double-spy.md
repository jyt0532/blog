---
layout: post
title: 測試替身(5) - Spy
comments: True 
subtitle: test doubles - spy
tags: unitTest
author: jyt0532
---

我們學會了用STUB來獨立測試SUT 可是我們用STUB並無法確定我們預期要被call的function是不是真的被call了 或是被call了幾次 這時候就需要一個專業一點的間諜來幫忙

Spy就是一個有記錄功能的Stub
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

不好意思 unit test會過 因為你的unit test並沒有保證一定要有人去呼叫databaseStub.authorize

## 怎麼跟mock長得一樣

接下來的部分是本系列最重要的內容 也是我寫這系列的主要原因

我查閱了非常多網路上的資料 大多數的人對於mock跟spy都不清楚 因為功能太過接近 

有時需要的是mock卻用成spy 有時需要的是spy但卻不知道怎麼從mock做到

接下來會說明我歸納出來的總結和工作上遇到的使用時機 可以的話可以開出[Mock](/2017/12/23/test-double-mock/)來比較一下

## 該怎麼用spy

你可以把spy想成是一個偷偷**紀錄**呼叫過程的STUB

我們可以定義一個被spy的物件 某個函式被呼叫的時候 回傳某個值

而且在測試函式的最後 可以問spy說哪個method被call了幾次 是給入哪些參數

## 那跟mock差在哪

1.當你在測試前 你就有辦法知道DOC預期會收到哪些呼叫 伴隨哪些參數 那你就要用mock
反之 如果某些參數 你無法在測試前知道 你就只能用spy 然後在測試的最後確認說那個函式有沒有被call

比如說你的server自己有個亂數產生id的方法 而這個亂數id會拿來call database的insert
那你在測試前就不知道你的databaseMock 會預期收到什麼值 

因為mock是針對一個物件的**行為**去寫測試(behavior verification)

而spy是針對物件的**狀態**(state verification)去檢查狀態是否正確

2.mock裡面可能有很多函式 我們可以事前定義會被呼叫的某幾個函式如果參數是什麼就回傳什麼 那其他沒有被定義的 就是回傳null 就是根本就不該被呼叫 一有人呼叫就噴錯 但如果是用spy 沒有被另外定義的 他就用原本的函數

講一堆四書五經 來點code吧

首先看一下正常的狀況
{% highlight java %}
public class SpyAndMock {
    @Test
    public void testStubbingMock() {
        List<String> mockList = mock(ArrayList.class);
	when(mockList.get(100)).thenReturn("HIHI");
	assertEquals(mockList.get(100), "HIHI");
    }
    @Test
    public void testStubbingSpy() {
	List<String> spyList = spy(ArrayList.class);
	doReturn("HIHI").when(spyList).get(100);
        assertEquals(spyList.get(100), "HIHI");
    }
}
{% endhighlight %}
當然對於spy或是mock 都可以給他們加上Stubbing 代表說我可以寫死這個物件的某個函式被呼叫的時候 
該回傳什麼

這點沒問題 從這點來看 這兩個替身長得一樣

現在來對這兩個Stubbing一下
{% highlight java %}
public class SpyAndMock {
    @Test
    public void testMock() {
        List<String> mockList = mock(ArrayList.class);
	mockList.add("test");
        assertNull(mockList.get(0));
	assertEquals(mockList.size(), 0);
    }
    @Test
    public void testSpy() {
	List<String> spyList = spy(ArrayList.class);
        spyList.add("test");
        assertEquals("test", spyList.get(0));
        assertEquals(spyList.size(), 1);
    }
}

{% endhighlight %}

對於mock來說 沒有預先定義的函式 全部都回傳預設值

對於spy來說 沒有預先定義的函式 全部都按照正常程序

所以你可以說spy是partial mock 

我比較喜歡把spy想成是 真的生了一個要測試的類別的物件 然後把這個物件包在一個wrapper裡面 監視一舉一動 除非必要不打草驚蛇(stubbing a spy)

mock就是生了一個跟要測試的類別長得很像的物件 只是所有函式回傳預設值 再把會用到的函式輸入輸出定義一下

spy就是偽君子岳不群 mock就是真小人左冷禪

## 使用時機

其實大多數的情況 你都應該用mock 應該是說你能用mock解決的就用mock 可是萬一下面幾點有一點符合 那就要考慮用spy

1.SUT中間跟DOC的互動時 需要的參數無法確定 

2.mock的asseretion在mock物件裡 有些assertion不明顯 你覺得測試跑完錯誤訊息不夠清楚(別忘了spy是在測試的最後 我們去檢查spy物件的狀態)

3.對assertion的equality control不夠: 因為很多時候 assertEqual對於equal的定義我們可能想要customize 
那如果你assertion發生在mock裡面 就無法做到這件事

## 為什麼我會用到spy

說來也離奇 我用到spy的地方並不是用來取代DOC

且聽我娓娓道來

我現在要寫SUT的methodA1的單元測試 methodA1會call DOC的methodA2
但他同時也會call SUT自己的methodB1

![Alt text]({{ site.url }}/public/spy1.png)

這裡的methodB1的測試已經寫好了 因為methodB1只depend DOC的methodB2 很好寫 簡單的mock

但methodA1就麻煩了 除了mock methodA2
{% highlight java %}
DOC mockDoc = mock(DOC.class);
when(mockDOC.methodA2(inputA)).thenReturn(retA);
{% endhighlight %}

我還必須mock methodB2
{% highlight java %}
DOC mockDoc = mock(DOC.class);
when(mockDOC.methodA2(inputA)).thenReturn(retA);
when(mockDOC.methodB2(inputB)).thenReturn(retB);
{% endhighlight %}

可是也是越看越不對 這樣子我methodA1跟methodB1 幾乎綁在一起 如果今天methodB1想call methodB3 那我的methodA1的unit test也要跟著改

這絕對是一個bad smell 可是網路上的文件翻來覆去 都沒找到滿意的解答 直到最近細看了spy的使用時機還有spy跟mock的差別 才發現從來就沒有人規定spy一定只能spy DOC

我spy自己！！

> 故用間有五：有因間，有內間，有反間，有死間，有生間。五間俱起，莫知其道，是謂神紀，人君之寶也
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;孫子兵法用間篇

說破哪值幾文錢 但其實我只要stubbing methodB1 就直接搞定

{% highlight java %}
SUT spySut = spy(SUT.class);
DOC mockDoc = mock(DOC.class);
when(mockDOC.methodA2(inputA)).thenReturn(retA);
doReturn(retAB).when(spySut).methodB1(inputAB);

//execute methodA1
spySut.methodA1();

verify(spySut).methodB1(inputAB);
verify(mockDoc).methodA2(inputA);
{% endhighlight %}



### 總結

看了網路上找得到的文獻 對於Mock跟Spy的差別都是強調於**驗證行為**vs**驗證狀態**
但這一直讓我想不通 因為我覺得所有spy能做到的事 mock都能做到 這也是大家對這兩個的概念越來越模糊的原因

直到我在工作上遇到剛剛所說的問題

我認為 區分這兩個替身最好的方式是:

mock是複製一模一樣的骨架 只stubbing你會call到的函式

spy是在一個生好的物件外面包了一層 你一樣stubbing你想要寫死的輸入輸出(這樣那些沒有寫死的函數就正常call)



