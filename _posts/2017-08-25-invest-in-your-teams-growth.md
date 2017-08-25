---
layout: post
title: Effective Engineer - Invest Your Team's Growth
comments: True 
subtitle: 投資團隊成長
tags: effectiveEngineer
author: jyt0532
---

這篇是Effective Engineer - Invest Your Team's Growth章節的讀書筆記

作者說他在ooyala的第一個project deadline是兩個禮拜 可是code都沒有test 那個語言他也沒學過 所以前兩個禮拜他一個禮拜工作80小時才把project生出來 他發現沒有一個好的onboard program 大家一開始都很難上手 所以他在ooyala的第一個學到的事就是 positive and smooth的onboard program是非常有價值的

投資在onboarding program只是其中一個讓你團隊成長的方式 這本書目前為止教的都是如何讓你個人成長 為什麼讓團隊成長也很重要呢 因為跟你合作的人跟事對於你的成長也有很大的影響 你不一定要是manager才可以投資團隊成長 即使你只是Individual contributor 投資團隊成長也是件levarage很高的事情

當你的職位做得越高 你的表現就會不只是你個人成就 而且還取決於你對於其他人的impact

> You are a staff engineer if you are making the whole team better than it would be otherwise
>
> You are a principle engineer if you are making the whole company better than it would be otherwise
>
> And you are a distinguished engineer if you are improving the industry
>
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Marc Hedlund

### 招人是每個人的責任

面試別人很無趣 會降低我們的生產力 而且寫feedback也花時間 每個人各自從讀resume但電話面試到onsite面試全部包辦實在浪費心力時間 所以好的公司會把hiring process標準化 節省每一個人的時間

好的面試流程要達成兩個目的 1. 找到適合公司的人 2.推銷公司的team, culture and mission 即使最後面試者沒拿到offer 他也會推薦朋友去 或是等更強的時候再來 所以推銷公司也是非常重要的一環

身為一個面試官 你要問的問題必須要**信噪比最低**(signal-to-noise ratio) 一個好的題目可以讓你在短時間內分辨出你想要的engineer 不好的題目會讓你即使花了時間 最後也難以決定

最典型的面試題目是algorithm跟coding 然後寫白板 這只能證明他們底子好 不能證明他們工作起來有效率 是否能完成project 

所以越來越多公司給你一台筆電 跟project skeleton 你可以自己找api 自己寫完後debug  他們看你怎麼用基本的UNIX command怎麼去從你不熟的library去找你需要的東西


根據作者面試的經驗 有五點建議

1. 花點時間跟你的team談一下哪個特質是公司最想要的 coding能力 了解語言 資料結構 產品認知 debug, communication skills, culture fit 討論完之後 確保你們的interview loop有cover到你們想測試的特質
2. 定期去確認現在的process適不適合 面進來的人表現如何 一直改進面試的流程
3. 讓你的問題可以輕易調整難度 比如說很容易加上follow up 不要只是一翻兩瞪眼的解的出來跟解不出來
4. 面試過程維持高信噪比 不要讓應徵者想太久或卡太久  給提示或跳過都可以
5. 先問幾題很快的可以了解他的觀念好不好的問題 比如說變數如何傳進函數裡等等
6. 不定期shadow 也就是在旁邊看別人怎麼面試
7. 別怕在你的面試流程加入非傳統面試 像airbnb就有專門culture fit的面試

要讓你們面試更有效率 只有透過一直改進 這一切都會值得 加入你公司的強者給你帶來的額外產值大於你把那些時間投資在面試之外的事情 

### 設計一個好的onboard流程

當一個組很小的時候 對於新進員工很容易就可以知道什麼是重要的 有什麼工具可以用等等 但是當你size慢慢成長時 新進員工會不知道該先學什麼 甚至對於每個組員認為的重要的東西不一樣 他們學習的知識會非常零散  而且沒有人帶領的話 他們會花更多時間在讀document 而沒有時間寫新feature 所以作者在Quora的時候自願lead一個onboarding program

作者之前沒有類似經驗 所以他研究了各大公司的制度 最後作者引入了mentorship制度 組織onboarding talk 寫onboarding material跟訓練mentor的制度

