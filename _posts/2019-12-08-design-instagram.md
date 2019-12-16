---
layout: post
title: 系統設計 - 設計Instagram服務
comments: True 
subtitle: Design instagram
tags: system design instagram
author: jyt0532
---

設計Instagram/Facebook Feed/Twitter是系統設計的常見考題 這篇文章我們就來Mock一下迷途書僮 看看這些年來他準備系統設系準備的如何

還記得上次我們跟迷途書僮聊天 聊了許多關於如何大方向的準備系統設計 請參考[系統設計 - Design cache](/2017/03/27/system-design/)

我們還討論了[如何設計縮網址服務](/2019/12/05/design-tiny-url/) 

至於為什麼把instagram/Facebook feed/Twitter集合在一起 是因為news feed的概念比較類似

# Design Instagram

jyt0532: 今天我們來設計Instagram 概念很簡單 

1.使用者可以發文 這個文章可以有文字/照片/影片  

2.可以follow別人

3.可以生成NewsFeed 代表你follow的所有人的所有post 按照重要性順序列出來給你看

## Requirement

迷途書僮: 好 現在我要問**系統的需求** Consistency跟Availability哪個重要

jyt0532: Availability 畢竟有一個人發文之後 過了一下子才出現在別人的動態牆上問題也不大

迷途書僮: 感覺最麻煩的地方是News Feed generation 這個需求的latency要求呢

jyt0532: 200ms

迷途書僮: 還有其他要求嗎

jyt0532: 系統要非常reliable 使用者上傳的照片跟影片不能丟失

## Constraints

迷途書僮: 我想知道我們要處理的是多大的系統 Daily Active User/Daily Post等等

jyt0532: 

1.300M DAU

2.平均每個人每天會去看timeline 5次

3.200M new post everyday

4.發的文有文字也有照片 就假設平均post大小是10KB吧

迷途書僮: 

300M * 5 / 24 / 3600 = 17361 QPS for feed generation request

200M / 24 / 3600 = 2314 QPS for create post

Space needed to store post a day: 200M * 10KB = 2TB

Space needed for 10 years: 2 * 10 * 365 = 7.3PB

## API design

jyt0532: 接下來來設計一下服務的API吧

第一個: `createPost(api_dev_key, user_id, text, media_ids)`

就這麼簡單 text是這篇post的文字 media_ids是array of photos/videos ids

api_dev_key 指的是允許呼叫你的前端的token 讓server知道現在是誰在發送請求 我們就可
以分配/控制每個前端發送請求的bandwidth 比如我們可以限制每個使用者的請求數量 用途是避免惡意對手DDOS搞爆你的服務

回傳feed id

第二個: `getUserFeed(api_dev_key, user_id, since_id, count)`

回傳array of feed id

## Database design

迷途書僮: 我們需要至少以下幾個table

User Table:

| UserId(Primary Key) | Name | Email | DateOfBirth | CreateDate  | LastLogin |
| --- | --- | --- | --- | --- | --- |
| int | varchar(20) | varchar(32) | datetime | datetime | datetime |

UserFollow:

| Follower(Primary Key) | Followee(Primary Key) |
| --- | --- |
| int | int |

Feed Table:

| FeedId(Primary Key) | UserId | Content | CreateDate | NumsOfLike |
| --- | --- | --- | --- | --- |
| int | int | varchar(256) | datetime | int |

Media Table:

| MediaId(Primary Key) | Type | Title | Path | CreateDate |
| --- | --- | --- | --- | 
| int | int | varchar(256) | varchar(256) | datetime |

Path就是實際存放這個照片/影片的路徑 我們可以放在S3的機器上面 

> 凡是跟檔案有關的系統設計題 只有檔案的meta可以用Database存儲 檔案本身必須用其他的存儲 比如HDFS或是S3

FeedMedia Table:

| FeedId(Primary Key) | MediaId(Primary Key) |
| --- | --- |
| int | int |

把這些Table都使用RMDB存雖然簡單 但很難scale 

對於不同的table 我們可以選擇不同的NoSQL 比如說在這個例子 `User` `Feed` `Media`就可以用key-value store的NoSQL
比如說Redis/Dynamo/Voldemort 在這裡key就是PhotoId, value就是Photo的metaData等等

