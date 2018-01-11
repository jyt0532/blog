---
layout: post
title: 測試替身(6) - Fake
comments: True 
subtitle: test doubles - fake
tags: unitTest
author: jyt0532
---
Fake就是一個真實DOC的簡單版 

一個常見的例子是DOC是真正的database 但是用一個簡單的in memory map來取代真正的database


## 實用性

不太實用 基本上很少人為了測試DOC 
去寫一個簡單版的DOC去取代真正的DOC 

因為你可能要另外為了這個簡單版的DOC在另外寫一個測試 得不償失

基本上之前提到的已經很夠用了

## 唯一好處

我能想到的唯一好處 就是用mock或是spy非常有可能會讓你寫出fragile test

什麼是fragile test呢 就是你的測試非常的脆弱 不堪一擊 比如說你一輕微改變你的實作 你的測試就要跟著改

這是不好的現象

對我來說Refactor code 就是改變一個東西的實作 但不改變一個東西的行為

寫測試有一個很大的好處 就是你在refactoring你的程式碼的時候(讓你的程式更易懂好讀) 你確信自己是對的 不會影響到外顯的結果

如果隨著refactor的進行 我們的unit test要跟著變 那就代表我們當初寫測試的時候 測試跟你的程式綁在一起

而用spy或是mock 很容易會讓你的測試程式 跟你的主要程式的implementation綁在一起

當你的測試跟著你主要程式碼的改變而跟著改變 那個不叫refactor 那個叫修改


所以如果你不想要你的測試跟主要程式綁在一起 你可以考慮用簡單的fake來取代DOC
