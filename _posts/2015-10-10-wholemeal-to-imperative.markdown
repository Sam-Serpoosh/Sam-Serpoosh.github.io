---
layout: post
title:  "Wholemeal to Imperative ONLY IF Necessary"
date:   2015-10-10 20:23:00
categories: programming design wholemeal imperative performance
---

Since 1950, in computer science and programming a general dichotomy was shaped. This dichotomy was regarding how to approach programming problems, what to sacrifice and what to favor first. One group was in favor of **performance**, also known as **Bottom-Up** and the other in favor of **abstraction** also known as **Top-Down**.

The `first` group would start with the hardware and machine language, then add abstraction to get closer to math, but **NEVER** at the expense of *performance*. The languages in that family are famous for being super fast like **C**, **C++**, **Java**, etc. But they have the downside of getting the programmer too involved in details and you have to specify everything in an imperative way. For instance to transform a collection, you have to loop over **every single** element, apply the transformation and collect it back up.

The `second` group would start with mathematics and *substaract* abstraction to get down to the machine, sometimes at the expense of *performance*. The languages in that category are being blamed for being too slow sometimes. But they have nice features like the ability to confirm to [wholemeal programming](http://stackoverflow.com/questions/6957270/what-is-wholemeal-in-functional-programming?answertab=active#tab-top) approach and not getting involved in details. For instance, you transform the entire collection by `map`-ing the transformation function on the collection, and the language will take care of the details for you.

**Brian Beckman** explains this dichotomy nicely in his [Don't Fear the Monad](https://youtu.be/ZhuHCtR3xq8?t=3720) talk (the link gets you to **that part** of the talk).

Both schools of thought have valid points and like anything else in the world of computer science, there is a trade off involved. I had a professor in college who always said: "**The world of Computer Science ... The world of Trade Offs**" and he never used a verb in that sentence :)

But personally, I kind of like the latter approach better. Starting with **wholemeal** and reducing **abstraction** ONLY if you absolutely need the performance improvements it brings. Because you're paying the cost of more *complicated* and *harder to maintain* codebase at that point.

Let's experiment this with a more concrete example rather than just abstract talk and preaching. 

*I'm aware that there are better implementations for all the sample codes shown below and feel free to mention them in the comments, I enjoy checking them out for sure. But keep in mind that these samples are written in this way in order to be aligned with the point I'm trying to make in this post. Also I'm using intermediate variables obsessively cause this is a blog post :)*

## Perfect Squares Problem

Imagine you wanna write a piece of code that takes two *Integer* numbers and count how many **perfect squares** are within that range. We start with the pure functional and wholemeal approach, then we reduce abstractions continuously until we achieve our desired performance. Just for fun and some obligations, we'll write this in 3 different steps and each step in a different language!

### Pure Functional & Wholemeal

In this approach, we solve the problem by following these steps:

- transform numbers to `True` or `False` representing whether a number is perfect square
- count how many `True` exists in the outcome

Here's the **Haskell** implementation:

{% highlight Haskell %}
findSquaresCount :: Int -> Int -> Int
findSquaresCount low up = let isSquares = map isSquare [low..up]
                          in length . filter id $ isSquares

isSquare :: Int -> Bool
isSquare num = let root    = sqrt (fromIntegral num :: Double)
                   floored = floor root
                   diff    = root - (fromIntegral floored :: Double)
               in diff == 0.0
{% endhighlight %}

#### Pros

- **succinct**
- **easy to understand**
- **easy to maintain**
- does not get involved in **details of collection handling**

#### Cons

- **VERY SLOW** in case of huge ranges (e.g 100 million numbers)
- Using **extra space** (maintaining `True/False` List)


### More Imperative and More Efficient

In this approach, we solve the problem by following these steps:

- Iterate over the range of numbers
- check each number for being a perfect square or not
- increment the `perfect square count` if it is

Here's the **Ruby** implementation:

{% highlight Ruby %}
def count_perfect_squares(low, up)
  perf_sq_count = 0
  (low..up).each { |num| perf_sq_count += 1 if square?(num) }
  perf_sq_count
end

def square?(num)
  root    = Math.sqrt(num)
  floored = root.floor
  diff    = root - floored
  diff == 0.0
end
{% endhighlight %}

#### Pros

- easy to understand
- easy to maintain
- **NO extra** space required

#### Cons

- Involved in collection *iteration* --> **less abstract**
- **SLOW** in case of huge ranges (it's going over EVERY single number to check)

### Most Efficient and Least Abstract

In this approach, we take leverage of a mathematical relation between consecutive perfect squares:

{% highlight %}
(n + 1)^2 - n^2 = 2 * n + 1
{% endhighlight %}

We solve the problem by following these steps:

- Find the first perfect square within the range
- From that point on, get the next perfect square by adding `2 * n + 1` to the current one
- until it gets out of the range and in the meantime increment the count

Here's the **C** implementation:


%{ highlight C %}
typedef struct {
  int square;
  int root;
} Square;

int count_squares(int, int);
Square find_first_square(int);

int count_squares(int low, int up) {
  Square first_square = find_first_square(low);
  if (first_square.square > up) return 0;
  if (first_square.square == up) return 1;
  
  int root           = first_square.root;
  int current_square = first_square.square;
  int squares_count  = 0;
  while (current_square <= up) {
    squares_count++;
    current_square += 2 * root + 1;
    root++;
  }

  return squares_count;
}

Square find_first_square(int low) {
  int root = (int) sqrt(low);
  if (root * root < low)
    root++;

  Square sq = { root * root, root };
  return sq;
}
{% endhighlight %}

#### Pros

- **extremely fast**
  - NOT iterating over every item and jumping
  - Use simple `addition` over more expensive `sqrt` check
- **NO extra** space required

#### Cons

- **harder** to read
- **more work** to maintain
- VERY involved in **details** of collection handling
- **least abstract** solution


### What's the Point?

As you noticed, we started with the most *abstract* and *wholemeal* solution which was elegant, easy to maintain and understand. But we **NEEDED** to improve the `execution time` dramatically and reduce the `amount of space` required by this solution. That was a **REQUIREMENT** in this case for us. So we had to start removing layers of abstraction **one at a time** until we get to our desired performance and space usage.

So, if you **absolutely need** the performance boost you achieve, it's worth reducing the *abstraction* at the cost of damaging readability and maintainability to a certain degree. But if that's **NOT** a requirement, then it's not worth it and it's a premature optimization which might **NEVER** payoff. 

This can be a terrible investment with no return. Because you damaged the elegancy of your program in the hope that the performance improvement will be needed in the future. But that future in A LOT of cases never come.

As I said at the beginning, I personally like the second school of thought better. But like anything else in programming, there is no such thing as **silver bullet** and both approaches have their *pros* and *cons*. You need to see what fits your *circumstances*. You probably have a good understanding up-front about some of your performance constraints and start somewhere lower in the spectrum. The key is to find the appropriate starting point for yourself in that spectrum.

![Imperative-to-Wholemeal Spectrum](https://dl.dropboxusercontent.com/u/100502983/wholemeal_to_imperative_blog/imperative_wholemeal_spectrum.png)

And that's it! Happy Hacking :)
