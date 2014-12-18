---
layout: post
title:  "Unit Tests should be for Units"
date:   2011-03-01 22:00:00
categories: programming refactoring
---

This is just a quick thing about DRY.

I think every person in the software development community know what is DRY. But I’m gonna explain its  essence in few sentences.

DRY stands for Don’t Repeat Yourself. Dave Thomas and Andy Hunt talked about it for the very first time in their Pragmatic Programmer book. You shouldn’t write even single line of code more than once in your application and also this is for logics in your software too (you shouldn’t repeat them). So DRY is for both lines of code and logics of your application. This rule is the base for the most Design Patterns and principles and practices in software development. It’s Even one of the main rules of XP(eXtreme Programming) which says **ONCE AND ONLY ONCE**.

I think we can conform this principle even in more aspects of software development and programming. You can conform DRY in your naming too. Martin Fowler in his [Refactoring](http://www.amazon.com/Refactoring-Improving-Design-Existing-Code/dp/0201485672/ref=sr_1_1?ie=UTF8&amp;qid=1298875768&amp;sr=8-1) book has a point about don’t have extra stuff in your method names which doesn’t give any useful information. For example:

{% highlight java %}
moveToPosition(position);
{% endhighlight %}

as you can see the position word in the method name is not giving you any useful information you can gain that information by the argument itself and there’s no need for this part in method name, so it’s better this way:

{% highlight java %}
moveTo(position);
{% endhighlight %}

Another case which I faced with it so much in people’s programs is negative names for Boolean functions. This only cause misunderstanding in the program. When you want to consider both negative and positive case of a Boolean function, name it in a positive manner. I saw this once:

{% highlight java %}
if (!notFound(file))
{% endhighlight %}

And further I saw this:

{% highlight java %}
if (notFound(file))
{% endhighlight %}

So in this case the programmer wanted both positive and negative cases, so it’s so much better and more readable if he/she has named it positive:

{% highlight java %}
if (found(file))
{% endhighlight %}

and then:

{% highlight java %}
if (!found(file))
{% endhighlight %}

Programming language designers can help programmers to conform this great principle(DRY) even more. The var keyword in C# helps to conform this principle in a very interesting way. Look at this line of Java code:

{% highlight java %}
FlashCardGame game = new FlashCardGame();
{% endhighlight %}

The equivalent of this line of code in C# can be:

{% highlight csharp %}
var game = new FlashCardGame();
{% endhighlight %}

I personally prefer latter one cause you don’t have to say what is the type of **game** variable twice. When you’re creating instance of FlashCardGame class and assign it to game variable you’re saying everything perfectly.

I really recommend to Java programming language designers to consider some equivalent for var in Java too cause it’s gonna help programmers to conform DRY even in another interesting aspect of programming.

Someone said to me once: “you know DRY? It’s really not only about line of code or logic in code. It’s about every possible aspect of the programming and code base, naming, combination of method name and its arguments, lines of code, logics and algorithms, variable definition and etc.”

Something that worth a try.
