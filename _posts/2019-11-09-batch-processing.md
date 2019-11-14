---
layout: post
title: Designing Data-Intensive Application - Batch Processing
comments: True 
subtitle: 批處理
tags: systemDesign 
author: jyt0532
excerpt: 本文講解批處理
---

這是Designing Data-Intensive Application的第三部分第一章節: 批處理

本文所有圖片或代碼來自於原書內容
{% include copyright.html %}


# 批處理

本書的前兩部份 我們探討了很多關於request和query的問題以及相對應回傳的結果 許多現有數據系統中都採用這種方式: 你發送一個請求 一段時間後系統給出一個結果 不論系統是數據庫 緩存 搜索索引 還是Web Server都一樣

這樣子的online系統 因為請求通常是由人類發起的(瀏覽網站等等) 他們正在等待回覆 所以我們非常在乎系統的response time

因為Web和越來越多的基於HTTP/REST的API的系統 使得上述的這種`請求/響應`風格變得如此普遍 讓大家以為這是理所當然 但這並不是建構一個系統的唯一方式 我們總共有三種不同類型的系統

1.Service(online system)

Server等待客戶的請求或指令到達 對於收到的每一個請求 server都會盡快處理它 並回傳一個響應 
**回傳響應**的時間通常是一個server性能的衡量指標 可用性非常重要

2.Batch processing system(offline system)

批處理系統有大量的輸入數據 需要跑一個job處理一些輸入 產生一些輸出 往往需要比較長的時間(幾分鐘到幾天都可能) 所以通常不會有用戶等待作業完成 

批量作業通常會定期運行(比如一天一次) 而主要性能衡量標準通常是**吞吐量** 也就是處理特定大小的輸入所需的時間 

3.Stream processing system(near-real-time system)

流處理介於online和offline之間 所以稱為near-real-time或是nearline

像批處理系統一樣 流處理消費輸入並產生輸出(並不響應需求) 但是 流式作業在事件發生後不久就會對事件進行操作 而批處理作業則需等待固定的一組輸入數據 這種差異使流處理系統比起批處理系統具有更低的延遲


&nsbp;

在本章中我們將會看到 批處理是構建可靠 可擴展和可維護應用程序的重要組成部分 比如2004年轟動武林的Map-Reduce 被稱為造就Google大規模可擴展性的算法 隨後也在各種開源數據系統中得到應用 包括Hadoop CouchDB和MongoDB

在本章中 我們將瞭解MapReduce和其他一些批處理算法和框架 並探索它們在現代數據系統中的作用

首先我們將看看使用標準Unix工具的數據處理 雖然可能你已經很了解了 但Unix的哲學也值得一讀 Unix的思想和經驗教訓可以遷移到大規模的分佈式數據系統中

## 使用Unix工具的批處理

來個例子 假設你有台Web服務器 每次處理請求時都會在日誌文件中附加一行 以下例子使用的是nginx預設的訪問日誌格式

{% highlight txt %}
216.58.210.78 - - [27/Feb/2015:17:55:11 +0000] "GET /css/typography.css HTTP/1.1" 200 3377 "http://martin.kleppmann.com/" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_5) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/40.0.2214.115 Safari/537.36"
{% endhighlight %}

事實上這只有一行 但因為排版問題看起來像是很多行 要讀懂一個日誌要先知道格式

{% highlight txt %}
$remote_addr - $remote_user [$time_local] "$request"
$status $body_bytes_sent "$http_referer" "$http_user_agent"
{% endhighlight %}

![Alt text]({{ site.url }}/public/DDIA/translatetranslate.jpeg)

所以這一行日誌的意思是

在2015年2月27日17:55:11 UTC的時候 服務器收到從ip位址216.58.210.78的地方 接收到對文件`/css/typography.css`的請求 但因為用戶沒有被認證 所以$remote_user被設置為`-` response code 200代表響應成功 response的大小是3377bytes 網頁瀏覽器是Chrome 40 `http://martin.kleppmann.com/` 頁面中的引用導致該文件被加載

### 分析簡單日誌

