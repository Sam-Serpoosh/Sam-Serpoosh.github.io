---
layout: post
title:  "Writing Katas in Different Languages"
date:   2011-02-08 23:00:00
categories: programming refactoring
---

I assume that anyone who reads this post is familiar with the concept of Kata in programming. But I’ll just introduce it in few lines. Kata is kind of practicing which originally belongs to martial arts. It’s a practice that each student does exactly what their master does over and over again till they can do those moves like their master, perfectly, beautiful and with high quality and efficiency.

The Kata in programming was introduced by Dave Thomas for the very first time. It’s one of the great ones in Agile practicing items. It helps you to be more productive and efficient. It makes you so much better in programming in a language and also working effectively with your IDEs and knowing key chords(a very fancy and interesting name for shortcut keys or key strokes which I read somewhere lately) for usual tasks that you have to do while programming and etc.

I saw lots of programmers which get interrupted by using mouse and go over a menu and clicking on one item and then the related dialog box appear and so on and so on, and I’m like “WHAT THE HELL IS THAT? YOU’RE A PROGRAMMER MAN, USE THE KEYBOARD FOR GOD’S SAKE FOR DOING THESE KIND OF STUFF. DON’T GET INTERRUPTED LIKE THIS.” There are lots of reasons which interrupt us while we’re programming and one of the most popular is googling about some stuff(which is necessary and we can’t escape from that one!), so I think it’s reasonable that at least don’t let ourselves to be interrupted by these stupid things cause interruption make us less efficient at what we're doing.

Writing Katas make your muscles strong for programming and keep your mind in the training mode. It’s one of the best ways for practicing TDD which is the most important programming attitude that every person who wants to be a better programmer should have it.

