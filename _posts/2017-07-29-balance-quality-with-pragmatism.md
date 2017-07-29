---
layout: post
title: Effective Engineer - Balance Quality with Pragmatism
comments: True 
subtitle: 質跟量的平衡
tags: effectiveEngineer
author: jyt0532
---

這篇是Effective Engineer - Balance Quality with Pragmatism章節的讀書筆記

當作者在Google的時候 Google對於code的quality要求非常高 排版什麼的都非常的一致 這讓code非常好讀也非常好maintain 但很多人最後都跳槽去startup 因為覺得在Google開發速度太慢 時間應該花在新code上面 還是應該讓舊code更好讀 **找平衡點這件事情 本身就是一個high-leverage的Task**

### 建立code preview的process

讓Code品質高的作法中最重要的 就是要有code review 每一個commit在被push之前 要確保至少有一個engineer review過 好處如下:

1.及早抓到bug或是design的缺陷: bug上了production之後才發現會花的時間比一開始就抓到還浪費非常多 **為了省review的時間 之後就會花時間在找production的bug** 2008年有個研究顯示有code review比沒有code review減少了85%的bug

2.增加engineer的責任感:有人在看你的code你就不會亂搞

3.好的code會被學習: review別人的時候 學習別人的好處 被review的時候 被給的每個comment都是一個進步的機會

4.讓其他人知道你在做什麼: 確保至少有一個人看過知道你在做什麼 之後你不在的時候還有人可以接手

5.高品質的code容易讀 容易改 iteration的speed就會變快

### 多樣的code review方式
但要不要code review不是兩個極端的選擇 有很多中間地帶可以選 以下提供幾個例子:

早期的Instagram: 跟你的reviewer share螢幕 然後講給他聽

Square跟Twitter: pair programming

早期的OOyala: 只看核心功能的困難部分 他們還在post-commit之後review 就是code已經上去了 才開始看 就是為了加快iteration速度

Quora:只review model跟controller, 每個commit優先度不一樣 比如說碰到infra的就要仔細看 new hire的commit也要認真看

現在其實市面上很多review的tool 也有很多tool可以直接看你的unit test coverage跟coding stylecheck

怎麼樣的code review process適合你們的team 需要不斷做實驗 通常隨著team size跟code size不同 通常review方式會不一樣

### 利用抽象化 簡化複雜度

> Procedural abstraction=> function development should separate the concern of **what** is to be achieved by a function from the details of **how** it is to be achieved.
。
>
>Data abstraction => separation of the logical view of a data object (what is stored) from the physical view (how the information is stored).
>
>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;《Using Abstraction to Manage Complexity》


作者提了MapReduce當做例子 在Google他只要花半小時 就可以寫一個C++的程式去爬billions的網頁 這是因為MapReduce把內部的實作抽象化了 你只需要在乎你的application logic 剩下的什麼reliability或fault tolerance你不用管

MapReduce的例子讓我們知道 只要正確地把一件事情抽象化 可以很大程度的放大我們的output

正確的抽象一個複雜的問題 可以讓我們

1.降低複雜度: MapReduce把一個大數據的處理問題簡化成兩個 map: 轉化input reduce:把轉化的data aggregate

2.減少未來維護成本: 因為你實際需要寫的code變少 理解容易

3.只需要解決一次複雜的問題(怎麼build MapReduce framework) 可以應用在無數的小問題 DRY(Don’t repeat yourself) 好的抽象可以分離出所有很常被共用但又很複雜的細節 不用之後處理每個問題都要再解決一次

好一點的公司 都有專門的team去開發這些tool

Google - Protocol buffer: encode structured data in a extensible way, Sawzal: simplify distributed logs processing, BigTable: store and manage terabyte of structured data

Facebook - Thrift: support cross-language service development, Hive: support relational queries over semi-structured data, Tao: simplify graph queries on top of MySQL 

這些tool可以讓大家開發新feature的時間從幾個月變成幾天

但比invest tool更重要的事情是 **invest invest tool這件事划不划算** 作者拿Asana公司舉例 他們曾經想開發一個猛語言 可以讓之後的工程師花更少時間上手 但花了太多時間 原本的重點product沒有預期出來 那個project也被迫中斷 

作者給的建議是 不要太早就開始抽象化 這樣你手上的example太少 有可能抽象完後會太overfit 根本很少case可以適用 不好的抽象不是浪費開發tool的時間而已 他還會浪費之後所有用它的人的時間

<br>
> 過早最佳化是萬惡的根源&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; -- Donald Knuth

<br>

一個好的tool應該要有什麼條件呢

1.很好用

2.即使沒有document也很好用

3.很難用錯

4.強大 可以滿足需求

5.很好擴展

想build好一個tool需要花費時間心力 以下有幾點建議

1.去看你工作環境的codebase的tools 讀document 讀source code 試著擴展

2.去看大公司的open source project 看看為什麼Protobuf Thrift Hive對他們公司那麼重要

3.讀好的API 像AWS的api 為什麼那麼多人那麼輕易就上手

### 自動化測試 

我們需要寫好unit test跟integration test因為

1.手動測試很麻煩 基本上一個事情需要重複做兩次以上 就應該自動化 何況是需要做無數次的test

2.每次有新feature或是refactoring的時候就不用怕舊的功能壞掉

3.測試寫得好 當有事情發生 可以馬上找到到底錯在哪裡 節省時間

4.test code本身就是document 新來的可以由test看出這code怎麼用

### 償還技術債

在軟體開發的過程 無可避免地會有很多時候急就章 比如說某個test先不寫 或是copy paste某部分的code而不是refactor 每一個決定不論是因為偷懶還是product的deadline壓力 都會加重我們的技術債 **指的就是那些被拖延但是是必須完成的事**

有債務就會有利息 如果你不快點償還 你之後就會越花越多時間在還利息 當然 就像之前每個主題所說 這不是個兩極的選擇 還或不還 你可以先從那些利息高的開始還起 比如說最接近你核心的code 一天會有很多人看的code 那些相關技術債先還 再還比較不重要的 

大多數的人都是遇到需要還技術債的時候 才會開始還 但effective engineer會規劃它的時間 從投資報酬率最高的開始
