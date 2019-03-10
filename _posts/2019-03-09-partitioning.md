---
layout: post
title: Designing Data-Intensive Application - Partitioning
comments: True 
subtitle: 分區
tags: systemDesign 
author: jyt0532
excerpt: 本篇文章介紹分區
---

這是Designing Data-Intensive Application的第二部分第二章節: 分區

本文所有圖片或代碼來自於原書內容
{% include copyright.html %}

# 分區

我們在第五章討論了複製 指的是數據在不同節點上的副本 但如果數據本身就很大 無法單獨存在一個節點 我們還要把數據進行分區(partitioning)又稱分片(sharding)

> 術語
> 在MongoDB, ElasticSearch和Solr Cloud稱為shard
> 在HBase中稱為Region
> 在Cassandra和Riak中稱為vnode
> 在Couchbase稱為vBucket
> 但約定俗成的說法就是分區

通常情況下 每條數據**屬於且僅屬於**一個分區 有很多方法可以實現這點 本章會進行深入討論 實際上 每個分區都是自己的小型資料庫

分區的主要目的是可擴展性(scalability)  對於單個分區上的Query 每個節點可以單獨執行 所以可以輕易藉由增加節點來擴大吞吐量(當然有些複雜一點的查詢可能會需要跨越節點處理)

這一章我們會先介紹分割大型數據的不同方法 並觀察索引如何和分區配合 然後會討論如何平衡分區(如果你想要添加或移除節點) 最後則是討論數據庫如何把請求route到正確的分區來執行查詢

## 分區與複製

分區通常與複製一起使用 意思就是你分完區之後 每個分區的數據也都同時備份到好幾個節點上 增加容錯能力

既然知道一個節點可以存儲多個分區 那在使用主從模型的前提下 每個節點可以有一個領導者分區跟若干追隨者分區

![Alt text]({{ site.url }}/public/DDIA/DDIA-6-1.png)

上圖的例子 你可以想成 我先把資料分區分成四個部分 Node1拿第一個部分 Node3拿第二部分 Node2拿第三部分 Node4拿第四部分 然後再進行複製 把第一部分分區分給Node3, Node4進行複製 第二部分分區分給Node1, Node3進行複製... 依此類推

這樣的配置即使你真的天選的衰 同時有兩個Node掛掉 你還是可以把你的資料救回來

複製的選擇跟分區的選擇沒有太大相關(你想怎麼分區跟你想複製幾份沒有太大關聯) 為了本章討論簡單起見 本章先不考慮複製的問題

## 鍵值數據的分區 Partition of Key-Value Data

來實際討論該怎麼分區 我們希望分區能達成的目的是

**數據和查詢負載均勻分佈在各個節點上**

如果每個節點公平分享data跟traffic 那你有十倍的節點應該可以增加十倍的吞吐(不考慮複製) 所以如果分區分的不公平 某些分區有著比較多的data或traffic 我們稱為偏斜(skew) 偏斜的分區會導致效率下降 而高負載的分區稱為hot spot

避免熱點的簡單方式就是讓數據**隨機分配**給不同節點 這樣保證數據平均分配 但當你查詢時 你就得每個節點都查詢 當然我們有些更好的方法

### 根據key的range分區

一種分區的簡單方法 就是為每個分區指定一段連續的key的範圍 下圖是百科全書的分區
![Alt text]({{ site.url }}/public/DDIA/DDIA-6-2.png)

你想要找Beautiful就知道是在第二區 想找jyt0532就是在第六區

注意因為key的範圍並不是均勻分布 所以數據分布的也不均勻 因為每個字母開頭的單字不一樣多 所以如果只是簡單的規定每兩個字母一區 很容易會出現偏斜 為了平均分配 分區的邊界需要進行調整 (比如上圖A-B在第一區 可是T-Z是在最後一區)

