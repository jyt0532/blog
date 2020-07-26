---
layout: post
title: Bend or Break - 彎曲或弄壞
comments: True 
subtitle: Bend, or Break
tags: pragmaticProgrammer
author: jyt0532
excerpt: 本文記錄了讀完The Pragmatic Programmer的讀後筆記
---

千呼萬喚 終於等到[Pragmatic Programmer 20週年紀念版](http://books.gotop.com.tw/v_ACL057200) 如果沒聽過這本書 你大概也聽過[程序員修煉之道︰從小工到專家](https://www.books.com.tw/products/CN10279423)這本暢銷了20年的書 終於等到了再版 

在再版裡面 刪掉了比較過時的內容和範例 收集了20年來收到的feedback 在讓這本書的內容也可以適用於2020年的程序員 但在我細細品嚐後發現 其實很多人生的哲學並不是只適用於程序員 各行各業看了都可以有所收穫

因為每個篇章的篇幅都不長 所以筆記也用條列式紀錄

本篇的圖片以及程式碼來自於原書內容

{% include copyright.html %}

# 第五章: 彎曲或弄壞

> 因為需求一直在變 所以你要馬讓你的程式有彈性而彎曲 或是彈性不足而直接被壓壞


## 去耦合

講的內容都跟[之前提的](/2020/03/27/coupling/)差不多 本書將稍微提到三個主題

1.方法呼叫鍊

2.全域化

3.繼承

這三個都是不被推薦的東西

### 方法呼叫鍊

我們在[重構 - 改善既有程式的設計 - Message Chain](/2020/04/17/message-chains/)討論過了 看那篇就可以

### 全域化

全域可存取的資料是應用程式元件之間耦合的潛在來源 每多一個全域變數 這代表**所有類別 所有方法都多了一個參數**

要特別注意 [Singleton](/2017/05/19/singleton/)也是全域變數的一種

### 繼承

繼承是所有依賴關係中 耦合性最強的 能免則免 我們會在[繼承稅](#繼承稅)中細談

## 在江湖走跳

### 事件

一個事件代表了一份可用的資訊 可能來自外部世界(使用者按一下按鈕 股票報價更新) 也可能來自內部(計算結果出來了 搜尋完成了) 甚至可以簡單到類似 "取得了list的下一個元素"

不管來源是什麼 我們應該撰寫回應事件的應用程式 並根據這些事件調整應用的行為

有四種策略可以有效的應對事件

1.有限狀態機(Finite State Machine)

2.觀察者模式(Observer Pattern)

3.發佈/訂閱(Publish/Subscribe)

4.回應式程式設計(Reactive Programming)和串流(Stream)

### 有限狀態機

就是由一堆狀態跟一堆動作所定義的處理事件的規範

![Alt text]({{ site.url }}/public/fsm.png)

從初始狀態開始 如果接收到 `header` 事件 我們就跑到 `Reading message`狀態 接收到`trailer`事件 就跑到`Done`狀態

更多內容可以參考[State Pattern](/2017/05/30/state/)

### 觀察者模式

請參考[Observer Pattern](/2017/04/12/observer/)

### 發佈/訂閱

Publish/Subscribe 其實就是更廣泛的觀察者模式 而且同時解決了耦合和性能問題

在這種模式中呢 發佈者和訂閱者是通過 channel 連接 那channel是怎麼實作的呢 這個實作本身跟發佈者或訂閱者獨立 可以是個函式庫 可以是個程序 可以是個分散式架構 但這些實作的細節都是被隱藏的

每個channel都有名字 訂閱者在一個或多個命名 channel 中註冊興趣 發佈者向其寫入事件 和觀察者模式的差別有兩個

1.發佈者和訂閱者之間的通信是在代碼外部處理的

2.可以是異步

### 回應式程式設計(Reactive Programming)和串流(Stream)

可以參考[DDIA的討論](/2019/07/14/stream-processing/)

### 事件無所不在

顯而易見 真實生活中的事件無所不在 本章節討論了四個 **圍繞著事件**來寫程式的方法 幫助你更容易開發 更容易去耦合以及更好的性能

## 轉換式程式設計 Transforming Programming

> 如果你不能夠把你這在做的事情描述成一個處理流程 那表示你不知道自己在做什麼

所有的程式都會做資料轉換 把輸入轉成輸出

想像一下我們想要列出目錄樹中擁有最多行數的五個檔案

你覺得一個工程師應該要寫幾個檔案 要花多少時間呢 寫C嗎 寫python嗎

不對 寫bash

{% highlight bash %}
> find . -type f | xargs wc -l | sort -n | tail -5
{% endhighlight %}

這個命令執行了一系列的轉換

### find . -type f

將當前目錄(.)中或其下的所有文件(-type f)的列表寫入標準輸出

### xargs wc -l

從標準輸入中讀取行並將它們作為參數傳遞給命令 wc -l (帶有 -l 選項的 wc 程序計算每個參數中的行數) 並將每個結果以 行數檔案名稱 寫入標準輸出

### sort -n

對標準輸入排序 假設每行以數字(-n)開頭 將結果寫入標準輸出

###  tail -5 

讀取標準輸入並將最後5行寫入標準輸出

### 執行看看

在本書的目錄中執行指令 會得到如下結果

{% highlight txt %}
470 ./test_to_build.pml 
487 ./dbc.pml
719 ./domain_languages.pml 
727 ./dry.pml
9561 total
{% endhighlight %}

哎呀 原來wc會在最後一行列出總共的行數 所以扣除最後一行我們只拿到了四個檔案

所以應該先拿最後六個 再拿前五個

{% highlight bash %}
> find . -type f | xargs wc -l | sort -n | tail -6 | head -5
{% endhighlight %}

好 現在來圖解一下我們的pipeline

![Alt text]({{ site.url }}/public/chapter-5-pipeline.png)

就像是工業中的裝配線一樣 由許多包含著輸入以及輸出的元件組成

我們喜歡這樣子思考所有的代碼

> Programming Is About Code, But Programs Are About Data

### 為什麼喜歡這樣呢

我們曾在[使用Unix工具的批處理](/2019/11/24/batch-processing/#使用unix工具的批處理)討論過Unix的哲學 有興趣的話可以回去複習一下 

在這裡提到轉換式程式設計的原因 是因為如果你有物件導向的背景 那你的直覺會告訴你要隱藏資料 把資料封裝在物件裡面 然後藉由物件彼此的互動改變彼此的狀態 

可想而知 這樣導入了很多的耦合 這也是物件導向的程式難以修改的重要原因

> 不囤積狀態 把狀態傳播出去


## 繼承稅

我們來複習一下[SOLID的L](/2020/03/22/lsp/) 也就是 當你要使用繼承的時候要非常小心 大多數的情況你都不該用繼承

再說一次 繼承是所有依賴關係中 耦合性最強的

那不用繼承的話 有什麼其他方式可以選擇呢

1.interface(介面)和委派protocal(協定)

2.delegatiom(委派)

3.mixin(混合)和trait(特性)

### 介面和協定

用介面取代類別 會靈活很多

{% highlight java %}
public class Car implements Drivable, Locatable {
  // Code for class Car. This code must include 
  // the functionality of both Drivable
  // and Locatable
}
{% endhighlight %}

這樣就可以把所有有實作Locatable的東西擺在一起

{% highlight java %}
List<Locatable> items = new ArrayList<>();
items.add(new Car(...)); 
items.add(new Phone(...)); 
items.add(new Car(...));
{% endhighlight %}

也可以呼叫同樣的函示

{% highlight java %}
void printLocation(Locatable item) { 
  if (item.locationIsValid() {
    print(item.getLocation().asString()); 
  }
}
{% endhighlight %}

> 用介面來表達多型

### 委派

如果一個父類有20個方法 即使子類只需要其中的兩個 繼承逼得子類的物件仍然擁有剩下不需要的18個方法 提過不少次了 可以參考[復合優先於繼承](/2018/05/05/favor-composition-over-inheritance/)

### 混合和特性

不同語言有不同名稱 mixin, trait, categories, protocol extensions等等

我們又想使用已經有的功能 可是又不想繼承 怎麼辦呢 就用trait 

用php舉例 來自[W3School](https://www.w3schools.com/php/php_oop_traits.asp)

{% highlight php %}
<?php
trait message1 {
  public function msg1() {
    echo "OOP is fun! ";
  }
}

class Welcome {
  use message1;
}

$obj = new Welcome();
$obj->msg1();
?>
{% endhighlight %}

你只要`use` 就可以用了 精美 不用繼承 就是直接使用你想要使用的部分

如果繼承是Is-a, 委派是has-a 那特性就是-able



## 設定(Configuration)

若程式碼需要的一些值可能會因為應用程式執行的環境而改變的話 就讓這些值存在應用程式的外部 這樣你的應用程式就可以適用在不同的環境中 給不同的客戶使用

> 使用外部的Configuration來參數化你的應用

有哪些東西可以放進Config呢

1.外部服務(DB, API)的帳號資訊

2.log存儲目的地

3.應用程式的IP, Port, Cluster

4.適用於執行環境的驗證參數

5.外部設定的參數 比如稅率

6.輸出格式的細節

7.授權金鑰

### Configuration-As-A-Service

把設定本身當作是一個服務 這樣有許多好處

1.多個應用可以共享設定資訊 

2.利用身份控制或是存取控制去限定每個應用程式可以看到的內容 還有哪些人有權限修改設定

3.可以有專門UI去存取資料

4.可以動態設定資料

第四點是最重要的 因為現在的服務都講求可用性 你不想要只是改個參數 就要把你主要的應用重新啟動(去重新讀取某個txt檔) 

應該讓Config service去告訴你的應用哪些參數被更新了




