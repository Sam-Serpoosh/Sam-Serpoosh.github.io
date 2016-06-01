---
layout: post
title:  "As Tests become more Specific, Code becomes more General"
date:   2012-02-17 21:00:00
categories: programming unit-test refactoring
---

[Just for the heads up, in this post when I say reversing decimal number I mean converting from 1.2 to 2.1]

Few days ago I was hanging out with my friend and his little sister, Sarah, asked me: “We can reverse decimal numbers with no trouble, when someone asks us to reverse 12.34, we immediately answer 34.12 with no thinking. It is kind of a reaction. But I’m sure there’s an algorithmic and mathematical approach for doing this which we do implicitly in our mind so quickly without knowing it. Do you know what is that approach?”

It was really interesting question and I’ve never thought about it before (well, there’s a reason she is so smarter than me even though she is a little kid). But at that moment it hit me that I can answer this question perfectly with Test-Driven-Development. Maybe you think this is apples and oranges? What is the relation here? Think about it. The whole point of TDD and baby steps approach is doing one little thing at a time until you find a pattern and general solution to a given problem. For finding the implicit and mysterious algorithm to this problem taking baby steps sounds great. (I know mysterious is a little out of the picture word for this scenario but I am just adding some spice for more excitement)

So I started to write a little program to take a decimal number as an input and reverse it and of course with TDD and taking the smallest possible steps at a time. So this was my first test and related production code:

{% highlight csharp %}
[Test]
public void ShouldDoNothingForZeor()
{
  Assert.AreEqual(0, DecimalReverser.Reverse(0));
}

public static decimal Reverse(decimal number)
{
  return 0;
}
{% endhighlight %}

The next small step was handling an integer number with one digit. Here is its test and production code which made it pass along with previous test:

{% highlight csharp %}
[Test]
public void ShouldReverseOneDigitIntegerNumber()
{
  Assert.AreEqual(0.1, DecimalReverser.Reverse(1));
}

public static decimal Reverse(decimal number)
{
  if (number == 0)
      return 0;
  return number / 10;
}
{% endhighlight %}

Now that we could handle one digit integer number it is time for more general case which is multiple digits integer number:

{% highlight csharp %}
[Test]
public void ShouldReverseMultipleDigitsIntegerNumber()
{
  Assert.AreEqual(0.123, DecimalReverser.Reverse(123));
}

public static decimal Reverse(decimal number)
{
  if (number == 0)
      return 0;
  while (number >= 1)
      number /= 10;
  return number;
}
{% endhighlight %}

I think at this moment we finished handling the number before the decimal point (for instance, 12 in 12.34). Now it is time for handling the number after the decimal point. Here is the related test and code (as you can see I did a little Extract Method Refactoring for the before decimal point part):

{% highlight csharp %}
[Test]
public void ShouldReverseOneDigitAfterDecimalPointNumber()
{
  Assert.AreEqual(1, DecimalReverser.Reverse(0.1));
}

public static decimal Reverse(decimal number)
{
  if (number == 0)
      return 0;
  if (number >= 1)
      return ReverseBeforeDecimalPointNumber(number);
  return number * 10;
}

private static decimal ReverseBeforeDecimalPointNumber(decimal number)
{
  while (number >= 1)
      number /= 10;
  return number;
}
{% endhighlight %}

Ok, now it is time for a more general case again and that is obviously multiple digits after the decimal point. You can see the test and code for it below with the related Refactoring:

{% highlight csharp %}
[Test]
public void ShouldReverseMultipleDigitsAfterDecimalPointNumber()
{
  Assert.AreEqual(123, DecimalReverser.Reverse(0.123));
}

public static decimal Reverse(decimal number)
{
  if (number == 0)
      return 0;
  if (number >= 1)
      return ReverseBeforeDecimalPointNumber(number);
  return ReverseAfterDecimalPointNumber(number);
}

private static decimal ReverseAfterDecimalPointNumber(decimal number)
{
  var afterDecimalPoint = ExtractAfterDecimalPoint(number);
  while (afterDecimalPoint != 0)
  {
      number *= 10;
      afterDecimalPoint = ExtractAfterDecimalPoint(number);
  }
  return number;
}

private static decimal ExtractAfterDecimalPoint(decimal number)
{
  return number - Math.Floor(number);
}
{% endhighlight %}

Now we handled main parts of the problem in isolation and separately, you can call it TDD, Divide and Conquer, Baby Steps or whatever you want but the concepts remain the same. At this moment we can use these parts to solve the whole problem:

{% highlight csharp %}
[Test]
public void ShouldReverseDecimalNumber()
{
  Assert.AreEqual(123.45, DecimalReverser.Reverse(45.123));
}

public static decimal Reverse(decimal number)
{
  var beforeDecimalPoint = (int) number;
  var afterDecimalPoint  = ExtractAfterDecimalPoint(number);

  return ReverseAfterDecimalPoint(afterDecimalPoint) + 
    ReverseBeforeDecimalPoint(beforeDecimalPoint);
}
{% endhighlight %}

The most important point in this code is that as our tests become more specific, our code becomes more general. This is one of the most important points that we should pay attention to during Test-Driven-Development which was mentioned at first by Uncle Bob Martin. There are lots of examples out there for this point and one of the best ones is Stack Kata.

Thanks to TDD, I finally found out the algorithm we implicitly use for reversing decimal numbers. Two days after that I went to my friend’s home and I was excited to tell little Sarah the answer to her question. It turned out she solved the problem herself. But the more interesting thing was that she used a completely TDD and baby steps approach for solving it. You can see her solution right here:

![Her solution steps](https://dl.dropboxusercontent.com/u/100502983/reverse_number_pictures/reverse_number.jpg)

I am sure she is going to be a great programmer soon :)
