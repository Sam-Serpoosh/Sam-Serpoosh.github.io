---
layout: post
title:  "Intermediate Variables, More Communicative Code"
date:   2012-03-26 19:00:00
categories: programming refactoring unit-test
---

As you know one of the [four rules of simple design](http://c2.com/cgi/wiki?XpSimplicityRules) is that the code should clearly express the intent of its author. So any attempt for making the code more communicative and expressive will pay off later. As Martin Fowler says: "Any fool can write a program that computer understands but the art is writing a program that humans can understand perfectly."

Kent Beck has a book called [Implementation Patterns](http://www.amazon.com/Implementation-Patterns-Kent-Beck/dp/0321413091/ref=sr_1_1?s=books&amp;ie=UTF8&amp;qid=1332794942&amp;sr=1-1) this whole book is about the techniques and patterns for making the code as communicative  as possible. I strongly recommend reading this book at least twice because you will see how dramatically your code becomes more expressive as a result of these patterns.

We should try to write a clear code which communicates its intent perfectly. Because when someone will read it in future (doesn’t matter close or far) can understand what it does and what was our intent when we wrote it at the first place. Remember, most of the times we will be that someone so it is better to write our code in a way which won’t make us to curse ourselves.

One of the simplest and at the same time great ways for making the code more communicative is using expressive intermediate variables. Unfortunately we forget about this little technique a lot and the main reason for that is the attraction to making things succinct. It is nice to write succinct code but remember succinct means **short AND useful**. It doesn’t just mean short. But sometimes we forget that and we are so happy while we are making things done in ONE line. Sometimes this one line thing comes at a price and that is less readability and understandability of the code. We fade in this fake beauty so much that we forget about the main goal which is making things clear and expressive. There’s an old saying: “don’t make much at once.” We should consider this continuously during our programming.

Few days ago at work, we found a problem in our filtering process. When in the UI we set some filtering for data retrieving it worked correct but when we removed that filter and retrieved data again that same filtering was considered as before. For instance when I wanted to get users that their first name was John, I wrote John in the FirstName filter text area and it worked as it should but when I removed John from that filter text area again it gave me the users with first name John.

Finally I found out that the filter mechanism didn’t clear itself in some special circumstances and I tried to write a unit-test for it which of course failed and then I made it pass. Sure enough the problem was solved and the filtering process worked like a charm since then. (Sometimes when you are doing TDD you cannot think of all possible cases, so later you will face a problem in your system. The best reaction to that is writing an automated unit-test for it and reproducing the problem in that test and then making it pass. With this approach you will be sure that this situation will be covered all the time through your test suite after that.)

I did this process while our new employee sat next to me. The test was almost something like this:

{% highlight csharp %}
[Test]
public void ShouldRetrieveAllDataWhenThereIsNoNeedForFiltering()
{
    var searchEngine = new SearchEngine();
    searchEngine.AddFilter(new Filter(“FirstName”, “John”));
    var users = searchEngine.GetUsers();
    Assert.AreEqual(10, users.Count);
    Assert.IsTrue(AllUsersNamesAre(“John”));
    searchEngine.AddFilter(new Filter(“FirstName”, null));
    users = searchEngine.GetUsers();
    Assert.AreEqual(50, users.Count);
}
{% endhighlight %}

When I asked her how she feels about this, she said: “what does it mean when you say new Filter(“FirstName”, null)? what is the meaning of null in this context? Does that mean users that their first name is null?” At that point it hit me. I was so excited about making the filtering process in one line that I forgot about the most important goal. I should make the code communicative so the reader can understand the intent perfectly. But apparently I screwed up. So I sat back and looked at the code and thought with myself: “why is this so obvious to me but not her?” and then I found out. I knew that when the search engine gets a filter it will only use it when that filter is effective. Somewhere in the GetUsers implementation the search engine will check this à filter.IsEffective() and IsEffective will return false when the value part of the filter is null etc. And when the filter is effective the search engine will use it otherwise it will ignore it. Because I knew about the implementation details of this code that test was clear to me. But that is the problem. The unit-test is supposed to be the live and executable documentation of the system and everyone should be able to find out about how the system works in general by looking at it not by knowing about the implementation details of the system.

So I made this one line change and everything turned out differently:

{% highlight csharp %}
public void ShouldRetrieveAllDataWhenThereIsNoNeedForFiltering()
{
    …
    var ineffectiveFilter = new Filter(“FirstName”, null);
    searchEngine.AddFilter(ineffectiveFilter);
    …
}
{% endhighlight %}

After this she said: “oh, it is an ineffective filter so it will do nothing and then the search engine should return all the users.”

With that one line and introducing an expressive intermediate variable with appropriate name the whole intent of the code became obvious even to someone who had no idea of the implementation details of the production code.

## Conclusion

The main goal is communicating the intent through the code as much as possible. Even simple techniques like expressive intermediate variables can change things dramatically. Remember that these little things make the difference between a crap and a well-crafted code. Do not forget about the goal along the way. Always keep in mind that by expressing the intent clearly the reader will thank you and be able to understand the code perfectly. **MOST OF THE TIMES THE READER WILL BE YOURSELF.**

Happy Hacking :)
