---
layout: post
title: 測試替身(2) - Dummy
comments: True 
subtitle: test doubles - dummy
tags: unitTest
author: jyt0532
---
{% include foodgod_youtube_tracking.html %}

首先登場的 就是最簡單的替身 也就是Dummy

當我們需要傳一個變數給某個method
所以需要一個跟signature一樣type的變數(因為java是strong type的language) **可是這個變數以後又不會用到**
為了加速跟省記憶體空間 我們可以丟一個Dummy替身給這個method

比如說 我們想要測試People的getNumberOfPerson這個函式
{% highlight java %}
public class People {
  private List<Person> persons;
  public void addPerson(Person p){
    persons.add(p)
  }
  public int getNumberOfPerson(){
    persons.size();
  }
}
{% endhighlight %}
那你的測試可能原本長成這樣
{% highlight java %}
public class PeopleTest{
  @Test
  public void testGetNumberOfPerson(){
    People people = new People();
    Person person1 = new Person("John Doe", 25, address1, phoneNumber1);
    Person person2 = new Person("Jane Doe", 23, address2, phoneNumber2);
    people.addPerson(person1);
    people.addPerson(person2);
    assertEquals(2, people.addPerson.getNumberOfPerson)
  }
}
{% endhighlight %}
建一個Person可能開銷過大 而且很麻煩
況且Person並不是這個test的重點

這時候就要來個簡單的Dummy
{% highlight java %}
public class DummyCustomer extends Customer {
   public DummyCustomer() {}
}
{% endhighlight %}
這個class的目的就是要通過addPerson的型別限制
所以只要extends Customer就可以

{% highlight java %}
public class PeopleTestWithDummy{
  @Test
  public void testGetNumberOfPerson(){
    People people = new People();
    people.addPerson(new DummyCustomer());
    people.addPerson(new DummyCustomer());
    assertEquals(2, people.addPerson.getNumberOfPerson)
  }
}
{% endhighlight %}
就是這麼簡單

## Customer的其他method呢
那Customer的其他method 我們都全部不管 那如果getNumberOfPerson 呼叫了Customer的method 我們無法知道 但這也不是這個unit test在乎的重點 

但如果你真的想確保其他method不會被call 那就在DummyCustomer裡面覆寫Customer的其他method 然後都throw Exception就可以

## Null

其實很多人在測試的時候 直接傳null進去 如果你要傳進去的function沒有nullCheck 這也是個可行的方式 但如果有nullCheck那還是只能用Dummy

## Mockito
如果你寫的是java 你會很常看到Mickito
在Mockito裡面 如何生一個Dummy object呢?

{% highlight java %}
public class PeopleTestWithDummy{
  @Test 
  public void testGetNumberOfPerson(){
    People people = new People();
    people.addPerson(mock(Customer.class));
    people.addPerson(mock(Customer.class));
    assertEquals(2, people.addPerson.getNumberOfPerson)
  }
}
{% endhighlight %}

小心不要被這裡的mock搞混 他只是syntax是mock 但如果你只mock一個class但沒有給他任何預期的行為 他就是個dummy

**我們是以用法來區分TestDoubles 不是syntax** 因為很多framework不會為每一個TestDoubles都給一個constructor 

都會把好幾個都參在一起用


<div id="youTubePlayer2"></div>

