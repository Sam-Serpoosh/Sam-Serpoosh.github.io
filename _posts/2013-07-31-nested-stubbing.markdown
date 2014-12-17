---
layout: post
title:  "Nested Stubbing => Shouts for Refactoring"
date:   2013-07-31 18:00:00
categories: programming refactoring ruby unit-testing
---

A lot of programmers write unit tests during the development and also a lot of programmers do Test Driven Development. One thing that we usually forget while programming is Listening to our Tests. If we listen carefully to our tests they will give us a lot of interesting hints and information and frankly that’s why a lot of people call it Test-Driven-Design because we can find useful points in our tests that will help us to have a better design.

I’m going to talk about one of those points that we can find out very easily by listening to our tests and will help us to have a better design and having a piece of information in its right place in our program. That certain point is Nested Stubbing which BTW happens a lot in our tests.

Lets say we are writing a twitter client app and all the communication through the network, OAuth related parts, calling Twitter API, fetching and storing tweets, etc. is being done in separated modules ([Separation of Concerns](http://en.wikipedia.org/wiki/Separation_of_concerns)  and [Single Responsibility Principle](http://en.wikipedia.org/wiki/Separation_of_concerns) etc.)

We have also different kinds of presentations (views if you will) for this application. One of them is a Console Based presentation for taking a look at the tweets in terminal. One part of this presentation is taking the latest 10 tweets and rendering them (in whatever way you like) to the user. Obviously the rendering related code lives in its own place separated from the logic, data storage etc. Imagine we have a Console class which gets those latest tweets and give them to the renderer object in order to show them to the user (for brevity I remove some parts):

{% highlight ruby %}
def show_latest_tweets
    last_ten = Tweet.order(‘created_at DESC’).limit(10)
    renderer.render_tweets(last_ten)
end
{% endhighlight %}

and lets say we have a test for this piece to make sure that this method is calling the right things:

{% highlight ruby %}
it “fetches last 10 tweets and renders them” do
    latest_tweets = [t1, …, t10]
    ordered_tweets = stub(:limit).with(10) { latest_tweets }
    Tweet.stub(:order).with(‘created_at DESC’) { ordered_tweets }
    renderer.should_receive(:render).with(latest_tweets)
    console.show_latest_tweets
end
{% endhighlight %}

^^

If you look at this code there is an obvious violation of  **Law of Demeter** happening. We know that when we call **order** on **Tweet** what is being returned has a limit method that we can call to limit the results. How can you easily detect this? *BECAUSE WE’RE DOING A NESTED STUBBING IN OUR TEST*. We are setting up a nested stub cause we setup *ordered_tweets* as a result of calling **order** on **Tweet** which is a stub itself. This tells us that we know **TOO MUCH** about the inside of **Tweet** class and its details at this level (Console class) which we should NOT!

Right now we’re using something like ActiveRecord as the ORM for storage part of the application but what if we change that later to something else? There’s a good chance in that new ORM the mechanism for doing the same thing (getting the latest 10 tweets) will be different and we need to change our code appropriately. But with this code that we wrote here we have to change **Console** class for changing our storage mechanism which does not make any sense. Console SHOULD NOT know anything about the details of storage and storage should NOT be a [REASON of change](http://en.wikipedia.org/wiki/Single_responsibility_principle) for Console.

We need to have a layer which hides this information from Console and give him what he wants instead of Console reaching for that information through method chains (Tell, Don’t Ask). As you can see Tweet is like a model (if you will) in this application. And he’s the one who should know about the storage mechanism in this app (how to be stored, retrieved etc.) [Or even taking it further there can be a TweetRepository class which is specifically handling storage-related stuff for Tweet, kind of like a Façade Pattern between Tweet and DB/File/etc.]

We can add a method to Tweet class like the following:

{% highlight ruby %}
def self.last_n_tweets(n)
    Tweet.order(‘created_at DESC’).limit(n)
end
{% endhighlight %}

So now lets rewrite our test based on this change:

{% highlight ruby %}
it “fetches last 10 tweets and renders them” do
    latest_tweets = [t1, …, t10]
    Tweet.stub(:last_n_tweets).with(10) { latest_tweets }
    renderer.should_receive(:render).with(latest_tweets)
    console.show_latest_tweets
end
{% endhighlight %}

And now the code for it:

{% highlight ruby %}
def show_latest_tweets
    last_ten = Tweet.last_n_tweets(10)
    renderer.render_tweets(last_ten)
end
{% endhighlight %}

First of all, we don’t have that nested stubbing in our test method but that’s not the point, if you pay attention you see that we eliminated the coupling and dependency of Console to the Storage mechanism and it doesn’t have any knowledge about that part anymore which is a big advantage. If we decide to change the data storage part of this application or change our ORM, that change will be hidden from Console or any client of this functionality (Tweet retrieving stuff etc.)! Console will still call **Tweet.last_n_tweets** and how’s that being implemented is none of its business and it doesn’t care.

As you saw listening to our test can have interesting results and nice design hints. Whenever you feel that your test is more work than it should be, or it doesn’t seem to be right or it’s hard to write the test for the target piece of code, that means there’s a design problem in our code (*MOST OF THE TIMES*). So we should fix the problem in the right place not struggle with making that test happen anyway. It's time to stop preaching.

Hope that helps and happy hacking.

^^ This test contains more than one assertion which is usually not a good idea (there are exceptions depending on the situation like anything else in programming)!  I wrote those here to show how things are supposed to work in that piece of code!