但對於Association table `UserFollow` `FeedMedia` 我們可以用wide-column的NoSQL比如說Cassandra 以`UserFollow`為例 key就是UserId, value就是所有這個user follow的人 以`FeedMedia`為例 key就是FeedId, value就是這個feed所擁有的mediaId

## Algorithm

jyt0532: 這個系統設計問題的難點有兩個 第一個是如何生成Feed 第二個是如何推送Feed

### 生成Feed

迷途書僮: 基本演算法如下 當Alex要看他的Timeline時

1.系統找出所有Alex的朋友`userIds` 

2.對於每個userIds 找出他們最popular或最重要的post

3.對於post排序

4.把結果存在cache 回傳前20個當作請求的回傳值

5.當前端讀完了前20個 可以再發個請求要接下來的20個

要注意的是如果Alex一直在線上 那我們可能每五分鐘要重新生成一次NewsFeed 更新Alex的Timeline給他

基本的SQL長這樣(當然我們不一定選擇的是SQL Database 但SQL語言是最好理解的 所以解說的時候用SQL 實作的時候在轉成選定的NoSQL syntax)

假設Alex的ID是123
{% highlight sql %}

SELECT 
	FeedId 
FROM 
	Feed 
WHERE
	UserId
IN (
  SELECT 
	  Followee
  FROM
	  UserFollow
  WHERE
	  Follower = '123'
) 
{% endhighlight %}

看起來簡單 但有幾個問題

1.慢 很慢 非常慢 特別是如果Alex follow了很多人的情況下

2.這個之後還要sort

3.每當被follow的人有了新的post 這個SQL都必須重跑 才跑得出新的那篇post

既然online生成news feed無法接受 那就試試offline吧 做法也簡單 就是有事沒事就先跑出來 Alex想看Feed的時候就直接從Memory拿出來給他 

那既然我們都已經花時間預先生成了News Feeds 我們勢必要有個資料結構可以讓我們online存取更快 這個資料結構需要達成幾件事

1. 每個使用者都預先生成好

2. 要知道上次替這個使用者預先生成的時間

3. 當使用者讀完前20個 要再往下看的時候 可以直接再給他20個

答案呼之欲出 我們需要的資料結構就是

{% highlight cpp %}
Map<
	UserId, 
	Struct {
		LinkedHashMap<FeedId, Feed> feeds,
		DateTime lastGeneratedTime
	}

>
{% endhighlight %}

