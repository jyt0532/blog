---
layout: post
title: Effective Engineer - Validate Your Ideas Early and Often
comments: True 
subtitle: 早期且頻繁的驗證你的想法
tags: effectiveEngineer
author: jyt0532
---

這篇是Effective Engineer - Validate Your Ideas Early and Often章節的讀書筆記

cuil 是個在2008年紅極一時的Google-killer 超過120b的網頁有indexed 號稱是google的3倍 
在眾多期待中 2008年7月28號 開站當天 當天因為流量過大 網站當了好幾次

當時crawling indexing和ranking等等的功能run在幾千台機器上 完全被traffic壓得喘不過來
當時AWS也沒有那麼純熟 使用者搜尋了很多不重複的query 比如說user自己的名字 因為重複性太低 in-memory cache也完全爆了
讓search變得更慢 某些index的shard也崩了 所以search result就有很多空缺 該出來的結果沒出來 總之就是爆了 挺有名的新聞

Cuil是個失敗的實驗 當作者問了創辦人之一Levy 從這個事件學習到了什麼 第一個就是validate the product sooner 
因為當初他們想讓cuil造成很大了市場衝擊 所以消息都很少洩露給媒體 他們也沒有任何的alpha tester去玩他們的產品 
在launch之前 沒有人跟他們說search result的quality不夠 user根本不管你們indexed的網頁數量是多少 結果沒出來就是沒出來 
沒有早期的驗證產品 就是花太多時間在user不在乎的事情上面

Levy下一個工作是在BloomReach 他需要建造一個marketing platform去幫助e-commerce優化搜尋結果跟營收 這個產品還有很多不確定性  但這次Levy不再花好幾年去建造一個不知道user喜不喜歡的東西 這次他們只做MVP(Minimal Viable Product)就給Beta測試了 拿到他們的feedback之後 他們再決定下一步怎麼走

越早拿到feedback越好 了解你客戶真正需求 [第四章](/2017/07/07/invest-in-iteration-speed/)講的是怎麼加快速度完成事情快一點 這一章講的是怎麼讓我們完成的是對的事情

### 找簡單的方式去validate你的work

每一個iteration cycle越短 你就越快學到經驗 知道自己的錯誤 每一個iteration cycle越長 你越晚知道你哪裡犯錯 或是做了錯誤的假設 你浪費的時間會成指數成長

當你處在一個大型專案時 要不定時的問自己 **我能不能花一小部分的時間去搜集一些資料去證明我做的東西會work** 我們太常因為要早點deliver 所以不願意花那10%時間去驗證 結果有時候90%都浪費

MVP的好處就是可以花最少的時間收集到最多的使用者feedback 這其實跟我們[leverage](/2017/07/31/focus-on-high-leverage-activities/)的定義有異曲同工之妙 當你建造一個MVP的時候 你希望專注在high leverage的任務 這代表你需要儘早驗證你們關於使用者的假說

有時候建造MVP也需要一些創造力 當Drew Houston創造Dropbox之前 市面上已經有太多共享文件的工具 但他相信使用者喜歡seamless的使用者體驗 但他怎麼驗證呢？他做了一個四分鐘的影片 展示了file怎麼在mac, windows跟網頁上無縫銜接 一個晚上使用者從5k變75k 當時他就確定這個產品會成 然後再開始投資之後的事情 演變成現在的Dropbox

也許不是每個人都在startup工作 但validate our work with small effort的準則卻通用 比如說你想migrate database因為MySQL的scalability已到上限 想轉成NoSQL 這種migration都會花很多時間 你要怎麼讓你自己在產品出來前就有信心這可以達成你的目標呢

常見的是花10%的時間去建一個small and informative的prototype 取決於你project的大小 prototype可以是測量performance 或是比較舊版新版的code評估新東西的好處 做完這個之後你就會有方向 這個project到底能不能達成目標 或是其實我們只需要小部分的migrate 根本不需要動全部的東西

