---
layout: post
title: Introduction to C++ multithread Part 3
comments: True 
subtitle: lock_guard, unique_lock 教學
tags: c++ multi-thread
author: jyt0532
---

深入淺出C++ multithread part3

### problem with lock/unlock


關於上一篇的問題你有答案了嗎 上篇例子用sum++你可能沒有feel 但如果改成一個function的話呢?
沒錯 如果mutex之間的operation噴出了exception

{% highlight cpp %}
void incre(){
    mu.lock();
    funA();//if function incre() terminate here, the lock will never release
    mu.unlock();
}  
{% endhighlight %}

**那噴exception的那個指令之後的指令都不會被跑** 它會直接看它現在是不是在try block裡面
是的話 就會找到相對應的catch block 執行catch block內的指令
找不到的話 就會直接return 再看它的parent function有沒有預期到這個exception 沒有的話就一路return出去
直到找到相對應的exception handler 再去執行相對應的catch 雖然這不是這個文章的主題
但stack unwinding是個很重要的概念
推薦各位看一下learncpp的教學 寫得很好
[Exceptions, functions, and stack unwinding](http://www.learncpp.com/cpp-tutorial/143-exceptions-functions-and-stack-unwinding/)

回到主題 重點呢 就是如果你噴exception 那麼你的鎖永遠不會release 
其他thread就等你等到死 那怎麼辦呢 簡單 用try/catch包住可能噴exception的所有地方

{% highlight cpp %}
void incre(){
    mu.lock()
    try{
        funA();
    } catch(std::exception &cException){
        mu.unlock();
        throw cException;
    }
    mu.unlock(); 
}  
{% endhighlight %}

但這樣的寫法只解決的眼前的問題 但當你寫大型的程式的時候 
一個function很可能會有很多個可能的exit point 
那你就必須在所有可能的exit point unlock你的鎖 你的程式就會越來越難maintain


那該怎麼解決這個問題呢?

### RAII

[Resource acquisition is initialization](https://en.wikipedia.org/wiki/Resource_acquisition_is_initialization)
> In RAII, holding a resource is tied to object lifetime: resource allocation is done during object creation by the constructor, while resource deallocation is done during object destruction by the destructor. Thus the resource is guaranteed to be held between when initialization finishes and finalization starts and to be held only when the object is alive.

這是個被非常推崇的習慣 特別是C++在處理resource leak或是excpetion safety的時候必須謹記在心的

那C++怎麼implement這件事情呢 剛剛推薦的stack unwinding有看嗎 叫你看你還不看


<br>

![Alt text]({{ site.url }}/public/iced.jpeg)


再給你一次機會
[What is stack unwinding](http://stackoverflow.com/questions/2331316/what-is-stack-unwinding)

借用一下stack overflow [Nikolai](http://stackoverflow.com/users/106671/nikolai-n-fetissov)大的例子

{% highlight cpp %}
void func( int x )
{
    char* pleak = new char[1024]; // might be lost => memory leak
    std::string s( "hello world" ); // will be properly destructed

    if ( x ) throw std::runtime_error( "boom" );

    delete [] pleak; // will only get here if x == 0. if x!=0, throw exception
}
{% endhighlight %}

C++的compiler會在一個變數要失效的時候(about to out of scope or exception thrown)去執行所有變數的destructor **in reverse order**
為什麼要相反順序呢 因為後面宣告的變數可能用到之前宣告的變數當作constructor的參數 先destruct後宣告的變數比較安全

所以在multi-thread的世界裡也是一樣 C++為了支援RAII 建議用這兩個寶貝

### lock_guard, unique_lock

有了lock_guard 世界變得明亮許多

{% highlight cpp %}
void incre(){
    lock_guard<std::mutex> lockGuard(mu);
    funA();
}  
{% endhighlight %}

簡直完美 要initialize lock_guard 只需要給一個mutex variable 然後呢 剩下的他全幫你搞定
其實內部的實作也是很簡單
lock_guard在constructor裡lock你給他的mutex, 在destructor裡release 同一個mutex 只是他保證destructor會被call到

unique_lock用途又更廣了 除了以上的基本款之外unique_lock還可以拿來當作function的return type
更重要的差別是unique_lock更佳的彈性 支援各種不同的lock

1. Deferred lock: 先不要acquire 等晚點再aquire
2. Time-constrained lock: 試著要lock 但過了一段時間都要不到就放棄
3. Recursive lock: 如果一個function會recursively call自己 如果你用的是一般的mutex, 那就會deadlock 因為你的parent正在佔用同樣的鎖 但如果是recursive mutex, 那就可以同一個thread一直acquire同樣的lock, 其他thread必須等到這個thread release 所有的recursive lock之後才可以acquire
4. condition variable都需要搭配unique_lock使用

有興趣的可以詳讀這篇
[Flexible locking with unique_lock<>](https://www.justsoftwaresolutions.co.uk/threading/multithreading-in-c++0x-part-5-flexible-locking.html)

下一篇進入本系列作的高潮 conditional variable
