---
layout: post
title: Pragmatic Paranoia - 務實的偏執
comments: True 
subtitle: Pragmatic Paranoia
tags: pragmaticProgrammer
author: jyt0532
excerpt: 本文記錄了讀完The Pragmatic Programmer的讀後筆記
---

千呼萬喚 終於等到[Pragmatic Programmer 20週年紀念版](http://books.gotop.com.tw/v_ACL057200) 如果沒聽過這本書 你大概也聽過[程序員修煉之道︰從小工到專家](https://www.books.com.tw/products/CN10279423)這本暢銷了20年的書 終於等到了再版 

在再版裡面 刪掉了比較過時的內容和範例 收集了20年來收到的feedback 在讓這本書的內容也可以適用於2020年的程序員 但在我細細品嚐後發現 其實很多人生的哲學並不是只適用於程序員 各行各業看了都可以有所收穫

因為每個篇章的篇幅都不長 所以筆記也用條列式紀錄

本篇的圖片以及程式碼來自於原書內容

{% include copyright.html %}

# 第四章: 務實的偏執

> 你無法寫出完美的軟體

接受這句話吧 接受之後我們再來好好討論 完美跟務實的軟體之間的距離


有些人把寫程式比喻成防禦駕駛 你需要假設其他人都會犯錯 在麻煩出現之前就做好準備 要能預測意料之外的事

但務實的程式設計師除了防禦駕駛之外 **他連自己都不信** 因為他們深知他們無法寫出完美的程式碼 所以他們就會為自己的錯誤建立防禦機制


## 死程式不說謊

(Dead Programs Tell No Lies 其實是在玩 Dead Men Tell No Lies 的梗)

你是不是偶爾會注意到  有時候當你自己發現問題之前 別人就已經發現了你的問題了 因為我們自己很容易陷入先入為主的假設之中

比如某件事是絕對不可能發生的 不需要去處理 但我們其實應該要依循防禦型程式設計原則 我們需要確認資料就如同我們所以為的那樣 確認已發佈的程式碼就是我們認為的程式碼 所有相依項目都是載入正確的版本

當然你還是可以說服自己那些例外都不會發生 但務實的程式設計師會告訴自己 如果發生錯誤 就會發生非常非常糟糕的事情

### 早期崩潰

某些開發人員認為應該先捕獲所有異常 log下來後再重新引發它們是一種很好的樣式 比如


{% highlight python %}
try do
  add_score_to_board(score);
rescue InvalidScore
  Logger.error("Can't add invalid score. Exiting");
  raise
rescue BoardServerDown
  Logger.error("Can't add score: board is down. Exiting");
  raise
rescue StaleTransaction
  Logger.error("Can't add score: stale transaction. Exiting");
  raise
end
{% endhighlight %}

但一個務實的程序員應該這樣寫

{% highlight python %}
add_score_to_board(score)
{% endhighlight %}

有兩個原因

1.你不該讓錯誤處理的光芒完全遮蓋了原本的主程式

2.新的程式碼耦合性變低 如果之後要新的例外要處理 就不需要加在這裡 應該直接拋出 直接傳播


> Crash Early

### 不拖泥帶水

儘早發現問題的好處之一就是你可以更早地讓程序崩潰 而讓程序崩潰通常是您可以做的最好的事情

Erlang 的發明者Joe Armstrong曾說: "防禦性程式設計是浪費時間 應該儘早讓程式崩潰"

基本原理不變 當您的代碼發生了**原本不應該發生的事情**時 你就應該趕快讓你的程式停止 因為你的程式是用來應付你預先知道會發生的事情

通常一個死的程式所造成的損害遠遠低於一個半殘的程式





## assertion式程式設計

> 請用assertion來避免不可能發生的事

每次當你在想說 "這是不可能發生的" 的時候 就請你加入程式碼來檢查 比如說

{% highlight python %}
assert(result != null)
{% endhighlight %}

某些語言你還可以寫註解
{% highlight java %}
assert result != null && result.size() > 0 : "Empty result from XYZ";
{% endhighlight %}

### 注意assertion的副作用

如果我們為了檢測錯誤的程式碼 本身就帶來錯誤 那就尷尬了

{% highlight java %}

while (iter.hasMoreElements()) { 
  assert(iter.nextElement() != null); 
  Object obj = iter.nextElement();
  // ...
}
{% endhighlight %}

我們在while的一開始確認了下一個元素不是null 但這個呼叫有副作用 就是把iterator往後指...

應該這麼做

{% highlight java %}

while (iter.hasMoreElements()) { 
  Object obj = iter.nextElement(); 
  assert(obj != null); 
  // ...
}
{% endhighlight %}


### 保持assert功能開啟

對於assertion有一個常見的誤解如下

"assertion給程式碼增加了負擔 因為他們的目的是去檢查一些不應該會發生的事 只是一個除錯工具 一但程式碼通過測試 發佈出去的版本中不應該有assertion"

這段論敘有兩個明顯錯誤的假設

1.它假設了**測試可以發現所有的bug** 但現實生活中對於一個複雜程式 你無法完整的測試你的程式碼會遇到的所有情況

2.他忘記了現實世界是多麼的危險 測試的時候不會有老鼠咬斷電纜 log檔案也不會塞滿硬碟分割

所以 你的第一道防線是自行找出所有可能的錯誤 第二道防線是使用assertion來嘗試找出你未能找到的錯誤


## 如何平衡資源

在寫程式的時候 我們都需要管理資源 比如說記憶體, transaction, threads, network connections, files等等 所有的東西都是有限的資源 大多數人對於資源的取得和釋放沒有一致的計畫 所以我們建議

> 由取得資源的人負責釋放資源

來個例子吧 

{% highlight python %}
def read_customer
  @customer_file = File.open(@name + ".rec", "r+") 
  @balance = BigDecimal(@customer_file.gets)
end
def write_customer 
  @customer_file.rewind 
  @customer_file.puts @balance.to_s 
  @customer_file.close
end
def update_customer(transaction_amount)
  read_customer
  @balance = @balance.add(transaction_amount,2)
  write_customer
end
{% endhighlight %}

上面的程式 由read_customer負責開檔案 write_customer負責關檔案 感覺很ok啊 反正都會各被叫一次

好問題來了 今天如果需求變了 只有更新後的值不為負數時 我們才更新 感覺這個超簡單 就交給一個新來的做吧

{% highlight python %}
def update_customer(transaction_amount) 
  read_customer
  if (transaction_amount >= 0.00)
    @balance = @balance.add(transaction_amount,2)
    write_customer
  end
end
{% endhighlight %}

測試看起來沒問題啊 就發佈出去吧 大概一小時內OS就爆了 OS跟你抱怨開啟的檔案太多 你仔細看了才知道 原來某些時候這個檔案只被開不被關

這時那個新手就說 抱歉抱歉 我馬上解掉這個bug

{% highlight python %}
def update_customer(transaction_amount)
  read_customer
  if (transaction_amount >= 0.00)
    @balance = @balance.add(transaction_amount,2)
    write_customer
  else
    @customer_file.close
  end
end
{% endhighlight %}

這時候就看準他的頭在哪裡 大力地給他巴下去

![Alt text]({{ site.url }}/public/hithim.gif)

原本只有兩個函式跟檔案開關有關 現在改完變三個 只會搞得越來越混亂

應該改成這樣

{% highlight python %}
def read_customer(file) 
  @balance=BigDecimal(file.gets)
end
def write_customer(file) 
  file.rewind
  file.puts @balance.to_s
end
def update_customer(transaction_amount) 
  file=File.open(@name + ".rec", "r+") 
  read_customer(file)
  @balance = @balance.add(transaction_amount,2) write_customer(file)
  file.close end
{% endhighlight %} 

檔案的開關都由update_customer處理


### 一次取得多個資源

兩個重點 就如同我們如何避免死結一樣

1.不同地方要一次取得多個資源 要照相同順序取: 比如你完全一個transaction需要A,B,C三個資源 那就永遠按照A->B->C的順序取得

2.釋放順序和取得順序相反: 按照C->B->A的順序釋放

### 平衡和例外

如果語言有支援exception 要如何保證程式結束前我們有把資源釋放乾淨呢 通常兩種方法

1.variable scope: 如C++或Rust 出了範圍外資源就自動回收

2.try-catch-finally


## 不要跑得比您的車頭燈還快

> 要做出預測是很困難的 尤其是未來的預測

在軟體發展中 我們的車頭燈能照亮的範圍是有限的 我們無法看到遙遠的未來 所以務實的程式設計師有一個堅定的原則

> 每次只走一小步

永遠都小步小步的走 再繼續往下走之前才可以檢查feedback和調整

你越是需要預測未來 你就越有可能犯錯 與其浪費精力為不確定的未來進行設計 不如將程式碼設計的容易修改 隨時準備因應未來的變化