[LinkedHashMap](https://www.geeksforgeeks.org/linkedhashmap-class-java-examples/)基本上就是Double-LinkedList 加上HashMap 就是我們在實作LRU的時候常用到的資料結構

當我們有了這個之後呢 我們可以輕鬆的iterate `feeds` 這個map 當使用者想要再看20個 我們就輸入最後一個feed的feedId 就可以輕鬆再iterate 20個

jyt0532: 看起來不錯 但你這個LinkedHashMap要為每個使用者存多少entry呢 50? 500? 5000?

迷途書僮: 可以先預設500 但也可以根據每個使用者改變 如果是重度使用者 我們可以很頻繁的更新 每次都500個 如果是個很少使用的人 那50個也無妨

jyt0532: 對每個使用者都這樣做未免也太累了吧

迷途書僮: 我們可以只對DAU做 我們也可以用一個LRU Cache來把很少login的使用者的預先計算的map踢出cache 再聰明一點 我們學習每個user的使用pattern 在他上線之前先算好就可以 這樣可以省去很多重複的計算


### 推送Feed

jyt0532:剛剛說明的都是一般的情況 Alex想看他的News Feed 系統就給他 

那如果Alex一直把Instagram/Facebook的頁面開著 如果Alex follow的人有了新的post 我們如何更新Alex的Timeline呢

換個角度來說 如果Alex發了篇新的文章 我們如何快速的讓所有follow Alex的人看到更新呢

迷途書僮: 基本上有兩種方式

#### Pull model

又稱為Fan-out on load 意思就是client定期的跟server要更新 缺點有兩個

1.當有新的post進來時 follower不會第一時間知道 要等到client去問的時候才知道 

2.大多數情況下 client跟server問更新時什麼都不會回傳 白白浪費bandwidth

#### Push model

又稱為Fan-out on write 意思就是只要有Followee寫了新post 就會發送給所有followers 這就只會在真的有更新的時候才需要使用到網路流量 但先決條件就是client跟server必須先維持一個[Long Poll]()

缺點也很明顯 如果一個名人發文的話 push model會在短時間內需要做很多的事


jyt0532: 那你的選擇是?

![Alt text]({{ site.url }}/public/choices.jpeg)

迷途書僮:

![Alt text]({{ site.url }}/public/mixed.jpg)

我們兩種一起用!

#### hybrid model

我們對於follower不多的人 使用Push model 

對於follower多的人 使用Pull model

這是最簡單的綜合解法 當然對於follower很多的名人 我們可以對所有在線上的followers使用Push model也可以 這也大幅的減少了對所有人Push所需要的bandwidth

???上個圖????

## Ranking 

jyt0532: 對於一個Timeline裡面的Feed排序 該怎麼做

迷途書僮: 當然就是把 createdTime, numberOfLikes, numberOfComments, numberOfShares, havePhotos/Videos等等的feature用機器學習的模型產生個分數 這就需要ML專家出馬 

當然好的Ranking算法是會自我調整 一直觀察使用者的使用情況(stickiness/retention/revenue)來分析改進的

## Data sharding

jyt0532: 我們一天產生2TB的文章要存 總不可能都放在同一個機器吧 所以勢必要處理sharding的問題 有兩個東西要分區

1.Sharding post and metadata

2.Sharding feed

迷途書僮:我們各個擊破

### Sharding post and metadata

迷途書僮: 最直觀的方式 Shard by **UserID**

假設有N台機器 就把發文的使用者UserId -> `Hash(UserId)%N` 就知道要存到哪台機器 要讀取的時候用一樣的算法找到我們把UserId的文章存在哪台機器中

jyt0532: 這個做法會有一些問題 最常見的就是hot user 這樣對於少量的機會很容易會有大量的請求 況且除了unbalanced Traffic之外 還有unbalanced Storage的問題 也就是發文的都是那幾個人 不平均分佈的存儲也是很惱人的地方 有其他解法嗎

迷途書僮: 不然就 Shard by **PostId**

PostId -> `Hash(PostId)%N` 

jyt0532: 這樣同一個發文者的Post分散在各台機器 你怎麼集合起來?

迷途書僮: 要找UserId的發文 我們要**對所有的機器找** 每台機器回傳它所存的UserId的發文

演算法變成 

1.Alex要看他的timeline

2.Application Server問UserFollow table 來找出Alex所有朋友/Followee

3.對所有FeedTable問說每個Alex的朋友的所有文章

4.Combine and Rank 

jyt0532: 這的確是解決了hot server的問題 但每次都問所有database 也會造成極高的latency 有其他解法嗎

迷途書僮: 不然就 Shard by **creation time**

jyt0532: 這麼狂的嗎

迷途書僮: 按照文章的creation time來存可以保證每次都只需要query一小部分的機器

jyt0532: 那unbalanced traffic的問題又出現了不是嗎 每次都寫進讀取那小部分的機器

迷途書僮: 那就只好再做一次撒尿牛丸了

jyt0532: 願聞其詳

迷途書僮: 我們把**PostId跟Creation time合起來 shard**

我們需要做的事 就是**把timestamp加進PostId裡面**

jyt0532: 你的PostId到底要多長?

迷途書僮: PostId = timeStamp + counter 假設我們系統維持50年

`50 * 365 * 24 * 3600 = 1.57*10^9 < 2^31`

我們總共需要31個bit來代表一個timestamp

每個秒需要多少counter呢 我們的QPS是2314 < 2^12 

12bit可以應付2314(平均)的QPS 但考慮到尖峰時段的QPS 還有給多留點後路加上湊個8的倍數 我們就選17吧

這樣PostId的長度就是31+17 = 48 bits

假設現在時間是1575936000

那這一秒產生的FeedId就是 

1575936000 000001

1575936000 000002

1575936000 000003

...依此類推

這個做法雖然還是需要問所有的database 但是第一 這個設計減少了寫的時間(因為我們不需要對createdTime index) 第二 read會變得非常的快 因為我們不用再filter by createdTime


如果我們資料庫選擇用的是RMDB的話 我們可以在`Feed` Table上index by (FeedId, CreateDate) 也是一樣的效果






