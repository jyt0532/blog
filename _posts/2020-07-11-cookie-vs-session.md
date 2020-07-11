---
layout: post
title: 如何理解Cookie和Session
comments: True 
subtitle: 
tags: cookie session
author: jyt0532
excerpt: 如何理解Cookie和Session
---

一直以來 對於Cookie和Session的概念一直很模糊 稍微研究了一下之後記錄下來

## Cookie

當我們在Amazon買東西的時候 到各個頁面把東西丟到購物車裡 最後結帳的時候跳到結帳頁面 他怎麼記得你之前把哪些東西放進購物車裡? 他怎麼知道你是誰呢? 或是為什麼你大概好幾個月才需要登入一次facebook? 每次開facebook時瀏覽器的網址都是`facebook.com` 他怎麼記得我的? HTTP是無狀態的協議啊 

這是因為facebook跟你的瀏覽器使用cookie合作 來讓你上網的體驗更好

### 什麼是Cookie

每當你登入facebook之後 facebook會給你一串 好幾個key-value pair 你只要下次來facebook的時候再提供同樣的key-value 我就認得出你來 你可以想成是短期的會員證 這樣下次你來就不需要再登入了

做學問就是要學以致用 就看上面這段沒有感覺 怎麼辦呢?

### 怎麼看目前瀏覽器存的Cookie

我們來看一下chrome怎麼存cookie

1.打開Chrome Setting

2.找到Privacy and security

![Alt text]({{ site.url }}/public/privacy-and-security.png)

3.點進Site Settings

4.點進Cookies and site data

![Alt text]({{ site.url }}/public/cookies-and-site-data.png)

5.點進See all cookies and site data

![Alt text]({{ site.url }}/public/see-all-cookies-and-site-data.png)

然後你就可以看到你瀏覽器曾經存下的所有Cookie資料 

6.找到facebook的cookie

![Alt text]({{ site.url }}/public/facebook-cookie.png)

哇 原來每次我的瀏覽器連到facebook 他就把facebook的相關Cookie全部一併給上去 

我們稍微仔細看一下

![Alt text]({{ site.url }}/public/fb-cookie-key-value-pair.png)

這麼多個不同的值是什麼意思呢 這就是我們剛剛說的很多個key-value pair 這裡看到的每一個值都是不同的key 分別代表的是不同的資訊 比如`c_user`代表使用者是誰 `act`是使用者登入的時間 等等 有興趣的話這篇文章[Facebook Cookies Analysis](https://medium.com/@TechExpertise/facebook-cookies-analysis-e1cf6ffbdf8a)解釋得不錯


修但幾勒

![Alt text]({{ site.url }}/public/wait-a-minute.jpeg)

如果把這些資料都存在瀏覽器前端 那任何人不就都可以假造資料嗎 我跟facebook說我是Mark也可以嗎?? 

好問題 所以通常值得信賴的大網站 都會把Cookie的值給hash 第一個好處是讓你看不懂 第二個好處是你想偷改也不知道怎麼Hash 只要facebook發現這個hash過後的值不對 就不跟你玩 

![Alt text]({{ site.url }}/public/cookie-hash.png)

### 實戰一下

我相信講到這邊你已經很期待 到底我們跟facebook發出一個請求的時候 Header長什麼樣子呢 我們打開可愛的developer console來看一下 

![Alt text]({{ site.url }}/public/cookie-in-request.png)

你會發現 cookie裡面就是把所有的key-value pair連在一起而已

![Alt text]({{ site.url }}/public/cookie-in-request-highlighted.png)

### Session

那用cookie的缺點是什麼呢 第一個是客戶可能偷改cookie資料的風險 第二個就是每次呼叫server都要帶上一大堆的字串 傳輸起來浪費時間空間

另一個解法就是session 最大的差別就是cookie存在客戶端 session相關的資料都存在server端 這樣只要客戶知道自己的sessionId(通常是包在cookie裡一起給server) 就可以請server幫忙找出自己的相關資料 就不用每次都傳輸那麼多的資料


### 比較總結

1.Cookie放在客戶端 Session放在服務器端

2.因為Cookie可以偽造(比如說你的電腦被hack 所有資料被偷走 駭客可以有機會假裝是你) 所以安全性 Session > Cookie

3.因為Session存在服務器端 所以當Session量一大 會增加服務器的負擔 所以性能Cookie > Session

4.你也可以兩個都用 通常比較注重安全性的資料(比如說登入資訊)放在session 其他不重要資料放在cookie
