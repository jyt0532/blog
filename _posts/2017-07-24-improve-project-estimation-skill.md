---
layout: post
title: Effective Engineer - Improve your project estimation skills
comments: True 
subtitle: prioritize regularly
tags: effectiveEngineer
author: jyt0532
---

這篇是Effective Engineer - Improve your project estimation skills章節的讀書筆記

作者先講了他在Ooyala重寫video player的經驗 他們預估要花4個月才能寫完 但這四個月的過程中遇到了當初沒想到的問題 結果這個project最後花了9個月 

這個事件結束之後 他找了些資料 window vista比預期的晚了三年 Netscape晚兩年 有study表示44%的project都會delay或是錯估預算 24%的project會失敗

作者認為 評估project是最難學習的skill之一但卻非常重要 
因為我們手上有的information通常都不夠精確 在這情況下很難增加準確率並且適應變化

### 用正確的評估去Drive project

我們很常被問到: 這個project多久可以做好

Steve McConnel的定義是: **好的估算 可以提供清楚的peoject現況 讓leader去做對於目標的取捨**

對於估算project 作者有幾個實際建議

1.把大task分離成小工作 rule of thumb是只要有一個task需要超過兩天完成 就要把它再分更細

2.完成一個項目的時間是how long it would take而不是how long someone want it to take 當你的老闆覺得這個project可以更快完成的時候 給他看你的estimation 如果你切得比較小塊的話估算也比較細 比較沒有爭議的空間

3.應該把你的分析想成一個機率分佈 只給一個確切時間的結果 就是PM預期”你可能完成的最早時間” 或是 “你無法證明你不可能完成的最早時間” 正確的估算應該是 50%的機率4week 70%的機率6weeks 90%的機率8weeks完成

4.讓實際做那個task的人估算那個task的時間

5.小心內心的既定印象: 有一個教授做了一個實驗要每個人寫下社會安全碼的後兩碼 之後估算一個酒的價格 發現後兩碼比較大的平均下來估算的比較高 所以不要相信自己的第一印象 要實際的去估算

6.用不同方式去估算同一個project: 第一個方法就是分成細小的tasks 每個單獨估完後加在一起(Bottom-up) 第二個方法是找公司其他人做的類似project花了多久(Top-down) 第三個方法是先算你需要建幾個subsystem 再加起來 

感覺起來就是把一個project切成不同的size去估 增加可信度

7.小心人月神話: 人月神話講的是人力和時間並不是線性關係 只有PM的世界裡面才會覺得九個媽媽一個月可以生出一個嬰兒 團隊溝通需要花費的時間成本跟團隊大小呈平方比
 
8.用歷史資料估算 比如說上季你用同樣方法 低估了20% 那這次你就應該高估25%

9.做research**的時間**要一開始就寫好 比如說你要選要用哪個database或library 在規劃的一開始就要畫出一個time box 比如說三天內要決定用哪個database 這樣就不會survey完就花了很大部分的時間

10.讓其他人去看你的規劃 讓其他人challenge


### 為未知保留預算 空間

作者當初在重寫video player的時候 會延遲那麼久是因為某些當初沒預料到的事

1. 寫單元測試的時間
2. 一些coding style的東西沒有統一 花了些時間制定
3. 客戶突然有更重要的事 耽擱了engineer的時間
4. 未預期的技術問題 需要花人debug
5. scalability的問題
6. 有人員跳槽走人
7. 需要重寫UI而不是用之前的UI
8. version control的tool換了

一個project越長 難以估計的偏差就會越多 

先估算那些雜事在一天當中的比例(meeting, 面試, oncall bug, 1-1, 員工訓練, 聽演將等等)去推一天到底有多少時間可以寫code  
再用總需要時數去除以一天可以工作多久 就得到需要幾天 最後不要說”one calendar month”要說”one engineer month“ 因為PM總是把情況想的樂觀

### 設定project明確目標

每個project開始之前要先清楚明確目標是什麼 要是目標籠統 過程中一些tradeoff的問題會比較難做決定

除此之外還有兩個好處
1.清楚分離must-have和nice-to-have

2.跟stakeholder講清楚需求

當時他們delay很久的video project的目標是”Rewrite video player in 4 months” 作者說現在回想起來 應該要改成

“As soon as possible, build a drop-in replacement for the video player that supports dynamically loadable modules, is unit-tested, and can later be extended to support additional ad integrations, analytics reports and video controls” 這樣他們在很多local tradeoff就可以省很多時間

提供一些具體目標的例子:

1.減少P95 latency for home page to under 500ms

2.launch search feature讓使用者filter by content type

3.手機app支援離線瀏覽

4.A/B test顧客的結帳流程 增加購買量

5.開發一個分析報告 可以看到各國家的metrics

### 設立一些可以評量的里程碑

每次有人問你你project進度如何 不要再說almost done或是90% 應該要設立一些里程碑 每個里程碑要是measureable

### 別再馬拉松的中途衝刺

一個常見的問題是 當manager發現再兩個月就是死線 但是進度落後兩個禮拜 所以要所有組員接下來兩個月 一個禮拜工作50小時 就可以趕上死線了

但其實scheduling不是個數學問題 因為

1.生產力隨著工時上升而下降

2.你可能比你想的還要落後: 因為按照你之前的估算 你都落後兩個禮拜了 那很可能你之後的時間也跟之前一樣低估

3.不是每個人都對時間那麼有彈性 有人有家人有小孩

4.離deadline越近可能會有越多的meeting, status update 那些都會減少工作時間

如果你真的非得要加工時不可 有幾個建議

1.一定要讓每個組員知道落後的主因是什麼 然後確保之後不會發生

2.修改timeline 重新定義里程碑

3.只有在完全確定加工時可以讓project如期完成的情況下 才加工時

4.做好心理準備 可能這個sprint要整個放棄


