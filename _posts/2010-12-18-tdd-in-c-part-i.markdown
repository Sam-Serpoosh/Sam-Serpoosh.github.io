---
layout: post
title:  "TDD in C Part I"
date:   2010-12-18 23:00:00
categories: programming tdd
---

A few days ago one of my friends told me: “do you think we can do TDD in C?” at first I thought this is kind of stupid cause TDD makes sense for Object Oriented programming and design so for a non-OO language like C it doesn’t make any sense. (which is NOT the case at all BTW :D)

One day after that my friend told me again: “I Googled about it a little and found something. Apparently we can do TDD in C”. You know that I’m a TDD geek so it was really interesting topic for me. When I got home I started to search about that issue and found some useful things on the Internet.

We can do TDD in C with “CppUnitLite”. Actually CppUnitLite is written for C++ as you can figure it out from its name. But you all know that we can call C code from C++ with a little trick which we’ll see later in this post series.

Although the first step for doing TDD in C is integrating CppUnitLite with your IDE. I’m using Dev-C++ as an IDE for writing C and C++ codes, so I tried to do this with Dev-C++. After a lot of searching I didn’t find a very useful and simple guide for doing this, so I tried and struggled so much with little things that I found in Internet(sporadic things here and there) and finally did it.

Then I decided to write about it here in the first part of this series, so if there will be someone like me interested in the topic can find useful things for doing this integration in one place in a simple manner.

You can download CppUnitLite [here](http://objectmentor.com/resources/downloads.html). After you did this for integrating it with your Dev-C++ IDE you can do the following steps:

1. After you extracted the Zip file you downloaded, there are some folders and some files(.h and .cpp files which are for the CppUnitLite test modules that is written in C++).

2. Launch your Dev-C++ IDE and then create a new project from File->New->Project path.

3. Choose static library as the type of your project and name it CppUnitLite:

<iframe src="https://drive.google.com/file/d/1_YxhBDAMUq3G4w0KZDPsNo-VfZz2cxWp/preview" width="640" height="480"></iframe>

4. Then add to your project files of CppUnitLite like the below pictures:

<iframe src="https://drive.google.com/file/d/1Ju6kDXVLx2JJ8lZ4kbRrKDNzDoNG3LLr/preview" width="640" height="480"></iframe>

<iframe src="https://drive.google.com/file/d/1L3TdRCdhBf28JYIRCh6ak1wiCEOuwnVk/preview" width="640" height="480"></iframe>

5. And finally Compile this project.

Ok first step was completed. You build a library for CppUnitLite on your system. If you go to the path which you saved this project, there’s a file there with name CppUnitLite.a, this is your **CppUnitLite** library that you can use in other projects for writing tests.

Now we want to write our first C project with TDD approach. Before I start that I should mention that when you download CppUnitLite from ObjectMentor website and extract that Zip file, there’s a sample Stack project which is written in C++ and in OO way that you can use for learning TDD in C++. So I’m gonna write Stack in C and non-OO way(I know it’s weird but it’s **just a SAMPLE** for showing you guys how we can do TDD in a non-OO language like C).

Ok, launch your Dev-C++ IDE again and create a new project and this time choose Empty Project as your project type and call it Stack. Now we should make this project depends on the library we just created. For doing this right click on your project name(top-left of the IDE, called Stack in the project tab) and choose Project Options, then add your library(CppUnitLite.a) to this project like the following picture:

<iframe src="https://drive.google.com/file/d/1q6KFWIk0YwUX-S5-p4V6gcH01WcQ9oSG/preview" width="640" height="480"></iframe>

Ok now that this project depends on the CppUnitLite library we can use this library’s modules in our project. So we add to our project the first file and save it as StackMain.cpp. Then type the followings in that file:

{% highlight cpp %}
#include "..\CppUnitLite\TestHarness.h"
//here is the path of TestHarness.h in your system
#include <stdlib.h>

int main()
{
  TestResult testResult;
  TestRegistry::runAllTests(testResult);
  system(&quot;pause&quot;);
  return 0;
}
{% endhighlight %}
 
Ok, now after you compile and run this project you should get the following result:

<iframe src="https://drive.google.com/file/d/1qPHZrNjK1n_ITzaQJIPRS5hSi-KA9Cp0/preview" width="640" height="480"></iframe>

Of course there were no test failures cause we didn’t write any test yet but we integrated CppUnitLite to our project and it works like a charm now. It’s completely ready for us to write our tests and code for the Stack project.

*Until the next post in this series ...*
