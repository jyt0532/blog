---
layout: post
title: Writing engineering strategy
comments: True 
subtitle: 寫工程策略
tags: staffEngineer
author: jyt0532
excerpt: 本文介紹如何Writing engineering strategy
---


只有少數的公司真正了解他們的engineering strategy和vision 主要原因是大家認為這些東西很難寫 有些人認為這些東西很神秘 有些人認為很世俗 但事實上一個好的engineering strategy很無聊 但是**很容易寫**


要寫一個engineering strategy 你需要先寫五個design documents 然後把相似的地方提取出來 這就是engineering strategy


要寫一個engineering vision 你需要先寫五個engineering strategies 仔細研究這些strategy對於未來的展望 這就是engineering vision


## When and Why

為什麼我們需要寫strategy and vision? 

Strategy是一個工具 這個工具可以讓團隊前進的更快更有信心 這個工具可以讓團隊的所有人在意見分歧的時候快速釐清方向

如果你發現你們組上三番兩次的在同一件事情上沒有結論 這就是時候去寫一個strategy

如果你發現未來太過模糊 讓大家不知道應該投資哪些值得做的事情 這就是時候去寫一個vision

如果你們組上都沒有這些問題 那你應該先去忙別的事 以後再回來處理strategy和vision的問題

## Write five design docs

> Design documents describe the decisions and tradeoffs you have made in specific projects. 

某些公司會稱design doc為RFC或是tech spec 

一個好的RFC應該

1.描述一個明確的問題(specific problem)

2.survey各種可能的解法

3.解釋不同解法的細節

當你不知道某一個專案是否需要值得寫一個RFC時 某些法則可以拿來檢視

1.當這個專案會被很多其他未來的專案所使用

2.當這個專案會對於你的使用者有著重大影響

3.當這個專案需要一個月以上的engineer effort

如剛剛所說 五個design doc就會是一個寫好strategy的元素 因為design doc有著strategy缺乏的東西 - **detailed specifics grounded in reality**

你知道的 常常兩個工程師對於一個抽象的strategy有著兩種不同的解讀 但對於實作一個解法則意見很難分歧

以下是作者提供的 寫好design doc的好建議

1.Start from the problem

Problem statement寫得越清楚 solution就會越清楚 當你覺得解法不那麼顯而易見時 回頭看看你的問題是不是定義的不夠好 

當你真的卡在定義問題的時候 找五個從沒看過這個問題的工程師 問他們what is missing 你會發現旁觀者清

2.Keep the template simple 

大多數的公司對於RFC都有一個模板 通常都很值得follow 但要注意公司的模板是不是同一時間注重太多goals 一個overloaded的模板會讓大家寫RFC的第一步感受到挫折 

建議一開始的模板應該要簡單 讓寫RFC的人可以選擇專注在most useful section. **只在風險度大的專案中堅持寫下所有細節**

3.Gather and review together, write alone: 

通常你不會一個人就了解所有的相關知識和細節 所以我建議當你在深入研究某個主題之前 先跟周圍的人收集feedback 特別是那些未來需要依賴你的output of design doc的人

> Gather perspectives widely but write alone

然而 收集完feedback之後 還是要記得自己把RFC寫完 因為每個人都認爲自己是個好writer 但事實上要所有人一起把所有想法組織起來是很困難的 最好就是由一個人寫 然後讓其他人review

4.Prefer good over perfect: 

建議大家 與其花很多時間寫到你認為完美再給大家看 不如先寫個ok的版本趕快先分享

別忘了 當你在看別人的design的時候 不要一開始就預期別人寫的就應該跟你心中的最好解法一樣 特別是當你越資深 你就要越注意不要對別人寫的design太過toxic 要有耐心的給別人建議 讓別人更好



寫一個好的design需要時間練習 如果你真的想進步的話 作者的建議是 當你實做完這個專案之後 **回頭重新讀你的design 找出那些你當初忽略的地方** 或是實作起來跟當初設計的時候不同的地方 然後找出當初忽略的原因

然後多寫一些design 你就會越來越好

## Synthesize those five design docs into a strategy

當你們的組織中有五個design doc之後 坐下來好好的看一下這五個design doc 找尋這些文件中 有爭議的部分 衝突的部分


好的strategy會引導大家如何從tradeoff中選擇 並提供很好的解釋和context

不好的strategy就只是告訴大家應該做什麼不應該做什麼 但卻沒告訴大家為什麼

沒有context的話 你的strategy就會越來越難以理解 而當context開始轉換的時候 大家也不知道strategy應該相對應的如何轉換

