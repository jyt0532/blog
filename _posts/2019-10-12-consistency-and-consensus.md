---
layout: post
title: Designing Data-Intensive Application - Consistency and Consensus
comments: True 
subtitle: 一致性和共識
tags: systemDesign 
author: jyt0532
excerpt: 本文講解一致性和共識
---

這是Designing Data-Intensive Application的第二部分第五章節Part1: 一致性和共識介紹

一致性和共識Part1 - [介紹](/2019/10/12/consistency-and-consensus/)

一致性和共識Part2 - [線性一致性](/2019/10/17/linearizability/)

一致性和共識Part3 - [順序保證](/2019/10/24/ordering-guarantees/)

一致性和共識Part4 - [分佈式事務與共識](/2019/11/03/distributed-transactions-and-consensus/)

本文所有圖片或代碼來自於原書內容
{% include copyright.html %}

# 一致性和共識介紹

正如[第8章](/2019/10/05/the-trouble-with-distributed-systems/)所討論的 分佈式系統中的許多事情可能會出錯 比如網路中的數據包可能會丟失/重新排序/重複遞送或任意延遲 時鐘只是盡其所能地近似 而且節點可以因為許多理由暫停或隨時崩潰

構建容錯系統的最好方法 是找到一些帶有實用保證的通用抽象，實現一次，然後讓應用依賴這些保證

在說什麼聽不懂? 沒關係 想一下我們在[事務篇章](/2019/04/21/transactions/)中做的事 藉由事務這個抽象 應用程式可以得到以下優點

1.應用程式可以假裝沒有崩潰發生(atomicity) 

2.應用程式認為沒有其他人同時訪問數據庫(isolation)

3.應用程式認為存儲設備是完全可靠的(durablility)

那事實是否真的如此呢 當然不是 但即使發生崩潰 競爭條件或是磁盤故障 事務抽象也隱藏了這些問題 應用程式不用擔心

現在把同樣的概念拿回來用在這 在分布式系統裡面 哪個麻煩的問題被抽象之後 可以解決應用程式最多的問題呢 給你三秒鐘

3

2

1


答案是: **共識(consensus) 也就是讓所有的節點對某件事達成一致** 之前提到的問題 網路問題 時鐘問題 進程問題 都會讓節點跟節點之間資訊不流通 所以很難做決定 所以只要共識的問題搞定了 分布式系統跟單節點系統沒什麼大差別

我們會在第四Part的[分佈式事務和共識](/2019/11/03/distributed-transactions-and-consensus/)中提到解決共識和相關問題的算法 
但在那之前 我們需要瞭解什麼可以做 什麼不可以做 什麼可能 而什麼不可能


## 一致性保證

在[複製延遲問題](/2019/02/12/replication/#複製延遲問題)中 我們看到了數據庫複製中發生的一些時序問題 如果你在同一個時刻觀察兩個不同數據庫節點 這兩個節點可能數據不一樣 因為寫請求可能在不同的時間到達不同的節點 無論數據庫用何種複製方法(單主 多主 無主複製) 都會出現這些不一致的情況

大多數複製的數據庫至少提供了**最終一致性**: 如果你停止向數據庫寫入數據並等待一段不確定的時間 那麼最終所有的讀取請求都會返回相同的值 所以你所看到的不一致性都是暫時的 最終都會解決 最終一致性的一個更好的名字可能是收斂(convergence) 因為我們預期所有的副本最終會收斂到相同的值

然而 最終一致性是個挺弱的保證 他並沒有保證你多久後會收斂 在數據庫真正收斂之前 讀操作可能會返回任何東西或什麼都沒有 最慘的是 當你在寫入之後馬上讀取 你還不一定會看到你剛寫的值 因為讀請求可能被導到別的副本上(參閱[讀己之寫](/2019/02/12/replication/#讀己之寫))

所以在跟只提供弱保證的數據庫打交道時 你需要始終意識到它的侷限性 不要太高興做出太多假設(比如假設你讀一個剛寫的東西 值一定正確) 這種錯誤非常難debug 而且大多數情況運行良好

當然有許多更強一致性模型 這些具有較強保證的系統可能會比保證較差的系統具有更差的性能或更少的容錯性 聽起來很吸引人(雖然效率低 但起碼我保證對)

本文將探索數據系統可能選擇提供的各種一致性強度 你必須在一致性強度跟效能之間做選擇 這跟我們之前討論[事務](/2019/04/21/transactions/)時很像 事務隔離主要是為了**避免由於同時執行事務而導致的競爭狀態** 分布式一致性主要關於 **面對延遲和故障時 如何協調副本間的狀態**

本章涵蓋了不少主題

1.我們會在Part2先研究最強的一致性模型之一 [線性一致性(linearizability)](/2019/10/17/linearizability/) 討論其優缺點

2.然後我們會在Part3檢查分佈式系統中事件[順序的問題](/2019/10/24/ordering-guarantees/) 特別是因果關係和全局順序的問題

3.在Part4[分佈式事務和共識](/2019/11/03/distributed-transactions-and-consensus/)中將探討如何原子地提交分佈式事務 這將最終引領我們走向共識問題的解決方案