有名的例子是Asana 有點像是trello的產品 當初在考慮要不要在首頁擺上Google Signup button 希望可以增加會員數 但他們並不知道這樣會不會達到目標 所以他們決定放一個假的signup button 當有人點的時候 就跳出訊息”謝謝你的支持 這個feature很快會上” 他們去track按這個按鈕的次數 然後只有在確定用戶會增加的情況 他們才開始花心力去做完整版

結論就是花一些時間 去收集資料 去驗證你project的assumption

### 用A/B test去驗證產品的變化

2012年的時候 歐巴馬競選團隊缺錢 他們想要請大家捐點錢 他們當時不知道email的標題要用什麼才可以得到比較多的募款 當時候選人有

“Deadline: Join Michelle and me”
“Change”
“Do this for Michelle”

他們當時有17個標題再選 他們測試在一小群他們的subscribers 
但最後 他們用了完全不一樣的標題”I will be outspent” 他們怎麼決定的呢

他們當時請了20人的engineer team來建造A/B test工具 當時測試了10000多種不同的變化 包含標題 捐款金額 格式 字體大小 按鈕等等 每封email都在18個小group測試 最好的那個總是比最差的那個好了5-7倍左右

測試想法的觀念不只應用在email上 在產品開發也是 即使是一個測試完整 設計良好而且可擴展的產品 如果使用者不engage或是顧客不買 那也是沒什麼用 

有一個在你改變產品時(add/remove feature)常見的問題 雖然你看到metric有變化 但你不知道有多少部分是起因於你產品的變化 還是因為某個新聞報導 還是舊的bug 該怎麼辦呢 一個常見的孤立其他變因的方式就是A/B test

A/B test裡 **隨機**的小部分使用者用新feature 選擇這些新fearure的使用者的時候不能有bias很重要 這樣的話你從實驗組跟控制組看到的metric差別就完整的是起因於產品的變化

A/B不只可以讓你選擇你要launch哪個feature 即使你已經確定某個feature很好要上prod 你還是可以由A/B測試知道實際上到底好了多少 可以讓你之後評估這個方向要不要繼續走 不要覺得這個feature很猛就直接上 還是要A/B test拿到數字

作者在Quora就是建構了A/B testing的framework 他們抽象了實驗定義 寫工具讓更多variation同時分配到不同group 還有自動化收集feedback 這些最大化了他們每個iteration可以拿到的feedback. 對於每個改動他們可以run幾百個實驗

A/B test讓我們validate我們的產品想法 把未知變成已知 讓我們對於產品的走向更有自信

### 小心一人團隊

既然validating early and often這麼重要 那一個很常見的anti-pattern就是一人團隊

作者在google實習的時候 他建了一個search的功能 他當時做了很認真 因為規模很大 他對於code review也沒什麼經驗 別人也不太管他做了什麼 實習結束前他教了一個上千行code的code review 當時google內部的系統還寄了一封信給他的manager 標題是”Edmond sent you a ginormous code review” 

當時作者還非常滿意 覺得自己的impact比其他intern大多了 但當時其他人都不可置信地問他 要是他的設計從根本上就有錯怎麼辦 mentor也許根本沒時間看你的code 就算他看了 你有時間修bug嗎 要是到最後你的code沒辦法check in怎麼辦 當時作者心都慌了 可能整個暑假做的都白費了

雖然最後他的code check in了 但作者學到的很大經驗 就是應該commit iteratively而不是一次commit很多code 一次commit很多code會讓你做白工的機會增加 而且要是他分批上傳的話 mentor也不用花那麼多時間一次看那麼多code 

很多時候你必須獨自完成一個project 這的確是省了一些溝通的時間 也帶來了個人的彈性 有某些公司升職的條件是能不能獨立帶領一個project 但你必須很小心一人project的風險 比如說你會比較不好拿到feedback 通常如果一個人對於你的project不熟 他不太會去給你review 而你也會傾向在project快要完成的時候才給他review 但要是他那時候才跟你說有些設計上的基本問題 你就會浪費很多時間