你可以看看大公司怎麼決定他們的strategy:

[Slack](https://slack.engineering/how-big-technical-changes-happen-at-slack/)

[Stitch Fix](https://multithreaded.stitchfix.com/blog/2019/08/19/framework-for-responsible-innovation/)

以下是作者給大家的建議

1.Start where you are

現在就開始寫 如果你的藉口是 有些文件根本還沒出來或根本不存在 你就必須知道 每個不存在的文件都有它的原因 不管你第一版寫出來的內容是怎麼樣 都會需要有很多次的改動 所以現在就開始寫

2.Write the specifics

盡量寫出specific一點的東西 當你發現你越寫越generalize 就該停筆 當你發現你沒什麼specific的東西可以寫 那就去多生一些design doc出來

> Specific statement create alignment, generic statement create the illusion of misalignment

3.Be opinionated

寫strategy時要武斷一點 如果不夠武斷 那麼這個strategy對於未來的大大小小的決定沒什麼幫助

4.Show your work

讓大家知道為什麼你做了這些決定 訂下這個strategy的理由是什麼 context是什麼

展現這些東西不僅讓大家對你的決定更有信心 在未來context shifts的時候 大家也知道該做什麼相對應的改變


講完了四個建議之後 常見的問題是

什麼事情值得去寫stragety?

我們來舉幾個例子

“什麼時候我們需要寫design doc” 這個問題本身就可以寫一個strategy

“在哪些use case的情況下我們要用哪種database” 這個問題也可以寫一個strategy

“我們什麼時候該從monolith migrate to services” 這個問題也可以寫一個strategy

你可以有很多的strategy 來幫助組織裡的人做出決定 節省未來的時間 當你發現某些strategy幾乎沒被用到 也可以簡單地把它deprecate掉 多寫總比沒有好



## Extrapolate five strategies into a vision

當你有越來越多的strategies 你就會發現很難釐清各個strategy之間的關聯互動或因果關係

也許某個strategy要我們少點自己寫的software 多用cloud上的solution

但另一個strategy也許是說 database的複雜性應該盡可能的被降低

那如果團隊遇到個選擇 發現一個複雜度高的database在cloud上 而複雜度低的不在cloud上 那該怎麼辦



拿出你們最近的五個strategy 把它們的tradeoff extrapolate到兩三年之後 你會越來越容易去解決那些彼此的衝突 進而寫成engineering vision

一個好的vision會讓大家很好理解目前存在的每個strategy之間的關係 也讓未來的新strategy更加經的起推敲

作者也要在此給幾個寫vision的建議

1.Write two to three years out

小組 大組 甚至公司都改變得太快 如果想得太遠的話 會讓人失去目標而憂心重重 但如果想得太近也不行 如果一個strategy六個月後就會失效 那也沒啥用 想想你們六個月內是能寫出幾個strategy? 試著把目標定在2-3年之後

2.Ground in your business and your users

一個好的vision應該不只跟business息息相關 跟使用者也應該息息相關

3.Be optimistic rather than audacious

vision應該要ambitious而不是audacious. 應該要考慮有限資源的情況下的best possible version

4.Stay concrete and specific

越specific的vision越有用 generic的vision很容易被大家同意 但組織裡真正遇到問題的時候 很難幫忙得出有用結論

5.Keep it one to two pages long

大家不喜歡讀冗長的文件 如果你光vision就寫了五六頁 那很容易大家都不會讀完

Vision文件本身小一些 提供一些reference讓那些有興趣了解更多的人去dive deep


寫完之後呢 你也不要太爽的就急著分享給組織裡的所有人

沒錯 5個design doc才有一個strategy 五個strategy才有一個vision 作者理解好不容易寫出一個vision 你有多興奮 但作者要先打個預防針 你其實很容易被澆冷水 因為其實大部分的人都不會有所回覆

你想想 基本上對你的vision有興趣的人 都是那些寫strategy的人(只佔了很小一部分) 而且一個好的vision基本上看起來都很無聊

所以 不要藉由大家多麼興奮來判斷你的vision的好壞 

那麼應該怎麼判斷呢

找一個兩年前的design doc 再找一個你的vision推出來之後才被寫出的新design doc 如果感覺起來有明顯的進步 那麼你的vision就很好


## 總結 

恭喜你 你已經了解了Staff-plus的人是如何寫出engineering strategy和vision 就算你不需要親自去替組織撰寫strategy 或 vision 你今後也不會對別人寫出的strategy和vision那麼的陌生

