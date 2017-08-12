---
layout: post
title: Chrome console Error-Mixed Content
comments: True 
subtitle: 消除所有Mixed content error
tags: security https chrome
author: jyt0532
---
{% include doctor-jin.html %}

今天要Push新的文章之前 赫然發現我網站上面的可愛綠色鎖頭不見了
但也不是完全不見 他一開始有 過兩秒之後就變成一個驚嘆號
這簡直是不可能的事 我前兩天還確認過 SSL是11月到期 而且這個綠鎖頭一開始有 然後又突然不見了

實在詭異 不能容忍

打開console一看

![Alt text]({{ site.url }}/public/console_error.png)

簡直亂七八糟 以下各個擊破

### Adsblock

先處理容易的
![Alt text]({{ site.url }}/public/console_error_highlight1.png)

這代表說某些Ajax call被Adsblock擋住了 為什麼我知道呢 因為右上角有個可愛的7

![Alt text]({{ site.url }}/public/adsblock.png)

然後他又說BLOCKED_BY_CLIENT 那就八九不離十是被擋住了 果然把Adsblock關掉後那些就不見了

仔細研究了一下 其實每個Adsblock會有不同的rule 比如說你的網址有ads, click, doubleclick等等的ajax call都會被擋掉

### disqus 

![Alt text]({{ site.url }}/public/console_error_highlight2.png)

當我重新load網頁之後 我發現這個才是讓我的鎖頭消失的主因

前面的紅色ERROR顯示完之後我都還是有https的 但一秒後黃色的這個出現了 我的綠鎖頭就變成驚嘆號了

連結點開 是一個黑底的圖加上一個白色pixel在中間

![Alt text]({{ site.url }}/public/white_dot.png)

可疑到炸了 

仔細找了源頭才發現是我掛的Disqus(就是可以在下面評論的東西) 你在網頁放這個的時候如果你沒有特別設定
他會預設幫你追蹤使用者event 因為Disqus有放廣告的功能 我猜這是用來記錄使用者偏好 然後再依照使用者偏好放廣告的功能

既然我沒有要放廣告 我就應該把它關掉 依照[DISQUS討論區](https://disqus.com/home/channel/discussdisqus/discussion/channel-discussdisqus/bug_reports_feedback_redirect_chain_from_rlcdn_idsync_loadus_and_many_others/best/)的教學關掉了之後 
連剛剛Adsblock擋掉的東西都不見了呢

知道為什麼一個網站上面的綠色鎖頭很重要了吧 **因為我是HTTPS的網站 而那個一個白點的gif的來源是HTTP** 所以chrome抱怨了:
誒你跟使用者說你是HTTPS 可是你卻在load一個HTTP的東西 這樣對嗎？ 看我把你的綠鎖頭拿掉

這就是我網站的綠鎖頭被拿掉的原因

### Mixed content

![Alt text]({{ site.url }}/public/console_error_highlight3.png)

這也是間接跟最後一個ERROR有關的東西 當我發現這件事的時候簡直崩潰

我上一篇[Tracking Youtube Event教學](/2017/08/09/youtube-tracking/) 裡面內嵌的youtube iframe api是HTTP

所以跟剛剛的問題一樣 我是HTTPS的網站 你卻LOAD一個HTTP script 所以我就把你擋掉不Load

那為什麼跟剛剛的CASE不一樣呢?

差別差在要load的是什麼東西 剛剛的情況 是我load了一個gif 只要你load的是圖片 影像 音源等等 那就是Passive mixed-content 它就會load 然後給你一個黃色的Mixed content Warning

可是現在我要load的是Javascript 那就是Active mixed-content 他就不會load 直接噴一個ERROR給你看

### 怎麼改

當然最簡單的改法 就是把你所有load http script的地方改成https 比如說我還有一個地方也是load了http的script

{% highlight javascript %}
<link rel="stylesheet" href="http://fonts.googleapis.com/css?family=PT+Sans:400,400italic,700|Abril+Fatface">
{% endhighlight %}

最簡單的改法就是加個s把它變成https

{% highlight javascript %}
<link rel="stylesheet" href="https://fonts.googleapis.com/css?family=PT+Sans:400,400italic,700|Abril+Fatface">
{% endhighlight %}

但其實你還有個選擇 就是把前面的protocol拿掉

{% highlight javascript %}
<link rel="stylesheet" href="//fonts.googleapis.com/css?family=PT+Sans:400,400italic,700|Abril+Fatface">
{% endhighlight %}

這樣做的話 他就會用跟你website 一樣的protocol 你是https他就load https 你是http他就load http

### 結論

所以最崩潰的是什麼呢 就是[Gatsby那篇](/2017/08/04/facade/) 其實是有放個Youtube影片的 為了track這個影片的點擊率 我還寫了另外一篇[用Google Analytics追蹤使用者的Youtube體驗](/2017/08/09/youtube-tracking/) 結果因為一個Mixed content的問題 youtube iframe api完全沒load到

沒發現的主因是 我在local端跑網頁測試的時候是用localhost測 所以理所當然是http 所以剛剛說的那些問題全部沒出現

還在Gatsby那篇說什麼上面的片段是影史經典的片段 結果居然沒load出來

要是我的網站是營利的網站的話 這就是個巨大的失誤 

簡直就跟野野村龍太郎+仁醫一樣崩潰

<div id="Jin"></div>
