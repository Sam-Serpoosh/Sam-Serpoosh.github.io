---
layout: post
title:  "Refactoring Legacy Code"
date:   2011-07-09 23:00:00
categories: programming refactoring
---

What would you do when you have a mess and legacy code and you need to add some functionality to it?

Of course there are different options here. But the one you choose can make adding functionalities and continuing the development, a smooth process or make it so damn hard until you break the monitor with a bat or something.

So it’s an important decision.

I want to refer to some quotes and then we try to decide what we’re going to do in this situation according to them:

Michael Feathers: “any code without test is a legacy code.”

Robert C.Martin: “how do you know something work when you don’t have test for it?”

Martin Fowler: “Refactoring without good test coverage is changing shit”

Rober C.Martin: “if you don’t have good test coverage and Refactor something how you are going to know that you didn’t break anything in the existing code?”

…

There are lots of other similar statements from professional software developers about this issue. But I think we can make a good decision with these(actually I made it before and tried it on a real project and I couldn’t be happier with the result so I’m cheating a little bit here)

First thing we should do is to put the existing code under test. So when we’re Refactoring it, we immediately understand when we broke something.

Actually here’s the tricky part, how you’re going to put an existing legacy code under good test coverage? So you can Refactor it till it becomes cleaner and more flexible and you can add some functionality to it.

As you know it’s a mess and legacy existing code which of course is not so testable. According to Michael Feathers [Working Effectively With Legacy Code](http://www.amazon.com/Working-Effectively-Legacy-Michael-Feathers/dp/0131177052/ref=sr_1_1?s=books&amp;ie=UTF8&amp;qid=1310236092&amp;sr=1-1)  book, when you’re dealing with legacy code and you want to Refactor it and put it under test you don’t have to put every single line of it under test till you start Refactoring. Not only it’s not possible(cause it’s not testable, specially that much) but also it’s not that useful for this situation.

We can just put the functionality of one code area under test to make sure that area works correct. Then we start Refactoring that particular area and make it cleaner and cleaner till it becomes gorgeous. During this process after each little change, we run the tests against that area to make sure that we didn’t break anything. And if we broke something we can roll back our last little change with only few ^z and do it again this time more carefully.

Now that we Refactored the existing code, we can put the cleaner and more testable new code under other tests for having more isolated micro tests for different methods and modules and etc.

Now we have clean, flexible code which we can add new functionality to it so much easier and smoother than before. And from now on, we develop new functionalities with TDD approach which leads us toward better, cleaner, more flexible code and design.

For example imagine we have a class for solving the TSP(Travelling Salesman Problem) and there’s a solve method in this class which takes a distance matrix as an input and return the path with the least cost in the given distance matrix.

It’s a really messy method with more than 100 lines of codes and bad naming and lots of other bad features. It’s completely obvious that this method is doing more than one thing, actually it does lots of things. For example, **it creates list of available cities in the path, find the best starting point for travelling, update the available cities during the travelling, in each step find the next closest city with its related cost, update the best path**, etc.

All of these different things buried in this method and we can’t test them in isolation and separately. In this case we write some tests for the functionality of this code area. Create some test distance matrices and give them to the solve method and retrieve the result and compare them with our expected results.

With this way we make sure that the **solve method works correctly**. At this point we start Refactoring the solve method with some routine Refactoring moves like extract method, renaming some stuff, etc. after that we have lots of methods each of them doing one thing. Maybe some of them belong to other class so we do some split class Refactoring and put them at their real home and etc.

Now we have chunks of code each of them has only one responsibility and one reason to change so we can put them separately under isolated micro tests:

{% highlight csharp %}
resetAvailableCities();
findeClosestCityForTheCurrentCity();
updateBestPath ();
increaseTotalCostOfTravelling();
…
{% endhighlight %}

Now we can add new functionalities to our code easier and also we would develop them with TDD approach like we talked about it above.

Something that worth a try.
