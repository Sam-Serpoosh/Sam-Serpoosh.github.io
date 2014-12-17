---
layout: post
title:  "Sometimes remove the tests after they served you!!!"
date:   2013-02-23 23:00:00
categories: programming unit-test
---

I know it sounds a little bit strange when I say removing tests after they served you specially because we all know that one of the great benefits of having automated unit-tests is that you’ll end up with a regression suit that you can always run whenever you make a change in your code! So as long as you’re maintaining a project they are serving you in that matter!

What I’m targeting here is the tests that we write during the development directly for methods that should be private in the class-under-test (CUT)! I’ll show you what I mean with a SUPER simplified example. Technically I eliminated everything in this code for showing my main point!

Imagine you’re writing a microblogger which is using a third party API as a twitter client. As we know one of the features of twitter is Direct Message  (known as DM). For implementing this feature you will write a method in your MicroBlogger class named “dm” for example. Of course we’re doing it TDD so one test for this class would be like the following:

{% highlight ruby %}
it “doesn't send message for someone who’s not following you” do
    twitter_client = stub(:followers). and_return([])
    microblogger.dm(“anyone”)
    twitter_client.should_not_receive(:dm)
end
{% endhighlight %}

Now we’re gonna write enough code to make this pass. When we start to implement the production code for this test we realize that we need to check whether the receiver is following us or not![1] This is a unit of work on its own! There are 2 options here for us. A) We start implementing the dm method and inside that method we insert the logic of this check and see whether the test will pass or not B) Make the current test pending and start TDD-ing that unit-of-work (checking whether the receiver is following us or not) and as soon as we have that functionality in place un-pending (weird word) the test and continue the development of dm method.

IMHO the second option is better for several reasons. First of all the first option is violating the rule of taking the smallest possible steps and going in the baby-step style that eXtreme Programming and TDD talk about it cause it’s obviously going through more than one unit-of-work. Also if the test fails after implementing the dm method with the first option we need to check more than one thing, either the check for followers has a problem or the other part of the dm method. And we should have the least number of reasons for a test to fail (1 is the best). And some other reasons which are not for this post.

Then we make the original test pending and start TDD-ing the check functionality for followers. Couple tests for something like that would be the followings:

{% highlight ruby %}
it “knows when someone is following you” do
    …
end

it “knows when someone is NOT following you” do
    …
end
{% endhighlight %}

Then the production code for making them pass can be in a method called followed_by?(someone)

Now it’s time to un-pending the original test that led us to here and implement the dm method to make it pass. Part of the dm method for making the test pass could be like the following:

{% highlight ruby %}
def dm(receiver)
    …
    return if !followed_by?(receiver)
    …
end
{% endhighlight %}

Perfect. Now think about the design of the class under test and how its interface is gonna be to the outside world (other classes/modules of the system)! The followed_by? method should not be visible to the outside world. This is a method that is being used internally in the class and we defined it cause we didn’t want our dm method to do more than one thing! Remember Uncle Bob definition of clean method: “A method should do one thing and only one thing and do it well!”

So what are we gonna do now? If we make the followed_by? method private then those couple tests will complain about it cause they can’t see it anymore and we have to remove them to shut them up! It’s A OK to do that in this case. Sometimes you can remove the test after they served you. We TDD-ed the functionality of followed_by? in our code and we’re using it with no problem in the dm method. And it’s completely a good choice to make that method private according to the circumstances and how it’s being used in the class. And followd_by? method is still being tested indirectly through the tests for the dm method, so we didn’t ruin the coverage of our automated tests.

And I know that we don’t have that ONE reason for failure anymore but again “Programming is the world of trade-off” and IMHO this time it’s a better idea to make the method private and remove the tests related to it cause they served their purposes during the development of that part. I learned this interesting technique and idea from Kent Beck TDD screencasts by Pragmatic Programmer which I highly recommend you to check them out! This technique came handy and very interesting in a bunch of places while I was writing code and I thought it’s a good idea to share it with you.

Hope that’ll be useful!

[1] When we’re doing Red-Green-Refactor cycle in this case we just define the dm method and the code will pass cause we’re not calling the dm method on the twitter_client (writing only enough production code to make the failure pass)! And then with more tests we drive the design and development of the dm method towards this whole idea of checking whether the receiver is following us or not! But again this is a blog post and I wanted to make another point here so I cut through the path a little bit :-)
