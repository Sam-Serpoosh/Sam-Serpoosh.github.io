---
layout: post
title:  "Communicative Name for Unit Tests"
date:   2011-05-10 11:00:00
categories: programming unit-tests refactoring
---

This is just a short post for mentioning a quick but very important issue in unit-testing(micro-testing). Some people don’t pay enough attention to naming their unit-tests!!! Actually some people don’t pay enough attention about quality of their unit-tests(but it’s a broader topic which I’ll cover later. But in a nutshell, don’t forget that more than half of our codes are test codes so I think it makes no sense that we don’t care about the quality of half of our code!!!)

I’ve seen lots of programmers with unit-tests like this:

{% highlight java %}
@Test
public void testOne();

@Test
public void testTwo();

…
{% endhighlight %}

For the love of God, what the hell is that?

I think we forgot about one of the most important benefits of unit-tests. Unit-tests are live and executable documentations. You should be able to read them and understand what the system under those tests is doing without any need to go deep into the production code.

But I want to go further than that. For understanding **WHAT** the system is doing, you should only need to read **NAME** of its unit-tests.

Imagine your teammate worked on a module and now he wants to go and work on something else and you should continue that module. You check-out the latest version of that module from version control(I hope you’re using version control tool), Then you immediately open it’s related tests cause you know that those are live documentations for that module and you can find out what that module is doing by reading them.

Then you face this:

{% highlight java %}
@Test
public void testOne();

@Test
public void testTwo();
{% endhighlight %}

You can’t understand what that module is doing with this kind of naming. You have to stick your nose into details of those unit tests for learning things. This is inefficient and time-consuming.

If you want to change a method in that module it’s ok to see its related unit-test in details because you need to see what’s its API and how you can use it and other stuff like that(which are more related to implementation details). But when you want to know WHAT the module is doing and nothing more, I think it’s better you can find it out with only reading unit-tests names. These are the names of unit tests for a simple Login module which I wrote in Java, compare them to the example at the top:

{% highlight java %}
shouldNotValidateWhenUserNameIsNull();

shouldNotValidateWhenPasswordIsNull();

shouldNotValidateWhenUserIsNull();

shouldNotValidateWhenPasswordIsWrong();

shouldValidateWhenPasswordIsRight();
{% endhighlight %}

By reading these names, you can easily understand the policy of this module for validation or invalidation a user. You just found out WHAT this module is doing with only reading it’s unit-tests names. That’s what I’m talking about.

Something that worth a try.
