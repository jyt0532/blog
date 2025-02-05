---
layout: default
title: Effective Engineer
---
# The Effective Java 心得筆記

![Alt text]({{ site.url }}/public/effective_java.jpg)

## 第二章 物件的創造跟銷毀

[Item1 - 靜態工廠方法](/2017/09/20/static-factory-method/)

[Item2 - 建造者模式](/2017/09/21/builder/)

[Item3 - 用Enum實作Singleton](/2017/10/20/enforce-the-singleton-property-with-a-private-constructor-or-enum/)

[Item4 - 透過私有構造器來禁止實例化](/2018/04/15/enforce-noninstantiability-with-a-private-constructor/)

[Item5 - 依賴注入優於硬連接資源](/2020/03/17/prefer-dependency-injection-to-hardwiring-resources/)

[Item6 - 避免創建不必要的對象](/2018/07/01/avoid-creating-unnecessary-objects/)

[Item7 - 消除過期的對象引用](/2018/07/22/eliminate-obsolete-object-references/)

## 第三章 對於所有對象都通用的方法

[Item10 - 覆蓋equals請遵守通用規範](/2018/06/16/obey-the-general-contract-when-overriding-equals/)

[Item11 - 覆蓋equals時總要覆蓋hashCode](/2018/06/23/always-override-hashcode-when-you-override-equals/)

[Item12 - 始終要覆蓋toString](/2018/06/24/always-override-tostring/)

## 第四章 類和接口

[Item15 - 使類和成員的可訪問性最小化](/2018/04/21/minimize-the-accessibility-of-classes-and-members/)

[Item16 - 在公有類中使用訪問方法而非公有域](/2018/04/22/in-public-classes-use-accessor-methods-not-public-fields/)

[Item17 - 使可變性最小化](/2018/04/28/minimize-mutability/)

[Item18 - 復合優先於繼承](/2018/05/05/favor-composition-over-inheritance/)

[Item24 - 優先考慮靜態成員類](/2018/08/04/favor-static-member-class-over-nonstatic/)

[Item25 - 讓每個java文件只有一個top-level類別](/2018/11/30/limit-source-files-to-a-single-top-level-class/)

## 第五章 泛型

[泛型篇章簡介及術語列表](/2018/12/01/generics/)

[Item26 - 不要使用原始類型](/2018/12/02/dont-use-raw-types/)

[Item27 - 消除非檢查警告](/2018/12/02/eliminate-unchecked-warnings/)

[Item28 - 列表優於數組](/2018/12/08/prefer-lists-to-arrays/)

[Item29 - 優先考慮泛型](/2018/12/09/favor-generic-types/)

[Item30 - 優先考慮泛型方法](/2018/12/15/favor-generic-methods/)

[Item31 - 利用限制通配符來提昇API靈活性](/2018/12/23/use-bounded-wildcards-to-increase-api-flexibility/)

[到底 <T extends Comparable<? super T>>是什麼意思](/2018/12/23/t-extends-comparable-questionmark-super-t/)

[類型參數和通配符的選擇](/2018/12/23/how-to-choose-between-wildcard-and-generic-method/)


## 第六章 枚舉和註解

## 第七章 Lambda和Stream

[Item42 - lambda表達式優於匿名類](/2018/08/05/prefer-lambdas-to-anonymous-classes/)

[Item43 - 方法引用優於lambda表達式](/2018/08/05/prefer-method-reference-to-lambdas/)

[Item44 - 優先使用標準的函數式接口](/2018/10/07/favor-the-use-of-standard-functional-interfaces/)

[Item45 - 謹慎的使用Stream](/2018/11/10/use-streams-judiciously/)

[Item46 - 優先考慮在流的中間操作中使用無副作用的方法](/2018/11/18/prefer-side-effect-free-functions-in-streams/)

[Item47 - 優先使用Collection而不是Stream來作為函數的回傳類型](/2018/11/25/prefer-collection-to-stream-as-a-return-type/)

[Item48 - 謹慎使用並行Stream](/2018/11/25/use-caution-when-making-streams-parallel/)

## 第八章 方法
[Item49 - 檢查參數的有效性](/2018/02/23/check-parameters-for-validity/)

[Item50 - 必要時進行保護型拷貝](/2017/09/26/make-defensive-copies-when-needed/)

[Item51 - 謹慎的設計方法簽名](/2018/05/06/design-method-signatures-carefully/)

[Item52 - 慎用重載](/2018/06/02/use-overloading-judiciously/)

[Item53 - 慎用可變參數](/2018/05/20/use-varargs-judiciously/)

[Item54 - 返回零長度的數組或集合 而不是null](/2017/12/14/return-empty-arrays-not-nulls/)

[Item56 - 為所有導出的API元素編寫文檔註釋](/2018/05/12/write-doc-comments-for-all-exposed-API-elements/)

## 第九章 通用程序設計

[Item57 - 將局部變量的作用域最小化](/2018/04/05/minimize-scope-of-local-variable/)

[Item58 - For-each 優先於傳統的for或while](/2018/04/14/prefer-for-each-loops-to-traditional-for-loops/)

[Item64 - 通過接口引用對象](/2018/06/09/refer-to-objects-by-their-interfaces/)

[Item66 - 謹慎的使用本地方法](/2018/06/10/use-native-methods-judiciously/)

[Item67 - 謹慎地進行優化](/2018/06/09/optimize-judiciously/)

[Item68 - 遵守普遍接受的命名慣例](/2018/01/28/adhere-to-generally-accepted-naming-conventions/)

## 第十章 異常

[Item69 - 只針對異常的情況才使用異常](/2017/11/16/use-exceptions-only-for-exceptional-conditions/)

[Item70 - 對可恢復的情況使用受檢異常 對編程錯誤使用運行時異常](/2018/03/03/use-checked-exceptions-for-recoverable-conditions-and-runtime-exceptions-for-programming-errors/)

[Item71 - 避免不必要的使用受檢異常](/2018/03/11/avoid-unnecessary-use-of-checked-exceptions/)

[Item72 - 優先使用標準的異常](/2018/03/16/favor-the-use-of-standard-excepions/)

[Item73 - 拋出與抽象相對應的異常](/2018/02/25/throw-exceptions-appropriate-to-the-abstraction/)

[Item74 - 每個拋出來的異常都要有文檔](/2018/03/24/document-all-exceptions-thrown-by-each-method/)

[Item75 - 在細節消息中包含能捕獲失敗的訊息](/2018/03/31/include-failure-capture-information-in-detail-messages/)

[Item76 - 使失敗保持原子性](/2018/04/01/strive-for-failure-atomicity/)

[Item77 - 不要忽略異常](/2018/04/07/dont-ignore-exceptions/)

## 第十一章 併發

## 第十二章 序列化

[Effective Java - 序列化](/2017/09/27/java-serialization-101/)

[Item86 - 謹慎實現Serializable介面](/2017/09/29/implement-serializable-judiciously/)

[Item87 - 考慮使用自己定義的序列化](/2017/10/03/consider-using-a-custom-serialized-form/)

[深入解析序列化byte stream](/2017/10/12/decrypting-serialized-java-object/)

[Item88 - 保護性的編寫readObject方法](/2017/10/19/write-readobject-method-defensively/)

[Item89 - 用Enum實現物件控制](/2017/10/22/prefer-enum-for-instance-control/)

[Item90 - 考慮用序列化代理代替序列化實例](/2017/11/05/consider-serialization-proxies/)
