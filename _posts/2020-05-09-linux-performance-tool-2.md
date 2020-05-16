---
layout: post
title: Linux 效能工具 - Intermediate
comments: True 
subtitle: Linux 效能工具 - 中階篇
tags: performance
author: jyt0532
excerpt: 本文講解常用的觀察Linux效能的工具
---

{% include strace_youtube_tracking.html %}

最近看到[大神 Brendan D. Gregg](http://www.brendangregg.com/)的演講 覺得自己對於Linux的工具實在是太不熟了 於是想稍微熟讀一下

本系列的目的不是把每個指令的每個option都寫出來 想知道的話`man`一下就知道了 

本文的目的是 希望你看完之後 對於每個有用的指令都有個印象 當你需要偵錯或是研究你的系統哪裡出了問題 你可以知道有哪個指令可以用 然後再去慢慢讀document

本文是照著[Linux Performance Tools, Brendan Gregg](https://www.youtube.com/watch?v=FJW8nGV4jxY)演講的順序講解
{% include copyright.html %}



### strace

system call tracer 就是可以上你追蹤系統調用

什麼是系統調用呢 就是你跟OS Kernel發出的調用 通常是在你需要

1.Process的創建或管理(fork(), wait(), kill()...)

2.Intormation管理(getpid(), alarm(), sleep()...)

3.檔案讀寫 資料夾管理(open(), close()...)

4.Device I/O(loctl(), read(), write()...)

5.Communication(pipe(), msgget()...)


知道什麼是System Call之後 再來看看我們該怎麼用`strace`

#### 第一種用法 

就是你可以看到一個你下達一個command之後 這個command到底幫你調用了多少system call

![Alt text]({{ site.url }}/public/performance/strace-1.png)

哇靠 原來只是`cat`一下居然發生了這麼多事 你可以看到所有調用的system call 還有每個system call的參數

那如果`cat`一個不存在的檔案呢

![Alt text]({{ site.url }}/public/performance/strace-2.png)

ok 了解 就是可以讓你完完整整地看到我們發給OS的函式和參數 還有OS給我們的回應

可是這是當我們知道command的情況下才有用 我們大多數時候並不知道是哪個command出了問題 那該怎麼debug呢

#### 第二種用法 

更高端的用法 `strace`可以綁定一個process id 然後監控這個process發出的所有system call

我先看一下我現在這個terminal的pid是多少

![Alt text]({{ site.url }}/public/performance/strace-3.png)

開另一個terminal監控

<div id="strace-demo"></div>

你會發現 啊 原來每一個鍵盤輸入 都會調用system call啊 你如果真的想深入研究 可以去看每個指令的document 但我們的demo中常見的`read` `write` 的第一個參數`fd = 0` 也就是輸入是來自standard input 就是我們的鍵盤

我已經示範了怎麼完完全全的監控某個進程的所有系統調用了 你手上的工具又比別人多了一個 以後別人問你

"咦 這個command(或是binary)怎麼跑不起來 error message沒有足夠的訊息 怎麼辦"

你就可以教他怎麼看這個指令跟OS交互的所有過程

#### 注意

讓我稍微提醒你一下 因為這個指令需要去監視 然後記下所有system call的細節 所以性能開銷非常大 如果是在production上使用要非常小心


