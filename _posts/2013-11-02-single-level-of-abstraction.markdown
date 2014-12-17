---
layout: post
title:  "Single Level of Abstraction (Don't mix things in the wrong place)"
date:   2013-11-02 08:00:00
categories: programming refactoring
---

This is going to be a short post on a very interesting and important programming concept that IMHO is really helpful and make a nice difference in the code! It’s about not mixing things that have different levels of abstraction in a method! The name of this technique is Single Level of Abstraction (not surprising really) by Kent Beck and it’s about not putting things that are in different levels of abstraction and details in the same method! It’s better to have same level of abstraction for the statements inside a method! It’s one of those somewhat fuzzy technique/principles in programming so I think an example would be good right now. For making the point lets imagine a VERY simple and almost-unreal piece of code! Lest say we store a matrix in a file and we want to compare two different matrices that are sitting in two files and check whether they’re equal or not. At one point, our compare method looks like the following:

{% highlight python %}
class MatrixFileComparer:
…
def compare(self):
  with open(self.file1) as f1:
    lines1 = f1.readlines()
  with open(self.file2) as f2:
    lines2 = f2.readlines()
  return self.compare_lines(lines1, lines2)
…
{% endhighlight $}

As I said it’s a very simple piece of code and obviously it reads lines of each file and compare those lines together one by one! If you pay attention to the content of compare method, it’s more like an integration and delegator point of the MatrixFileComparer class! That means it is delegating different responsibilities to appropriate methods inside/outside of this class each of which doing one thing and do it well, then integrates results/effects of them! Such a method in a class should not have any duplication and any low level task or implementation detail and also it’s mostly invoking/delegating to other methods as I mentioned before! But if you look at our compare method, it’s also doing a lower-level task which is opening a file and reading all of its lines via the file handler object! Right there you can notice the different levels of abstraction in this method! One task is reading the content of a file, which is obviously more fine-grained and lower-level than the other thing, which is JUST delegating to compare_lines method!

Now lets fix this mixture of abstraction levels and see what it looks like:

{% highlight python %}
…

def compare(self):
  lines1 = self.read_all_lines(self.file1)
  lines2 = self.read_all_lines(self.file2)
  return self.compare_lines(lines1, lines2)

def read_all_lines(self, file_name):
  with open(file_name) as f:
  return f.readlines()
…
{% endhighlight %}

Now if you look at the compare method, ALL it does is just method invocation and delegating the job to the appropriate method and getting its result in order to use in another step! The interesting thing is that all the steps in compare method are at the SAME LEVEL of ABSTRACTION now, unlike before that one step was doing something lower-level and the other step was doing something in a higher level of abstraction! It happened to eliminate the duplication that we had in our previous version of compare method as well but that’s just a bonus and it could have happened the other way around!

IMHO it’s a really interesting principle and helps to improve the code further! And the nice thing about it is that you don’t have to jump between different levels of abstraction and details in your mind while reading the code inside a method! Because when you have to do that and all of a sudden the method goes into more details and lower-level implementation tasks, it’s just a distraction from understanding WHAT that method is doing in each step and show you HOW it’s doing something in the middle of your higher level view! That’s a different level of abstraction and belongs to another method of the class!

Anyway, I should stop preaching now! I hope you find it interesting and helpful as well!

Happy Hacking!
