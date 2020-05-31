---
layout: post
title: Linux 效能工具 - 實用篇
comments: True 
subtitle: Linux 效能工具 - 實用篇
tags: performance
author: jyt0532
excerpt: 本文講解常用的觀察Linux效能的工具
---

[前](/2020/05/09/linux-performance-tool-1/)[三](/2020/05/17/linux-performance-tool-2/)[篇](/2020/05/24/linux-performance-tool-3/)文章 我們複習了許多Linux查看效能的指令 但真正發生問題的時候 到底該從哪裡開始一步步往下檢測呢

Netflix有分享一篇文章[6萬毫秒內對Linux性能診斷](https://netflixtechblog.com/linux-performance-analysis-in-60-000-milliseconds-accc10403c55) 文中教學了從你登入到Ec2之後 應該按照哪些順序 使用哪些標準的Linux工具可以用來診斷性能


{% highlight sh %}
uptime
dmesg | tail
vmstat 1
mpstat -P ALL 1
pidstat 1
iostat -xz 1
free -m
sar -n DEV 1
sar -n TCP,ETCP 1
top
{% endhighlight %}

總共十個推薦指令 你可以把這篇文章當作一個索引 每個指令的細節我有提供連結 可以連回比較細節的教學文章

{% include copyright.html %}






### [uptime](/2020/05/09/linux-performance-tool-1/#uptime)

![Alt text]({{ site.url }}/public/performance/uptime-2.png)

**這個指令快速地告訴你系統的平均負載** 

因為他秀出的是最近一分鐘 最近五分鐘 跟最近十五分鐘 比如說當你最近一分鐘的值比其他的都大 你就知道大概不太妙了

### [dmesg -T | tail](/2020/05/24/linux-performance-tool-3/#dmesg)

![Alt text]({{ site.url }}/public/performance/dmesg-Ttail.png)

**這個指令顯示最近的10條系統消息**

如果是發生了很明顯的系統錯誤 就會直接顯示在這邊 當你在診斷性能的時候 這個指令絕對值得一試

### [vmstat 1](/2020/05/09/linux-performance-tool-1/#vmstat)

![Alt text]({{ site.url }}/public/performance/vmstat1.png)

**列出關於process, memory, paging, blockIO, disk, CPU的資訊**

這個指令列出的第一行 是系統開機到現在的平均值

注意 

1.如果si或so不為0 代表系統把記憶體用完了

2.如果bi或bo不為0 代表有IO正在跑

### [mpstat -P ALL 1](/2020/05/09/linux-performance-tool-1/#mpstat)

![Alt text]({{ site.url }}/public/performance/mpstat-1.png)

**列出每個CPU的使用情況** 可以知道process是不是正常的在使用CPU

如果有某顆CPU特別忙的話 那大概就是正在跑一個很耗CPU的單線程應用

### [pidstat 1](/2020/05/17/linux-performance-tool-2/#pidstat)

![Alt text]({{ site.url }}/public/performance/pidstat.png)

**列出每一個process使用CPU的情況** `1`代表每秒印一次

### [iostat -xz 1](/2020/05/09/linux-performance-tool-1/#iostat)

![Alt text]({{ site.url }}/public/performance/iostat-xz1.png)

**查看設備(磁盤)的狀況** `1`代表每秒印一次

### [free -m](/2020/05/09/linux-performance-tool-1/#free)

![Alt text]({{ site.url }}/public/performance/free-m.png)

**系統中已被使用的和可以被使用的記憶體大小**

### [sar -n DEV 1](/2020/05/17/linux-performance-tool-2/#sar)


![Alt text]({{ site.url }}/public/performance/sar-nDEV1.png)

**這個指令檢查每個網路接口的統計數據**


| 代號  | 解釋 |
| --- | --- |
|IFACE|網路介面的名稱|
|rxpck/s|每秒收到的package數量|
|txpck/s|每秒傳送的package數量|
|rxkB/s|每秒接收到的kB|
|txkB/s|每秒傳送到的kB|
|rxcmp/s|每秒收到的壓縮過的package數量|
|txcmp/s|每秒傳送的壓縮過的package數量|
|rxmcst/s|每秒收到的multicast package數量|
|%ifutil|Interface的使用率|

你可以大概念的知道這台機器使用網路的情況


### [sar -n TCP,ETCP 1](/2020/05/17/linux-performance-tool-2/#sar)

其實就是把 `sar -n TCP 1` 和 `sar -n ETCP 1`合在一起

![Alt text]({{ site.url }}/public/performance/sar-nTCPETCP1.png)

關於`TCP`選項 就是秀出有關TCP的traffic 

| 代號  | 解釋 |
| --- | --- |
|active/s|每秒本地發起 TCP 連接數 比如說通過`connect()`|
|passive/s|每秒遠程發起的 TCP 連接數 比如說通過`accept()`|
|iseg/s|每秒收到的segments數量|
|oseg/s|每秒送出的segments數量|


至於`ETCP`選項 就是秀出有關TCP的traffic的Error

| 代號  | 解釋 |
| --- | --- |
|atmptf/s|每秒嘗試連線但失敗的連接數|
|estres/s|每秒從ESTABLISHED/CLOSE-WAIT狀態轉換到CLOSED狀態的TCP連接數目|
|retrans/s|每秒重傳的segments數量|
|isegerr/s|每秒收到的error的segments數量 比如說checksum錯誤|
|orsts/s|每秒送出的的包含RST flag的segments數量|


想要真正的搞懂每個值的意義 你要先熟悉TCP的[三次握手四次揮手協議](https://zhuanlan.zhihu.com/p/108504297)

### 三次握手四次揮手

大概念就是 網路上的兩個人 要建立TCP連線之前 會先確認對方聽得到

A: 安安 聽得到嗎 我有話說

B: 聽到了 請說

A: OK 我也聽到你了 我要說的是...

要結束對話前 則是

A: 誒我講完了 我要掛電話囉
{% include copyright.html %}

B: 好的了解

B: 那我也講完了 那就掰囉

A: 好的掰

流程圖大概是長這樣

![Alt text]({{ site.url }}/public/performance/tcp-handshaking.png)

### 好的 我們回來 ETCP

有了TCP State的概念之後 再回來看這些值

#### active/s

就是每秒鐘 從CLOSED狀態 轉到SYN-SENT狀態 的次數

![Alt text]({{ site.url }}/public/performance/tcp-active.png)

#### passive/s

就是每秒鐘 從LISTEN狀態 轉到SYN-RCVD狀態 的次數

![Alt text]({{ site.url }}/public/performance/tcp-passive.png)

這樣有沒有比較有感覺了 基本上就是準備開始要對話囉 被動主動差別就是誰初始這個對話

#### atmptf/s

就是每秒鐘 從SYN-SENT狀態 **直接**轉到CLOSED狀態 的次數(如果你是呼叫方)

加上 每秒鐘 從SYN-RECEIVED狀態 **直接**轉到CLOSED/LISTEN狀態 的次數(如果你是接收方)


![Alt text]({{ site.url }}/public/performance/tcp-atmptf.png)

#### estres/s

![Alt text]({{ site.url }}/public/performance/tcp-estres-1.png)

就是每秒鐘 從ESTABLISHED/CLOSE_WAIT狀態 **直接**轉到CLOSED狀態 的次數



### [top](/2020/05/09/linux-performance-tool-1/#top)

**秀出系統中所有process的資訊**


## 總結

當然有更多其他的指令 可以詳讀[大神 Brendan D. Gregg](http://www.brendangregg.com/)的[演講](https://www.youtube.com/watch?v=FJW8nGV4jxY) 這篇文章算是個出發點 讓你知道你應該從哪個角度去切入診斷一個系統的效能
