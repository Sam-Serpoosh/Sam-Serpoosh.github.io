---
layout: post
title:  "Report System Part II"
date:   2010-09-26 18:00:00
categories: programming technology-issues
---

In the previous post I told you we decided to write our own report system, so we’ll have an html page for each letter or report fixed contents and some kind of a special pattern for their interchangeable parts.

*From now I will only say report but remember it could be a letter too.

Our program read through those html pages and find their interchangeable parts according to the pattern and replace them with the actual values(which were determined by users).

So after some discussions between me and my boss we found out we should have a Parser module which extract out interchangeable parts. After that we pass those parsed parameters to a Getter module which gets their actual values and then we give those values back to the parser again and it replaces them in the report,  So we’ll end up with a complete report with actual values in it, PERFECT.

Just so you know our patterns were these:

{% highlight csharp %}
public const string ParameterPattern = @"~(?<param>[\w\d.]+)~";
public const string TemplatePattern = @"\|(?<key>[\w\d\.]+)\[\](?<value>[^\|]*)\|";
{% endhighlight %}

**Because  “~” and “|” don’t have any meaning in html we used them.

We used Regex for our patterns. First one is for simple things like ~LetterNo~ or ~Title~ and etc which will be set by users directly. Second one is for repeatable things like collections or one-to-many and many-to-many relationships between our Entities in the system.

For example imagine we have a File entity which has a one-to-many relationship with Person entity(each File contains a collection of Persons) so we should be able to get values for all of those persons and put them in the report, ergo the string in the page is something like this which matches with the pattern:

|Person[]< ~Name~><~Family~>|

now for each person in the collection we get Name and Family values and put them in the report (in this case we have two persons for simplicity of example):

<Robert><Madson><Dave><Hanson>

So I started to write my first test for this Parser module(you should always have tests and yet better than that you should do TDD as much as possible in your applications

Here’s my first test (BTW, I wrote this module with TDD approach):

{% highlight csharp %}
[Test]
public void ShouldGetAllSimpleParameters()
{
  var parameters = FieldParser.GetAllParameters("This is a ~test~ input! ~bye~");
  Assert.AreEqual(2, parameters.Count);
  Assert.IsTrue(parameters.Contains("bye"));
  Assert.IsTrue(parameters.Contains("test"));
}
{% endhighlight %}

So I wrote this function for getting all the parameters:

{% highlight csharp %}
public static IList<string> GetAllParameters(string text);
{% endhighlight %}

The next two tests that I wrote were these:

{% highlight csharp %}
[Test]
public void ShouldGetTemplateKey()
{
  var templateKey = FieldParser.GetTemplateKey("|Persons[] Name|");
  Assert.AreEqual("Persons", templateKey");
}

[Test]
public void ShouldGetTemplateValue()
{
  var templateValue = FieldParser.GetTemplateValue("|Persons[] Name|");
  Assert.AreEqual("Name", templateValue");
}
{% endhighlight %}

So for making these tests pass I wrote these two methods and also used them later in other parts:

{% highlight csharp %}
public static string GetTemplateKey(string template)
{
  var match = Regex.Match(template, TemplatePattern);
  if (match.success)
    return match.Groups["key"].Value;
  return null;
}

public static string GetTemplateValue(string template)
{
  var match = Regex.Match(template, TemplatePattern);
  if (match.success)
    return match.Groups["value"].Value;
  return null;
}
{% endhighlight %}

At this point, I could get all the parameters from the input content(which in our case is html string) and also could extract Key and Value from any string which matches with my “TEMPLATE_PATTERN”;

Now I should be able to replace the actual values with parameters  which I detected with “GetAllParameters” method, so that led me to this test:

{% highlight csharp %}
[Test]
public void ShouldBeAbleToSetParameters()
{
  var text = "Please replace ~this~ and ~that~";
  var replacingValueDic = new Dictionary<string, string>();
  replacingValueDic.Add("this", "me");
  replacingValueDic.Add("that", "you");
  var replacedText = FieldParser.SetParameters(text, replacingValueDic);

  Assert.AreEqual("Please replace me and you", replacedText);
}
{% endhighlight %}

Then I wrote this method for replacing values:

{% highlight csharp %}
public static string SetParameters(string text, IDictionary&lt;string, string&gt; replacingValues)
{
  if (replacingValues == null)
    return text;

  var replacedText = text;
  foreach(var replacingValue in replacingValues)
    replacedText = replacedText.Replace("~" + replacingValue.key + "~", replacingValue.value);

  return replacedText;
}
{% endhighlight %}

At this step I wrote another test for Parser which checked handling a string that matches with “TEMPLATE_PATTERN” and replace values in it as many times as needed:

{% highlight csharp %}
[Test]
public void ShouldReplaceCorrectlyInRepeatTemplates()
{
  var text = "|Persons[]<tr><td>~Name~</td><td>~Family~</td></tr>|";
  var replacedText = FieldParser.RepeatTemplates(text, new Func<object, string, object>(StubGetter));
  //SubGetter is for testing purposes which returns a collection with two persons in it
  Assert.AreEqual("<tr><td>mark</td><td>johnson</td></tr><tr>" +
      "<td>john</td><td>markson</td></tr>", replacedText);
}
{% endhighlight %}

This test led me to this function:

{% highlight csharp %}
//this method replace actual values for all the parameters in a template
//and do that as many times as needed according to the related collection
//count like Person collection or whatever;

public static string RepeateTemplates(string text, Func<object, string, object> valueGetter);
{% endhighlight %}

Ok, everything was fine and I almost had whatever I needed at the first place for my Parser Module.

Now was the time for writing the main Parse method which takes a string and a getter then return a result string which is filled with actual values. For this purpose I handled all the templates at first and after that I handled all the other parameters in the input text:

{% highlight csharp %}
public static string Parse(string text, Func<object, string, object> valueGetter)
{
  text = HandleAllTemplates(text, getter);
  text = HandleOtherParameters(text, getter);
  return text;
}
{% endhighlight %}

As you saw, after I found out what I need, I tried to write it step by step. I never made a big decision and never wrote something big, instead I tried to do everything little bit at a time, any decision I made was for at most 5 minutes later. I wrote my test first then I wrote a production code which made that failing test passed. At last coverage of this module was 98%(which is a great coverage by the way).

Of course the code you saw is not the first version I wrote. After I made each test passed, I stopped writing tests and code and did some Refactoring on the code I just wrote, So I ended up with a CLEAN CODE with small methods(each of them does one thing and does it well) and that’s one of the enormous benefits of TDD.

*There's even more about this report system, I'm gonna talk about it soon ...*