很多工具可以從這些日誌文件生成關於網站流量的漂亮的報告 但我們來自己練習一下Unix的基本工具 假設你想要在你的網站上找出五個最受歡迎的網頁

{% highlight sh %}
cat /var/log/nginx/access.log | #1
	     awk '{print $7}' | #2
	     sort             | #3
	     uniq -c          | #4
	     sort -r -n       | #5
	     head -n 5          #6
{% endhighlight %}

這一行命令做了什麼呢

1.讀取日誌文件

2.每行只輸出第七個字段(請求的URL) 在上例是`/css/typography.css`

3.按字母順序排列請求的URL表 如果一個URL出現過n次 那排序後這個URL會連續出現n次

4.`uniq` 命令通過檢查兩個相鄰的行是否相同來過濾掉輸入中的重複行 `-c` 表示還要輸出一個計數器 表示出現過幾次

5.按每行起始處的數字排序(`-n`) 這是URL的請求次數 大的數字在前(`-r`)

6.只輸出前五行(`-n 5`)

輸出如下所示

{% highlight sh %}
4189 /favicon.ico
3631 /2013/05/24/improving-security-of-ssh-private-keys.html
2124 /2012/12/05/schema-evolution-in-avro-protocol-buffers-thrift.html
1369 /
915 /css/typography.css
{% endhighlight %}

Unix工具非常強大 他可以在幾秒內處理幾GB的日誌文件 還可以根據需要輕鬆修改命令 只要你能輕鬆自在的運用`awk` `sed` `grep` `sort` `uniq`和`xargs` 那你幾乎可以做出和資料分析

### 自定義程序

除了Unix的chain of command之外 你還可以寫一個簡單的程序來做同樣的事情

{% highlight ruby %}
counts = Hash.new(0)         # 1
File.open('/var/log/nginx/access.log') do |file| 
    file.each do |line|
        url = line.split[6]  # 2
        counts[url] += 1     # 3
    end
end

top5 = counts.map{|url, count| [count, url] }.sort.reverse[0...5] # 4
top5.each{|count, url| puts "#{count} #{url}" }                   # 5

{% endhighlight %}

1.`counts` 是一個計數器的HashMap 保存了每個URL被瀏覽的次數 初始值為0

2.逐行讀取日誌 每行第七個被空格分隔的字段為URL

3.把HashMap中相對應的值加一

4.按計數器值對哈希表內容進行排序 並取前五位

5.印出前五個URL

這段程式不像Unix command那麼簡潔 但卻好懂很多 執行的流程也有很大的差異

### sorting VS in-memory aggregation

Ruby腳本在內存中保存了一個URL的HashMap 但Unix沒有 Unix依賴的是先對所有URL排序 排序完之後再把重複出現的加起來

哪種方法好呢 如果你只是個刷題機器 你可以輕鬆地回答 這也太簡單了吧 Ryby時間複雜度`O(n)` Unix時間複雜度`O(nlogn)` 哪有什麼好比?

![Alt text]({{ site.url }}/public/DDIA/chou-10-1.jpeg)

但正確答案是取決於你的應用 如果造訪你網站的都是那一兩個網站 來了一兩百萬次 那對Ruby來說 就只是一個HashMap的Entry 那即使在一個性能很差的筆電也跑得起來

但如果造訪你的網站的網站有上千萬個每個都只來一兩次 那可能每個網站的網址加起來放不進你的內存 整個程式Crash 這種時候 Unix sort就可以派上用場了 
為什麼呢 因為排序的優點就是可以高效地使用磁盤 我們在[SSTables和LSM樹](/2019/01/19/storage-and-retrieval/#sstables和lsm樹)中討論過的原理是一樣的 
數據可以一次提出一些進內存 排序完再丟回硬碟 每一段排序好的再merge成一個大的排序文件

Linux的sort就利用這種方式來應對大小超過內存的數據 並且還能同時使用多個CPU平行排序 這代表我們的Unix sort方法更容易可以擴展到大數據集 而且還不會耗盡內存 當然瓶頸可能是從硬碟讀取文件的速度

## Unix哲學
