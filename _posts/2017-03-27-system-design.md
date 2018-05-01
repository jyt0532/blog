---
layout: post
title: 系統設計 - Design cache
comments: True 
subtitle: Design cache
tags: system design cache
author: jyt0532
---

這篇是我寫在一畝三分地的文章 轉貼過來分享給大家 是前一陣子準備面試的心得之一 

系統設計是最近面試挺常見的題目
雖然system design的題目很難準備 但你花在這個的前10-20小時投資報酬率非常高！
基本的觀念有了以後可以apply到所有的問題 
面試之前不管HR跟你說有沒有system design你都不怕 廢話不多說 直接進入主題

我的所有資料來源都是來自以下連結 用自己的話整理出來

[checkcheckzz/system-design-interview](https://github.com/checkcheckzz/system-design-interview)

[interviewbit](https://www.interviewbit.com/courses/system-design/)

這篇文章主要是把所有手上資源照priority順序加上一些自己的整理 用簡單的方式介紹怎麼Ace system design 依照priority順序分成
明天面 下禮拜面 下個月面

### 明天要面system design

迷途書僮:誒jyt0532 我明天就要面試了 沒空讀那麼多你給的文章 該怎麼辦？

jyt0532: 很好 今天幸好你遇到我 我用最少的時間讓你知道明天面試的時候大方向怎麼走

**Step1: 先問所有requirement, spec 這個系統需要提供什麼功能**

**Step2: Constrains: 問他我們需要處理多少traffic, 多少data, latency重不重要 A和C選哪個**

**Step3: 計算需要多少機器 要用什麼storage**

**Step4: Abstract design: 先畫出大架構！ 每個會出現的component都要畫出來 再看面試官希望你深入講哪個component**

**Step5: Scale: 讓你的system有fault tolerance, scale成大公司的系統架構**

迷途書僮: 等等 step2的A和C是什麼

jyt0532: CAP的A和C 如果你明天要面試你不知道CAP是什麼 那我的建議是你今天早點休息 明天才有精神 地球是很危險的

迷途書僮: 別這樣啦 我是你部落格的忠實讀者 拜託給我一個最簡單最好的解釋

jyt0532: 既然你都這麼說了 今天幸好你遇到我 把這個看完你就通透了[A plain english introduction to CAP Theorem](http://ksat.me/a-plain-english-introduction-to-cap-theorem/)

迷途書僮: 喔原來是這個 那這4個steps 你這樣講真的很抽象 可不可以舉個例子 我可以apply到所有system design的題目

jyt0532: 光說不練假把戲 我帶你跑一次system design process

來個基本的 假設現在要design一個cache

**Step1, 2: 問requirement**

多少資料需要進cache? 30TB

expected QPS? 10M

eviction strategy? LRU

Access pattern? Write back(有空的話Write through, write around, write back都要知道什麼意思 利弊)

Latency重要嗎? cache的用途就是降低latency

C or A: A

**Step3:算一下你需要多大的machine多少台**

[Latency Numbers Every Programmer Should Know](https://gist.github.com/jboner/2841832) 這裡面的數字要有點sense

假設我們現在要用72GB RAM 4 core的machine
那總共以儲存data來說 需要30TB/72GB = 420台
這樣的話每台的QPS = 10M/420 = 23000, 即使所有core都用了 每個core要處理6000QPS
代表說 1/6000 = 167us 搭配上面那個link可知道即使是ram sequentially read 1MB要250M 所以我們如果用這個size的machine 會無法負荷

改變主意 假設現在用16GB RAM  4core的machine
30TB/16GB = 1875台, QPS per CPU = 10M/1875/4 = 1400QPS = 700us per queries. 這個數字負擔小多了

![Alt text]({{ site.url }}/public/how_to_play.gif )
看完上面的流程知道我們在幹嘛了吧? **先用data constrain算出要幾台機器 再用traffic constrain算看看這樣的配置合不合理**
這樣做完你就知道你的system是需要猛的機器少台一點 還是差一點的機器多台一點

**Step4: 畫出大架構**
這時候就必須推薦[CS75 (Summer 2012) Lecture 9 Scalability Harvard Web Development David Malan](https://www.youtube.com/watch?v=-W9F__D3oY4) 

這根本太精采
什麼你又沒時間 好啦別說我虧待你 有人把重點整理好給你了[Harvard_CS75_Notes](http://ninefu.github.io/blog/Harvard_CS75_Notes/)
但最後那個畫圖的地方還是要看一下

**Step5: 喇賽時間**

這時候system畫完了 如果要scale的話需要什麼東西 不外乎就是load balancer啦 DB就是可能要master-slave或是multi-master 這種東西

至於怎麼fault tolerance呢 常見的處理就是replication 就是一樣的資料存很多地方 假設有P個replication
因為每次寫和讀都寫進/讀出這P個地方非常花時間 那該怎麼辦呢
假設寫的時候 只要有W個replication confirm update我就return to user
假設讀的時候 只要有R個replication給我一個一樣的value, 我就return這個value給user
depends on design的use case(這就是為什麼use case很重要) 你要看read跟write哪一個operation可以承受高一些的latency

如果要求read很快 write可以慢一點沒關係 那就可以設R = 1, W = P, 反之可以設R = P, W = 1
總之 只要R+W > N 那這database就是strong consistent! 

如果真的要求高速度的話就必須犧牲consistent 那R+W就會<P(weak consistent)

### 下禮拜要面system design

[從小公司到上億用戶公司的架構演進](http://highscalability.com/blog/2016/1/11/a-beginners-guide-to-scaling-to-11-million-users-on-amazons.html)

[InterviewBit](https://www.interviewbit.com/courses/system-design/)這是個非常好的互動式網站 他是一步一步漸進式的問你每個你在面試中該問的問題 帶你走過一遍system design interview的process 非常建議這裡面的[八題](https://www.interviewbit.com/courses/system-design/)都要寫過

[Scalable Web Architecture and Distributed Systems](http://www.aosabook.org/en/distsys.html)

### 下個月要面
把[這裡](https://github.com/checkcheckzz/system-design-interview#intro)和[這裡](https://github.com/checkcheckzz/system-design-interview#blog)的文章都K過你就比大多數candidate強很多了


### 總結
其實有工作經驗的都知道 你很常需要去design一個新的project 而釐清use case這些事情是基本 連use case都沒問那面試官根本不會覺得你是個好的工程師 主要考察的是communication and problem solving, 給你一個開放性問題 你怎麼分析step by step, 你如何跟別人討論你的idea, 如何optimize你的system 往這個方向想就覺得其實system design其實沒想像中的難
