---
layout: post
title: 程式碼的氣味和啟發(一)
comments: True
subtitle: 註解/開發環境/函式/命名/測試
tags: cleanCode
author: jyt0532
excerp: 本文介紹程式碼的氣味和啟發
---

這篇文章討論 《Clean Code》 裡面收官的章節 - *程式碼的氣味和啟發* 以及《重構 - 改善既有程式的設計》裡第三章 - *程式碼的壞味道*

因為這篇是搭配兩書的類似概念 所以你看到

英文字母配個數字(C1, G2, T3)的就是《CleanCode》 的項目

3.X(3.1, 3.12)的 就是《重構 - 改善既有程式的設計》也有提及的項目

圖片以及程式碼來源自[Clean Code](https://www.amazon.com/Clean-Code-Handbook-Software-Craftsmanship/dp/0132350882)以及[重構 - 改善既有程式的設計](https://www.tenlong.com.tw/products/9789861547534)


## 註解

C1 - 不適當的資訊: 有些註解的資訊應該放在別的地方而不是程式碼裡 比如說作者 最後修改日期等等

C2 - 廢棄的註解: 當註解變成過時或是不正確 就是個廢棄的註解 當你發現廢棄註解就趕快移除它

C3 - 多餘的註解: 如果原本的程式碼已經夠清楚了 就不要再寫多餘的註解 比如說

{% highlight java %}
i++;// increment i
{% endhighlight %}

或是 

{% highlight java %}
/**
* @param sellRequest
* @return
* @throws ManagedComponentException
*/
public SellResponse beginSellItem(SellRequest sellRequest) throws ManagedComponentException
{% endhighlight %}

註解就是要寫程式本身無法表達的資訊

C4 - 寫得不好的註解: 不要說廢話 不要說些顯而易見的事 去確保這是你所能寫出的最好註解

C5 - 被註解掉的程式碼: 當你看到你的程式碼裡面有段被註解掉的Code 你會真的不知道該怎麼處理 能刪嗎? 要留嗎? 因為沒人敢動 所以日復一日就讓你的程式越來越難維護

所以 下次當你看到被註解掉的程式碼 把它刪掉就對了 你真的需要的話 version control會幫你找回來的

C6(3.22) - 過多的註解: 當你看到註解時 你可以先想想是不是可以Refactor 常常在你重構完之後 你會發現註解完全不必要 因為程式碼簡潔到不需要註解

## 開發環境

E1 - 需要多個步驟來建立專案/系統: 建立一個專案應該要是一個步驟的自動化行為 你不應該東找西找jar檔或是XML檔拼拼湊湊

E2 - 需要多個步驟來進行測試: 你應該只需要一個步驟就可以跑完所有的測試 可以參考[無瑕的程式碼 - 單元測試](/2020/03/15/unit-test/)

## 函式

F1(3.4) - 過多的參數: 函式的參數不能太多 最好是不要有參數 要是有超過三個參數 那必要性是非常值得懷疑的

請參考[重構 - 改善既有程式的設計 - Long Parameter List](/2020/04/10/long-parameter-list/)

F2 - 輸出型參數: 讀者通常預期參數是用來輸入的 而不是用來輸出結果的

比如說如果你看到
{% highlight java %}
appendFooter(s);
{% endhighlight %}

ㄜ... 這是什麼意思 是把東西接在s之後 還是s會被接在某個東西之後呢 你必須被逼著看函式的宣告
{% highlight java %}
public void appendFooter(StringBuffer report)
{% endhighlight %}

你就知道 ok 有東西要接在s後面 這種就是屬於輸出型參數 因為一般人都會直觀地認為 函式的參數就是一些input 直覺的不會再把input參數拿來繼續用

如果你想要改變物件的狀態 那就改成這樣

{% highlight java %}
report.appendFooter();
{% endhighlight %}

F3 - 旗標參數(Flag Argument)

{% highlight java %}
doSomething(boolean flag)
{% endhighlight %}

這種宣告就是很不好的習慣 你應該分成兩個函式 然後取個更好的函式名稱

F4 - 被遺棄的函式: 那些不再被呼叫的函式就大膽的把它刪除 version control會幫你記得他們的

## 命名

N1 - 選擇具描述性質的名稱: 任何命名都不應該急著確定 要確保這個選定的名稱有足夠的描述性

下面的程式碼 簡直就是天書
{% highlight java %}
public int x() { 
  int q = 0; int z = 0;
  for (int kk = 0; kk < 10; kk++) { 
    if (l[z] == 10) {
      q += 10 + (l[z + 1] + l[z + 2]);
      z += 1; 
    }
    else if (l[z] + l[z + 1] == 10) {
      q += 10 + l[z + 2];
      z += 2; 
    } else {
      q += l[z] + l[z + 1];
      z += 2; }
    }
  }
  return q; 
}
{% endhighlight %}

