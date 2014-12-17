---
layout: post
title:  "Refactor till Drop!"
date:   2012-10-13 22:00:00
categories: programming refactoring unit-test
---

Sometimes we think that our code is good enough and clean! It’s totally fine to continue developing instead of looking it from one step back and clean it up before we move one! After all it is called Red-Green-Refactor! It’s not Red-Green-[Move immediately after Green to the next Red]

Currently I’m working on rails application for maintaining banking transactions and generating some special reports for the data on a frequent basis and do a lot of stuff with that data, alerting, grouping, etc. which doesn’t matter for now!

I’m trying to a have good and SOLID design in my application and separate the logic from rails framework for having a better separation of concerns, fast-isolated stuff and some other reasons that I talked/tweeted about them many times!

One of the features for this application is doing several operations on transactions of the current month. So we need to get the transactions of the current month first then do the operations, render views, etc.

This is a good candidate for a class/module which does this specific task and nothing more! Single Responsibility Principle is our friend! I went through the Red-Green-[ Move immediately after Green to the next Red] cycle for this piece of code and I ended up with the following two rspec examples:

{% highlight ruby %}
describe CurrentMonthTransactions do
    it “gets the transactions for the current month which are in the current year too” do
        calendar = stub(:today =&gt; Date.new(2012, 10, 5))
        tr1 = stub(:date =&gt; Date.new(2012,  9, 1))
        tr2 = stub(:date =&gt; Date.new(2012, 10, 1))
        Transaction.stub(:all =&gt; [tr1, tr2])
        transactions = CurrentMonthTransactions.get_transactions(calendar) #this is for dependency injection
        transactions.should == [tr2]
    end

    it “does not get the transactions for the current month which are in the current year” do
        calendar = stub(:today =&gt; Date.new(2012, 10, 5))
        tr1 = stub(:date =&gt; Date.new(2012,  9, 1))
        tr2 = stub(:date =&gt; Date.new(2011, 10, 1))
        Transaction.stub(:all =&gt; [tr1, tr2])
        transactions = CurrentMonthTransactions.get_transactions(calendar)
        transactions.should == []
    end
end
{% endhighlight %}

First thing I like to mention here cause I want to be a positive person is that it’s a good idea that I have 2 tests here and they are testing this “year thing” specifically! Because in this case my tests have better granularity and more focused and since they are supposed to be documentation of the system they are shouting this tricky point about the year in getting the current month transactions that you should not just check the month!

Ok enough with the positivity! This code is terrible, horrible, awful and ugly :D and the reason for it is exactly the third step in my cycle! (look above)

So I tried to tackle this code one step at a time and I show you the steps here. I started with removing duplications and rspec will help you to do such a thing perfectly through different constructs!

Here is the first attempt for both removing the duplication for the 10 magic number that appeared a lot in my tests and also express its meaning with an expressive name:

{% highlight ruby %}
let(:october) { 10 }
# Then I replaced all those 10s with “october” variable
…
{% endhighlight %}

After that I removed the duplication in stubbing the calendar concept like the following:

{% highlight ruby %}
let(:calendar) { stub(:today =&gt; Date.new(2012, october, 5)) }
# Then I removed this line from both tests
…
{% endhighlight %}

After that I removed the duplication of declaring that stub “tr1” in exact same way in both tests:

{% highlight ruby %}
let(:tr1)  { stub(:date =&gt; Date.new(2012, 9, 1)) }
# Then I removed its declaration from both tests
…
{% endhighlight %}

Then I attacked the duplication for getting current month transactions which was again the exact same line in both tests:

{% highlight ruby %}
subject do
    @transactions = CurrentMonthTransactions.get_transactions(calendar)
end
# Then I replaced this line with subject
…
{% endhighlight %}

We eliminate a lot of mess! So far we made the code look like the following:

{% highlight ruby %}
describe CurrentMonthTransactions do
    let(:october) { 10 }
    let(:calendar) { stub(:today =&gt; Date.new(2012, october, 5)) }
    let(:tr1) { stub(:date =&gt; Date.new(2012, 9, 1)) }

    subject do
        @transactions = CurrentMonthTransactions.get_transactions(calendar)
    end

    it “gets the transactions for the current month which are in the current year too” do
        tr2 = stub(:date =&gt; Date.new(2012, october, 1))
        Transaction.stub(:all =&gt; [tr1, tr2])
        Subject
        transactions.should == [tr2]
    end

    it “does not get the transactions for the current month which are in the current year” do
        tr2 = stub(:date =&gt; Date.new(2011, october, 1))
        Transaction.stub(:all =&gt; [tr1, tr2])
        subject
        transactions.should == []
    end
end
{% endhighlight %}

A lot better! But still there are some duplications and it can be better. That’s why I say “Refactor till Drop!” because you should not say: “Ok, it’s clean let’s move to the next failing test!” you should make it REALLY clean! Otherwise these little messes inside the code will gather and bite you usually sooner than you think and sometimes it’s too late to do something about them and they make the development really painful and hard! So I’ll do another change for eliminating even more duplication and this is the final result:

{% highlight ruby %}
describe CurrentMonthTransactions do
    let(:october) { 10 }
    let(:calendar) { stub(:today =&gt; Date.new(2012, october, 5)) }
    let(:tr1) { stub(:date =&gt; Date.new(2012, 9, 1)) }
    let(:tr2) { stub }

    subject do
        @transactions = CurrentMonthTransactions.get_transactions(calendar)
    end

    before do
        Transaction.stub(:all =&gt; [tr1, tr2])
    end

    it “gets transactions in current month and year” do
        tr2.stub(:date =&gt; Date.new(2012, october, 1))
        subject
        transactions.should == [tr2]
    end

    it “does not get transactions in current month but other years” do
        tr2.stub(:date =&gt; Date.new(2011, october, 1))
        subject
        transactions.should == []
    end
end
{% endhighlight %}

As you can see I even changed the name of my tests for being more succinct and at the same time expressive enough! Now you compare the first version and this one then you will realize why I called that one terrible, horrible, awful and ugly! I was not mean! That is the absolute truth!

Now you might argue that we can even make it better (first thing is better names for tr1 and tr2) and I totally agree with you! That’s why you can never say my code is cleanest! There is always a way for making it better! But you should make it clean enough for having a smooth development with no roadblocks! And this code from my point of view for the thing that I’m currently doing is innocent enough and makes me happy!

And I’m not scared at all! Why? Because even if the next reader of this code will be a serial killer and knows where I live, I’m sure he’s not gonna come after me cause this code won’t make him angry! It’s clean enough! I encourage you any time you’re writing a piece of code, imagine its next reader will be a serial killer and he knows where you live! That will help, believe me! :D

Something worth trying!
