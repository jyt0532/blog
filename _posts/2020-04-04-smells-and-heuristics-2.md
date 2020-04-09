---
layout: post
title: 程式碼的氣味和啟發(二)
comments: True
subtitle: 一般狀況
tags: cleanCode
author: jyt0532
excerp: 本文介紹程式碼的氣味和啟發
---

這篇文章討論 《Clean Code》 裡面收官的章節 - *程式碼的氣味和啟發* 以及《重構 - 改善既有程式的設計》裡第三章 - *程式碼的壞味道*

因為這篇是搭配兩書的類似概念 所以你看到

英文字母配個數字(C1, G2, T3)的就是《CleanCode》 的項目

3.X(3.1, 3.12)的 就是《重構 - 改善既有程式的設計》也有提及的項目

圖片以及程式碼來源自[Clean Code](https://www.amazon.com/Clean-Code-Handbook-Software-Craftsmanship/dp/0132350882)以及[重構 - 改善既有程式的設計](https://www.tenlong.com.tw/products/9789861547534)


## 一般狀況

G1 - 同份原始檔存在多種語言: 一個Source File就是一個語言 有超過一個語言就會讓人困惑

G2 - 明顯該有的行為未被實現: 比如一個將星期幾轉成enum函式

{% highlight java %}
Day day = DayDate.StringToDay(String dayName);
{% endhighlight %}

你可能會預期輸入"Monday"會回傳Day.MONDAY 但其他使用者可能會覺得"monday", "MON", "mon", "MONDAY"等等都要回傳正確的值 
如果你沒有全部實作的話 會失去大家對這個函式的信賴

G3 - 邊界上的不正確行為: 為你的演算法在所有情況都寫好測試 不要依賴直覺

G4 - 無視安全規範: 比如手動控管`[serialVersionUID](/2017/10/12/decrypting-serialized-java-object/)` 或是把Compiler的警告關掉 都是很危險的事

G5 - 重複的程式碼: 本書最重要的規範之一 要非常認真看待

如果你看到相同的程式碼 代表在過去發生了該進行抽象而未進行抽象的情形

常見的情況如下

1.如果兩個函式有一樣的程式碼 就提取這些一樣的程式碼到新的函式 讓人家呼叫

2.如果兩個subclass有一樣的程式碼 那可以提取這些程式碼到parent class

3.那如果兩個subclass只是有類似的程式碼 比如說演算法有一點點差異呢? 恩你懂的 [Template Method](/2017/09/12/template/)

G6/7 - 在錯誤抽象層次上的程式碼: 要建立能劃分 **高層次的一般概念** 和 **低層次細節概念**的抽象模型 我們想把高層次的概念都放在base class 低層次的概念都放在derived class


來看一個可愛的Stack抽象
{% highlight java %}
public interface Stack {
  Object pop();
  void push(Object o); 
  double percentFull();
}

{% endhighlight %}
有沒有哪個函式看起來怪怪? `percentFull` 因為不是所有的Stack都有 多滿 的概念 這種東西就應該放在比如`BoundedStack`之類的衍生類別裡

G8 - 過多的資訊: 限制類別和模組中曝露的介面數量 當一個類別擁有的方法數量越少越好 當一個函式知道的變數越少越好 當一個類別擁有的實體變數越少越好

專心的在讓你的介面tight and small. Help keep coupling low by limiting information.

G9 - 被遺棄的程式碼: 比如永遠不會跑到的if區 或是永遠不會拋出例外的catch區 這些區塊永遠不會跑到 但若一直放著 久了就會有味道

G10 - 垂直分隔: 變數和函式應該要定義在靠近使用的地方 比如說

1.區域變數應該要在第一次被使用的位置的正上方 垂直距離越短越好 

2.私有函式應該要被定義在第一次被呼叫的地方的下面 讓人可以馬上找到

G11/G24 - 不一致性: 不論是命名還是其他事項 都應該遵從一樣的慣例

G12 - 雜亂的程式: 用不到的東西 不需要的東西 都全部拿掉 比如default constructor就可以直接拿掉

G13 - 人為的耦合(artificial coupling): 比如說把enum或是某個static變數宣告在某個類別裡 這樣會迫使使用者必須了解整個類別 

這種人為的刻意把兩個東西的關聯性變強 都應該避免

G14(3.7) - 特色留戀(Feature Ency): 一個類別的方法 應該只對同一個類別的變數和函式感興趣 而不是對其他類別的感興趣

來看個例子
 
{% highlight java %}
public class HourlyPayCalculator {
  public Money calculateWeeklyPay(HourlyEmployee e) {
    int tenthRate = e.getTenthRate().getPennies();
    int tenthsWorked = e.getTenthsWorked();
    int straightTime = Math.min(400, tenthsWorked);
    int overTime = Math.max(0, tenthsWorked - straightTime); 
    int straightPay = straightTime * tenthRate;
    int overtimePay = (int)Math.round(overTime*tenthRate*1.5);
    return new Money(straightPay + overtimePay); }
}

{% endhighlight %}

嗯... 這明明就是在`HourlyPayCalculator`裡面的函式 但卻一直去跟`HourLyEmployee`要資料 這可能就代表`calculateWeeklyPay`這個函式本身就該存在`HourlyEmployee`裡

G15 - 選擇性參數: 如同F3(旗標參數)

G16 - 模糊的意圖: 我們需要程式可以盡可能地具有表達力 跨行的表達或是匈牙利命名都會讓人困惑

{% highlight java %}
public int m_otCalc() { 
  return iThsWkd * iThsRte +
    (int) Math.round(0.5 * iThsRte * 
      Math.max(0, iThsWkd - 400)
    ); 
}
{% endhighlight %}

這什麼鬼 原本2秒可以看懂的程式變成要花10秒


G17 - 錯誤的職責: 軟體工程師能夠做的最重要決定之一 就是應該把程式碼放在哪

比如說`PI`這個常數應該放在`Math`裡 還是`Trigonometry`裡 還是`Circle`裡?

有時候我們會用一點小聰明來把程式碼放在我們覺得方便使用的地方 但這並不是對讀者來說最直觀的地方

G18 - 不適當的靜態宣告: 我們喜歡把一些常用的函式當作靜態函式來用 比如


{% highlight java %}
Math.max(double a, double b);
{% endhighlight %}

這是很合理的 畢竟我們不想要每次需要比較的時候需要new一個物件
{% highlight java %}
new Math().max(a, b)
{% endhighlight %}

或是
{% highlight java %}
a.max(b)
{% endhighlight %}

這些都不太實用 但也不是所有東西都該宣告能靜態函數 比如說
{% highlight java %}
HourlyPayCalculator.calculatePay(employee, overtimeRate)
{% endhighlight %}

乍看之下合理 但是仔細想一下 我們應該**很有可能** **遲早**會需要support這個函式的polymorphism 比如第一個參數如果給進employee的子類 我們也要會算

這樣的話 每當我們需要support一個子類 我們就得把這個子類的`calculatePay`加進子類的靜態函式 可想而知 重複性太高

總而言之 你應該偏向使用非靜態函式 當你真的需要把它變成靜態的時候 先考慮你不需要在未來支持多型

G19 - 使用具解釋性的變數: 就是變數命名要清楚

G20 - 函數名稱要說到做到

![Alt text]({{ site.url }}/public/cleanCode-tips-1.png)

如果你必須仔細看這個函式的實作 才知道它到底做了什麼 那你應該取個好點的名稱

G21 - 暸解演算法: 你必須先瞭解你要實作的算法 在去寫實作的函式

G22 - 讓邏輯相依變成實體相依: 其實跟G17很像 就是當兩個模組有依賴關係的時候 哪個模組要負責那些資料要跟清楚

G23(3.10, 9.6)/G27 - 用多型取代if/else 或 switch: 當你在物件導向的語言看到switch 很有可能可以用多型來取代

{% highlight java %}
class Employee {
 int payAmount() {
   switch (getType()) {
     case EmployeeType.ENGINEER:
       return _monthlySalary;
     case EmployeeType.SALESMAN:
       return _monthlySalary + _commission;
     case EmployeeType.MANAGER:
       return _monthlySalary + _bonus;
     default:
       throw new RuntimeException("Incorrect Employee");
   }
 }
}
{% endhighlight %}
當你看到這個 就可以用可愛的[Strategy](/2017/04/07/strategy/)來替換
{% highlight java %}
class Salesman{
  int payAmount(Employee emp) {
    return emp.getMonthlySalary() + emp.getCommission();
}
class Manager{
  int payAmount(Employee emp) {
    return emp.getMonthlySalary() + emp.getBonus();
  }
{% endhighlight %}

別忘了把base class的函式改成抽象函式
{% highlight java %}
class EmployeeType{
 abstract int payAmount(Employee emp)
}
{% endhighlight %}

G25 - 用帶有名稱的常數取代Magic number: 比如用`SECOND_PER_DAY`而不是86400

G26 - 要精確: 模稜兩可和不精確的程式碼 是由懶惰以及不一致所造成的 比如

單純用浮點數來代表貨幣利率幾乎是犯罪行為 或是因為不想處理同步問題所以不用lock或是transaction management 或是明明宣告List就夠了你卻硬要宣告為ArrayList來過度限制 或是讓所有變數都沒有access modifier(直接變成package protected)

當你在程式碼中下決定時 都要有充足的理由

G28 - 把條件判斷封裝

比如這個判斷
{% highlight java %}
if (shouldBeDeleted(timer))
{% endhighlight %}
就比這個清楚很多
{% highlight java %}
if (timer.hasExpired() && !timer.isRecurrent())
{% endhighlight %}

G29 - 避免否定的條件判斷: 否定的判斷比肯定的難了解

你自己來看看這兩個哪個需要讀懂的時間少一點
{% highlight java %}
if (buffer.shouldCompact())
{% endhighlight %}
{% highlight java %}
if (!buffer.shouldNotCompact())
{% endhighlight %}

G30 - 函式應該只做一件事

我們來看看付錢給員工的函式
{% highlight java %}
public void pay() {
  for (Employee e : employees) {
    if (e.isPayday()) {
      Money pay = e.calculatePay(); 
      e.deliverPay(pay);
    } 
  }
}
{% endhighlight %}

這個函式做了三件事 檢查所有員工 檢查哪些員工應該拿錢 給錢 那我們就應該分成三個函式寫

{% highlight java %}
public void pay() {
  for (Employee e : employees)
    payIfNecessary(e); 
}
private void payIfNecessary(Employee e) { 
  if (e.isPayday())
    calculateAndDeliverPay(e); 
}
private void calculateAndDeliverPay(Employee e) { 
  Money pay = e.calculatePay(); 
  e.deliverPay(pay);
}
{% endhighlight %}

G31 - 不應該隱藏時序耦合: 不要為了讓函式簡單 而隱藏了函式之間的依賴關係


{% highlight java %}
public class MoogDiver{
  Gradient gradient;
  List<Spline> splines;
  public void dive(String reason){
    saturateGradient();
    reticulataSplines();
    diveForMoog(reason);
  }
}
{% endhighlight %}

當我們`dive`的時候 **一定**需要按照這三個順序 `saturateGradient` -> `reticulataSplines` -> `diveForMoog`

你程式這樣寫很合理 但這就可能讓任何人其他的函式 用任何他想用的順序呼叫 你無法保證呼叫的順序 

當你函式的時序需求的時候 你可以這麼搞
{% highlight java %}
public class MoogDiver{
  Gradient gradient;
  List<Spline> splines;
  public void dive(String reason){
    Gradient gradient = saturateGradient();
    List<Spline> splines = reticulataSplines(gradient);
    diveForMoog(splines,reason);
  }
}
{% endhighlight %}

這樣就會逼著一定要按照順序了

G32 - 不要隨意: 你的程式碼的每個地方的結構都要清楚且一致 當你只要某個地方不一致 看的人或改的人就不會覺得保持一致很重要 就會越改越亂

G33 - 封裝邊界條件

當你要處理邊界條件的時候 可能會有這種情況

{% highlight java %}
if(level + 1 < tags.length) {
  parts = new Parse(body, tags, level + 1, offset + endTag);
  body = null; 
}
{% endhighlight %}

這兩個`+1`就很惱人 把他用另一個變數包起來

{% highlight java %}
int nextLevel = level + 1; 
if(nextLevel < tags.length) {
  parts = new Parse(body, tags, nextLevel, offset + endTag);
  body = null; 
}
{% endhighlight %}

好看多了

G34 - 函式內容應該下降抽象層次一層: 在一個函式裡的statement 都應該屬於同一個層次的抽象 哪一層呢 就是這個函式本身的低一層

比如說一個兩級抽象深的函式 裡面的內容就只能是三級抽象深的表達式 當你要使用四級抽象深的表達式 你需要再呼叫一個函式

上個例子 如果`size > 0` 就把hr(水平線)多一個size attribute

{% highlight java %}
public String render() throws Exception {
  StringBuffer html = new StringBuffer("<hr"); 
  if(size > 0)
    html.append(" size=\"").append(size + 1).append("\""); 
  html.append(">");
  return html.toString(); 
}
{% endhighlight %}

這個函式混雜了至少兩種以上的抽象層次 一種是水平線的粗細概念 一種是水平線的syntax

重構之後這這樣

{% highlight java %}
public String render() throws Exception {
  HtmlTag hr = new HtmlTag("hr"); 
  if (size > 0)
    hr.addAttribute("size", hrSize(size)); 
  return hr.html();
}
private String hrSize(int height) {
  int hrSize = height + 1;
  return String.format("%d", hrSize); 
}
{% endhighlight %}

這個重構把水平線應該要多粗 跟 如何畫粗水平線的概念分開 如果所有`render()`裡面的表達式都是第n層深的抽象的話 那`hrSize()`裡面的表達式都是第n+1層抽象

劃分抽象程式概念 是進行函式重構時 最重要的行為之一

G35 - 可以configure的資料放在高階層次

如果你有一個常數 比如說有預設值或是設定值 可以configurable的 那就要擺在高階的抽象層次上 不要藏到低層次的函式裡

G36(3.15) - 避免過度耦合的訊息鏈:

如果模組A跟模組B合作 且模組B跟模組C合作 我們不希望模組A了解模組C的資訊 比如說


{% highlight java %}
a.getB().getC().doSomething()
{% endhighlight %}

我們稱為Message Chain 這意味著物件之間的緊密耦合

這樣的話 如果你未來想在B跟C之間插入一個Q 你會難以進行架構上的變動 你必須找到所有的`a.getB().getC()` 改成`a.getB().getQ().getC()`

那要怎麼做呢 在[重構 - 改善既有程式的設計](https://www.tenlong.com.tw/products/9789861547534)中 建議的作法為[Hide Delegate](https://refactoring.guru/hide-delegate)

{% highlight java %}
class Person {
  Department _department;
  public Department getDepartment() {
    return _department;
  }
  public void setDepartment(Department arg) {
    _department = arg;
  }
}
class Department {
  private String _chargeCode;
  private Person _manager;
  public Department (Person manager) {
    _manager = manager;
  }
  public Person getManager() {
    return _manager;
  }
}
{% endhighlight %}

這種情況下 你想知道一個人的老闆 只能`john.getDepartment().getManager()` 這樣增加了`Person`跟`Department`的耦合 畢竟`Person`根本就不Care `Department`

那怎麼辦 就在Person裡面加一個Delegate method

{% highlight java %}
class Person {
  Department _department;
  public void setDepartment(Department arg) {
    _department = arg;
  }
  public Person getManager() {
    return _department.getManager();
  }
}
{% endhighlight %}

注意這時候我就可以拿掉`getDepartment()`了
