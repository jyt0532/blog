---
layout: post
title: Introduction to C++ multithread Part 4-2
comments: True 
subtitle: condition variable 範例
tags: c++ multi-thread
author: jyt0532
---

打鐵要趁熱 來看multithread第二常見的example

### Read/Write Lock


[How to make a multiple-read/single-write lock](http://stackoverflow.com/questions/27860685/how-to-make-a-multiple-read-single-write-lock-from-more-basic-synchronization-pr)

{% highlight cpp %}
#include <iostream> 
#include <cstdlib>
#include <thread>
#include <mutex>
#include <vector>
using namespace std;
class RWLock {
    public:
        RWLock()
            : shared()
              , readerQ(), writerQ()
              , active_readers(0), waiting_writers(0), active_writers(0){}

        bool no_one_writing(){
            return active_readers > 0 || active_writers == 0;
        }
        bool no_one_read_and_no_one_write(){
            return active_readers == 0 && active_writers == 0;
        }
        void ReadLock() {
            std::unique_lock<std::mutex> lk(shared);
            readerQ.wait(lk, bind(&RWLock::no_one_writing, this));
            ++active_readers;
            lk.unlock();
        }

        void ReadUnlock() {
            std::unique_lock<std::mutex> lk(shared);
            --active_readers;
            lk.unlock();
            writerQ.notify_one();
        }

        void WriteLock() {
            std::unique_lock<std::mutex> lk(shared);
            ++waiting_writers;
            writerQ.wait(lk, bind(&RWLock::no_one_read_and_no_one_write, this));
            --waiting_writers;
            ++active_writers;
            lk.unlock();
        }

        void WriteUnlock() {
            std::unique_lock<std::mutex> lk(shared);
            --active_writers;
            if(waiting_writers > 0)
                writerQ.notify_one();
            else
                readerQ.notify_all();
            lk.unlock();
        }

    private:
        std::mutex              shared;
        std::condition_variable readerQ;
        std::condition_variable writerQ;
        int                     active_readers;
        int                     waiting_writers;
        int                     active_writers;
};
int result = 0;
void func(RWLock &rw, int i){
    if(i % 2 == 0){
        rw.WriteLock();
        result += 1;
        rw.WriteUnlock();
        rw.ReadLock();
        rw.ReadUnlock();
    }else{
        rw.ReadLock();
        rw.ReadUnlock();
        rw.WriteLock();
        result += 2;
        rw.WriteUnlock();

    }
}
void not_safe(int i){
    if(i%2 == 0){
        result += 1;
    }else{
        result += 2;
    }
}
int main(){
    RWLock rw;
    std::vector<std::thread> threads;
    for(int i = 0; i < 1000; i++){
        threads.push_back(std::thread(func, ref(rw), i));
        //threads.push_back(std::thread(not_safe, i));
    }
    for(int i = 0; i < threads.size() ; i++)
    {
        threads[i].join();
    }
    cout << result << endl;
    return 0;
}
{% endhighlight %}

老話一句 跑起來加速理解

![Alt text]({{ site.url }}/public/imp.gif)

main裡面initiliza一個rw lock
把他丟給所有的thread
注意這裡要用reference丟 這樣所有thread才會用到同一個rw lock

func也簡單 就是要改動result前call WriteLock 改完後call WriteUnlock

### Read/Write Lock Deep dive

RWLock這個data structure裡面有一個共用的mutex
跟兩個condition variable readerQ跟writerQ

active_readers, waiting_writers, active_writers 
是紀錄現在的state的變數

基本上就是4個function
* ReadLock
* ReadUnlock
* WriteLock
* WriteUnlock

**_ReadLock:_**

1.Acquire lock

2.確認condition 不該做事的話就進CV wait 該做事就往下走 
這裏的寫法只是比較fancy一點 但下列這兩個寫法是等價的 

{% highlight cpp %}
while (!f()) {
        wait(lk);
}
{% endhighlight %}

{% highlight cpp %}
wait(lk, f)
{% endhighlight %}

第一個寫法是我Part4-1的寫法 對初學者比較好懂 
但要用第二個寫法的話 就把那個function解讀成wait until f
雖然比較難理解但第二個寫法漂亮一些 
就可以把判斷要不要做事的logic寫在另一個function而不是全部塞進while的condition

至於bind和this是什麼意思 有興趣可以詳讀這篇文章
[淺談C++ bind function](/2017/01/05/bind/)
在這裏先把它看成

{% highlight cpp %}
readerQ.wait(lk, no_one_writing);
{% endhighlight %}

3.所以ReadLock要做事的condition 就是沒人在寫或是有人在讀 這個情況下就可以安心進去讀

4.把active_reader++

5.解鎖(optional)

**_ReadUnLock:_**

1.Acquire lock

2.改變shared data

3.解鎖(optional)

4.去叫writer thread queue的**一個**writer thread起床

**_WriteLock:_**

1.Acquire lock

2.讓自己先去等waiting_writer++;

3.確認condition 不該做事的話就進CV wait 該做事就往下走
這裏的condition是沒有人讀而且沒有人寫 那我就可以安心write

4.\-\-waiting_writers, ++active_writers, 讓所有人知道有人在寫

5.解鎖(optional)

**_WriteUnLock:_**

1.Acquire lock

2.改變shared data

3.確認現在的state 如果有writer再等 叫一個writer起床
如果沒有人在等 去叫reader thread queue的**所有**reader thread起床
因為我們默許 多個reader可以同時read 所以就是個部隊起床的概念

4.解鎖(optional) 

講解到這邊 我還寫了個不安全的版本 只要把main裡面的第一個for loop改一下thread call的function就可以

{% highlight cpp %}
for(int i = 0; i < 1000; i++){
    threads.push_back(std::thread(not_safe, i));
}
{% endhighlight %}

![Alt text]({{ site.url }}/public/areyoureallywithme.gif)


打完收工 問問題時間 如果你夠認真的話 你應該會有兩個問題

Q1. 為什麼ReadUnlock的時候是先解鎖再notify 但WriteUnlock是先notify在解鎖呢

A1. 好問題 因為WriterUnLock在判斷要notify誰的時候 需要access shared data, 所以不能把鎖放掉

Q2. 既然你reader拿鎖的條件 其中一個是active_reader > 0, 那如果reader thread很多的話 很有可能active_reader會一直不為0 writer可能永遠拿不到鎖怎麼辦 

A2. Perfect question! 如果你沒念過read/write lock但卻能自己想到這問題 那代表說你極為聰明

[RW Lock Priority_policies](https://en.wikipedia.org/wiki/Readers%E2%80%93writer_lock#Priority_policies)

本篇文章的實作是Read-preferring的RW Lock 每當有新的reader進來 他發現有人在read 那他也跟著read 都不管writer受得了受不了

![Alt text]({{ site.url }}/public/dondondon.gif)

**所以可能writer會starve**

那Write-preferring的RW Lock怎麼改呢 很簡單 就改變reader做事的condition即可

{% highlight cpp %}
bool no_one_writing(){
    return waiting_writers == 0 && active_writers == 0;
}
{% endhighlight %}

這樣reader就會等到沒有人寫也沒有人等著寫的時候 才會read 只要有writer來 writer之後的reader都要等

當然你可以定義自己的application想怎麼玩 你也可以在WriterUnlock的時候先判斷有沒有waiting_readers 
有的話先notify_readerQ 或是你可以限定最多同時有多少個reader在read 
隨便你愛怎麼玩就怎麼玩 只是可能需要更多variable記錄現在的state 然後記得要更動這些state variable的時候鎖要拿著就好了

恭喜你已經通透了multi-thread最難的部分 相信之後不論是任何語言的multi-thread問題你都能快速上手

