---
layout: post
title: The Pragmatic Approach - 務實的方法
comments: True 
subtitle: A Pragmatic Approach
tags: pragmaticProgrammer
author: jyt0532
excerpt: 本文記錄了讀完The Pragmatic Programmer的讀後筆記
---

千呼萬喚 終於等到[Pragmatic Programmer 20週年紀念版](http://books.gotop.com.tw/v_ACL057200) 如果沒聽過這本書 你大概也聽過[程序員修煉之道︰從小工到專家](https://www.books.com.tw/products/CN10279423)這本暢銷了20年的書 終於等到了再版 

在再版裡面 刪掉了比較過時的內容和範例 收集了20年來收到的feedback 在讓這本書的內容也可以適用於2020年的程序員 但在我細細品嚐後發現 其實很多人生的哲學並不是只適用於程序員 各行各業看了都可以有所收穫

因為每個篇章的篇幅都不長 所以筆記也用條列式紀錄

本篇的圖片以及程式碼來自於原書內容

{% include copyright.html %}

## 第二章: 務實的方法

本文會提到作者認為在所有等級的軟體開發中都適用的方法


### 優秀設計的精髓

什麼是好的設計呢 好的設計就是**容易被改動的設計 ETC(Easier to Change)**

有了這個大方向後 我們就可以輕易地判斷 比如說

為什麼降低耦合是好的設計? 因為減少相關性我們就更好更改

為什麼[SRP](/2020/03/18/srp/)是好的設計? 因為如果需求更改 我們只需要改變一個類別

為什麼好的命名重要? 因為你需要閱讀程式碼才可以去改 而好的命名讓你更好閱讀

#### ETC是個價值觀 不是規則

價值觀幫助你做決定 當你不知道A選項跟B選項哪個比較優 你總是應該選擇那個**容易修改**的那個 這樣無論發生什麼(最差情況是要改選另外一個選項) 這段程式都不會成為你未來的絆腳石


### DRY(Don’t Repeat Yourself)

我們認為 要可靠的開發軟體 使我們的開發更容易理解和維護 唯一的方法就是遵循DRY

> 在一個系統中 每一條知識都必須有一個單一的明確的權威的描述
>
> Every piece of knowledge must have a single, unambiguous, authoritative representation within a system. 

如果不遵循DRY 代表說在兩個或多個不同地方表達相同的內容 那如果改變了其中一個 其他的也要跟著改 很不方便

#### DRY的範圍不只是程式碼

DRY不是只是單純地告訴你不要複製貼上那麼簡單 DRY指的是**知識**以及**意圖**上的重複 因為你可能在兩個不同的地方表達相同的東西 只是使用了不同的表達方式

判斷方式就是當你改一個東西 你需不需要同時修改其他地方

#### 重複程式碼

請在下列程式碼中找到重複的地方

{% highlight python %}
def print_balance(account)
  printf "Debits: %10.2f\n", account.debits 
  printf "Credits: %10.2f\n", account.credits 
    if account.fees < 0
      printf "Fees: %10.2f-\n", -account.fees
    else
      printf "Fees: %10.2f\n", account.fees
    end
    printf "          --\n"
    if account.balance < 0
      printf "Balance: %10.2f-\n", -account.balance 
    else
      printf "Balance: %10.2f\n", account.balance 
    end
end
{% endhighlight %}

問題還不少 我們一個一個解決

首先是負數的問題

{% highlight python %}
def format_amount(value)
  result = sprintf("%10.2f", value.abs) 
  if value < 0
    result + "-" 
  else
    result + " " 
  end
end
def print_balance(account)
  printf "Debits: %10.2f\n", account.debits
  printf "Credits: %10.2f\n", account.credits
  printf "Fees: %s\n", format_amount(account.fees)
  printf "          --\n"
  printf "Balance: %s\n", format_amount(account.balance)
end

{% endhighlight %}

再來下一個問題 就是printf中的欄位寬度 應該使用現有的函示

{% highlight python %}
def format_amount(value)
  result = sprintf("%10.2f", value.abs) 
  if value < 0
    result + "-" 
  else
    result + " " 
  end
end
def print_balance(account)
  printf "Debits: %s\n", format_amount(account.debits)
  printf "Credits: %s\n", format_amount(account.credits)
  printf "Fees: %s\n", format_amount(account.fees)
  printf "          --\n"
  printf "Balance: %s\n", format_amount(account.balance)
end

{% endhighlight %}

還有沒有呢 有的 如果今天客戶想在標籤跟價格之間保留額外空格 我們必須改五行 聽起來不優

{% highlight python %}
def format_amount(value)
  result = sprintf("%10.2f", value.abs)
  if value < 0
    result + "-" 
  else
    result + " "
  end
end
def print_line(label, value) 
  printf "%-9s%s\n", label, value
end
def report_line(label, amount)
  print_line(label + ":", format_amount(amount))
end
def print_balance(account) 
  report_line("Debits", account.debits) 
  report_line("Credits", account.credits) 
  report_line("Fees", account.fees) 
  print_line("", "———-") 
  report_line("Balance", account.balance)
end

{% endhighlight %}

這樣我們幾乎就沒有DRY的問題了 


第二個範例 我們再來看看下面這段程式碼

{% highlight python %}
def validate_age(value): 
  validate_type(value, :integer) 
  validate_min_integer(value, 0)
def validate_quantity(value): 
  validate_type(value, :integer) 
  validate_min_integer(value, 0)
{% endhighlight %}

這程式碼驗證了使用者的年齡以及購買數量 都應該要是大於零的整數 

這看也知道重複了吧?? 長得一模一樣還不重複???

答案還真的是沒重複 因為這兩個表達的是不同的概念 一個是年紀 一個是購買數量 是兩個不同的**知識**


記得 **不是所有重複的程式碼都是知識複製** 我們要避免的是知識的重複 而不是程式碼的重複

#### Documentation中的重複

如果程式碼就可以清楚地知道意圖 那就不用再寫註解 因為要是你需要修改意圖的話 需要修改兩個地方

#### 資料結構中的重複

來看一下這個可愛的類別
{% highlight python %}
class Line {
  Point start;
  Point end;
  double length;
};
{% endhighlight %}

乍看之下好像合理 但其實長度是由起終點定義的 這樣如果你改變起點或終點 長度很可能要跟著變 不優 應該要改成這樣


{% highlight python %}
class Line {
  Point start;
  Point end;
  double length() { return start.distanceTo(end); }
};
{% endhighlight %}

#### 開發者的重複

最難被處理和檢測的類型 大概就是某些功能不經意地被重複了 這個問題最好的解法就是開發人員之間必須頻繁地進行交流

你想創造的是容易被找到和重用現有內容的環境 而不是自己去寫 如果沒有這樣的環境 人們就不會去重用程式碼


### 正交性

如果你想產出一個易設計 建構 測試 擴展的系統的話 正交性(Orthogonality)是一個關鍵的概念

什麼是正交性呢 這是個幾何學的概念 如果兩條直線成直角那就是正交的 比如說x軸跟y軸 當有物體在平面上往北移動不會影響到x值 往右移動不會影響到y值

在計算機領域中 這代表著去耦合 其中一個面向的變化不影響另外一個的話 兩個事物就是正交的 比如說設計良好的系統中 資料庫和使用者介面就是正交 

> 消除不相關的東西對彼此的影響

正交有什麼好處呢

1.每次需要修改的時候 只需要修改特定部分 減少開發測試時間

2.當合併一些正交元件時 會有意想不到的收穫 假設元件A可以做M件事 元件B可以做N件事 如果元件A和元件B是正交的話 把它們組合起來 他們就可以做M*N件事 如果兩個不是正交 代表有重疊 那組合起來能做的事就比較少

3.如果元件的一部分的程式碼掛了 不會影響到其他正交的元件

#### 寫程式時

當你寫程式的時候 你應該時時注意要降低應用程式正交的風險 否則你會無意中重複了其他模組中的功能 有幾種技術可以保持正交

1.程式碼去耦合

2.避免全域資料:每次你的程式碼引用的全域資料 你都把自己跟那些也使用全域資料的元件綁在一起

3.避免相似功能: 當程式碼很接近 只是演算法不同 就可以考慮使用[Strategy Pattern](/2017/04/07/strategy/)

養成不斷批評自己程式碼的習慣 尋找機會重構它 改善結構和正交性

### 可逆性

一個專案裡面 總是有著某些"關鍵決定" 這些關鍵決定做出來之後 幾乎其他剩下的決定都是圍繞著這個關鍵決定 所以這些關鍵決定很難被逆轉 

比如說你選擇後端使用特定廠商的資料庫 之後你的很多決定都會跟著這個走 你要反悔這個關鍵決定非常困難 成本會非常大

本書會提出非常多的觀點 比如說DRY, 去耦合以及外部設定 這些觀點讓我們不必太早做出關鍵的 不可逆轉的決定

這很重要 因為我們很難在一開始就做出正確的決定 甚至當下是正確的決定但過一陣子之後就不正確了 比如說你當初選擇的資料庫公司倒閉了等等

> 沒有所謂的最終決定


#### 靈活的架構

要如何在架構上保持靈活性呢 就是要把抽象做好 比如說把第三方API隱藏在抽象層之後 或是把程式碼分配成一個個元件 套用[SOLID](/toc/clean_architecture/)

### 曳光彈

> 曳光彈是在子彈底部裝有少量煙曳光藥劑的子彈或砲彈。發射時，曳光藥劑點燃發亮，使彈丸的軌跡在白天也可見，而在夜間發射時則非常明亮。 - [WIKI](https://zh.wikipedia.org/wiki/%E6%9B%B3%E5%85%89%E5%BD%88?fbclid=IwAR3U-csiu2y5U6xqIbiBYz_7ddksBpV3igUNOsNKH6gM33pTZEu0RlbfhlQ)

在這個章節 讓我們把開發軟體比喻成射擊中目標

軟體開發中有個術語叫做曳光彈開發(tracer bullet development) 用來直觀的說明在目標不明確的實際狀況下**得到即時回饋**的重要性

另外一種做法 是在發射前 製作大量的文件 逐條列出每一項要求 把每一項未知列出 控制環境  經過許多精密的計算之後再開槍希望可以擊中目標 

務實的程式設計師更傾向於使用曳光彈開發 


#### 在黑暗中發光的程式碼

考慮到當今專案的複雜性 以及大量的外部依賴項目工具 曳光彈就變得更重要了 

來個例子 我們想要建立一個專案可以讓我印出"hello world" 就是這麼基本的需求

![Alt text]({{ site.url }}/public/tracer-bullet.png)

即使是最簡單的功能 我們還是需要把每一層需要的程式碼完成 對角線代表的是這個功能會通過程式碼的路徑 我們只需要完成這個箭頭會經過的地方就可以

這其實就是常聽到的MVP概念



### 評估

因為跟你合作的人喜歡問你沒有標準答案的問題 比如壓縮100Mb的文字需要多久 完成專案需要幾個月 等等的問題 所以你需要熟練評估的技能

#### 多準叫夠準

一個有趣的事情是 你使用的**單位**會影響人們對結果的解釋 比如你說

"喔大概需要130天" 那對方大概會認為120-140天可以完成

"喔大概4個月" 那對方大概會認為三個月到五個月才會完成

所以本書建議這樣給單位

| 期間  | 使用單位|
| --- | --- |
|1-15天 | 天 |
|3-6週|週|
|8-20週|月|
|20週以上|回答之前要審慎評估|

#### 評估值從何而來

1.多去問問已經做過類似事情的人: 借鑒他人經驗

2.引入不確定性: 比如"假設沒有交通事故並且車內有汽油 我應該在20分鐘之內到達"

3.了解範圍: 比如"您要求對X進行估算 但是這個看起來像 X 的變體 的 Y 可以在大約一半的時間內完成 而您僅損失一項功能"

4.每次評估過後 記錄你的評估成果: 如果評估錯誤不要無視 而是要找出原因


