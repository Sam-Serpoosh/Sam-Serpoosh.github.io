---
layout: post
title:  "Wishful Thinking & Test Driven Development"
date:   2013-08-13 11:00:00
categories: programming unit-test wishful-thinking
---

In the past few months I’ve been doing something a little bit different from the approach that I usually take while programming/developing software. If you’ve read the GREAT book [SICP](http://www.amazon.com/Structure-Interpretation-Computer-Programs-Engineering/dp/0262510871/ref=sr_1_1?ie=UTF8&amp;qid=1379517612&amp;sr=8-1&amp;keywords=SICP+book), you should be familiar with the term “Wishful Thinking”! (If you haven’t, I HIGHLY recommend reading it and also watching the talks by Professor Sussman and Abelson on the topics which you can find [here](http://www.youtube.com/playlist?list=PLB63C06FAF154F047))!

There are tons of places that you can see/hear that term but that book is probably one of the first places that talk about this idea. And it’s basically about not thinking about too many levels of abstraction at the same time and not jumping around with your brain for solving a problem. That means lay down the steps that you need to take to accomplish something and don’t think about HOW to do each of those steps at that very moment. It’s like a 1000 feet view of that specific task at hand. When you have the map for the steps, when you know WHAT are those steps, then try to figure out HOW to attack each of them. Then you focus in! Think of your whole application as a map and what you do here is basically taking a piece out and focus in in few steps! First laying down the steps of work needs to be done in order to accomplish that task and then figuring out HOW to do each of those steps! Hopefully following picture will give you some ideas.

<iframe src="https://drive.google.com/file/d/1-mYoPvNM0jqZjJOw-LCzOXMzzhBCip0O/preview" width="640" height="480"></iframe>

This sounds a lot like up-front design and frankly, if the level of abstraction that you are dealing with at one point is a whole application or even something really big in your application, you ARE doing an up-front design. And anybody knows that up-front design is not a good idea in most cases and I’m not going to talk about that, thousands of people said that way better than I ever can. But at the same time if we use this technique (Wishful Thinking) at a correct level of abstraction or at the right point, it can be really helpful and powerful. Imagine we’re writing a twitter client^ application and the feature we’re trying to implement (the task at hand) is getting the last tweet of all the people you are following (what a useful feature BTW)! The naïve (for blog purposes) implementation steps for this feature is going to be something like the following:

  - Get the list of people who the current user is following.
  - Iterate through each of those people.
  - Get the last tweet for them.
  - Pack those tweets and Tada!

That’s like a 1000 feet view of this specific feature. And if you look at them, those can be exactly the methods that we can have in our code for this feature:

{% highlight ruby %}
 def following_last_tweets
   following = get_following_of(current_user)
   last_tweets = []
   following.each do |person|
   last_tweets << get_last_tweet_for(person)
   end
   last_tweets
 end
{% endhighlight %}

And again this is not an up-front design or anything like that. We’re completely focused at a granular feature of the application at this point and laid down the steps needed to implement that. So the level of granularity in doing this matters and you should be careful of not flying so high! So now it’s the time for the question HOW we’re going to attack each of those steps? If you try to answer that question completely right away, the result of your code probably won’t be very elegant. What do we do then? The thing that we usually do! We answer that question for each of those steps via TDD. Actually we even comment out that code that we just wrote, because that code does not even work, it’s just a map for the steps that we need to take to accomplish that feature. Think of it as a TODO list in your code instead of having it in a separate text file or so!

The nice thing about this approach which is basically a combination of “Wishful Thinking” and “Test-Driven-Development”, is that we get some guidelines from our wishful thinking and then try to come up with a nice solution for each of those steps in a bottom-up and incremental fashion using our unit-tests and letting them DRIVE us toward that solution and a good design.

We don’t do any over-design or over-engineering and this is not against YAGNI either. If we are going through one of the steps and then realize: “This is not the way to go for it” it’s just a matter of reverting couple minutes of work (and thank God for having version controls to make it as easy as it can get), even if we realize that our map/guideline has problems, we haven’t gone too far with this approach and we can easily think of new steps and start over again. Because it’s just a matter of few minutes and our map wasn’t even functional, it was just few lines of comments for directing our overall approach for that specific piece of functionality.

That’s why it’s VERY important to use this technique or this combination of techniques at an appropriate level of abstraction and granularity in the application! Doing it at a VERY high level is just an up-front design and can make you do hours of effort and reverting all that back because of over-engineering/design and not considering some aspects which you’ll find later on and you KNOW that HAPPENS! Doing it at a VERY low-level is not going to provide any value (almost) and you might as well do the complete bottom-up approach without having any map or guideline cause your steps will become too much primitive. (e.g print a username or the like!!!)

It’s similar to Kent Beck’s rule of [Single Level of Abstraction](http://www.markhneedham.com/blog/2009/06/12/coding-single-level-of-abstraction-principle/) which we can only realize what that really means and how to get it done correctly by practicing A LOT and watching the code and thinking about it deeply for a while to see what’s going on at each point and is there a nice harmony between those lines of code at each method or module. For instance if all you’re doing in a method is calling some other methods and all of a sudden there’s a very primitive line of code like a = 2; in there, there’s a good chance that line does NOT belong there!

I’ve been trying this approach for a while and it’s been serving me well. I ended up writing more interesting and cleaner and better-designed code because of that. Maybe a lot of people write their code exactly like this or maybe we’ve been always doing this but we just lay down the steps in our mind unconsciously instead of writing them down IN THE CODE and attacking them one by one!

Anyway, I think it’s a good time for me to stop preaching, I found this approach interesting and useful and I thought maybe it is useful for someone else as well!

Hope you find it interesting as well and happy hacking ☺

^ The reason that I recently use this kind of application as an example, is because I’m writing one just for fun and it’s full of interesting points and examples. You can find the code for it [here](https://github.com/Sam-Serpoosh/twitter_client)!
