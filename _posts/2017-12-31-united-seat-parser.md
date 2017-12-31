---
layout: post
title: 寫腳本幫你自動找聯合航空網站的窗戶位
comments: True 
subtitle: script to parse united airline website
tags: script automate selenium python
author: jyt0532
---

{% include united_youtube_demo.html %}

最近回台灣買了一班470的便宜機票 但崩潰的是沒有靠窗戶的座位 下載了UA的app才知道原來可以隨時連上UA去看有沒有好的位置 只要有就可以換

連續看了兩三次之後都沒有座位 覺得這種可以自動化的東西我手動的一直看 實在愚蠢 於是便稍微研究了一下 才發現其實寫了自動化的程式真的挺簡單的

看完這篇文章後 你就可以自動化腳本去爬任何網站 比如搶iphone預購等等 

這篇文章會用python教學

### Selenium

[Selenium](http://www.seleniumhq.org/)是一個挺成熟的開源網頁測試框架 你可以用它來測試你的網頁怎麼點會跑到哪個網頁等等是否符合預期

它就是可以幫你的前端測試自動化 不用再手動去點你的網頁看有沒有出錯

既然可以自動化 我們就可以拿來用 在Mac裡 就打下列指令就可以安裝

(介紹Selenium)
{% highlight zsh %}
brew install geckodriver
{% endhighlight %}

現在來試著打開瀏覽器

{% highlight python %}
browser.get('https://www.google.com')
browser = webdriver.Firefox()
{% endhighlight %}

完美 可以開始爬網站了

### 登入

如果你要爬的是靜態的網頁 那可以跳過這一步 但常常我們要爬資訊之前 需要先登入

直接跳到聯合航空的登入頁面
{% highlight python %}
browser.get('https://www.united.com/ual/en/us/account/account/signin')
{% endhighlight %}

首先要先找到登入頁面中 輸入帳號密碼的那兩個HTML input element

![Alt text]({{ site.url }}/public/united_parser_1.png)

{% highlight python %}
username = browser.find_element_by_id("MpNumber")
password = browser.find_element_by_id("Password")

username.send_keys(UNITED_ACCOUNT)
password.send_keys(pswd)

browser.find_element_by_id("btnSignIn").click()
{% endhighlight %}

send_keys就是模擬我們打字 把你要打的字當作input丟進去

登錄之後找到你班機的網頁中 換座位的地方

![Alt text]({{ site.url }}/public/united_parser_2.png)

{% highlight python %}
browser.get('https://www.united.com/ual/en/tw/mytrips/trips/flightseats?cn=XXX')
{% endhighlight %}

### 找Pattern

這大概是之後你要寫自動化腳本 必經且最重要的一步 就是你要自己去研究這個網頁的排版 你感興趣的資訊是在哪個tag 是哪一個類別或id

像我感興趣的 是available的位置 

![Alt text]({{ site.url }}/public/united_parser_3.png)

{% highlight python %}
availables = browser.find_elements_by_class_name('available')
{% endhighlight %}

但是這會找到所有艙等的位置 這時候就簡單的字串過濾一下

{% highlight python %}
    for item in availables:
        text = item.text
        if "Economy Plus" not in text and "Window" in text and "premium cabin" not in text:
            print "Got it"
            send_email()
            break
{% endhighlight %}

如果找到了經濟艙的窗戶位 就可以寄信給自己了
### 寄email

要在script裡面寄信 需要用到SMTP(Simple Mail Transfer Protocol)協定

python都有強大的支援 只要輸入帳密就可以

{% highlight python %}
server = smtplib.SMTP( "smtp.gmail.com", 587 )
server.starttls()
server.login(GMAIL_ACCOUNT, GMAIL_PASSWORD)
server.sendmail( 'BoYu Mac', 'jyt0532@gmail.com', 'New seat open' )
{% endhighlight %}

### 關掉瀏覽器
{% highlight python %}
browser.close()
{% endhighlight %}


### DEMO

跑腳本 只要找到窗戶的位置就寄信給我

<div id="youTubePlayer3"></div>


[Github](https://github.com/jyt0532/united_parser/blob/master/find.py)
