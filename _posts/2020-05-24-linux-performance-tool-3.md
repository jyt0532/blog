---
layout: post
title: Linux 效能工具 - Advanced
comments: True 
subtitle: Linux 效能工具 - 高階篇
tags: performance
author: jyt0532
excerpt: 本文講解常用的觀察Linux效能的工具
---

{% include copyright.html %}

{% include perf_top_youtube_tracking.html %}

最近看到[大神 Brendan D. Gregg](http://www.brendangregg.com/)的演講 覺得自己對於Linux的工具實在是太不熟了 於是想稍微熟讀一下

本系列的目的不是把每個指令的每個option都寫出來 想知道的話`man`一下就知道了 

本文的目的是 希望你看完之後 對於每個有用的指令都有個印象 當你需要偵錯或是研究你的系統哪裡出了問題 你可以知道有哪個指令可以用 然後再去慢慢讀document

本文是照著[Linux Performance Tools, Brendan Gregg](https://www.youtube.com/watch?v=FJW8nGV4jxY)演講的順序講解



### ss

socket statistic

就是所有跟socket相關的數據 就看他就對了 也因為[`netstat`](/2020/05/17/linux-performance-tool-2/#netstat)被deprecated掉了 現在大多都是用`ss`

什麼option都不給的話 秀的是所有 有建立連線但non-listening的sockets

![Alt text]({{ site.url }}/public/performance/ss.png)

那麼有哪些選項可以用呢?


`-n`:不要解析服務名稱

`-r`:將 IP 位址解析為主機名稱

`-l`:列出listening的sockets

`-a`:顯示所有的 sockets 包含listening 和 non-listening

`-t`:列出 TCP 的 sockets

`-u`:列出 UDP 的 sockets

`-x`:列出 Unix 的 sockets

`-4`:列出 IPv4 的 sockets

`-6`:列出 IPv6 的 sockets

`-p`:顯示使用 sockets 的程式資訊

`-e`:顯示 sockets 細部資訊

`-i`:顯示 TCP sockets 內部資訊

`-o`:顯示 sockets 的計時器資訊

`-s`:輸出 sockets 的使用統計表


哇賽 連IPv4 IPv6都可以選 知道這些選項後就可以任意排列組合啦

比如說`ss -tai` 就是列出所有TCP socket的內部資訊

我們先看一下目前在聽多少TCP的socket

![Alt text]({{ site.url }}/public/performance/ss-ta-before-remove-https.png)

那我們在ec2要怎麼做實驗呢 這時就可以打開AWS的Security Group

![Alt text]({{ site.url }}/public/performance/inbound-rules-before-remove-https.png)

試著把HTTPS拿掉(當然拿掉後jyt0532.com就連不上了)

`ss -ta`

![Alt text]({{ site.url }}/public/performance/ss-ta-after-remove-https.png)

有趣 下面一整排都不見了 因為沒有人連得上了

等等 如果下面一整排不見代表的是所有連線都被踢掉的話 那我們再仔細看一下


![Alt text]({{ site.url }}/public/performance/ss-ta-before-remove-https-close.png)

左半部是我ec2的ip沒問題 那右半部難道是.....


答對了 右半的那些ip就是正在跟我的https socket通訊的人 也就是正在看我的部落格的人的ip

好玩吧

### iptraf

互動式網路流量監測工具

這可能是介紹到現在 最酷的指令

![Alt text]({{ site.url }}/public/performance/iptraf.png)

你沒看錯 輸入`iptraf`就是看到這個介面 接下來你就可以選你要觀測哪一個數據

![Alt text]({{ site.url }}/public/performance/iptraf-2.png)

然後他就開始跑起來了...

![Alt text]({{ site.url }}/public/performance/iptraf.gif)

原來是wireshark啊 我以為是linux呢

非常建議大家裝起來玩一玩

### iotop

就是I/O版本的[`top`](/2020/05/09/linux-performance-tool-1/#top)

![Alt text]({{ site.url }}/public/performance/iotop.png)

所以前兩行換成硬碟的使用資訊 而不是記憶體的使用資訊

需要特別注意的是 預設是每個thread都列出來 如果想以Process為單位的話 就要加`-P`

![Alt text]({{ site.url }}/public/performance/iotop-P.png)

### perf_events

`perf` 就是可以直接測量性能的指令

用途很多 比如你想知道某個指令跑了多久 出現了多少次context-switch等等

`perf stat [command]`

![Alt text]({{ site.url }}/public/performance/perf stat pwd.png)

`perf top`

就像是[`top`](/2020/05/09/linux-performance-tool-1/#top) 列出每個process的性能狀況

一樣來做個實驗 我們刻意寫個CPU很吃重的C程式

{% highlight c %}
#include <stdio.h>
#include <unistd.h>

void just_loop() {
  int i, j;
  for(int i =- 0; i < 100000000; i++) {
    j = i;
  }
}

void just_sleep() {
  sleep(10);
}

void many_print() {
  for(int i = 0; i < 100000; i++) {
    printf("Hello\n");
    printf("pid: %d\n", getpid());
  }
}
int main() {
    printf("pid: %d\n", getpid());
    sleep(5);
    many_print();
    just_sleep();
    printf("After sleep\n");
    just_loop();
    return 0;
}

{% endhighlight %}
編譯一下
{% highlight bash %}
> g++ -c example.c
> g++ example.o -o example
{% endhighlight %}

這個程式一執行之後 會印出process id 我們再把pid丟進`perf top -p [pid]`指令裡面 
來看看這個程式表現如何

<div id="perf-top-demo"></div>

我們首先先大量的`print` 也就是使用I/O而不是CPU 最後在大量的使用CPU

這是結算畫面
![Alt text]({{ site.url }}/public/performance/perf-top-example.png)

居然細節到我的函數名稱 他都可以分析 從結算畫面我們可以知道 這個程式的bottleneck就是`just_loop`

有趣吧 當然你要不給定pid整個系統的看也是可以的

隨便來個無限迴圈的程式

{% highlight cpp %}
int main(void)
{
  int i;
  while (1) i++;
}
{% endhighlight %}

跑起來

{% highlight bash %}
> gcc -o t2 -g test2.c
> ./t2
{% endhighlight %}

用另一個terminal觀測 `perf top`

![Alt text]({{ site.url }}/public/performance/perf-top-2.png)

無所遁形 一目了然


### dmesg 

可以看到系統從開機到現在的所有system message

![Alt text]({{ site.url }}/public/performance/dmesg-1.png)

通常加上個`-T` 顯示人類看得懂的時間戳

![Alt text]({{ site.url }}/public/performance/dmesg-2.png)

## 總結

這篇文章涵蓋了這個演講提到的部分Advanced指令 為什麼只提部分呢 因為有些指令並不是linux內建的 有些指令功能跟前兩篇提到的都有重複 當然也有些指令實在太細節了 我之後需要用到的時候再`man`就好了 所以關於那些指令 這篇文章就不再贅述了


基本上你有疑問的話 就是看下表 需要的話再去深入看每個指令應該怎麼用

![Alt text]({{ site.url }}/public/performance/performance-advanced.png)




