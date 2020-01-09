---
layout: post
title: Effective Engineer - Minimize Operational Burden
comments: True 
subtitle: 最小化營運負擔
tags: effectiveEngineer
author: jyt0532
---

這篇是Effective Engineer - Minimize Operational Burden章節的讀書筆記


眾所皆知Instagram是個很猛的公司 初期的時候只有五個engineer 
基本上時間都花在開發新feature 根本沒有太多時間心力去維護舊系統 
最小化營運成本是他們最優先的事

讓一個系統營運 讓一個feature scale或是教新的engineer舊的東西 都需要時間成本 
**很不幸的** 許多聰明的有天份的工程師 在聽到新的technology或framework出來的時候 
都會迫不及待想去學 整合進現在的系統裡 
或在下一個project就開始用這個新的東西 
他們想嘗試用新的技術或是還在實驗階段的infrastructure 但沒有去想到其實很多其他人都不懂 或是運用了以後維護的成本會很高等等的問題

所以Instagram每一個決定都是安全的路 每個使用的technology都是被證明很猛的他們才用 
所以很多startup選擇如同雨後春筍般的NoSQL的同時 Instagram用PostgreSQL, Memcache, Redis 這讓他們非常輕易的維持擴展他們的app

([High Scalability](http://highscalability.com/blog/2011/12/6/instagram-architecture-14-million-users-terabytes-of-photos.html)裡面就有關於Instagram architecture的介紹 值得一看)

### 經營簡單化

對於Instagram的工程師 simplicity就是他們的信條 運用在產品 運用在招人 運用在所有事 每個design做完後都會問 能不能再簡單一點 直到不能再簡單為止

當我們問賈伯斯設計IPod之後學到了什麼 他說 你一開始要解決一個問題的時候 想出來的答案會是複雜的 然後很多人就停止思考了 但如果你繼續思考 把洋蔥表皮一層一層剝下來 你會發現非常優雅簡單的答案 但大多數人沒有花心思到達那個步驟

Pinterest在早期也犯了類似的錯誤 剛開始的兩年內 每月page views從0到100億 他們為了因應這快速的成長 database曾經是七種不同technology的綜合體(MySQL, Cassandra, Membase, Memcache, Redis, Elastic Search, MongoDB) 太過複雜的架構會帶來的負擔如下

1.Engineer專家需要一次分心在太多系統: 每個系統有他自己的處理問題的方式需要花時間去通透

2.越複雜的系統 會帶來越多的潛在single point of failure: 你很難保證每個系統都會有兩個以上的工程師通透 要是唯一通透的生病了就gg

3.新工程師要花時間學習新系統

4.當每個人都需要精通很多系統 就很難去正確的[分離抽象](/2017/07/29/balance-quality-with-pragmatism/#利用抽象化-簡化複雜度) 寫library或tool

總之最後Pinterest最後大砍特砍 最後只用MySQL, Memcache, Redis跟Solr 結果他們的scale果然繼續成長 因為operational burden降低了

從Instagram跟Pinterest我們學到幾件事

1.要玩要試新科技可以玩一些toy project 如果你要用在new system上 就要考慮其他人是不是也了解這個tech, 為什麼要選擇這個 跟 好不好上手

2.用一個新tech之前先看其他team有沒有在用 他們是不是maintain的很好 跟standard的解法來比 值不值得花這個投資

3.當你在處理海量數據 考慮清楚需不需要用到分布式系統 還是單獨一個機器就夠 分散式系統維護的成本高很多

再處理問題的時候 常問自己有沒有更簡單的解法可以減少未來的負擔 找到複雜的來源 想辦法簡單化

### 讓你的系統快點fail

很多工程師 評斷一個系統穩不穩定 是不是reliable的方式是看crash了幾次 這是不對的

如果以這樣為評斷的標準 他們會寫code去自動處理failure/error 讓系統能繼續跑 比如說 當偵測到錯誤設定(misconfiguration)的時候 就設回default value 或是一個

{% highlight java %}
catch(Exception e){
}
{% endhighlight %}

來處理所有exception 

這樣會讓你的系統fail slowly 短期內你的系統會跑得起來沒錯 但你的bug非常難抓 甚至你有bug都不知道

**你feedback和code的連結越直接 你就可以越快reproduce bug然後改善它** 所以如果你的系統可以fail fast 那你可以減少你的feedback loop

提供幾個fail fast的例子

1.如果有configuration error 在startup的時候就要crash

2.validate software input

3.如果你call別的service得到error 要秀出error 不要自己忽略他

4.當你要改變一個data structure的時候 如果他會讓其他依賴你的data structure壞掉 要馬上噴exception 比如說改一個collection 會不會影響到之後的Iterator

5.如果一個data structure corrupted了要趕快噴錯

6.在重要或是複雜的business logic前後加一些Assert 而且錯誤訊息要寫得很清楚

7.如果你的系統是在invalid或inconsistent state 要噴Alert

系統越大越複雜 fail fast可以幫你省的時間越多

但fail fast也不是說系統一有錯 你就要crash你的系統給user看 你可以fail fast在一些很接近actual source of error的地方 不要真的去影響使用者

比如說你的網頁有幾百個component需要load 每個component都會fail fast 但也有個global exception handler 處理例外 log下來之後 gracefully render剩下成功的component 

你還可以建一個dashboard 讓你的exception照頻率sort 讓engineer從最高頻的開始解起

### 不間斷的自動化

每個公司每個product都會需要有人oncall 但沒有人希望半夜三點接到電話 只是跑一些machine可以幫你跑的指令 

手動快速地跑某些指令 短時間感覺很快就過了 但長期下來自動化會節省很多的operational burdern

Engineer要常問自己 **這個事情值不值得一直手動作 需不需要自動化** 當你手動做很多事的時候 這個答案就相對簡單 但大多數情況不是非黑即白 Engineer通常選擇不自動化因為

1.現在沒有時間: deadline快到了 上頭壓力很大 所以為了快點ship產品 放棄自動化

2.Tragedy of the commons: 越多人共享的事情 會得到最少的照顧 如果一些manual的task需要由oncall member來做 因為你久久輪一次 你就比較不願意去負責把它自動化

3.不熟悉自動化的工具: 大多數自動化 需要跟command line script 很熟 對一般來說人來說門檻較高 但熟能生巧

4.低估了未來使用它的頻率

5.低估了節省的時間可以複利 節省很多未來的大時間

每次當你做了一些機器能幫你做的事的時候 就問自己能不能自動化 能夠被自動化的事情包括

1.unit test, integration test 眾多test

2.Extracting, transforming, summarizing data

3.偵測error的spike

4.build跟deploy

5.capture and restoring資料庫的snapshot

6.定期的batch operation/computation

7.重啟web service

8.檢查code的style有沒有符合guideline

9.Training機器學習的model

10.管理使用者帳號跟data

11.幫你的service加減機器


自動化的cost一開始很高 因為還包含了學習怎麼自動化的時間 但隨著你慢慢自動化之後 你之後會越來越傾向自動化 因為leverage又更大了

最後 其實自動化有兩種 自動化跑固定指令 跟 自動化作決定 對於自動化作決定這一部分要特別小心 因為有時候很難做的完全正確

### 讓批次自動化Idempotent

通常我們的自動化程式都是batch processes 代表說一連串的指定 動作 **沒有人為的干預** 通常我們用這個來跑海量數據 當然大多數的時候都會成功
但當失敗的時後 也需要花不少時間復原 要是你不夠注意的話 **你花在修復自動化程式的時間 會比當初手動跑這個task的時間還多** 本末倒置

讓你的自動化程式更堅強更不容易壞 有一個很重要的技巧 就是讓你的程式Idempotent 這代表說不管這個program run一次還是run很多次 他都是跑出一樣的結果

(之前看到這個字是在rest api的時候 GET PUT就是idempotent POST就不是)

為什麼一個自動化程式Idempotent 比較好呢 代表說他可以重複跑很多次 而不會有預期之外的side effects

比如說你需要一個程式 會每天爬你的application log去計算你的每週使用者和他們的活動 non-idempotent的作法就是 一行一行爬 然後去修改database裡的counter 這樣要是哪天這個程式壞了需要重跑 你就很可能會重複計算 得到錯誤的資料 

稍微robust一點的解法 就是你先有一個script去跑每個user每天的活動 等一天的跑完 再用另一個script去加總 這樣即使需要重跑 也只是把之前得overwrite掉 

如果不可能idempotent的話 **最起碼追求Retryable或是Reentrant**  這代表說即使之前跑的時候失敗了 之後再跑一次還是會成功 如果你不是Reentrant 代表說你跑失敗之後會留下一些副作用on global state 所以你再也跑不成功

Idempotent還有其他好處 你可以比你預期的多跑幾次 用來提早發現問題 比如說今天你有個自動化程式會每個月生出user report 但是你當初寫這個程式的時候做的許多假設 可能幾個月後就不再成立 像是data size, code base size architecture那些可能都跟當初寫自動化程式的時候不一樣了 等到你發現問題的時候你上頭的人可能已經在跟你要資料 你就會措手不及

但如果你的script是idempotent 你可以先run每個禮拜的或是每天的 都先跑起來放著 你會更早拿到feedback發現說codebase變大了等等 不會等到每個月生資料的時候才發現爆了

如果你可以run很多次的話而沒有副作用的話 還有另一個好處 可以減少false positive 比如說你在check你system的state 如果你每5min確認一次metric 一發現異常就噴錯 那即使真的噴錯了 也可能只是偶爾的系統異常 過一兩分鐘就修復了 你卻還花人去處理 浪費時間 但若是你每分鐘跑一次 連錯個3分鐘-5分鐘你才噴異常 那很可能就是真的有什麼問題 而不是偶爾的小系統異常

寫idempotent的程式可以讓你對於batch process更好maintain 你有更多時間做其他事

## 訓練你respond和recover的速度

在Netflix有個挺反骨的傳統 別的公司都是能盡量避免failure最好 但他們有一個系統(Chaos Monkey)每過一段時間就會隨機kill掉某service 而且通常是上班的時間 這樣讓engineer知道要如何修復系統或是找到他們系統的弱點 然後修復它 這樣不但系統比較robust 當真正發生問題的時候大家比較知道怎麼應對

>
> Best defense against major unexpected failures is to fail often
>
>
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;《Chaos Monkey Released into the Wild》

經過這樣的不定時訓練 當AWS爆的時候 Netflix修復的最快 而其他公司的service都停擺了好幾小時 

儘管我們做了很多事情來避免failure 但我們永遠不可能100%保證不會爆 我們能訓練的就是爆的時候 我們可以用最快的速度修理 

谷歌每年都會有數天的Disaster Recovery Testing他們模擬地震海嘯等等 直接把某些data center斷電 看各組怎麼溝通處理解決

Dropbox會固定在production上有一些人造的traffic 當真正traffic+人造traffic超過上限的時候 他們可以先關掉人造traffic爭取一點時間
 
但並不是每間公司都是那麼大規模 但你還是可以常問自己:

1.要是有個重要bug被deploy 多快的時間我們可以roll back

2.如果database爆了 我們需要多久重啟另一個然後把失去的資料復原

3.如果traffic爆了 要怎麼最快速分流 不讓使用者體驗爆掉

4.如果測試環境掛了 多久可以生一個新的

5.要是顧客回報重要bug 從接電話後多久知道可以通知到engineering team 

當然你也可以把問題想得遠一點

6.如果大老闆或大客戶對你的project提出反對 你要怎麼回覆
 
7.如果一個重要的組員生重病或是受傷了 怎麼讓組繼續運作

8.要是project趕不上deadline 多久之內可以發覺 怎麼處理

當大家比較沒壓力的時候主動規劃 會讓你對於failure recovery更有信心 這也會讓你開發新feature更加大膽