但要是稍微命名的好一點 就很好理解
{% highlight java %}
public int score() {
  int score = 0;
  int frame = 0;
  for (int frameNumber = 0; frameNumber < 10; frameNumber++) {
    if (isStrike(frame)) {
      score += 10 + nextTwoBallsForStrike(frame); frame += 1;
    } else if (isSpare(frame)) {
      score += 10 + nextBallForSpare(frame); frame += 2;
    } else {
      score += twoBallsInFrame(frame); frame += 2;
    } 
  }
  return score; 
}
{% endhighlight %}

N2 - 在適當的抽象層次選擇適當的命名: 不要選擇透露**實現訊息**的名稱

書中的例子 數據機
{% highlight java %}
public interface Modem {
  boolean dial(String phoneNumber); 
  boolean disconnect();
  boolean send(char c);
  char recv();
  String getConnectedPhoneNumber();
}
{% endhighlight %}

乍看之下合理 但仔細看 並不是所有數據機都是用`dial` 因為可能有些是用USB 有些是用cable 這個命名把實現的細節給透露出來了

改成connect比較符合這一層抽象的名稱
{% highlight java %}
public interface Modem {
  boolean connect(String connectionLocator); 
  boolean disconnect();
  boolean send(char c);
  char recv();
  String getConnectedLocator();
}
{% endhighlight %}

N3 - 盡可能使用標準命名法: 比如你寫的是一個[裝飾器](/2017/04/18/decorator/) 就應該取Decorator 比如`AutoHangupModemDecorator` 如果要把物件變成字串 就用`toString` 這些都是大家一看就懂的 可以省很多時間去閱讀你的程式

N4 - 不要取模稜兩可的名稱

{% highlight java %}
private String doRename() throws Exception {
  if(refactorReferences) 
    renameReferences();
  renamePage();
  pathToRename.removeNameFromEnd(); 
  pathToRename.addNameToEnd(newName); 
  return PathParser.render(pathToRename);
}
{% endhighlight %}

`doRename`是什麼意思 這個裡面的`renamePage`又是什麼意思 沒有人看得懂 但如果你取名為`renamePageAndOptionallyAllReferences` 雖然比較長 但一看就懂 

N5 - 較大範圍的程式使用較長的名稱

名稱的長度和視野的範圍有關 比如說一段程式只有五行 那你用i或j就沒什麼問題 大家都知道這個馬上就不會再用了

比如以下的迴圈

{% highlight java %}
private void rollMany(int n, int pins) {
  for (int i=0; i<n; i++) 
    g.roll(pins);
}
{% endhighlight %}

這程式就夠清楚了 不用取太有意義的名稱

N6 - 避免編碼命名

型態的資訊 或是scope的資訊不要拿來命名 比如`m_`或是`f_`等等 或是針對專案的命名比如`vis_`(針對視覺化影像系統) 在當今的開發環境這些都多餘了 不要被[匈牙利命名法](https://zh.wikipedia.org/wiki/%E5%8C%88%E7%89%99%E5%88%A9%E5%91%BD%E5%90%8D%E6%B3%95)污染

N7 - 命名應該描述可能的程式副作用

這段程式來自TestNG
{% highlight java %}
public ObjectOutputStream getOos() throws IOException { 
  if (m_oos == null) {
    m_oos = new ObjectOutputStream(m_socket.getOutputStream()); 
  }
  return m_oos; 
}
{% endhighlight %}

這個函式做的事比獲取一個m_oos還要多 因為要是m_oos是null 那就會生一個 

這應該取名為`createOrReturnOos` 因為`getOos`看起來就是個沒有副作用的函式

## 測試

T1 - 不足夠的測試: 只要有任何的條件沒有被測試到 那就算測試不夠完整

T2 - 使用可以顯示測試覆蓋率的工具: 更快更輕易的找到沒被測試到的 if 或是 catch

T3 - 不要跳過簡單的測試: 這些簡單測試文件的說明價值高於撰寫的成本

T4 - 被忽略的測試可以用來表達對需求的疑問: 有時因為需求不明確 使得我們無法確定某個行為細節 我們可以利用被註解掉的測試或是@Ignore 來表達我們的疑問

T5 - 測試邊界條件: 邊界條件要特別測試 我們很常遇到一般情況正常但是邊界情況異常的程式碼

T6 - 在程式的錯誤附近進行詳盡的測試: Bug都是喜歡擠在一起的 當你在某個函式發現了Bug 你就應該對這個函式寫詳盡的測試 你很可能會發現Bug不只你發現的那個

T7/T8 - **失敗的模式**本身要能夠給你資訊: 一個厲害的測試 你只要看到測試失敗的Pattern 你就知道哪裡寫錯了

比如說對於所有字串長度超過五的測試都錯 或是這個函式第二個參數是負數的測試都錯 你的測試寫得好的話 你可以藉由**失敗的Pattern**找出bug而不是仔細去看每一個單獨的錯誤

T9 - 測試一定要跑得快

{% highlight java %}
{% endhighlight %}
{% highlight java %}
{% endhighlight %}
{% highlight java %}
{% endhighlight %}
{% highlight java %}
{% endhighlight %}
{% highlight java %}
{% endhighlight %}
{% highlight java %}
{% endhighlight %}
{% highlight java %}
{% endhighlight %}