至於在每個分區中 我們可以[按照順序來存儲key](參閱[SSTables和LSM-樹](/2019/01/19/storage-and-retrieval/#sstables和lsm樹)) 好處是進行範圍掃描非常簡單 你也可以把key接起來當作連接索引處理(參閱[多列索引](/2019/01/19/storage-and-retrieval/#multi-column-indexes)) 你就可以一個查詢得到多筆數據 

比如說我們的數據是有關網路感測器的數據 primary key是測量的時間(year-month-day-hour-minute-second) 那範圍掃描就很好用 我們可以輕鬆獲取每個月的所有數據

但是key range的分區缺點也很明顯 就是某些特定的訪問模式會產生熱點 剛剛的例子 primary key是時間戳 那不幸的是 每日的寫入都在同一個分區 這樣就會造成某一個節點過載 其他節點空閒

為了避免這個問題 我們還得加上除了時間戳以外的其他東西作為primary key的第一部份 比如時間戳前加上sensor名稱 這樣primary-key變成

sensorName-year-month-day-hour-minute-second

這樣不同的感測器同時寫入 最終就會平均的分佈在不同節點 

但當你想要獲取一個時間內所有的感測器的資料 你就得要每個節點都下一樣的範圍查詢


## 按照Key的Hash分區

因為偏斜和熱點的風險 使得許多分布式數據存儲都使用hash來決定一個key的分區

一個好的hash function可以讓數據平均分佈 因為hash function的目的只是用來分區 所以其實我們不需要用到太強的hash演算法(MD5就差不多夠用了) 當你定義好了hash function 你就可以為每個分區分配一個hash範圍(不是key的範圍)
![Alt text]({{ site.url }}/public/DDIA/DDIA-6-3.png)


@@@@@@@@@@@@Consistent hash

缺點也很明顯 我們失去了對於key的**範圍搜索**的能力 曾經相鄰的主鍵被分散在不同的分區中

MongoDB裡面 如果使用了這個方法 那所有範圍搜索都必須發送到所有分區

Riak, Couchbase, Voldemort則不支持主鍵上的範圍查詢

Cassandra則是採取折衷的策略 Cassandra中的表可以使用由多個列組成的**復合主鍵(compound primary key)** key裡面的第一個列拿來hash 其他列用來當作SSTable中排列的連接索引 雖然查詢無法在復合主鍵的第一列中做範圍查詢 但如果第一列已經被指定固定值 其他列就可以做範圍查詢

Cassandra的連接索引也為一對多數據提供了一個優雅的數據模型 比如說一個社交網站 一個用戶會發布很多更新 如果更新的主鍵被選擇是(user_id, update_timestamp) 那麼你可以有效的查詢一個特定用戶的一定範圍內的更新 因為每個用戶存在不同分區 每個分區內可以做範圍查詢

## 負載傾斜與消除熱點

雖然由hash分區可以減少熱點 但還是無法完全避免 畢竟大多數情況下 所有的讀寫操作都是針對同一個key 所有的請求都會被route到同一個分區

比如常見的例子是一個社交名人發了一篇文章 這個事件會導致大量的寫入到同一個key 這還是很可能導致負載爆掉 常見的解決辦法是讓應用程式primary的結尾加一個隨機數 當你加的是兩個位數的十進位數 就可以把主鍵分成100個不同的分區

那當然也有缺點 就是你的讀取就比較痛苦了 你必須把100個分區的數據合併 所以你需要其他分法來追蹤哪些鍵需要被分割和怎麼分割

## 分區和次級索引(Partitioning and Secondary indexes)

目前為止的討論方案都是依賴於key-value模型 如果只透過key來訪問紀錄 我們可以從key來決定分區 並且將請求導到相對應的分區來處理

但如果涉及secondary index 情況就會變得複雜 次級索引的問題是他們不能整齊的映射到分區 所以有兩種針對二級索引數據進行分區的方法

1.基於文檔的(document-bases)分區

2.基於關鍵詞(term-based)的分區

### 由文檔來分區二級索引

假設你正在經營一個賣車的網站 每個記錄都有一個文檔id 並用文檔id對數據進行分區(分區0分配id 0-499 分區1分配id 500-999等等)

你想讓用戶搜索汽車 並讓他們由顏色或是廠商來過濾 那你就必須在顏色和廠商上面創造二級索引
![Alt text]({{ site.url }}/public/DDIA/DDIA-6-4.png)

你每在一個分區中加入一個新的記錄 這記錄也都會更新你的二級索引表 那你下次就可以知道分區二中紅色的車只有768這台

這種索引方法中 每個分區是完全獨立的 每個分區維護自己的二級索引 不在乎其他人的二級索引 所以文檔分區索引又稱為本地索引(local index)

要注意的是 當你要找所有紅色的車 你還是必須對每一個分區都下一樣的查詢 這種查詢分區數據庫的方法稱為scatter/gather 並且會讓二級索引的查詢非常昂貴 即使你平行的對每個分區都進行一樣的查詢 scatt
er/gather會導致[尾部延遲放大](/2019/01/05/reliable-scalable-and-maintainable-application/#section-8) 但這個方法卻被廣泛使用 MongoDB, Riak, Cassandra, ElasticSearch, SolrCloud和VoltDB都使用文檔分區進行二級索引

通常你的數據庫會建議你建構一個可以剛好從一個分區提供二級索引的方案 比如說紅色車都在分區1 藍色車都在分區2 但這通常不可行 特別是你需要提供不同的二級索引需求(比如顏色跟廠商)

### 由關鍵詞來分區二級索引

相對於每個分區擁有一個自己的二級索引 我們也可以創建一個**全局索引** 這個全局索引包含了所有分區的所有數據的索引 單是我們不能只把這個索引存在單獨一個節點上 因為這可能會讓那個節點成為bottleneck(同時分區也失去意義) 

所以全局索引也要進行分區 

直上例子

![Alt text]({{ site.url }}/public/DDIA/DDIA-6-5.png)

所有紅色的車都存在紅色索引中 而紅色索引本身也被分區到Partition0(color索引開頭a-r在partition0 s-z在partition1 廠商索引a-f在partition0 廠商索引g-z在partition1)

我們稱這種索引稱為**關鍵詞分區**(term-partitioned) 因為我們尋找的關鍵詞決定了索引的分區方式

更厲害的是 **關鍵詞的分區** 跟 **主鍵的分區** 方式不用一樣 你主鍵分區可以用hash 關鍵詞分區可以直接照term分區 或是hash過再分區 你可以自己比較優劣 

比如你二級索引有價格的話 那你直接照價格(term)分區 還可以輕鬆的提供範圍查詢 但如果你想要平均的分佈traffic 你也可以把關鍵詞Hash過後再分區

關鍵詞分區的全局索引優於文檔分區索引的地方當然就是查詢的效率問題 不需要scatter/gather 客戶只需要向包含關鍵詞的分區發出請求 當然缺點就是寫入比較慢而且複雜 因為你寫入單個文檔可能會影響索引的多個分區 比如你這台車的顏色紅色索引存在分區1 這台車的廠商索引存在分區2 等等

理想情況下 索引總是最新的 寫入數據庫的每個文檔都會立即反映在索引中 但關鍵詞分區的全局索引就比較複雜 需要跨分區的分佈式transaction 並不是所有數據庫都支持

實際情況下 對全局二級索引的更新通常是異步(asynchronous)的 意思就是如果在寫入之後很快就讀取 很可能會讀不到 Amazon DynamoDB聲稱在正常情況下二級索引會在不到一秒的時間內更新 但在infrastructure有故障的時候會有延遲

全局關鍵詞分區索引還有其他用途 比如說Riak的搜索功能和Oracle的數據倉庫







![Alt text]({{ site.url }}/public/DDIA/DDIA-6-1.png)
![Alt text]({{ site.url }}/public/DDIA/DDIA-6-1.png)
![Alt text]({{ site.url }}/public/DDIA/DDIA-6-1.png)
![Alt text]({{ site.url }}/public/DDIA/DDIA-6-1.png)
![Alt text]({{ site.url }}/public/DDIA/DDIA-6-1.png)



















