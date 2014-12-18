---
layout: post
title:  "Report System Part III"
date:   2010-10-08 23:00:00
categories: programming technology-issues
---

This is the last part of Report System blog series. I’m going to talk about a testability point that happened to me when I was writing our Report System.

As you know, before we decided to write our own report system, we've worked with stupid Crystal Report. We had a Configuration module which handled configuration issues for reports back then. When we wanted to change the whole system and write our own report system I decided to reuse that configuration module for the new system and change it accordingly.

I wanted to write my first test, unfortunately we didn’t write that module with TDD or TAD(Test After Development-this phrase is not very official-) so it had no tests(Icky) and also wasn’t testable.

There was a method in this module which got the Entity related to the report:

{% highlight csharp %}
public IEntity GetEntity()
{
  object id = HttpContext.Current.Request["Id"];
  var entityRepository = new EntityRepository();
  return entityRepository.GetEntity(id);
}
{% endhighlight %}

And we had another method which got the values for replacing in the interchangeable parts of the report according to that Entity:

{% highlight csharp %}
public object GetValue(string propertyName)
{
  object entity = GetEntity();
  return entity.GetPropertyValue(propertyName);
}
{% endhighlight %}

I couldn’t write a test for GetValue method in isolation because it called GetEntity and in the GetEntity method we used EntityRepository module(which is a concrete class) and I couldn’t use it in my tests because it depends on other things in application like DB and etc(application details). So I decided to change the design of this Configuration module for more testability(as you know, there is a deep synergy between good design and testability).

I wanted to conform to the Jeremy D.Miller’s second law of TDD and Dependency Inversion Principle for solving this problem so I wrote the following interface:

{% highlight csharp %}
public interface ICurrentEntity
{
  IEntity GetCurrentEntity();
}
{% endhighlight %}

And I used that in the Configuration module by injecting it in the constructor:

{% highlight csharp %}
public class ReportConfig
{
  private ICurrentEntity _currentEntity;
  public ReportConfig(ICurrentEntity currentEntity)
  {
      _currentEntity = currentEntity;
  }
}
{% endhighlight %}

After that, I eliminated the GetEntity method in the Configuration module and refactor the GetValue method like this:

{% highlight csharp %}
public object GetValue(string propertyName)
{
  var entity = _currentEntity.GetCurrentEntity();
  return entity.GetPropertyValue(propertyName);
}
{% endhighlight %}

Ok now I’m conforming Jeremy’s second law of TDD and DIP so the configuration module is more testable, then I went for writing my test, first of all I wrote an Entity for my testing environment:

{% highlight csharpt %}
public class DummyEntity : IEntity
{
  public object Id { get; set; }
  public string Name { get; set; }
}
{% endhighlight %}

After that, I Mocked the ICurrentEntity for pushing it to the Configuration module and test GetValue method:

{% highlight csharp %}
public class StubCurrentEntity : ICurrentEntity
{
  public IEntity GetCurrentEntity()
  {
      return new DummyEntity { Id = 1, Name = &quot;James&quot; };
  }
}
{% endhighlight %}

Finally this is the test case for GetValue method in configuration module:

{% highlight csharp %}
[Test]
public void ShouldGetValueFromRelatedEntityToReport()
{
  var stubCurrentEntity = new StubCurrentEntity();
  var reportConfig = new ReportConfig(stubCurrentEntity);
  Assert.AreEqual(1, reportConfig.GetValue(&quot;Id&quot;));
  Assert.AreEqual(&quot;James&quot;, reportConfig.GetValue(&quot;Name&quot;));
}
{% endhighlight %}

At the first place I couldn’t  write a test for GetValue method in isolation because the configuration module depended on a concrete class(EntityRepository) and I couldn’t use it in my tests, so, according to DIP, I inverted the dependency to an abstraction(interface in this case). Then I mocked that easily in my tests for testing purpose and I was able to test the behavior of that method(GetValue) in isolation without anything else.

One more time we saw the benefits of tests, if I didn’t want to write a test MAYBE I understand the problem of this design and ignore it, but when I wanted to write a test for this I felt the pain as a user of this code so I changed the design and make it more testable and easier to use.

## Bottom Line

When you want to use a technology for some of your requirements in your application consider all the dimensions and aspects of your requirements and also that technology, then decide whether use it or not. If that technology doesn’t match with your system and requirements perfectly, forget about it and write yours. It’s really worth the time and effort and also you can handle it manually with your own system and requirements and more importantly you don’t have to trace the new releases and updates of that technology and match yourself with them(because frankly that takes a lot of time and effort, believe me).