一個新進員工對於公司的第一印象很重要 可以在剛進公司最純潔的時候跟他說公司的文化 理念 跟他介紹公司的工具等等 比讓他直接去讀doc還要有影響力的多

當你已經是組裡面很熟練的人 你也許會想 為什麼要花額外時間放下手上的project去訓練人呢 要永遠記得 投資在你組的成功 也極有可能對你帶來正面影響 讓新進員工快速上手會讓你之後選擇high-leverage的工作更有彈性 因為你的team變強代表code review更好 更快的修bug 還有oncall的支援

相反的 如果onboarding的系統很差 code可能會花資深工程師比較久的時間review就只是因為api用的不對或是工具用的不對 更糟糕的是 當一個人表現不好的時候 你不知道他是真的能力不足 還是你們的onboarding太差導致他表現不好 這樣對於interview loop的feedback會有偏差(別忘了上一節說的我們要從新進員工表現來改進interview process)

不論你資深與否 你都可以為onboarding流程貢獻 如果你剛進來沒多久 你知道你進來的時候哪些東西對你最有幫助 哪些doc無關緊要 直接去修改那些doc 如果你很資深 你應該去注意哪些環節新進的員工總是做不好 去改善onboarding流程

要如何設計一個好個流程呢 首先要先定好組的目標 再來要設定機制去達成目標

作者在Quora設計的時候 訂了以下目標
1. 讓新進員工快速上手: 要教新人要花資深的人的時間 他們上手越快不但資深員工花的時間少 新進員工也可以趕快有產出
2.   傳授組的文化跟價值: 新進員工可能準備面試時稍微知道你們公司的文化價值 在他剛進來的時候教給他具體的例子 讓他更清楚
3. 教他所有它需要的工具跟知識: 每個工程師都要會的tool 
4. 讓他有機會跟組員社交: 讓未來組員跟他互動 他認識越多人以後可能產出越高


每個公司的目標不一樣 以上參考就好 列出目標之後 作者用以下的方法去達成目標

1. Codelabs: 一個解釋公司tool的文件 包含怎麼用它 為什麼這樣設計 帶你走遍裡面的code最後還有測驗看你是不是精通了
2. onboarding talks: 讓資深工程師現場講或是錄一些講解codebase跟architecture 解釋跟展示內部的工具如何使用 怎麼跑各種test 以及各種我們認為重要的東西
3. mentorship: 每個新進員工背景不同 很難只用一個on boarding program去給所有人 所以他們安排了mentor提供個人的training 第一個禮拜mentor跟mentee每天1-1 之後每個禮拜1-1 責任包含了review code 討論design利弊跟prioritize工作 Quora甚至還有mentor的workshop去討論mentor的心得分享 他們還會讓mentor跟mentee坐的靠近一點
4. starter tasks: 每個人在上工第一天就會有任務 可能是修改bug或是小feature 為了要達成這個目標 Quora的on boarding program就會移除很多不必要的東西 因為第一天就要有產出

以上只是個例子 on boarding program也是個iterative的process 一開始可能只是寫個doc教新進員工怎麼build, deploy等等 隨著team的大小變化 可能要錄製video等等 還要請新進員工提供feedback 哪些對他們有用哪些是浪費時間 哪些希望可以早點知道 持續地進行

### share ownership of code

作者在ooyala的時候在某個去夏威夷的假期中 接到CTO的簡訊 說logs processor壞了 可是他當時正在火山附近 也沒有電腦網路 他就回覆給他說晚上會看
可以當天的心情都毀了 回到飯店後看了一下 的確是個全公司只有他會修的bug 這次經驗雖然他感覺到自己很重要 但也學到了很多

人們常常會誤解 覺得你手上lead很多project 特別是只有你懂的peoject代表說你很重要 無可取代 越少人會的知識價值越高 但其實share ownership給其他人不只對你好對公司也好 就像當你越來越資深 會有越來越多人來問你問題 雖然感覺不錯但卻也伴隨著代價

