---
layout: post
title:  "Few more characters better than jumping around"
date:   2013-05-12 20:00:00
categories: programming ruby unit-test
---

This is a short post for pointing out a quick and handy little thing that I found helpful in some cases while programming.

Some people are religiously against comments and they know the existence of comments in the code as a sign that means the code is not readable! (I was in that group for a while as well :P)

That’s not necessarily a true thing to say or believe. Comments can also exist in a readable and clean code. And they can be very helpful. In one specific case that I want to mention here, they can save you some scrolling and jumping around the file in order to get a sense out of the code you’re reading.

I’m a huge fan of the statement: “Imagine the next reader of your code is a serial killer and he knows where you live!” I always try to make my code as readable as I can at the moment. I like to be able to get a sense of my code with a glance without any need to dig deeper. Of course this is hard to achieve 100 percent but I try to make it really close when I can. One of the things that I noticed in my test code is that some times in order to have DRY (Don’t Repeat Yourself) I extract out a helper method or a setup block for setting up some data that I need to reuse in different tests that I’m writing.

Imagine you’re having a model named Event and you’re testing some validation rules for this model. Instead of keep repeating some values for its attributes in every single test it’s a good practice to put that as a setup in a “let” block in rspec as following:

{% highlight ruby %}
let(:attributes) do
    {
        event_type: “CreateEvent”,
        generated_at: sometime,
        event_no: 1
    }
end
{% endhighlight %}

and  just change the one that is relevant to the validation rule that you’re about to test in your current test like the following:

{% highlight ruby %}
it “requires a type” do
    event = Event.new(attributes.merge(event_type: nil))
    event.should_not be_valid
end
{% endhighlight %}

This makes your tests less polluted with repetitive values for the attributes over and over again. It’s a small thing with almost a big effect in the code cleanness.

Sometimes having this setup block makes you to scroll and jump around the test code to realize what are the values cause you’re depending on them in order to understand the test you’re reading and trying to understand. Not always you need to know the values. For instance in the above test with a quick glance you see what’s going on and you don’t care what are the values for the attributes cause the only thing that is matter (being tested) here is that event_type can’t be nil and it’s a required attribute.

But sometimes you need to know what’s going on in the attributes. Imagine a scenario that you need each event have a unique value for the event_no attribute. And you wanna test it like the following:

{% highlight ruby %}
it “needs a unique event_no” do
    event1 = Event.create!(attributes)
    event2 = Event.new(attributes.merge(event_no: 2))
    event2.should be_valid
end
{% endhighlight %}

Now here you cannot understand the complete picture with just a glance. You can guess it obviously and I want to make the code completely readable so the reader won’t have to think/guess as much as possible or she won’t have to jump around the code/file to realize what’s going on. When you see this you can guess that the first event (event1) has a different event_no value so that’s why the second event (event2) is valid. Which is totally fine but I don’t think there is any harm in being even finer when it’s the least amount of effort required in order to get there:

{% highlight ruby %}
it “needs a unique event_no” do
    event1 = Event.create!(attributes) #=> event_no = 1
    event2 = Event.new(attributes.merget(event_not: 2))
    event2.should be_valid
end
{% endhighlight %}

That’s it!!! Now with this few characters you saved your life (if the reader was a serial killer and he was pissed by having to scroll up and see the value for event_no in attributes)

I’m sure you think this is TOO MUCH. But again since it’s the least amount of effort and it makes the code even cleaner and more readable than before why not?! It’s so tempting to make it like this and save some scrolling and jumping or unnecessary guessing! Cause this makes the reader to get COMPLETELY what’s going on with just a glance which in my opinion is the best level of readability in the code => Glance-Understandable!!!

Anyway, I stop preaching right now and that’s all I have to say about this. I hope that will be useful.