Fortunately there are lots of resources you can use for writing Katas. Lots of screen casts by professional programmers and lots of [websites](http://codingdojo.org) for sample Katas are out there. You can see them freely and write programs with their authors simultaneously which can help you learn lots of stuff from professional programmers like how they write their micro tests(it’s a better name for unit tests or it’s a name for a good unit test) and when they stop and go to write production code and then back in writing tests again, What kinds of decisions they make in programming, how they name things, what’s their reaction in different situations and lots of other stuff.

We all should do that over and over till we can write programs so much more efficient, better and beautiful like them. This is the whole philosophy behind Kata(in martial arts) which is useful and compatible in programming too.

But there are some other points about writing Katas which we should pay more attention to them. One of them is writing a Kata in different languages. First let me clear something here. You may tell that why should I know more than one programming language at all?

I want to refer to a wonderful quote from Robert C Martin(aka Uncle Bob) at NDC09 in his keynote speech: “Know more than one language. There’s not enough to know about Java or C#. You should also try to program in more strange languages like Lisp or etc. If you think you’ll pickup and read a book about C# and you’re a programmer, you’re a fool!”

I think this is a wonderful and completely accurate belief. We should learn more than one language. Let me explain it little more. Going from C# to Java or vice versa is not what Uncle Bob meant. He meant learn a new language which has different style, thinking and issues. For example going from C# to Ruby or Lisp or Clojure or Perl or Python or etc. This is a change, because you have to think in different ways and program in different ways in these languages.

Learning new languages and write programs in them have lots of other benefits. One of the greatest is, when you try to learn a new language and write a program in it and you found yourself having problems, that’s maybe because you’re bringing your thinking ways from your previous language here. This is because you don’t conform one of the most important principles as a programmer: “PROGRAM INTO YOUR LANGUAGE, NOT IN IT”. If you don’t try to learn new languages and write programs in them you won’t be able to find out are you violating this important principle of programming or not?

So, as you can see, one of the most important rules about programming and for being a professional programmer shows itself in learning new languages and If you think about it more carefully this principle(Program Into Your Language, Not In It) has a hidden principle inside it which says: “Learn Different Languages and Write Programs In Them.”

I highly recommend you to read “Program Into Your Language, Not In It” chapter of [CODE COMPLETE](http://www.amazon.com/Code-Complete-Practical-Handbook-Construction/dp/0735619670/ref=sr_1_1?s=books&amp;ie=UTF8&amp;qid=1297185911&amp;sr=1-1) by SteveMcConnell. It has really interesting points.

If you’re still thinking you shouldn’t learn DIFFERENT languages you SHOULDN’T read the rest of this post.

Also you should write a same Kata in different languages. It’s so much interesting and also shows you in a comparing mode how different languages handles different situations in programming with what logic and concepts. It’s also teach you how to program into your language better.

For example I wrote Kata Stack in Java lots of times(I’m still writing it in Java cause it’s a KATA, don’t forget) but the first time I was writing the Kata Stack in Ruby I learned something interesting about Ruby and Java:

There’s one test case in Kata Stack which we describe when the pop from empty stack happened it should throw Under Flow Exception. In Java you simply write this micro test like the following:

{% highlight java %}
@Test(expected = UnderFlowException.class)
public void shouldThrowUnderFlowExceptionWhenPopFromEmptyStack()
{
    stack.pop();
}
{% endhighlight %}

The logic that Java uses for handling exception checking is having some keywords that you should use after @Test annotation. Then it runs the code in the test method and check that the production code it’s calling from test, throws the expected exception or not.

When I tried to write this test in the Kata Stack in Ruby for the very first time I found out a very interesting thing about Ruby which I didn’t know till then. Look at the equivalent test in Ruby:

{% highlight ruby %}
it "should throw under flow exception when pop from empty stack" do
    lambda { @stack.pop }.should raise_error(UnderFlowException)
end
{% endhighlight %}

So the mechanism Ruby uses for handling the same situation is totally different. It uses the concept of Lambda Expressions. In Ruby in a lambda expression you should specify that when some method is called or some operation is executed at a specified point of execution by micro test, then some kind of error should be raised. It’s absolutely fascinating. I personally love the way Ruby handles this situation. Or another example(imagine the capacity of stack is 2):

{% highlight ruby %}
it "should throw over flow exception when push to full stack" do
    lambda do
        @stack.push(1)
        @stack.push(2)
        @stack.push(3)
    end.should raise_error(OverFlowException)
end
{% endhighlight %}

It tells when you push for the **THIRD** time then the production code of push method should raise OverFlowException error cause the stack is full at **THAT POINT**.

Or, for checking the equality of two number collections(equality in this case means having same items in the same order), in Ruby you can simply write:

{% highlight ruby %}
numbers = [1, 2, 3]
numbers == [1, 2, 3]
{% endhighlight %}

But in java for checking the equality of two arrays or number collections you can’t do this:

{% highlight java %}
numbers = new int[] {1, 2, 3};
numbers == new int[] {1, 2, 3};
{% endhighlight %}

cause those arrays are reference types and the second line new int[] collection and numbers has different references so you have to write a method like the following in your test code which take expected and actual collections and check if they’re equal(with the same meaning as before):

{% highlight java %}
public void assertArrayEqual(int[] expected, int[] actual) {
    if (expected.length != actual.length)
        fail(&quot;size is not same!&quot;);

for (int i = 0;  i < expected.length; i++)
    if (expected[i] != actual[i])
        fail(String.valueOf(expected[i]) + &quot; is not equal to &quot; +
             String.valueOf(actual[i]));
}
{% endhighlight %}

And then using it like the following for the above example:

{% highlight java %}
assertArrayEqual(new int[] {1, 2,  3}, numbers);
{% endhighlight %}

So writing Katas in different languages help you to understand difference between languages in more practical ways and see power and weakness of each language in different actual cases.

## Conclusion

I really recommend you for being more efficient in a new language and also speed up your learning process, write different Katas that you wrote in your previous languages in it. It teaches you equivalent techniques in the new language in a very efficient and interesting way.

With this way you can learn how to handle similar situations(if there's any) in the new language with its new logic and concepts so much better and faster because of real practices(Katas) than if you try to learn and do the same thing in a normal way by only reading some books or other ways like that(I am absolutely not saying don’t read books, of course you should read them for learning new languages but write Katas in them too for better results).

Nothing teaches you better than examples and Katas are wonderful examples for TDD and programming.

It also makes you learn how to write your micro tests better and with better testing frameworks in the new language. I was using UnitTest built-in module for writing micro tests in Ruby till I watched Kata PrimeFactors in Ruby video from Uncle Bob. After that I’m using rspec for that purpose and it’s so much better and more readable and manageable.

Something that worth trying.

P.S there's another and simpler way for comparing collections in Java which curious reader can find out about it with a simple search.
