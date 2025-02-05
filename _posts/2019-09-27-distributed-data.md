---
layout: post
title: Designing Data-Intensive Application - Part2 Intro - Distributed Data
comments: True 
subtitle: 分佈式數據
tags: systemDesign 
author: jyt0532
excerpt: 本篇文章介紹分佈式數據
---


在本書的[第一部分](/2019/01/26/foundation-of-data-systems/)中 我們討論了數據系統的各個方面 但僅限於數據存儲在單台機器上的情況 現在我們到了第二部分 進入更高的層次 並提出一個問題: 如果多台機器參與數據的存儲和檢索 會發生什麼?

你可能會出於各種各樣的原因 希望將數據庫分佈到多台機器上

1.可擴展性 Scalability

如果你的數據量/讀取負載/寫入負載超出單台機器的處理能力 可以將負載分散到多台計算機上

2.容錯/高可用性 Fault tolerance/High availability

如果你的應用需要在單台機器出現故障的情況下仍然能繼續工作 則可使用多台機器 給你提供一些redundancy

3.延遲 Latency

如果在世界各地都有用戶，你可以在全球範圍部署多個服務器 讓每個用戶可以從地理上最近的數據中心獲取服務 避免了等待網路數據包穿越半個世界


### 擴展至更高的負荷

#### 共享內存架構

如果你需要的只是擴展至更高的負荷(load) 最簡單的方法就是購買更強大的機器 稱為垂直擴展(vertical scaling)或向上擴展(scale up) 一個超級電腦的許多處理器 許多內存 跟硬碟都可以在同一個OS下連接 在這種**共享內存架構(shared-memory architecture)**中 所有的組件都可以看作一台單獨的機器

共享內存方法的問題在於 **成本增長速度快於線性增長**: 一台有著雙倍處理器數量 雙倍內存 雙倍磁盤容量的機器 通常成本會遠遠超過原來的兩倍 而且可能因為存在瓶頸 並不足以處理雙倍的載荷

共享內存架構可以提供有限的容錯能力 高端機器可以使用熱插拔的組件(不關機更換磁盤，內存模塊，甚至處理器) 但它必然受限於單個地理位置

#### 共享磁盤架構
另一種方法是共享磁盤架構(shared-disk architecture) 它使用多台具有獨立處理器和內存的機器 但將數據存儲在機器之間共享的磁盤陣列上 這些磁盤通過快速網路連接 這種架構用於某些數據倉庫 但競爭和鎖定的開銷限制了共享磁盤方法的可擴展性

#### 無共享架構

無共享架構(shared-nothing architecture) 也稱為水平擴展(horizontal scale) 或向外擴展(scale out)已經相當普及 在這種架構中 運行數據庫軟件的每台機器/虛擬機都稱為節點(node) 每個節點只使用各自的處理器,內存和磁盤 節點之間的任何協調 都是在軟件層面使用傳統網路實現的

無共享系統不需要使用特殊的硬件 所以你可以用任意機器 你也許可以跨多個地理區域分佈數據從而減少用戶延遲 或者在損失一整個數據中心的情況下繼續提供服務 
隨著雲端虛擬機部署的出現 即使是小公司 現在無需Google級別的運維 也可以實現異地分佈式架構

本書的第二部分 會將重點放在無共享架構上

### 複製 vs 分區

數據分佈在多個節點上有兩種常見的方式：

1.複製(Replication):

在幾個不同的節點上保存數據的相同副本 可能放在不同的位置 

複製提供了冗餘(redundancy): 如果一些節點不可用 剩餘的節點仍然可以提供數據服務 複製也有助於改善性能 我們會在[第五章](/2019/02/12/replication/)討論

2.分區(Partitioning):

將一個大型數據庫拆分成較小的子集 稱為分區 從而不同的分區可以指派給不同的節點(node, 也稱分片shard) 我們會在[第六章](/2019/03/12/partitioning/)討論

理解了這些概念之後 就可以開始討論在分佈式系統中需要做出的困難抉擇

[第七章](/2019/04/21/transactions/)將討論事務(Transaction) 這對於瞭解數據系統中可能出現的各種問題 以及我們可以做些什麼很有幫助

第八章和第九章將討論分佈式系統的根本侷限性


在本書的第三部分中，將討論如何將多個數據存儲集成為一個更大的系統 以滿足複雜的應用需求 但在那之前 必須先聊聊分佈式的數據



