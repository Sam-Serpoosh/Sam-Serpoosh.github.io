---
layout: post
title:  "Terseness VS Elegance"
date:   2015-08-08 19:11:00
categories: haskell functional programming
---

One of the **factors** used by many programmers in the analysis of a program from *quality*, *design* and *understandability* perspective, is `LOC (Lines Of Code)` and sometimes that's taken to another level - specially in the functional programming world and the analysis of each function individually - to `NOW (Number Of Words)`!

I understand those metrics and they make sense to a certain degree for sure. One of the many cool things that you experience when you re-write an *imperative* program in the *functional* way, is the massive reduction in the number of lines. And we owe that to the **WholeMeal Programming** approach which is the essence of functional programming. Solving a problem in a general way instead of getting caught in the middle of quite irrelevant details. The best and most famous example of it I guess is **Iteration** VS **Map**. Something like:

{% highlight java %}
public List<Integer> squareNumbers(List<Integer> numbers) {
  List<Integer> squaredNums = new ArrayList<>();
  for (Integer num : numbers) {
    int squared = Math.pow(num, 2);
    squaredNums.add(squared);
  }
  return squaredNums;
}
{% endhighlight %}

Can be written in `Haskell` as following:

{% highlight haskell %}
squareNumbers :: [Int] -> [Int]
squareNumbers = map (\num -> num * num)
{% endhighlight %}

That's a huge win and you don't need to get into shenanigans of iteration over a collection. This particular operation is so common in programming that **map** is almost part of ANY programming language these days. (even Java 8 added that and it's been in Google Guava library for quite some time)

So far so good. But sometimes having less number of lines and words does **NOT** necessarily mean you have a better and more **elegant** code. We'll demonstrate that with a very simple example in `Haskell`.

## Payment Sample

Imagine you are writing a simple monthly payment tracking application. Obviously in the design of such an application one of the data-types would **Payment**:

{% highlight haskell %}
data Payment = Payment { value :: Double,
                         ... -- other properties like category etc.
                       } deriving (Show, Eq)
{% endhighlight %}

Also a common feature/capability of this application should be giving you the **total amount** of money you've paid so far. That can be done with a function like the following:

{% highlight haskell %}
totalPayment :: [Payment] -> Double
totalPayment = sum . map value
{% endhighlight %}

That's a very terse and succinct implementation for that function! Thanks to cool features of *Haskell*, spceially in this case the `math-equation-like-parameter-elimination` which even allows us to get rid of `payments` argument from both sides of the function equation.

*NOTE* that the same thing would be more verbose even in terse languages like Ruby or Python, let alone a language like Java.

## Abstraction is Missing

When I look at that code, it feels like something is missing. The understanding of the data type **Payment** is not complete in that implementation IMHO. One of the basic operations that you need to be able to perform on multiple *Payments* in this application, is adding them together. So a better solution is for the data type to provide that capability instead of us reaching into the belly of a Payment and grab what we need in order to perform that operation. That seems like a much more elegant design and doesn't smell like the inside of a Payment's stomach at all cause everything would be handled by the **Payment** and we'll get back what we need instead of opening something up and mess around with what's inside of it.

### Monoid to the Rescue

If you're famiiar with [**Monoid**](https://wiki.haskell.org/Monoid) typeclass in Haskell, you would immeidately realize that it's a perfect match for such requirements. So we need to make the Payment data type an instance of a Monoid:

{% highlight haskell %}
import Data.Monoid

instance Monoid Payment where
  mempty = zeroPayment -- a payment with value set to ZERO
  Payment { value = v1 } `mappend` Payment { value = v2 } = Payment { value = v1 + v2, ... }
{% endhighlight %}

You can read about *Monoids* in detail but the main thing you need to know about those functions implemented above are:

- *mappend* is an ASSOCIATIVE BINARY operation regarding the data type
- *mempty* is an identity value regarding the *mappend* (e.g 0 in mathematical ADD)

Now that *Payment* is a *Monoid* we can re-write our little `totalPayment` function as following:

{% highlight haskell %}
totalPayment :: [Payment] -> Double
totalPayment = value . foldr mappend zeroPayment
{% endhighlight %}

It's still pretty terse (one WORD more than previous version if that metric is important to you). BUT, IMHO this version is much more elegant and abstracts away a detail which was involved in the previous implementation. Before we were reaching into EVERY single Payment and got its value and then sum them up. Now we're ADDing Payments themselves and they know how to take care of that internally and finally we just grab the **value** out of the result payment.

## Elegance over Terseness

If you can improve your design and your codebase with a nice abstraction like the one we did in our example, by all means go ahead and take care of it. We added few more lines of code (instantiation of a Monoid) and more words (our function definition) but at the end of the day we ended up with a more **elegant, abstract and understandable** design and codebase. And that is much more valuable than the number of lines and words. So if you have to trade-off between `Terseness` and `Elegance` I'm sure you know that you should favor `Elegance` **EVERY SINGLE** time!

Ok, I stop preaching at this point and I hope that was interesting. Happy Hacking :)
