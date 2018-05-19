---
layout: post
title: Effective Java Item70 - 對可恢復的情況使用受檢異常 對編程錯誤使用運行時異常 
comments: True 
subtitle: effective java - 對可恢復的情況使用受檢異常 對編程錯誤使用運行時異常
tags: effectiveJava
author: jyt0532
---

這篇是Effective Java - Use checked exceptions for recoverable conditions and runtime exceptions for programming errors章節的讀書筆記 本篇的程式碼來自於原書內容

### Item70: 對可恢復的情況使用受檢異常 對編程錯誤使用運行時異常

標題有點長 但重點是分得出什麼是unchecked exception跟checked exception

JAVA有三種類型的可拋出結構 分別是 unchecked exception, checked exception和error 使用時機分別如下

*Error*: 通常指的是運行時的環境發生了錯誤 跟你的程式沒有太大的關係 比如說OutOfMemoryError和StackOverflowError 基本上在運行時沒有恢復的方法 你只能終止你的程式 所以寫try-catch也沒有用(因為這些error只有運行時才會發生 compiler在編譯的時候也沒有裡理由要求你寫try-catch)

*Unchecked exception(非受檢異常)*:  指的是compiler不會檢查的異常 因為這些異常也只有在執行時才會發生 compiler在編譯的時候也沒有理由要求你寫try-catch 所以unchecked exception又稱為runtime exception

通常情況是用在**錯在程序員**的情況

比如說 ArrayIndexOutOfBoundException, ClassCastException, NullPointerException 這些東西你都應該有辦法避免

記得Error也是非受檢的

如果你自己想要實作一個非受檢異常 那這個非受檢異常一定要繼承自RunTimeException

*Checked exception(受檢異常)*: 就是compiler可以預期的異常 既然compiler可以預期 那麼就會強制程序員要預先處理防範它 處理方式就是try catch 大概念就是如果你的這個方法可能會丟出異常 不論是你自己丟的還是你呼叫的人拋出的 你都必須在你的signature裡面講清楚 這樣別人呼叫你的時候就知道要預先處理

比如說FileNotFoundException, IOException

通常情況是用在**錯在使用者或使用者給的資料**的情況 身為一個好的程序員 當然要把使用者的錯誤輸入考量到寫的程式裡面

受檢異常代表你期望使用者去恢復處理這個異常 那麼給清楚這個異常的相關資訊就顯得更為重要 你也可以提供遇到這個異常的解決方法 

### 個人理解

以上的定義是大多數書籍對於受檢異常跟非受檢異常的區分 比如說你在實作一個方法的時候 你預期可能會發生某個錯誤 你到底應該丟出哪個異常給呼叫你的人 這就需要一個明確的準則

其實異常就是跑程序的時候出了問題 但我個人的理解是 受檢異常是你(程序員)無法暗中幫忙的 需要明確告訴程序如何幫忙 如何處理消化(try/catch) 真的幫不了就再往上拋 但非受檢異常是你(程序員)應該要在執行時有辦法考慮到的 所以大部分的IDE都可以在你忘記處理的時候給你warning

