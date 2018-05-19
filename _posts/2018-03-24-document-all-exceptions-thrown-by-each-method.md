---
layout: post
title: Effective Java Item74 - 每個拋出來的異常都要有文檔
comments: True 
subtitle: effective java - 每個拋出來的異常都要有文檔
tags: effectiveJava
author: jyt0532
---

這篇是Effective Java - Document all exceptions thrown by each method章節的讀書筆記 本篇的程式碼來自於原書內容

### Item74: 每個拋出來的異常都要有文檔

如同標題所說 對於每個你的方法拋出的受檢異常 都要用Javadoc的@throws標註 
且寫下會拋出這些異常的條件

並且不要寫說會拋出你的異常的superClass 比如說 throws Exception 更慘的是throws Throwable 這種程式沒有提供任何資訊 並且還掩蓋了可能拋出的其他異常

至於未受檢異常 並沒有那麼強烈的要求要有文檔 
但如果有文檔也通常是一件好事 當你連所有未受檢異常都寫完的時候
你就會清楚的看到一個方法能被成功執行的所有先決條件(當然這裡就是只加在JavaDoc的文檔上 而不是方法的signature裡)

如果一個類中很多方法都可能丟出某個異常 你可以把這個異常寫在這個類的文檔裡


### 總結
為你每個可能拋出的異常建立文檔 不論受檢非受檢 不論抽象方法具體方法 只為受檢異常提供throws子句 

如果沒做到的話 別人很難使用你的方法或接口
