---
layout: post
title: Effective Java Item49 - 檢查參數的有效性
comments: True 
subtitle: effective java - 檢查參數的有效性
tags: effectiveJava
author: jyt0532
---

這篇是Effective Java - Check parameter for validity章節的讀書筆記 本篇的程式碼來自於原書內容

### Item49: 檢查參數的有效性

當你在寫一個方法或是constructor 要在做任何事之前檢查參數的有效性 比如說 index不能是負數 reference不能是null等等 

給錯什麼樣的參數會丟出什麼樣的exception
都應該在javadoc裡用@throws寫好 
通常是IllegalArgumentException, IndexOutOfBoundException跟NullPointerException

**重要的法則是 應該在錯誤發生後盡快檢測出錯誤** 
沒做到這一點的話 就會比較難檢測出錯誤 
而且即使檢測出錯誤 找到錯誤的根源的可能性也降低

簡單的例子 非常淺顯易懂的Doc
{% highlight java %}
/*
 * @param m the modulus, which must be positive
 * @return this mod m
 * @throws ArithmeticException if m is less than or equal to 0
*/
public BigInteger mod(BigInteger m){
  if(m.signum() <= 0)
    throw new ArithmeticException();

  ...
}
{% endhighlight %}

對於private的方法 還是要確認 但就用assert就可以

{% highlight java %}
// private function for recursive sort
private static void sort(long a[], int offset, int length){
  assert a != null;
  assert offset >= 0 && offset <= a.length;
  assert length >= 0 && length <= a.length - offset;

  ..
}
{% endhighlight %}

Assert如果失敗會丟出AssetError

對於那些當下沒有用到的參數 參數檢查更是重要 不要等到要用的時候才檢查 那時候可能已經浪費了不少資源 這是在concstructor中很常犯的錯誤

通常我們希望在一個方法執行計算任務之前 
先檢查一下方法的參數 
但有個唯一的例外 就是檢查這件事本身花費很多資源 
而且在計算過程中有包含了檢查 

比如說關於Collection.sort 一個List裡面的東西應該要是可以被比較的 但你在這個sort之前先去比較每一個東西是不是comparable沒有太大意義 因爲如果不能比的話 其中的某一個比較就會拋出ClassCastException

### 總結

每次編寫方法或構造器 都要考慮參數有哪些限制 並且寫在文檔 在函數的一開始 
透過顯性的檢查來檢視這些限制 這個習慣很重要 

只要這個檢查有一次檢查到錯誤 且提早噴錯誤 你之前對於寫這個檢查花的所有時間都會被加倍償還