當你做到這個project瓶頸的時候  你對於其他的任務彈性就變低 因為你最懂你的project所以bug給你修當然最快 之後你大多數時間都是在回應問題 maintain 跟調整feature等等 你會越來越少時間學新東西 這也是為什麼要投資在你的team 不管是教學或是mentor 對你來說都有長期價值

從公司的角度來看 sharing ownership代表**巴士指數**(指的是一個team裡面如果有多少人同時被巴士撞 然後你們team就不能正常運轉)可以大於一 巴士指數代表說只要公司有人放假或生病 你們team就慘了 這也代表 組裡面的engineer都不是fungible

fungible指的是沒有一個特定的人一定要去做特定的事 任何一件事都可以被其他人做 這給了所有人一定程度的自由 更多彈性去調整開發速度 更少對於oncall或支援
的限制

為了增加shared ownership 以下有幾點建議

1.避免一人團隊
2.review別人的程式跟系統設計
3.Rotate各種不同的task 背負不同的責任
4.讓code好讀 品質高
5.給一些關於軟體設計的演講 
6.把你做的是寫好doc
7.寫好複雜的workflow或是不明顯的workaround 目標是讓其他人可以照著走也可以把事情做完
8.花時間教別人 mentor別人


### 建立事後分析檢討機制

在每次事件發生後 比如說網頁掛了等等 修復完後都要開會 討論為什麼會發生這件事 要怎麼樣預防未來再發生這件事 

所有人要有很好的概念 這不是在找戰犯 而是一起想更好的解法 最後的想法解法要分享給全team 讓所有人知道發生了甚麼以及我們打算怎麼預防

事後檢討並不是只有在壞事發生的時候 通常團隊表現不錯的時候  大家慶祝完後就接下一個project 但你們到底是多麼有效率 為什麼這樣做會有效率 你們是在朝公司的goal前進嗎 開完會後還分享給其他team 不用讓他們花時間去學一樣的lesson 

也許這種公司的文化很難一系之間改變 你可以先從你的組開始 從小project開始事後檢討 再慢慢擴展到大project 分享你的經驗給別人

### Build great engineering culture 

作者目前為止看過上千份履歷 面試了好幾百個應徵者  每次面試他都會問他們 你最喜歡跟最不喜歡你前公司的哪些文化 作者把同公司的回答拼湊在一起 去描繪一個公司的樣子 作者也去猜想自己的公司應該會被描繪成什麼樣子

Engineering culture 代表著組裡面共同的價值 而且好的engineering culture會帶來很多好處 工程師會更有動力把事情做完 這會讓他們更有生產力 也代表強的工程師會更願意留下來 強的工程師留下來 engineering culture會更穩固的被實踐 良性循環

那作者interview最常聽到的正面文化包括
1.[Iteration speed 很快](/2017/07/07/invest-in-iteration-speed/)
2.[不間斷的自動化](/2017/08/14/minimize-operational-burden/#不間斷的自動化)
3.[建立正確的軟體抽象](2017/07/29/balance-quality-with-pragmatism/#利用抽象化-簡化複雜度)
4.[利用code review提升程式碼品質](/2017/07/29/balance-quality-with-pragmatism/)
5.[建造一個令人尊敬的工作環境]
6.[程式碼所有權共享](/2017/08/24/invest-in-your-teams-growth/#share-ownership-of-code)
7.投資在自動化測試
8.分配一些experimentation時間 比如InDay或是hackthon
9.建立持續學習進步的文化
10.雇用最好的人

你會發現大多數的文化都已經包含在這本書裡了 這並不意外 因為好的工程師很享受把事情完成的感覺 而高levarage的投資會讓我們把事情更快做完 好的工程師希望能在高質量而且有著良好測試的環境開發 他們希望iteration跟validation的cycle短一點 而且相信自動化會省很多時間 省下來的時間可以做更多有意義的事 

好的文化不是一天兩天造成的 是從一個組一開始的時候就開始 每個工程師一起努力shape的 隨著團隊做的每個決定 我們說的每個故事跟我們養成的每個習慣 慢慢改變 好的文化引導我們做對的決定 快速適應變化 還有吸引人才

當我們專注在high leverage的任務時 我們變成effective engineer 也為effective engineering culture打下地基
