---
layout: post
title:  "Pick the Right Tool for the Task at hand"
date:   2014-02-14 10:30:00
categories: programming ruby
---

We’ve been told many times in our life that each tool is useful for certain tasks and using a tool for something unrelated or even somewhat related is not a very smart decision. For instance you do *NOT* use a hammer in order to cut a piece of wood! Maybe after a lot of struggling you will be successful (breaking it from a certain place you wanted maybe! :) ) but when you say it out loud (like I did now) it seems even more ridiculous! But sometimes it is not as irrelevant as the hammer situation. You can achieve what you’re looking for or you can get to where you want, but with extra effort and more time, cost and energy that is necessary which can be avoided! Imagine you want to get from point A to point B in the following picture:

![Point A -> B](https://dl.dropboxusercontent.com/u/100502983/right_tools_blog_pics/point_a_to_b.png)

There are different ways to do that. One is the blue path and the other is the red path (and many more). Both are doing the job and get you to the destination but I’m sure everyone agrees that blue path is better and more effective (from time, cost, energy, etc. perspectives). It’s kind of the same concept as using the appropriate tool in order to do the task at hand. When you do not use the appropriate tool, you’re taking the red path, probably you would get to your destination but with much more struggling that is necessary! On my machine, I have a directory which I store the episodes of a show I’m watching in and each episode files are stored in a subdirectory named in the form of a four-digit number ####! For some weird reasons I need to know what are the last 3 episodes of the show that I have. There are two different tools that I can use in order to get this small task done. One is a simple program in whatever language I want. Lets say **Ruby** as an example and the code can be something like the following:

{% highlight ruby %}
class LatestEpisodesFetcher
 def initialize(entries)
  @entries = entries
 end

 def fetch
  return [] if @entries.empty?
  last_three = @entries.map(&:to_i).sort[-3..-1]
  last_three.select { |num| num != 0 }
 end
end
{% endhighlight %}

And the other is a simple **Bash** script which can look like the following:

{% highlight bash %}

#!/bin/bash

set -e
ls | grep -v "whatever you don't want" | 
     sort -n | 
     tail -3 | 
     xargs

{% endhighlight %}

It’s super clear which is the red path and which one is the blue path. You can tell that EVEN by just looking at them and not more detailed technical reasons (e.g Bash is taking advantage of some simple, very efficient and super-fast UNIX commands for working with file system and its structure directly but in case of the Ruby code in best case they are some thin wrappers around the appropriate system calls and some other data processing on the results) Of course you can write the Ruby code in a more succinct way and even probably a one liner in more idiomatic Ruby style but the code I put here is showing my point more explicitly, so I’m gonna leave it that way. **UNIX** commands are pretty great for doing fast and quick analysis on files and directories and you can extract some interesting information (not just the weird one I showed here) very efficiently in order to get some perspective. And the great philosophy of UNIX -- Everything is doing ONLY one thing and does it WELL! – gives you the ability to mix/compose all these small programs to achieve great stuff. The point I’m trying to make is pretty simple: **pick the right tool for the task at hand**! 

To be honest it did not need me to preach this much about it and bore you! So, sorry for that and happy hacking :)

