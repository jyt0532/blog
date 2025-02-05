---
layout: post
title: Designing Data-Intensive Application - Part3 Intro - Derived Data
comments: True 
subtitle: 衍生數據
tags: systemDesign 
author: jyt0532
excerpt: 本篇文章介紹衍生數據
---


在本書的[第一部分](/2019/01/26/foundation-of-data-systems/)和[第二部分](/2019/09/27/distributed-data/)中 我們討論了所有關於分佈式數據庫的主要考量 從數據在磁盤上的佈局 一直到出現故障時分佈式系統一致性的侷限 但所有的討論都侷限於應用中只用了一種數據庫

現實世界中的數據庫往往更佳的複雜 大型應用程序經常需要以多種方式訪問和處理數據 幾乎沒有一個單獨的數據庫可以滿足所有這些不同的需求 應用程序通常需要組合使用多種組件: 數據存儲, 索引, 緩存, 分析系統等等 並且需要實現在這些組件中**移動數據**的機制

本書的最後一部分 會研究將多個不同數據系統 集成為一個協調一致的應用架構時會遇到的問題

## 記錄系統(system of record)和衍生數據系統(derived data system)

從高層次上看 存儲和處理數據的系統可以分為兩大類:

### 記錄系統(System of record)

也就是source of truth 持有數據的權威版本 當新的數據進入時 首先會記錄在這裡

每個事件正正好好被標準化的表示一次 如果其他系統和記錄系統之間存在任何差異 那麼記錄系統中的值才是正確的


### 衍生數據系統(Derived data system)

衍生系統中的數據 通常是另一個系統中的現有數據以某種方式進行轉換或處理的結果 如果丟失衍生數據 可以從原始來源重新創建

常見的例子是緩存,索引和物化視圖

從技術上講 衍生數據是redundant的 因為它重複了已有的信息 但衍生數據會讓讀查詢更加的快速 讓你由不同的**視角**來觀察數據


&nbsp;

並不是所有的系統都在其架構中明確區分記錄系統和衍生數據系統 但這是一種有用的區分方式 因為它明確了系統中的數據流: 系統的哪一部分具有哪些輸入和哪些輸出 以及它們如何相互依賴


## 章節概述

我們將從[第十章](/2019/11/24/batch-processing/)開始 研究例如MapReduce這樣 batch-oriented 的數據流系統 對於建設大規模數據系統 它們提供了優秀的工具和思想

[第十一章](/2019/07/14/stream-processing/)將把這些思想應用到流式數據中 使我們能用更低的延遲完成同樣的任務

[第十二章](/2020/05/03/the-future-of-data-system/)將對本書進行總結 探討如何使用這些工具來構建可靠,可擴展和可維護的應用 





