---
layout: default
title: Effective Engineer
---
# The Effective Java 筆記

## 第二章 物件的創造跟銷毀

[Item1 - 靜態工廠方法](/2017/09/20/static-factory-method/)

[Item2 - 建造者模式](/2017/09/21/builder/)

[Item3 - 用Enum實作Singleton](/2017/10/20/enforce-the-singleton-property-with-a-private-constructor-or-enum/)

[Item4 - 透過私有構造器來禁止實例化](/2018/04/15/enforce-noninstantiability-with-a-private-constructor/)

## 第三章 對於所有對象都通用的方法

[Item11 - 覆蓋equals時總要覆蓋hashCode](/2018/06/23/always-override-hashcode-when-you-override-equals/)

[Item12 - 始終要覆蓋toString](/2018/06/24/always-override-tostring/)

## 第四章 類和接口

[Item15 - 使類和成員的可訪問性最小化](/2018/04/21/minimize-the-accessibility-of-classes-and-members/)

[Item16 - 在公有類中使用訪問方法而非公有域](/2018/04/22/in-public-classes-use-accessor-methods-not-public-fields/)

[Item17 - 使可變性最小化](/2018/04/28/minimize-mutability/)

[Item18 - 復合優先於繼承](/2018/05/05/favor-composition-over-inheritance/)
## 第五章 泛型

## 第六章 枚舉和註解

## 第七章 Lambda和Stream

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
