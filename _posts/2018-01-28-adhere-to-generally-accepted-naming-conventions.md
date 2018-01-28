---
layout: post
title: Effective Java Item56 - 遵守普遍接受的命名慣例
comments: True 
subtitle: effective java - 遵守普遍接受的命名慣例
tags: effectiveJava
author: jyt0532
---

這篇是Effective Java - Adhere to generally accepted naming conventions章節的讀書筆記 本篇的程式碼來自於原書內容

### Item56: 遵守普遍接受的命名慣例

Java平台有許多命名的慣例 其中分為兩大類 字面的(typographical)和語法的(grammatical)

### 字面類

package的命名要是層次狀 用句號分隔每部分 每部份只包含英文字母和數字 任何將在你組織之外使用的package 要以你的組織開頭 比如edu.cmu或com.sun 

用戶創建的package不能以java或javax開頭 這些是留給標準類庫的

package名稱除了組織之外 剩下就是包的組成部分 且組成部分通常比較簡單 不超過8個字母 比如使用util而不是utililities 也可以只取第一個字母的縮寫 比如awt 句號跟句號之間的部分通常都是單詞或是縮寫組成

#### 類和接口

當一個類或接口名稱超過一個單詞 則每個單詞的第一個字母大寫 且盡量避免縮寫

比如HttpUrl, TimeTask

#### 方法和變數名稱

第一個字母應該小寫 其他單詞第一個字母大寫 比如ensureCapacity fetchHttpContent

#### 常數

都用大寫字母加上底線 比如NEGATIVE_INFINITY

#### 局部變量

跟變數名稱差不多 只是允許變量更不模糊一點 因為還可以參考上下文 比如i, xref, houseNumber

#### 類型參數

在泛型的時候會用到 單個字母 偶爾字母加數字

T: 任意類型, 不同類型也可以用T1, T2, T3

E:表示集合的元素

K,V: map的key跟value


### 語法類

語法的命名比較多爭議 以下列出比較被接受的命名

類別用名詞 比如說 Timer, BufferedWriter

接口命名用名詞或是 -able 比如說Comparable或Runnable, Iterable

執行某個動作的方法就用動詞開頭 比如appendImage或drawImage

如果方法回傳一個物件的非boolean屬性 則直接用屬性命名 或是get屬性 比如size或getSize

回傳boolean的 就可以用is或has開頭 比如isDigit, isEmpty

### 易讀程式之美學

關於命名 推薦大家可以看這本書[易讀程式之美學](https://www.tenlong.com.tw/products/9789862767191)