一人團隊 還有其他的風險 當你遇到問題的時候 只能依靠自己 因為別人要花時間去懂你的context 或是遇到bug的時候 有人可以分享這個bug是怎麼回事 瞭解你為什麼struggle 都比你自己獨自找bug 解bug都還容易忍受 而且當你知道你的組員需要你的時候你會更有責任感 種種的好處都是個人團隊無法擁有的

即使你真的必須要一人工作 還是有些建議可以給你

1.Be open and receptive to feedback: 如果你防禦心太高 會讓你很難聽到feedback 而且同事未來會更不想給你feedback 要把評論跟建議當作進步的機會而不是批評

2.Commit code early and often: 大量的code非常難review 需要花更久拿到feedback 而且如果有設計上的問題 會非常浪費時間 學著make iterative process 從iterative process裡面得到feedback 

3.Request code reviews from thorough critics: 每個工程師對於code review的態度跟內容都不一樣 如果你很急著ship一個feature 你可能會找的看得沒那麼仔細的人給你shipit 但如果今天你想要真的精進你的code 或是你想要證明你的approach能work 你就應該找review的很仔細的 甚至會帶些批評的 這些都是進步的機會

4.Ask to bounce ideas off your teammates: 最直接得到feedback的方法就是去跟他要 看到有閒著的組員就去跟他白板討論你的idea 跟別人解釋你的idea會讓你更了解他 甚至有時候在你解釋的過程中你會找到一些解釋不通的地方 大多數的人都希望be helpful而且參與一些有趣的問題 如果你想要常常得到feedback 你要尊重你同事的時間 先自己練習一下如何講解這個題目 討論完後還可以把別人的feedback整理好share給其他組員 讓他覺得被重視

5.Design the interface or API of a new system first:在interface設計完之後 先設想client會怎麼用你的 建立一個具體的藍圖 可以提早知道有沒有錯誤的假設或是錯誤的requirement

6.Send out a design document before devoting your energy to your code: 文件不需要是非常正式 也可以是一封詳細的email 但要完整到讓你的讀者知道你想要做什麼 可以問問題

7.Structure ongoing projects so that there is some shared context with your teammates: 當然就是讓你的project能見度高一點 不要都是你自己玩

8.Solicit buy-in for controversial features before investing too much time: 在早期就應該把你的idea分享出來 有時候工程師會誤解覺得不該分享太多(因為某些office politics) 但就leverage來說還是非常值得分享 沒有從那些真正了解領域的人拿到feedback很可能就會導致浪費時間

這些策略都是讓你獨自工作的時候可以提早拿到feedback 但同樣的策略在團隊工作也很有用

> Software development is a team sport   
>
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; - Brian Fitzpatrick and Ben Collins-sussman

### Build feedback loop for your Decisions

在之前的文章中提到 無論你是在實作還是在開發產品 去驗證你的想法是非常重要的 但你也可以把validation從驗證想法推廣到**驗證每一個做的決定**

隨著你的職位往上爬 你的每一個決定會越來越重要 而且也越來越難做決定 而你若是沒有validation feedback 你每個決定就只是在猜而已

當Hoofien是Ooyala的senior VP的時候 他做了非常多個實驗 比如說 最完美的team size, tech lead應不應該同時當manager, reliability engineer, designer, PM應不應該跟development team綁在一起, 什麼時候應該用Scrum

甚至他嘗試把最好的工程師的薪水double 想看看能不能讓team更猛
他做每個實驗後都積極的去拿feedback

這裡就不分享他的實驗結果 畢竟每個公司每個組對於同樣問題的答案都會不一樣 重要的是我們常常會把我們認為對的東西apply到其他人身上 但即使是這種management相關的東西 都應該要跟code一樣謹慎 需要定時地拿feedback 然後改進

Validation 就是先有個可能可以work的假說 設計實驗去test假說 知道實驗結果好壞長什麼樣子 跑實驗 從結果學習 你不一定可以像A/B test那樣有非常明確的結論 但你還是可以把”猜猜看” 變成 “ 合理的決定”


