---
layout: post
title: 整潔的架構
comments: True 
subtitle: 整潔的架構
tags: softwareArchitecture
author: jyt0532
excerp: 本文介紹整潔的架構
---

這篇文章討論 Clean Architecture 裡面集大成的一個章節 The Clean Architecture

圖片以及程式碼來源自[Clean Architecture](https://www.amazon.com/Clean-Architecture-Craftsmans-Software-Structure/dp/0134494164)以及[Uncle Bob的Blog](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)


## 整潔的架構

開宗明義 好的架構 必須要把關注的點分離(separation of concerns) 怎麼做到呢 藉由分層 比如說Business Rule一層 使用介面一層 等等

這樣有什麼好處呢??

1.獨立於框架: 這樣的架構並不依賴於任何一個功能強大的框架 對你而言 框架只是工具

2.可測試: 業務規則能夠在沒有UI 沒有數據庫或沒有Web服務器的情況下被測試

3.獨立於UI: 不必改變系統的其餘部分的情況下 可以輕鬆改動UI

4.獨立於數據庫: 你能夠在Mongo/BigTable/CouchDB之間切換 你的業務規則不會和數據庫綁定

5.獨立於任何外部代理: 你的業務規則可以對其外面的技術世界毫無所知


### 上圖

講了那麼多優點 一個理想的架構圖如下:

![Alt text]({{ site.url }}/public/clean-architecture-1.png)


同心圓的越內部層次越高 所有的依賴關係只能往內指 

內部沒有任何東西可以或需要對於外部的情況有任何的了解 包含函式名稱 類別名稱 變數名稱等等

現在我們來一層一層討論一下

### 實體層 Entities

實體層封裝的是企業的業務規則 

我們試著不要把問題想的那麼複雜 一個Entity可以是一個帶有方法的物件 或是一堆Data Structure 只要這些東西會被企業中的眾多應用使用都算

如果我們根本沒有到企業規模也沒關係 即使我們只是一個應用程式 那這層的東西就是應用程序中的business objects 比如說Messenger的訊息 或是臉書的Post 

即使外部發生變化(比如說安全性變更或是UI更動) 這一層的東西都不該被更動

### 使用案例層 Use Cases

在這個層的軟體包含應用指定(application-specific)的業務規則 這一層包裝並且實作了所有使用案例 比如一個Post可以被reply 或是Message可以被修改 等等的業務規則

這一層的改動不應該影響實體層 也就是如果今天你想要所有Post都不可以被reply 那實體層不應該被變動

當然 Database或是UI的外圍改動也不該影響到這一層 

### 介面轉接層 Interface Adapters

這一層主要是一堆Adaptor 可以把資料的格式從 **最適合use case層使用的資料格式** 跟 **最適合外部代理(Web或資料庫)使用的格式** 彼此互換

### 框架和驅動層 Frameworks and Drivers

最外面一圈通常是由一些框架和工具組成 比如說資料庫或網頁框架


### 只有四層？

當然不一定 也許你的規模可以只用兩層 也可以用超過四層 只要每一層的依賴關係都是由外往內就可以

### 跨邊界


![Alt text]({{ site.url }}/public/crush-1.gif)


架構圖的右下方說明了如何跨越邊界 

![Alt text]({{ site.url }}/public/clean-architecture-2.png)

從Controller往內呼叫使用案例層 在這層做完了該做的事之後 由Presenter呈現結果

注意 箭頭的方向都是往內指 至於怎麼往內指 就是我們熟悉的DIP 

你說你忘了 沒關係我再幫你複習一次 原本我在使用案例層做完了我該做的事(比如說一個Post的按讚數+1 或是有人回覆的話回覆數要加一等等的use case) 我要怎麼Render出來呢? 我們知道要是往外呼叫就破壞了依賴關係 內部不應該知道怎麼往外呼叫

所以我們需要一個可愛的抽象 在圖中叫做`Use Case Output Port` 然後`Presenter`實作這層抽象

這個就是我們跨邊界的方法 在任何兩個相鄰的層都一樣


### 哪些東西可以跨邊界

那我們要用什麼東西來互傳呢 通常都是簡單的資料結構 也許是一個struct 也許就是一個方法的參數 這個結構要是內部那層看得懂的

為什麼呢 因為要是這是外部才看得懂的(比如說SQL的Database row) 內部又被逼著要轉換 這個轉換本身也代表著內部依賴外部 因為如果外部從SQL換成Mongo 內部也要跟著改

所以不管是內往外還是外往內 傳的東西都得是**對於內部最方便的資料結構**

## 建議搭配

推薦一下這篇文章 [架構整潔之道導讀（四）第25章層次與邊界-圖 25.3 疑惑澄清](https://www.jianshu.com/p/e442cfdd9a1a) 

這篇是本書譯者寫的文章 關於第25章的層次和邊界 的問題 譯者還寄信問Bob相關問題 Bob回答得非常清楚 簡單來說 我們都知道需要定義抽象介面來讓內外層溝通 但這個抽象介面本身該放在哪裡呢? 

答案是不論是InputBoundary還是OutputBoundary 都應該放在內層裡

## 總結

只要遵守這些簡單的規則 

1.將軟體分層

2.外層依賴內層 

就是整潔的架構





