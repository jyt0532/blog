---
layout: post
title: Effective Java Item67 - 謹慎地進行優化
comments: True 
subtitle: effective java - 謹慎地進行優化
tags: effectiveJava
author: jyt0532
excerpt: 本篇文章介紹進行優化的時機以及如何優化
---

這篇是Effective Java - Optimize judiciously章節的讀書筆記 本篇的程式碼來自於原書內容


## Item67: 謹慎地進行優化(Optimization)

有三條與優化有關的格言 大家應該都要知道

1.很多計算上的過失都被歸咎於效率(沒有必要達到的效率) 而不是其他原因 包含盲目的愚蠢

> More computing sins are committed in the name of efficiency (without necessarily achieving it) than for any other single reason—including blind stupidity.
—William A. Wulf [Wulf72]

2.不要去計較效率上的小小得失 在97%的情況下 不成熟的優化才是一切的根源

> We should forget about small efficiencies, say about 97% of the time: premature optimization is the root of all evil.
—Donald E. Knuth [Knuth74]

3.在優化方面有兩個規則要遵守
	
1) 不要進行優化
2) (對專家來說) 在你沒有絕對清晰的優化方案之前 不要進行優化

> We follow two rules in the matter of optimization:
Rule 1. Don’t do it.
Rule 2 (for experts only). Don’t do it yet—that is, not until you have a perfectly clear and unoptimized solution.
—M. A. Jackson [Jackson75]


不要為性能而犧牲了合理的結構 要努力編寫好的程序 而不是快的程序 原因是如果你一開始就想要寫快的程序 那可能一寫完就已經以後沒戲了 
但如果你一開始是想要寫好的程序 比如說你有好的結構使得各個模組之間訊息隱藏(information hiding) 那之後要單獨加速優化某個模組都是非常簡單的

這不代表說寫程序不必在乎性能問題 而是你心裡要清楚知道 實現上的問題如果你有好的結構 很以輕鬆地透過後期的優化修正 但結構上的問題幾乎是不可能改的

### 努力避免那些限制性能的設計決策

當一個系統設計完成之後 其中最難更改的組件就是那些**指定了模塊之間交互關係以及模塊與外界交互關係的組件** 
比如說API或是存儲格式 這些設計在之後難以改變 而且很可能會是之後優化的bottleneck

舉幾個API設計決策的例子 

1.使公有的類型設計成可變的 這會導致大量不必要的[保護性拷貝](/2017/09/26/make-defensive-copies-when-needed/)

2.在適合使用複合模式的公有類中使用繼承 會把這個類與他的超類永久束縛在一起 [Item18](/2018/05/05/favor-composition-over-inheritance/)

3.在API實現類型而不是接口 會把你束縛在一個具體的實現上 即使將來出現更快的實現你也無法使用 因為你的設計不夠彈性[Item64](/2018/06/09/refer-to-objects-by-their-interfaces/)

### 結構確立之後

一旦設計了一個清晰 簡明 且結構良好的設計之後 如果你對性能不滿意 再來決定優化

別忘了在每階段的優化前與後 要對性能進行測量 你可能會驚訝優化對於性能沒有太大影響 甚至性能會變更差 主要原因在於 你要找出bottleneck並不容易


### 總結

不要努力去寫快的程序 應該要努力去寫好的程序 

設計系統的時候(特別是API 線路層 或存儲)一定要考慮性能的因素 當建構完之後 要記得測量性能

如果性能不夠快 小心地找出問題的根源 然後優化相關部分(第一個步驟就是檢查所選擇的算法 再多的低層優化也無法彌補算法的選擇不當) 必要時重複這個流程 並且每次改變之後都要繼續測量性能 直到滿意





