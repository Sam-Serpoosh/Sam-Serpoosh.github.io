---
layout: post
title:  "Unit Tests should be for Units"
date:   2011-03-17 18:00:00
categories: programming unit-tests
---

Unit Tests should be more focused and specialized:

Writing unit tests is a really interesting and important craft and art which every software developer should understand and practice it very well for being more effective and successful.

In Test Driven Development you write lots of unit tests (or we can call it Micro Test) for deriving the design of your code and verifying the behavior and functionality of it.

The one thing that most of the time is seen is that people don’t write **Unit Test**. They think they do but if you look at them carefully while they’re programming they write a test (it’s not unit you can call it whatever you want but it’s absolutely is for broader scope than a unit) and spend lots of time on the production code and write lots of production code to make that test pass.

I want to refer you to a quote from Robert C.Martin (a.k.a Uncle Bob): "according to three laws of TDD we can’t spend so much time on tests or production code and we should switch between them quickly. Spending time on each mode (either production or test) is in order of extremely 5 minutes even 10 minutes is too long and wrong. Yeah even 10 minutes is **TOO LONG**"

But I saw lots of developers that write a test and spend even rest of their working day on making it pass and all of their time is in production code. So they’re not doing TDD at all.

When you ask them why they are doing this they say: “I’m doing top-down or “out to in” approach. I know the problem and write a test for it and then try to implement the solution to make that test pass.”

As you can see in the TDD work of Kent Beck he mentioned that you can do both “out to in” and “in to out” approach with TDD. It’s just a matter of trade off. Personally if I know the problem well I do “out to in” approach but if I don’t know it, I take the smallest and simplest possible part and work on it and through this road learn more and more about the problem. (Thanks to Brett L.Schuchert for some points about "in to out" and "out to in" approach)

But the thing is even if they’re doing “out to in” approach, this way is not TDD “out to in”.

When you’re doing “out to in” with TDD you write a test for the problem (which maybe is not unit and cover the whole problem maybe we can call it acceptance test, it’s up to you) and then go to the production code and try to make it pass, but as soon as you understand that you need for example a helper method for part of your solution, you go back to test mode immediately, write a **UNIT** test for that helper method or that unit of code and again go to write production code for that **UNIT** and make this test pass and so on and on, till you completely implement the solution and make the main test pass too. With this way you’re moving right in the TDD **cycle**.

(you absolutely know it but I just mention it: when you’re writing test for that unit you thought you need you should ignore that main test and make this new one pass and then again and again until the moment that you think the only test which remain and should be passed is that main test which you wrote at first, that acceptance test actually)

Let me make it clearer with an example:

Imagine you want to write a class which converts an Entity to Xml formatted string. You know the problem well and you can work “out to in” on it. This class should take the Entity and return a string in standard Xml format. With the wrong way people often write their test (hopefully at least they write this kind of test first, cause in most cases they don’t write test at all!!!) like this:

{% highlight java %}
@Test
public void shouldConvertEntityToAppropriateXml()
{ … }
{% endhighlight %}

And after that they spend so much time try to make this test pass. They write lots of methods and lines of codes in production code. They have to use debugger a lot because there’s so much code without specialized tests for each part and they can’t understand what’s going on exactly and why.

When their test fails they have to use debugger and check different things (awful) cause they don’t have “focused unit tests” for units of their code.

As you can see they almost have all the problems that non-TDD approaches have, except they at least have a little automated test. And most of the times these are the people who complain about TDD and say it’s not working well, it’s making us slower and takes lots of times without any benefits.

According to Jason Gorman: “If TDD makes you slower or you don’t achieve good results with it, there’s a good possibility that you’re not doing it right and you’re not good at it.”

And the important thing is that, this has nothing to do with **coverage**. You can have 100% coverage with this one test but it’s not enough and it’s not good at all. You got even slower than when you didn’t write test at all and also reduced your productivity. This is so not TDD.

So you should move through the TDD cycle correctly and switch frequently between test and production mode. You can work “out to in” and write this test which covers the whole problem. But when you’re trying to make it pass you understand that you should write some helper methods and units of code in that class for solving this problem. Immediately you should go to test mode, write a unit test for it and then make it pass and so on. For example in this case we can have some unit tests like:

{% highlight java %}
@Test
public void shouldGetEntityNameAndSetItAsRoot()
{ … }

@Test
public void shouldAddSimplePropertiesAsAttributesOfRoot()
{ … }

@Test
public void shouldAddCollectionPropertiesAsChildrenOfRoot()
{ … }

… //other needed unit tests;
{% endhighlight %}

As you can see each of these tests are more focused and specialized on a part of a class or even part of a method or in other word they are focused on a **UNIT** of code because they’re unit tests for God’s sake. We should write unit tests in TDD!!! The first test is more like an acceptance test for this behavior or functionality (converting an Entity to Xml formatted string in this case) but the rest of them are unit tests because they’re testing a unit in the code.

Something that worth to try.
