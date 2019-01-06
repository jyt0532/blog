---
layout: post
title: Designing Data-Intensive Application - Data Models and Query Languages
comments: True 
subtitle: 數據模型與查詢語言
tags: systemDesign 
author: jyt0532
excerpt: 本篇文章介紹何謂數據模型與查詢語言????
---

這是Designing Data-Intensive Application的第一部分第二章節: 數據模型與查詢語言

本文所有圖片或代碼來自於原書內容

## 數據模型與查詢語言

數據模型(Data Model)大概是軟件開發中最重要的部分 你選擇的數據模型不僅影響著軟件的編寫方式 也影響著如何思考並解決問題

大多數的應用 使用層層疊加的數據模型來構建 舉例來說

第一層:身為應用開發人員 你對於你的應用選擇了適當的對象或是資料結構來存儲 並定義了適當的API來操作(CRUD)這些資料結構

第二層:存儲資料結構時 你用更廣義的data-model 像是JSON或是XML文件 關係數據庫中的table等等

第三層:Database的工程師可以決定要如何以內存,硬碟或網路上的bytes來表示這些Json/Xml/relational/graph數據 選擇正確的表達形式可以讓數據能被查詢 搜索 處理

第四層:硬件工程師可以使用電流 光脈衝 磁場或是其他東西來表示bytes

再複雜一點的應用可能會有更多層 比如API的API 不過基本思想一樣 透過一個明確的data model來隱藏低層次的複雜性 這些抽象允許了不同層級的合作(Database工程師 應用工程師 硬件工程師彼此合作)

數據模型種類很多 要掌握任何一個都需要花費很大心力 但正確的選擇可以讓你不論是對上層還是對下層都更方便地溝通 
本章節會講解一系列用於數據存儲和查詢的通用數據模型(第二層) 特別是比較relational model vs document model vs graph based model 
第三章則會討論搜尋引擎如何運作 還有數據模型是如何實現(第三層)

## Relational Model vs Document Model

眾所皆知的著名Data Model就是SQL 數據被組織為relations(tables) 而relation是由unordered tuples所組成 這個模型已經稱霸軟件界30年 算是極其穩定的設計 當時其他的數據模型迫使應用開發人員需要思考數據庫內部的數據表示形式 但relational model把上述細節隱藏在更姐單的接口之後 造就了它的成功

### NoSQL的誕生

2010年開始 NoSQL試圖推翻關聯模型的領導地位 NoSQL全名是Not only SQL 優勢如下

1.比關係數據庫更好的可擴展性 應付大數據以及高吞吐量

2.免費和開源軟件更受偏愛

3.關係模型不能很好地支持一些特殊的查詢操作

4.更具多動態性(dynamic)與表現力(expressive)的數據模型

不同的應用程序有不同的需求 甚至關聯模型也可以混合著非關聯數據庫一起使用 稱為混合持久化(polyglot persistence)

### 對象關係不匹配 Object-Relational Mismatch

現在大多數的應用都是使用面對對象(object-oriented)的語言來開發 這也造成了一些對於SQL的批評: 我們需要一的笨拙的轉換層 在每次讀寫的時候轉換

Object in application code <-> database rows and columns

模型之間的不連貫有時被稱為[阻抗不匹配(impedance mismatch)](https://en.wikipedia.org/wiki/Object-relational_impedance_mismatch)

以下是用Relational DB表示比爾蓋茲的履歷

![Alt text]({{ site.url }}/public/DDIA/DDIA-2-1.png)

這份履歷可以被一個唯一的標識符`user_id`來識別 但是大部分的人在職涯中會有超過一分的工作 所以這其實存在著一對多的關係 有幾種解法

1.以前的SQL(before 1999) 就是全部normalize 職缺一個表 教育程度一個表 聯絡資訊一個表 每個表上的row都有user_id 你需要的時候再join

2.比較新的SQL 可以支援**結構化數據類型**或XML或Json 這讓你一個row裡面可以有很多data 支持XML的有Oracle, IBM DB2, MS SQL Server, PostgreSQL 支持JSON的有IBM DB2, MySQL, PostgreSQL

3.第三個方法 就是把所有資訊存在一個JSON或XML document 然後存在數據庫中 讓應用去自行解讀Json的結構跟內容 只是這個做法通常不能search document裡面的值

來看看第三個方法 對於一個像簡歷這種本身就是文檔的數據結構而言 JSON表示是非常合適的
(支援Document的數據庫有MongoDB, RethinkDB, CouchDB, Espresso等等)

{% highlight json %}
{
  "user_id": 251,
  "first_name": "Bill",
  "last_name": "Gates",
  "summary": "Co-chair of the Bill & Melinda Gates... Active blogger.",
  "region_id": "us:91",
  "industry_id": 131,
  "photo_url": "/p/7/000/253/05b/308dd6e.jpg",
  "positions": [
    {
      "job_title": "Co-chair",
      "organization": "Bill & Melinda Gates Foundation"
    },
    {
      "job_title": "Co-founder, Chairman",
      "organization": "Microsoft"
    }
  ],
  "education": [
    {
      "school_name": "Harvard University",
      "start": 1973,
      "end": 1975
    },
    {
      "school_name": "Lakeside School, Seattle",
      "start": null,
      "end": null
    }
  ],
  "contact_info": {
    "blog": "http://thegatesnotes.com",
    "twitter": "http://twitter.com/BillGates"
  }
}
{% endhighlight %}

JSON表達式比RMDB的多table格式具有更好的局部性 比如說你想獲取簡介 你在RMDB裡需要再另外Join 在這裏我所有東西都在document裡了

剛剛一對多的問題也在JSON格式中很好的被處理

![Alt text]({{ site.url }}/public/DDIA/DDIA-2-2.png)













