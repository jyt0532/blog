---
layout: post
title: Effective Java Item63 - 在細節消息中包含能捕獲失敗的訊息
comments: True 
subtitle: effective java - 在細節消息中包含能捕獲失敗的訊息
tags: effectiveJava
author: jyt0532
---

這篇是Effective Java - Include failure-capture information in detail messages章節的讀書筆記 本篇的程式碼來自於原書內容

### Item63: 在細節消息中包含能捕獲失敗的訊息

當一段程式因為一個沒被捕捉到的異常而失敗時 
系統會印出有關這個異常的stacktrace 包含這個異常的字符串表示法(string represenation) 
也就是這個異常類別呼叫toString()的輸出 然後是細節 這訊息通常是給程序員看 然後debug

所以你的toString方法要印出盡可能多的關於失敗的訊息 因為某些錯誤並不是那麼容易reproduce 錯過了就沒了

一個異常的細節訊息應該包含所有**可能對異常有貢獻**的參數

來個例子 IndexOutOfBoundsException 就應該印出上界 下界 和 超出上下界的index 因為這三個都可能是導致這個異常拋出的原因 比如說上下界太小或太大 下界大於上界 索引小於下界 或大於上界 有太多的可能情況

這個訊息是給程序員看的 所以不需要寫到用戶都可以理解 在一個異常訊息中 訊息內容比可理解性重要的多
