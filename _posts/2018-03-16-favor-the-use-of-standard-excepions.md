---
layout: post
title: Effective Java Item60 - 優先使用標準的異常
comments: True 
subtitle: effective java - 優先使用標準的異常
tags: effectiveJava
author: jyt0532
---

這篇是Effective Java - Favor the use of standard exceptions章節的讀書筆記 本篇的程式碼來自於原書內容

### Item60: 優先使用標準的異常

好的程序員跟初階的程序員一個很大的差別在於 好的程序員追求代碼的重複利用性 這包含了異常的重複利用性 

Java提供了一系列的非受檢異常 他們通常可以滿足API的需要

重複用異常有兩大好處

1.和其他程序員已經習慣的用法一致 學習門檻較低

2.可讀性好 因為每個非受檢異常大概都是自己人 

3.異常類越少 裝載時間的開銷就越少

想知道更多裝載時間的開銷問題 歡迎詳讀 [每個Java程序員都應該了解的JVM](/toc/jvm/)

以下列出一些常見的非受檢異常

IllegalArgumentException	非null的參數值不正確

IllegalStateException	對於方法的調用來說 對象狀態不合適

NullPointerException	再不應該為null的時候 值為null

IndexOutOfBoundsException	下標越界

ConcurrentModificationException	禁止並發修改時 檢測到修改

UnsupportedOperationException	對象不支持用戶請求的方法

ClassCastException 非法的Cast

NumberFormatException 非法的把字串轉換到數字型態

以上的異常應該都要很熟 一看就要知道什麼意思 不應該再去google

