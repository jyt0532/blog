---
layout: post
title: malloc的虛擬內存分配
comments: True 
subtitle: 
tags: bash
author: jyt0532
excerpt: XX
---

在前一篇[time指令的教學文](/2018/06/01/time-command/)中 我們為了測試system call 
而使用了malloc指令 但用完之後越看越不對

{% highlight cpp %}
int main () {
   char *str;
   str = (char *) malloc(400000000000);
   free(str);
   return(0);
}
{% endhighlight %}

這個程式居然可以執行！！！！？？？

我等於是跟heap要了400G 居然默默地執行完沒有任何complain

越想越不對勁 於是來稍微研究一下malloc在搞什麼鬼

### 虛擬內存

也有人說是虛擬記憶體 OS使用虛擬記憶體有兩個好處

1.有時候我們的內存空間並不是連續 

比如說總共有10MB的可用空間 但卻被切成3MB, 3MB, 4MB的三個空間

如果你要申請一個5MB的空間 就需要虛擬記憶體 讓你(使用者)以為你被配置到了連續的空間

2.有時候heap空間根本不夠 虛擬記憶題還可以分配Disk的空間給你 讓你覺得你真的被配置了很多記憶體


### malloc回傳的是虛擬空間

由上面的程式我們知道 malloc配置給我們的是虛擬地址空間 


### 用途

測量一個command花了多少時間

### 用法 
{% highlight bash %}
> time command
{% endhighlight %}

直接上例子
{% highlight bash %}
> time ls
{% endhighlight %}

會看到如下的輸出
{% highlight bash %}
real	0m0.020s
user	0m0.002s
sys	0m0.009s
{% endhighlight %}

你總共會看到三個值 

real: 總共的時間 從開始到結束 就像有個人拿著碼錶等著這個指令跑完 所會看到的總時間

user: CPU所花費的**運算**總時間

sys: CPU所花費的在**系統**(kernel)相關的時間 比如說配置記憶體

來看個例子 先來睡個三秒

{% highlight cpp %}
int main () {
   sleep(3);
   return(0);
}
{% endhighlight %}

執行它
{% highlight bash %}
real	0m3.012s
user	0m0.001s
sys	0m0.003s
{% endhighlight %}

看起來是沒有用到CPU的時間 

現在來用看看CPU

{% highlight cpp %}
int main () {
   int total;
   for(int i = 0; i < 1000000000; i++){
     total += i;
   }
   return(0);
}
{% endhighlight %}

執行它

{% highlight bash %}
real    0m2.328s
user    0m2.237s
sys     0m0.017s
{% endhighlight %}

跟預期的一樣 CPU花了些時間運算 但沒有任何的system call

現在我們來試試system call 跟OS要一點記憶體來用用

{% highlight cpp %}
int main () {
   char *str;
   str = (char *) malloc(400000000000);
   free(str);
   return(0);
}
{% endhighlight %}

執行它
{% highlight bash %}
real    0m0.007s
user    0m0.001s
sys     0m0.003s
{% endhighlight %}

沒fu 加個0

{% highlight cpp %}
int main () {
   char *str;
   str = (char *) malloc(4000000000000);
   free(str);
   return(0);
}
{% endhighlight %}

執行它
{% highlight bash %}
real    0m0.018s
user    0m0.001s
sys     0m0.013s
{% endhighlight %}

再加個0

{% highlight cpp %}
int main () {
   char *str;
   str = (char *) malloc(40000000000000);
   free(str);
   return(0);
}
{% endhighlight %}

執行它
{% highlight bash %}
real    0m0.168s
user    0m0.002s
sys     0m0.151s
{% endhighlight %}

這樣有感覺到線性的成長了 所以sys是代表花在system call的時間

## 系統呼叫(System call)

講那麼多 什麼是system call呢

一個電腦上可以跑很多個應用(Application) 有些Application可以自己玩自己的 但有些application就需要OS提供服務來達成目的 

所以OS提供了很多API讓應用程式來呼叫 OS所提供的服務包含

1.進程控制: create process, abort process, allocate memory, free memory

2.檔案管理: file open, file close, delete file, read file

3.裝置管理: request device, delete device, get device attribute

4.資訊管理: get system time or data

5.通訊: create/delete communication, send/receive messages

這些東西都算在system call的範疇


## 結論

user time + system time 會告訴你你的CPU花了多少時間處理這個命令 特別注意這個是加總了所有的CPU

所以如果你的進程(process)有兩個以上的線程(thread) 那user time + system time就會是所有線程的時間相加 所以你看到加起來大於你的real time也不要太驚訝 因為不同的線程可以同時運行

有興趣的讀者可以用time跑跑看[C++多線程教學](/2016/12/23/c++-multi-thread-p1/)的範例程式

