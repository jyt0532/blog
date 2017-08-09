---
layout: post
title: Youtube tracker in google analytics
comments: True 
subtitle: 用Google Analytics追蹤使用者的Youtube體驗
tags: googleAnalytics Youtube
author: jyt0532
---
{% include youtube_tracking.html %}

如果你的網站有掛Google Analytics 他可以幫你track許多event
最近在研究如果放了youtube影片 Google Analytic可以track到什麼地步

基本的就是有多少人播放 暫停 放完等等 但只要你會寫程式
你可以track非常多事情 網路上好像還有不少人開課教PM怎麼track這些東西
但其實非常簡單 連Tag Manager都不需要用到 只要你的網頁有掛GA的都可以

Let's Go!

### Youtube iframe

[Youtuve iframe](https://developers.google.com/youtube/iframe_api_reference)是最好的在網站嵌入影片的方式 他的API非常的健全 可以自動播放 允不允許全螢幕等等[Iframe DEMO](https://developers.google.com/youtube/youtube_player_demo) 裡講得很清楚 有興趣的自行研究

你在你的html裡需要加上的就只有一個div
{% highlight html %}
<div id="youTubePlayer_test"></div>
{% endhighlight %}

剩下的交給javascript處理

我們看一下API Doc 

{% highlight javascript %}
<script>
  var tag = document.createElement('script');
  tag.src = "https://www.youtube.com/iframe_api";
  var firstScriptTag = document.getElementsByTagName('script')[0];
  firstScriptTag.parentNode.insertBefore(tag, firstScriptTag);
</script>
{% endhighlight %}

...

...

...
<br><br><br>
我看到這段第一印象就是
![Alt text]({{ site.url }}/public/chou1.jpg)
這不就只是這樣嗎
{% highlight javascript %}
<script src="http://www.youtube.com/iframe_api"></script>
{% endhighlight %}

認真研究了一下 原來是有差別的 原本的那段會asyncy load script

Asynchronous load代表那些script可以同時下載

Synchronous 也就是通常的版本 一個script tag接著一個script tag下載 會比較慢

原來google用心良苦 連load一個script都考慮得這麼細節

讓我們繼續看下去

{% highlight javascript %}
function onYouTubeIframeAPIReady(event) {
  player = new YT.Player('youTubePlayer_test', {
    videoId: 'jIDRwcB2SPA',
    events: {
      'onReady': PlayerReady,
      'onStateChange': PlayerStateChange
    }
  });
}
{% endhighlight %}

這一段code可以customize你嵌入的youtube影片 長寬等等
把videoId改成你要播的 別忘了Player的第一個參數要跟一開始的div id一樣

好戲上場

### 發送事件到GA

{% highlight javascript %}
function PlayerReady(event) {
  // do nothing
}
var pauseFlag = false;
function PlayerStateChange(event) {
  // when user click Play
  if (event.data == YT.PlayerState.PLAYING) {
    ga('send', 'event', 'Videos', 'Play', 'Test Video');
    pauseFlag = true;
  }
  // when click Pause
  if (event.data == YT.PlayerState.PAUSED && pauseFlag) {
    ga('send', 'event', 'Videos', 'Pause', 'Test Video');
    pauseFlag = false;
  }
  // when video ends
  if (event.data == YT.PlayerState.ENDED) {
    ga('send', 'event', 'Videos', 'Finished', 'Test Video');
  }
}
{% endhighlight %}

PlayerReady 就是當onReady這個event發生之後的callback function 這裡我們不做事

PlayerStateChange 就是當youtube狀態改變的時候要做什麼 User按Play的時候就send一個Play Event 按暫停的時候就send Pause Event 影片播完就send Finish Event

打完收工

你在你網站上的youtube亂玩一下後 就可以看到Google analytic搜集到你的使用情況了

![Alt text]({{ site.url }}/public/ga.png)

### GA debugger

這裡要推薦一個很好用的extension[Google analytics debugger](https://chrome.google.com/webstore/detail/google-analytics-debugger/jnkmfdileelhofjcijamephohjechhna?hl=en)

裝了之後你就可以開你的console看到底什麼event被發送給GA


![Alt text]({{ site.url }}/public/ga2.png)

這就是每個人的GA都有的pageview event

非常好用

### 嘿嘿嘿

當你知道你可以在所有event都放callback function之後 
就可以做很多壞事了
我先來拋磚引玉一下

要做壞事前要先看好Doc:[Event Tracking](https://developers.google.com/analytics/devguides/collection/analyticsjs/events)

他除了給eventCategory, eventAction, eventLabel之外 還可以給eventValue

那就精彩了 我就可以tracking每個user到底實際上看了多久的影片 在user把頁面關掉的moment傳回我的GA

![Alt text]({{ site.url }}/public/ga_code1.png)

夠邪惡吧 連我用一個GA都可以輕鬆track了 基本上你常用的網站都在做同樣的事 你停在一個feed多久 影片看多長 
都可以輕鬆紀錄
