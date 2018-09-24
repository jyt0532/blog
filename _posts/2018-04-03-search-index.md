---
layout: post
title: 搜尋引擎
comments: True 
subtitle: search engine
tags: search
author: jyt0532
---

這篇來講一下關於搜尋引擎最基本的知識 看完這系列文章後 你就可以針對你手上有的許多文本
直接寫一個簡單的搜尋引擎 

要建置一個搜尋引擎包括三個階段 Index, Query, Ranking

### 文本前置作業

以下的前置作業都是選擇性的 不過如果是在系統設計的面試中 你能夠提出越多pre-processing的考量就越加分 

1.把所有文件大小寫先統一 先固定成小寫(APPLE = apple)

2.把所有文件的文法統一 (research = researching = researches)

3.把stop words拿掉(the, a, an等等)

以上的這些前置作業又稱為正規化(normalization)

## 倒排索引(Inverted Index)

本文介紹的是最常見的倒排索引 又稱為反向索引

直上例子 假如我有兩個文件

Doc1: "web search course information retrieval course"

Doc2: "information retrieval web information"

我們就把每個字出現在哪個文件的哪個位置 記成一個Array

information: [[1,[3]], [2,[0,3]] 

retrieval: [[1,[4]], [2,[1]]

course: [[1,[2,5]]

web: [[1,[0]], [2,[2]]

search: [[1,[1]]

第一個值是文件id 第二個值是這個詞在文件中的位置 比如說[[1,[3]] 代表在第一個文件的第三個位置 依此類推

怎麼使用這些Array待會就會說明

### Map/Reduce

看完倒排索引的介紹後 你會發現這個索引方式非常的適合用Map/Reduce

//稍微介紹map-reduce
//成就google

## Query

## Ranking
