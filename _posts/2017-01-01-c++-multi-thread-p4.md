---
layout: post
title: Introduction to C++ multithread Part 4-1
comments: True 
subtitle: condition variable 教學
tags: c++ multi-thread
author: jyt0532
---

condition variable 教學

<br>
在看Part4之前 先確認你有15分鐘的連續時間
以收到最大效率的學習效果 如果現在沒空也沒關係 
有空再看 我的blog不會拿掉 
但我曾經因為這部分的學習太斷斷續續而走冤枉路

準備好了嗎 Go!


### condition variable是什麼

之前所有的例子 用的鎖都是 **mut**ual **ex**clusive的lock
其實在正常情況下 我們不會同時只讓一個thread進入critical section

甚至是不同的thread有不同的access條件
比如說 現在critical section是一個文件 
如果是write thread 一次只能一個thread進入critical section修改 
但如果是read thread 一次可以讓很多個thread進去讀 反正你也不會改東西

或者是一個thread想拿東西的時候 發現沒東西可以拿的話就要等
或是一個thread想放東西的時候發現沒地方可以放的話就要等
有太多太多可能的application 需要自己定義做事的condition
這件事要怎麼做到呢 就是用condition variable 我的理解是把它想成一個thread queue 

一個thread要做事之前呢 看看有沒有符合可以做事的condition
不符合的話就乖乖push進queue裡面睡覺 等人家叫你醒來
如果有人叫你醒來 你就確認一下有沒有符合可以做事的condition 符合的話就可以做事
不符合就繼續進queue睡覺

有點抽象 以下用一個最最常見的例子講解

### producer/consumer


{% highlight cpp %}
#include <thread> 
#include <iostream>
#include <vector>
#include <mutex>
#include <deque>
using namespace std;
struct QueueBuffer {
    deque<int> deq;
    int capacity;
    mutex lock;
    condition_variable not_full;
    condition_variable not_empty;
    QueueBuffer(int capacity) : capacity(capacity){}
    void deposit(int data){
        unique_lock<mutex> lk(lock);
        while(deq.size() == capacity){
            not_full.wait(lk);    
        }
        deq.push_back(data);
        lk.unlock();
        not_empty.notify_one();
    }
    int fetch(){
        unique_lock<mutex> lk(lock);
        while(deq.size() == 0){
            not_empty.wait(lk);
        }
        int ret = deq.front();
        deq.pop_front();
        lk.unlock();
        not_full.notify_one();
        return ret;
    }
};
void consumer(int id, QueueBuffer& buffer){
    for(int i = 0; i < 20; ++i){
        int value = buffer.fetch();
        cout << "Consumer " << id << " fetched " << value << endl;
        this_thread::sleep_for(chrono::milliseconds(200));
    }
}

void producer(int id, QueueBuffer& buffer){
    for(int i = 0; i < 30; ++i){
        buffer.deposit(i);
        cout << "Produced " << id << " produced " << i << endl;
        this_thread::sleep_for(chrono::milliseconds(100));
    }
}

int main(){
    QueueBuffer buffer(4);

    thread c1(consumer, 0, ref(buffer));
    thread c2(consumer, 1, ref(buffer));
    thread c3(consumer, 2, ref(buffer));
    thread p1(producer, 0, ref(buffer));
    thread p2(producer, 1, ref(buffer));

    c1.join();
    c2.join();
    c3.join();
    p1.join();
    p2.join();

    return 0;
}

{% endhighlight %}



強烈建議 把上面那段code跑起來 加速理解


<br>
![Alt text]({{ site.url }}/public/doit.gif)


各個擊破 先high level看main
兩個producer 三個consumer共用一個pool
這個pool最多可以放4個東西
producer製造東西 丟進pool
consumer從pool拿東西

再來看consumer跟producer function
再簡單不過 一個call pool的deposit
一個call pool的fetch

現在來看怎麼implement這個Pool


### producer/consumer Deep dive

QueueBuffer裡面 除了基本的deque放東西 跟capacity以外
需要一個所有thread共用的mutex 跟兩個condition variable
這兩個CV就是thread要被丟進去睡覺的地方
not_full放producer thread, not_empty放consumer thread

來看deposit
一個producer thread先拿到lock之後
**看一下現在的condition 確認一下該做事還是該睡覺**
while裡面的就是等待條件 在這裏 就是如果pool的size滿了就睡覺
not_full.wait(lk) 解讀成把自己push進not_full這個thread queue裡面

**在CV裡睡覺之前 release 手中的lock**

**在CV裡睡覺之前 release 手中的lock**

**在CV裡睡覺之前 release 手中的lock**

這就是CV最神妙的地方 也是這個系列文最重要的一個概念

而且他並不是一直在while裡面確認condition 
他是一直在CV裡面睡覺直到有人叫他起床
起床之後做幾件事
1. acquire lock (別懷疑 就是樓上那個unique_lock 我們睡覺前有release它)
2. 確認while condition符不符合 
3. 是個可以做事的condition的話 跳出while loop 做事
4. condition還是不對的話 一樣release剛剛拿的lock 進CV睡覺

好如果現在可以做事的話 我就可以把我的data push進pool
然後release lock(看到了嗎 我們在改動shared data的時候 手上一定要有鎖)
最後叫consumer thread queue(not_empty)的其中一個thread起床(notify)
因為你不知道有沒有consumer thread再等你叫他起床 
如果沒有人在not_empty裡面也沒關係 
你再放東西進去pool後就要負責叫not_empty這個CV裡面的thread起床

fetch裡面也大同小異

拿鎖 確認條件 有東西可以拿的話就拿 拿完後release lock
叫producer thread queue的人起床

看到這邊你融會貫通了嗎

![Alt text]({{ site.url }}/public/youweresaying.gif)

如果你夠認真的話 你應該會有個問題

Q. 之前不是有講RAII嗎 為什麼你要call一個lk.unlock() 不call會不會怎麼樣

A. 不會怎麼樣 如同上一篇所講 destructor會unlock 但notify CV的時候 我們並不用拿著鎖
不如先放掉給別人用 所以差別只有performance 


請接著看下一個例子
[Read/Write Lock](/2017/01/02/c++-multi-thread-p4-2)
