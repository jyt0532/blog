---
layout: post
title: Bash shell - time command
comments: True 
subtitle: 了解如何使用time命令
tags: bash
author: jyt0532
excerpt: 本篇教學如何使用time命令以及看懂time命令的輸出
---

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

