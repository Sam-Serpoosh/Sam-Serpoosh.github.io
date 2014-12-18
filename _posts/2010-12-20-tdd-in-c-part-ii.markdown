---
layout: post
title:  "TDD in C Part II"
date:   2010-12-20 21:00:00
categories: programming tdd
---

In the previous part we saw how we can create CppUnitLite library for using in other projects and also how we can make our projects to depend on that library and use. Now it’s time for writing a very simple Stack project with TDD approach in C which is a non-OO language(Odd) as a first practice.

Before I start to write about the codes, I just wanna say if the indentation in codes have problems or layout of them is not right it's because of wordpress problems in handling these kind of stuff in both HTML and Visual style, sorry about that anyway.

I just want to remind the StackMain.cpp file for the project which we wrote in the previous part. This is how your StackMain.cpp file should look like before you write any test:

{% highlight cpp %}
#include "C:\software\CppUnitLite\om\CppUnitLite\TestHarness.h"
#include
int main()
{
  TestResult tr;
  TestRegistry::runAllTests(tr);
  system("pause");
  return 0;
}
{% endhighlight %}

When you compile and run your project you’ll get the the result which says: “There were no test failures” which is obvious cause there are no tests yet.

Ok, now it’s time for writing our first test case for the Stack. We include Stack.h file here(which doesn’t exist yet). At first we want to handle size of the stack, so here’s our first test:

{% highlight cpp %}
TEST(Stack, sizeShouldBeEmptyAtCreation)
{
  LONGS_EQUAL(0, getSize());
}
{% endhighlight %}

In front of TEST keyword we have two arguments. Both of them are names. First is a GroupName which determine the name of a group that our tests belong to(it’s just a name, no confusion) and the second one is the name of test which should be as expressive as possible.

Of course it doesn’t compile because there is no getSize() method in there yet, so we make it compile by adding Stack.h file to the project and write the followings in it:

{% highlight cpp %}
#ifdef __cplusplus
extern "C"
{
  #endif
  int getSize();
  #ifdef __cplusplus
}
#endif
{% endhighlight %}

This block is the magic! Any code which is written in this block should be in C and we can call it from C++ with this trick(extern “C”). After that, we should add Stack.c file to the project and write the followings in it for making the project compile:

{% highlight cpp %}
#include "Stack.h"
int getSize()
{
  return -1;
}
{% endhighlight %}

Now it compiles but sure it will fail because we expect 0 but it’s **-1**. It’s so easy to make this pass:

{% highlight cpp %}
int getSize()
{
  return 0;
}
{% endhighlight %}

It’s time for the next test case:

{% highlight cpp %}
TEST(Stack, sizeShouldBeOneAfterOnePush)
{
  push(4);
  LONGS_EQUAL(1, getSize());
}
{% endhighlight %}

Ok for making this pass first of all we determine push method in Stack.h and Stack.c files for making it compile and then write enough code to make the test pass so it’ll be like:

{% highlight cpp %}
static int itsSize = 0;
void push(int element);
int getSize();
{% endhighlight %}

And in the Stack.c will have:

{% highlight cpp %}
void push(int element)
{
  ++itsSize;
}
{% endhighlight %}

The interesting point is here. When you run your tests in some platforms(like mine) you’ll get failure message at this point(in others maybe later) and size is not what you expected. Here’s why, your tests run in no predictable order, so you can’t rely on them in this way. Maybe the second test runs first which increase the itsSize variable and then the first one runs which will make failure cause itsSize is not “Zero” at that time!

When you do TDD in a non-OO language like C sometimes you need to handle some concepts of OO programming. For example initialization which handled by Constructors in those languages. At here we need some kind of initialization that set the itsSize variable to Zero anytime we want to use Stack functionality in a context(like our tests here). So we write this function and we call it at the top of each unit test we write:

{% highlight cpp %}
void initialize();
{% endhighlight %}

And in the Stack.c:

{% highlight cpp %}
void initialize()
{
  itsSize = 0;
}
{% endhighlight %}

Now everything is fine and all the tests pass. So we go to the next test case:

{% highlight cpp %}
TEST(Stack, sizeShouldBeZeroAfterOnePushAndPop)
{
  initialize();
  push(3);
  pop();
  LONGS_EQUAL(0, getSize());
}
{% endhighlight %}

We do the following in Stack.h:

{% highlight cpp %}
void pop();
{% endhighlight %}

And then in Stack.c:

{% highlight cpp %}
void pop()
{
  --itsSize;
}
{% endhighlight %}

You think this is stupid, cause we all know that pop method in stack should return an element, so what’s with the void in it’s prototype? I know, but we don’t have a test for this yet so we shouldn’t write production code for that. Remember the Third Rule Of TDD by Robert C.Martin(Uncle Bob): “we should write enough code to make a failing test pass, more than that is just wasting time.” Ok now that everything is fine and all tests pass we should move on and go to the next test case. I think we handle all the scenarios for size of the Stack so here’s our next test case:

{% highlight cpp %}
TEST(Stack, shouldPopOneAfterPushingOne)
{
  initialize();
  push(1);
  LONGS_EQUAL(1, pop());
}
{% endhighlight %}

For making it pass we can do the following:

{% highlight cpp %}
int pop()
{
  --itsSize;
  return 1;
}
{% endhighlight %}

Don’t forget to change the prototype of pop method in the Stack.h file too. Now it passed but it’s hard coded(return1;) we should make it more general according to this rule of TDD by Uncle Bob: “as tests become more specific, code becomes more general.” So here’s the next test:

{% highlight cpp %}
TEST(Stack, shouldPopTwoAfterPushingTwo)
{
  initialize();
  push(2);
  LONGS_EQUAL(2, pop());
}
{% endhighlight %}

We make it pass so easily like this:

{% highlight cpp %}
static int itsElement;
{% endhighlight %}

And then in Stack.c:

{% highlight cpp %}
void push(int element)
{
  ++itsSize;
  itsElement = element;
}

int pop()
{
  --itsSize;
  return itsElement;
}
{% endhighlight %}

Everything is fine now. It’s time for the last test case which make our Stack completely general with all of Stack’s functionality:

{% highlight cpp %}
TEST(Stack, shouldPopInReverseOrderAfterMultiplePushes)
{
  initialize();
  push(1);
  push(2);
  LONGS_EQUAL(2, pop());
  LONGS_EQUAL(1, pop());
}
{% endhighlight %}

For making it pass we’ll do this:

{% highlight cpp %}
#define MAXSIZE 100
static int itsElements[MAXSIZE];
{% endhighlight %}

And then in the Stack.c we’ll make following changes for making the new test pass while keep previous ones passing:

{% highlight cpp %}
void push(int element)
{
  itsElements[itsSize++] = element;
}

int pop()
{
    return itsElements[--itsSize];
}
{% endhighlight %}

At this moment all tests pass without any problem and our Stack has the full functionality of the stack data structure.

One invaluable thing for me about this experience, was when you counter a problem which is odd or strange or it’s odd or strange from your point of view, don’t say NO so quick and don’t give up, try to figure out is there any way for handling that situation or not and then go for it.

That’s almost what I did for this problem and it ended up with learning lots of interesting things. So this was how we can do TDD In C with a very simple Stack example. I hope you enjoyed this series.
