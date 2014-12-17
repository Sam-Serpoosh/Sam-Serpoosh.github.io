---
layout: post
title:  "Expressive Literals in Tests"
date:   2011-09-20 23:00:00
categories: programming refactoring unit-test
---

This is just a quick thing about making unit tests (or micro tests, you can call it whatever you want the important things is the concept) more readable and less confusing. In [TDD by Example](http://www.amazon.com/Test-Driven-Development-Kent-Beck/dp/0321146530/ref=sr_1_1?s=books&amp;ie=UTF8&amp;qid=1316553752&amp;sr=1-1) Kent Beck says that we should be careful about the data we use in our unit tests. If we use a data for a specific purpose we should always use that data for that specific usage, because it makes our tests more consistent and readable. Also it doesn’t cause any confusion for the reader. For instance, if we want to have one sample student entity in some of the unit tests, set its Id to 1 for all the places, do not set it 1 for a unit test and 2 for another one (of course this depends upon the goal of the test and sometimes we should use different values but clearly I’m not talking about those situations and I mean situations that using different values not only have no benefit but it also makes inconsistency and confusion).

We can go further than this and try to make our unit tests even more communicative and expressive by just a little thinking about the values of the data in tests. If we care about the hard-coded values in our tests like the way we care about naming of the things in our code then we will end up with a great improvement in the readability of our unit tests.

Imagine we are writing unit tests for a log in system. One of the cases is that if username was nil and password was whatever (it can be anything, doesn’t matter) system should not validate this combination of username and password:

{% highlight ruby %}
it “should not validate when username is nil” do
    result  = @user_validator.validate(nil, “blabla”)
    result.should == false
end
{% endhighlight %}

By reading the content of this unit test we cannot understand about the point that when username is nil then it doesn’t matter what the password is because its value is “blabla”, something that is not expressive at all. For solving this issues there are two solutions a) express this fact in the description of the unit test b) use more communicative and expressive value for password. I’m gonna focus on the second solution here while the first one is important equally. But sometimes you cannot say every little thing in the description or name of the unit test because they should be more story-like and talk about scenarios. In this example if we use “any password” instead of “blabla” anyone can understand the point by reading it.

In another scenario we will have something like this:

{% highlight ruby %}
it “should not validate when password is wrong” do
    result = @user_validator.validate(“john”, “blabla”)
    result.should == false
end
{% endhighlight %}

It is a good unit test and you can understand a lot by its description but the combination of the username and password is not very communicative and can be a little confusing. Now what about this one:

{% highlight ruby %}
result = @user_validator.validate(“correct username”,  “wrong password”)
{% endhighlight %}

At first when we see the previous test we say it is perfect and completely expressive but when we see this new one, we will be like: “aw, this one is much better”. This is exactly the Corey Hains point about the fact that no code is in its cleanest state and there is always room for more Refactoring. With this little change we made a great impact on the readability and expressiveness of the test and no one will be ever confused by these values. Also we can conform the Kent Beck’s rule that I mentioned at the beginning, perfectly. Whenever we mean wrong password we can use “wrong password” or whenever we mean password doesn’t matter we can use “any password” and so on. This little thing can show its wonderful effects in huge code bases with thousands of unit tests.

Sometimes we think that are these extra efforts worth it? Do they return worthwhile values? But always remember that these little things can make a huge difference. By doing these kinds of improvements a good code or even a great code can become **WONDERFUL**. No one will ever say: “that is good don’t make it better”!!! There is always room for becoming better and making improvement in **EVERYTHING**.

Thanks for reading.
