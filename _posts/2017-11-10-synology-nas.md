---
layout: post
title: 不必再花錢買Iphone容量 用群暉科技NAS自動備份手機照片
comments: True 
subtitle: NAS自動備份手機照片
tags: nas synology
author: jyt0532
---

最近iphoneX上市 256GB跟64GB的價格差了美金150 越想越覺得把錢花在手機的容量實在有點蠢 可是手上的這台舊手機一直跟我說iCloud記憶體滿了 實在煩人 更不用說iphone要把照片存進電腦實在麻煩 所以最近開始研究NAS 才發現現在的NAS跟我第一次看到這個詞的時候 已經長得不太一樣了 這兩天敗了一台NAS [DS718+](https://www.synology.com/en-us/products/DS718+) 功能比想像中的完整許多

![Alt text]({{ site.url }}/public/718.JPG)

先簡單介紹一下NAS是什麼

### NAS
Network Attached Storage

顧名思義 就是個讓你掛在網路上的硬碟 你在任何可以上網的地方 都應該要可以存取你自己的NAS 
不需要再用網路上的其他付費service 像是dropbox等等 而且這顆硬碟就放在你自己家 

那如果是手動的備份 那就跟買一個一般外接硬碟沒有差別了 原本在NAS寄來之前 我都已經找好要sync的App了 
結果沒想到群暉都已經出了相對應的App [DS Photo](https://itunes.apple.com/us/app/ds-photo/id321493106?mt=8)幫我省了不少事

只要手機連上wifi 就可以把手機的照片同步到家裡的NAS去 還可以選擇用Https

當然現在已經是2017 資料講求多重備份 只要你的nas有兩個以上的插槽 還可以做RAID的備份

### RAID

Redundant Array of Independent Disks 

中文是容錯式磁碟陣列 中心思想是把好幾個硬碟組合起來 並妥善利用存儲空間的話 即使某一個硬碟爆了 還是可以靠其他硬碟救回來 增加容錯 

各種RAID的比較這邊就不詳述了 wiki都有 我買的718+ 只有兩個插槽 所以只能做到RAID 1 就是資料一比一的備份

![Alt text]({{ site.url }}/public/RAID_1.svg)
圖片來源:wiki

身為一個骨灰級工程師 只把照片放在家裡容錯率還是不夠高 如果突然地震等等天災把硬碟毀了那也完全沒救

### Glacier

這是後就要考慮AWS的Glacier服務 這是Amazon的一個成本非常低的雲端存儲 可以自己設定每個禮拜替自己的硬碟做snapshot 做長期的備份 不像s3要求高速的存儲 glacier上傳下載都非常花時間 所以非常便宜 每個月每GB 0.004USD 算是備案中的備案 但可以保證你的照片不會從這世上消失 

[Amazon Glacier](https://aws.amazon.com/tw/glacier/)

好消息是 群暉也幫你想好了 直接下載他們的app 再給他AWS glacier的Access Key 就可以自動安排備份頻率

### 總結

裝了NAS之後 從今之後買iphone只要買最低容量的就可以 現在每天回家連上wifi 就會把手機相簿裡還沒備份相片的都上傳進NAS硬碟 然後每個禮拜amazon glacier備份一次

買雲端硬碟之前我沒有預期到群暉科技對於NAS的整合這麼好 不得不說很多我原本以為買回來可以玩的feature都已經先被做掉了 
使用NAS的門檻實在低到沒什麼發揮空間
現在只要開browser都可以直接連到NAS的OS直接存取 實在方便 推薦給大家

