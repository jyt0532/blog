---
layout: post
title: Blackjack - 算牌教學和程式模擬
comments: True 
subtitle: blackjack - counting and simulation
tags: effectiveJava
author: jyt0532
---


今天來聊聊21點

21點大概是全世界賭場裡最受歡迎的遊戲
第一因為它很好學
第二它的賭場期望值(house edge)是相當低的0.3~0.6%
(當然這是在你用最佳策略的前提之下)

0.3%是多低呢 
有名的的輪盤大概是5.26% 
第二低的百家樂也是1.06%
你在blackjack每賭一塊 賭場只賺0.3cent
通常賭場免費提供酒和飲料 這些錢算起來大概是break even
賭場最主要的收入來自非理性的判斷 那些會大幅增加house edge 

BUT 
0.3~0.6%指的是你不受情緒影響 一直照最佳策略玩的賠率

最佳策略如下(資料來源: [WizardPdds](https://wizardofodds.com/games/blackjack/strategy/4-decks/))
![Alt text]({{ site.url }}/public/blackjack1.gif)

如果你今天到賭場只想花錢買那種刺激感 
那這篇文章看到這裡就可以 照著最佳策略玩保證不會輸太多
又很痛快 基本上你在賭場要直接拿出來看著玩也行 我甚至還看過有人直接跟manager要cheatsheet的


一個讓blackjack風靡幾百年的一個更重要的因素是
blackjack不像其他任何的casino game 

house edge是隨時在變動的

house edge是隨時在變動的

house edge是隨時在變動的

很重要所以說三次
偶爾house edge很大 偶爾house edge負的(換言之 玩家有利)
我們的必勝策略是 

在house edge正的時候 下小注 

在house edge負的時候 下大注 

那要如何知道大約的house edge咧
就是鼎鼎大名的算牌

![Alt text]({{ site.url }}/public/21.jpg)
### 算牌原理

算牌原理非常簡單 介紹原理之前 先來了解一下玩家跟莊家的規則差在哪

1.莊家沒到17 需要強制補牌(玩家有利)

2.玩家可以double, split(玩家有利)

3.玩家莊家同時爆 莊家勝(莊家有利)

4.玩家拿到blackjack賠率3:2 莊家拿到blackjack賠率1:1(玩家有利)

知道規則的不同之後 再來要了解算牌的原理是建立在

**10 or ace越多對玩家越有利**

以上論述有兩個理由

1.規則差別一 因為莊家必須強制補 所以10多 很容易爆

2.規則差別二 所有的double都是發生你的牌是10或11(偶爾9) 這些都是希望你能補到10

3.規則差別四 你拿到blackjack賠率好 那當然10跟A越多越好

我們把所有牌分成三種

2~6: 1分

7~9: 0分

10~K,A: -1分

把所有**出現過的牌**照分數加起來 
正越大代表牌堆裡剩下的10,A越多

重點來了 一個可怕的數字出現了
只要**每個deck**的牌堆裡大牌比小牌多一張
也就是true count = 1的時候

house edge就少了0.5%

house edge就少了0.5%

house edge就少了0.5%

!!!

因為很驚人所以說三次
這個波動非常非常的大 
每一張發出來的牌 都在改變這個遊戲的賠率
所以即使是0.6%的遊戲 只要true count是2或以上
就可以下大注了

### true count

這裡要說明一下什麼是true count

true count = (current count) / (remaining deck)

舉了例子

比如說總共三副牌(156張) 已經發出了24張小牌 10張中牌 18張大牌 共發出了52張

那現在的true count就是 (24-18)/(104/52) = 3

那現在你下注的期望值就是正的 house edge是負 這時候就是下大注的好機會

### 道高一尺

所以你很少看到賭場發blackjack只用一兩副牌在發 
通常都是6-8副 然後發到一半(3-4副)左右就重新洗牌

剩下的牌越多副 true count就越容易被稀釋 這就是賭場因應的方法

所以你大概80-90%的時間期望值都是負的

### true count是正的怎麼下

所以這個遊戲對一般賭客來說 house edge永遠是0.6%
但是對於算牌者來說 這是個完全不同的遊戲
那至於對於所有不同的count
count= 2要下多少 count=3要下多少
才會造成max profit呢

這時候就要推薦一個非常用心的(blackjack simulator)[https://github.com/jyt0532/blackjack]啦
如果你是cs的或是你寫過c++
這應該是市面上少數非常好懂的blackjack simulator了
當初想寫這個其實是對於8-pair against 10要分牌很無法理解
就寫了一個全面性的simulator
你可以照你喜歡的strategy玩
照你想要的算牌方法賭
算算期望值 如果滿意 就可以去賭場試試手氣啦

還支援sidebet 

不過先說好side bet的house edge永遠是負 
跟賭場其他遊戲一樣 不像blackjack會變
寫sidebet只是我想練功而已 想贏錢就不要賭sidebet

買保險就沒寫了 鐵定不買的

### 介紹一下我的程式
{% gist 1083024b49d9e26ece7646e118070a53 %}
BlackjackGame的三個參數分別是

幾副牌 發到剩多少percent要洗牌 用哪一個sidebet

Player 的兩個參數分別是

下注的一單位是多少錢 sidebet下多少錢

宣告完Player之後 就可以設定他們的玩牌策略 分成hard soft跟split

這三個都是寫死的2D array

{% gist 53e157c621f2dede59bc78268f033423 %}

hard[12][3] = STAND; 就代表說你12點 莊家3點的時候 你的策略就是stand

soft[13][4] = D_H 就代表你soft 13點(帶張Ace) 莊家4點的時候 你可以double要double 不能double要hit

split[8][10] 代表你拿兩個8 對到莊家10要分牌

你可以任意改動你的策略 然後跑模擬程式 看你最後會賺多少

當然最重要的 如果你算牌的話 這個程式還可以讓你隨著true count改變下注大小

{% gist 66c50c18354546f97355c8b72a67051c %}

把你要下注幾個單位 寫在這個程式裡 就可以照你想的模擬

來看一下跑出來的結果

Dealer通殺
![Alt text]({{ site.url }}/public/blackjack1.png)
Dealer爆掉
![Alt text]({{ site.url }}/public/blackjack2.png)
這是有分牌的情況 
![Alt text]({{ site.url }}/public/blackjack3.png)
Player 2拿了兩張Ace 分牌


大概就是這樣 你就可以模擬個100萬局 並隨著你的true count調整籌碼 看你的策略最後會賺多少

### 建議

如果真的要去賭 再給你一些建議

1.全世界都知道house edge負的時候下大注會贏 你知我知獨眼龍知賭場也知 
要是你每次都在house edge負很大的時候下10倍的注 
你大概五手以內會被請進小房間 
許多書籍是說最大注和最小注只能差到5倍 一個常見的賭法是
count < 2 : 1*
count = 2~3 : 2*
count = 3 ~4: 3*
count = 4 ~5: 4*
count > 5 : 5*
就這麼簡單 跑一次simulator就知道 
結果會明顯優於不算牌 且大多時候為正

2.算牌需要練習 不過很容易可以練得起來

3.當你熟悉這一切真的要進賭場的時候
就不可以再把錢當錢看了 不然會很容易被情緒影響
即使前100次16 against 10你hit爆了 
第101次還是要hit 
你做對的事即使輸了 你會確信只是難免的勝負波動
之後會贏回來
但要是一次本來該贏 做錯事結果輸了 
那house edge就會整個崩潰 
就和其他人一樣成為賭場中blackjack的收入來源

結論:好repo 不fork嗎？ 
[https://github.com/jyt0532/blackjack](https://github.com/jyt0532/blackjack)

<div id="blackjack"></div>

{% include hangover_blackjack_tracking.html %}
