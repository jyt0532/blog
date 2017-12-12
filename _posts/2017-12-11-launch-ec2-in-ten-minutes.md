---
layout: post
title: 十分鐘在AWS架好個人部落格
comments: True 
subtitle: launch ec2 instance in 10 min
tags: blog growth
author: jyt0532
---

接下來就是實戰篇 想寫這篇文章的原因是上個禮拜我的ec2完全掛掉 登都登不進去 然後domain也連不進去 只好忍痛再開一個新的instance 整個重新migrate

為了讓接下來migrate的時候不用重新survey 就把過程記錄下來 

過程中有很多細節跟參數可以調整 但這篇的目標就是架一個靜態的部落格網站 所以硬體需求不高 有其他需求的讀者可以在每個步驟都仔細研究一下


### Launch instance

到AWS申請個帳號這些細節就不說了

到AWS Console 選EC2(Elastic Compute Cloud)

Launch Instance之後就會看到很多AMI可以選

![Alt text]({{ site.url }}/public/aws1.png)

我之前就是選了Amazon Linux AMI然後壞掉 發現他很多library的支援都不是很強 就想跳槽到Ubuntu 
可是Ubuntu 14.04裝jekyll會有很多問題 最後我發現最好的還是Ubuntu16.04 選下去就對了

下一步是依照你們需求選擇CPU Memory選項等等 我們架的是靜態網站 就用t2.micro就可以

下一步有很多選項 比如說要不要直接allocate一個Public IP 或是每個instance最多付多少錢(因為AWS是照流量計費 default是沒有上限 如果你被DDOS可能帳單會爆表) 還有需不需要Auto Scaling

等等會一步步帶到所有需要調整的設定 直接Launch就可以


### 選private/public key

在Launch之前 你需要創建一個key pair 把它建好的private key存在你自己電腦安全的地方 你用Mac或Linux的話 就存在 ~/.ssh/MyKeyPair.pem

![Alt text]({{ site.url }}/public/aws2.png)

public key他會幫你存在你剛創的instance裡

對非對稱式加密有興趣的可以參考[基礎密碼學](https://www.jyt0532.com/2017/02/25/simple-cryptography/)跟[SSH](https://www.jyt0532.com/2017/03/09/ssh/)

注意當你創建了一組之後 之後每一個新建的instace 都可以用之前的key pair登入

Launch之後 就會看到你可愛的instance正在running了

### Security group

這時候就要先開一下有什麼方式可以連到你的instance 預設是ssh 但你必須要把http跟https打開

找到左邊的security group
![Alt text]({{ site.url }}/public/aws3.png)

找到你的instance之後 下面有個inbound 按Edit -> Add Rule 

把HTTP跟HTTPS加上去 IP用Anywhere

### 設定固定ip

你新開的instance的ip 預設會是浮動的 就是有可能會變 如果你想要一個自己的只屬於你的ip也很簡單  左邊的選單有一個Elastic IP

![Alt text]({{ site.url }}/public/aws4.png)
 
裡面就給他allocate 你就有一個屬於自己的ip了 YAY 

但別忘了要把你剛剛allocate的ip associate到你的instance 如果你給你的instance掛上ip的話不用錢 但如果你要了一個ip但是卻放著不用 那就要收錢

### 自己的Domain

你可以在GoDaddy或是在Route53買自己的domain 如果你有買自己的Domain的話

一年12塊 我非常建議寫部落格不要放在別人的平台上 放在別人平台上所有流量都是別人的

因為我是在Route53買的 所以就在aws的介面搞定

![Alt text]({{ site.url }}/public/aws5.png)

就把你剛剛allocate的ip放在你買的domain上 等個五分鐘左右 就可以從你買的domain連到你的instance了


### 連上自己的主機吧

從任一一個terminal 打以下指令

{% highlight zsh %}
ssh -i ~/.ssh/你的私鑰.pem ubuntu@ip
{% endhighlight %}


ubuntu是你instance的預設user name

當然你應該要把下面這段加入你的~/.bash_profile

{% highlight zsh %}
alias sshaws="ssh -i ~/.ssh/你的私鑰.pem ubuntu@ip"
{% endhighlight %}

這樣只要打sshaws就可以

### 安裝需要的東西

連上去之後 

{% highlight zsh %}
sudo apt-get update
sudo apt-get install apache2
{% endhighlight %}


好了搞定 打開一個browser在網址打上你的固定ip或是你的domain 就會秀出一個網頁 這個網頁在

/var/www/html 裡面

你也可以指定你的網址的root是哪裡 到下面這個file

{% highlight zsh %}
sudo vim /etc/apache2/sites-available/000-default.conf
{% endhighlight %}

DocumentRoot /var/www/html

改成你放html的地方就可以


### 裝Jekyll

{% highlight zsh %}
sudo apt install jekyll
{% endhighlight %}

有沒有覺得用Ubuntu裝東西實在是太輕鬆了

Jekyll的教學就看[Jekyll x Github x Blog](https://rhadow.github.io/2015/02/18/Jekyll-x-Github-x-Blog-Part1/) 這個作者寫的很清楚 一開始學習曲線有點高 但之後就可以體會到自動化+hot reload的威力



### HTTPS 
[Let's Encrypt](https://certbot.eff.org/#ubuntuxenial-apache)

把上面的command run一下就可以

恭喜你 可以開始在自己的domain寫部落格囉 之後會再講到一些安全性的設定
