---
layout: post
title: Effective Engineer - 投資迭代速率
comments: True 
subtitle: prioritize regularly
tags: effectiveEngineer
author: jyt0532
---

這篇是Effective Engineer - Invest in Iteration Speed章節的讀書筆記

這篇文章講的是迭代(Iteration)速度的重要 
迭代指的可以是從code committed 到上production的過程 
也可以是code改完到build起來的過程 

### 持續交付

作者先提到Continuous Delivery的重要性 
CD指的就是每次一有小改動就release出去 
利用完善的unit test/integration test/regression test來保證code的正確性 
每次release小部分的好處是有bug或是business metric被影響了馬上可以找到

那要是有個feature是比較大的 
一定要一起release怎麼辦 

通常大公司都有一些A/B wrapper 可以把每個incoming request assign一個tag 
tagA就走flowA tagB就走flowB 所以你可以一個個commit release後但是只ramp公司內部 全公司公測 
所有commit都release後 再改變tag的比例慢慢ramp到所有user
等到所有user都是走你新的flow後 在push code把舊的flow拿掉

拿一個很常見的Scenario為例子
假設今天你想要改變你database某個table的schema
你需要以下步驟

1.創一個新table

2.deploy **同時寫進兩個table 讀old table**的code(source of truth是舊table)

3.搬資料 從舊table複製到新table(offline process)

4.deploy **寫new table 讀new table**的code (source of truth是新table)

5.拿掉舊table

這種migration都要非常小心 這麼多的release 只要任何一步走偏了  
就有可能會永遠失去user data或是user讀不到data等(那你可能永遠失去這個user)
我們希望migration越快越好 但又要保證資料的正確性 那CD就是必然的結果 
因為CD的好處就是用完善的測試加速迭代時間

### 投資在省時間的工具

作者說他問過不少engineer leader 把時間花在什麼上面回報最高
最常聽到的答案是**工具**
同時工具也是判斷一個人成不成功的一個metric
成功的人會去寫工具 給其他人使用

**大方向是只要某一件事情需要手動做兩次以上 你就應該寫個工具**


好的公司都有專門的team去寫tool 即使是節省1分鐘的build長期下來也相當可觀

hot reload和continuous integration都是省時間的工具 包括有提供interactive programming environment的語言都比沒提供的更有優勢 你只是想測試一個小api你不需要向java一樣要寫好 compile過 run等等

> hot code reload: application server 可以自動換到最新版本而不需要重新build 對於漸進式開發非常方便

> CI: 每個commit都會rebuild code base, 跑所有test

### 熟悉你的programming環境

這個章節主要在說明 每天我們都花很多時間在基本但又無可避免的操作 包括

1.用version control tool看version的變化

2.compile, build code

3.跑unit test
 
4.code改變以後 Reload 網頁

5.測試一個expression的結果

6.讀funciton的doc

7.jump到一個函數的definition

8.reformat code

9.找誰會call過這個function

10.rearrange你的桌面視窗

11.navigate到一個file的某個地方

這些動作你的生涯中可能會做上萬次 你越早把這些時間省下來越好

作者也列出一些建議 如何精通你的開發環境

1.非常精通你現在在用的IDE: 問一個比你有效率的工程師 
看可不可以在旁邊看他開發一下子 學習他用的技巧並練習

2.學至少一個high level的語言: Python Ruby處理一些雜事比C++ JAVA快太多

3.熟悉UNIX Shell command: grep, sort, uniq, wc, awk, sed, xargs, find等等 
學習怎麼pipe在一起

4.能不要用滑鼠就不要用: 把手移去滑鼠的時間很浪費

5.把你目前需要手動的workflow自動化

6.善用interactive interpreter: 有什麼idea或是想試試某個api 就用你那個語言的interpreter 可以省不少時間

7.讓你的unit test可以只run跟你的改動有關的code: 不知道有什麼tool可以做到這件事 有誰知道的話麻煩留言給我 我可以研究一下

### 非engineering的bottleneck

最後一個section講到 往往一個project最大的瓶頸都不在engineer 而是engineer和PM,team leader或是decision maker的溝通 很多時後都是project快做完了之後 上頭的人才跟你說要改feature 那你之前省吃儉用的時間都沒了 

> 過早最佳化是萬惡的根源&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; -- Donald Knuth

### 結論
找出你project iteration cycle裡面的最大瓶頸 看是開發者工具, 團隊之間的溝通, decision maker的同意或是公司組織的process

找到之後再去最佳化
