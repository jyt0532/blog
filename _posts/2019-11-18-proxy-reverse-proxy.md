---
layout: post
title: 系統設計 - 正向代理跟反向代理
comments: True 
subtitle: Forward Proxy and Reverse Proxy
tags: systemDesign proxy 
author: jyt0532
excerpt: 本文講解Forward Proxy跟Reverse Proxy的差別
---

# Proxy 代理

什麼是代理呢 在系統設計裡面 代理就是Client跟Server之間的一個中間層

有兩種Proxy 一種是Forward Proxy 一種是Reverse Proxy

## Forward Proxy

![Alt text]({{ site.url }}/public/forward-proxy.png)

就是長這樣 眾多Client跟一個Forward Proxy溝通 讓它幫我們把請求導到Server 

Proxy的概念就是 Server不知道發請求的Client是誰

你可以想成高中時期的數學小老師或是英文小老師的角色 那為什麼需要一個小老師呢?

### 好處

1.匿名: Server這樣就不知道消息是誰發的 他只知道是由Proxy負責問 就像老師就只回答小老師問題 他並不知道是誰問的 就不會有任何的偏見

2.過濾請求: 比如有些人不該訪問Server就直接過濾掉 比如說別班的同學來問數學小老師問題 他就可以直接不理

3.過濾response: 比如某些人不該看到某些response 通常是政治上或是倫理上的考量 比如說某些國家不該看到某些內容 或是某些人不該看到18禁的內容一樣

3.可以log request: 可以統計所有的請求 這樣老師以後想知道這個班都在問什麼問題或是問了多少問題就只要問小老師

4.可以改變request(改變header, 加解密等等): 比如同學問了不精準的問題 他可以幫忙修正成老師想看到的問題 然後再去問老師

5.Caching: 這樣比如有三個同學問一樣問題 如果小老師這邊知道答案 就根本不用去問老師 可以節省非常多問老師問題的時間

## Reverse Proxy

剛剛說到 Proxy的概念就是 Server不知道發請求的Client是誰 那當然Reverse Proxy的概念就是Client不知道響應的Server是誰

至於現實生活中的例子 就一樣是小老師只是老師現在有非常多個

![Alt text]({{ site.url }}/public/reverse-proxy.png)

好處如下

1.Load balancing: Reverse Proxy可以決定哪個Server最閒 讓他來處理請求 或是輪流也可以

2.Caching: 可以把答案不變的問題給記錄下來 不用問老師直接回答

3.你的server可以做任何你想做的事 不用讓Client知道 比如說今天你想換個port 從8080到8081 你的client根本不用改變 只需要跟你的reverse proxy說一聲 學生根本不用知道某個老師突然換辦公室

4.可以log request: 可以統計所有的請求

5.Canary deveploment: 可以ramp你的change給不同的client 就是說今天如果我的Server有個新的feature要推出 我通常不會一次把所有server都部署新的程式 我可以一開始先部署一台 然後Reverse Proxy依照UserId來決定哪些人要試新feature 就把那些請求導到這唯一的一台有新程序的機器上

當我覺得1%穩定之後 再上到5%等等 一路往上直到100% 這樣要是新feature真的有什麼問題或是使用者其實很不滿意 我們也可以降低影響


相信看完之後你也像我一樣 不會再搞不清楚Forward Proxy跟Reverse Proxy
