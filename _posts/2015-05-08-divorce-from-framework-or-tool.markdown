---
layout: post
title:  "Divorce from Framework or Library"
date:   2015-05-08 21:20:00
categories: programming design storm framework testability
---

*Heads up: for the main point of this post, library and framework are used interchangeably even though those two are inherently different things.*

In our programs we leverage many different libraries and frameworks. We use them in order to address certain problems and challenges that we're facing in our system and already has been solved. This increases our productivity (coughing - in some cases) and prevent us from reinventing the wheel. Also it gives us the ability to dedicate the majority of our attention to our *specific problem at hand in the domain we're dealing with*.

That sounds great, but there is a potential downside and drawback to this approach as well. Using a library or a framework means using someone else's code and something that usually you don't have control over it or a full understanding of how it works. What are the nuts and bolts and life cycle of the events and pieces that are involved in that library or framework. And if you're using something that is not open source or not well documented then you're in an even bigger toruble. I know you're probably thinking: **Then don't use that library** and you're kind of right but in certain cases there is not much of a choice out there.

## Should NOT use Libraries & Frameworks?

Of course that's not what I'm advertising here. There is a way to balance this out and bring the best of both worlds together. Again you don't wanna solve a problem which has been solved already and slow yourself down. There are tons of libraries and frameworks out there which are super useful and invaluabe (e.g Authentication & Authorization libraries, Web Development Frameworks, Realtime Stream Processing Libraries/Frameworks, Data Analysis libraries, etc.)

The way to control the situation and get the best out of those foreign dependencies in your program is **Separation of Concerns** and getting your logic as far as possible from those dependencies. I like to think of it as **Core Logic** VS **Dependency/Foreign Shell**.

The library or the framework is **NOT** the logic of the application, that is not part of the solution for the main problem at hand. You need to separate those things. You solve the main problem at hand in the particular domain you're dealing with and then try to leverage the benefits of those libraries and frameworks in a manageable way which is totally under your control and isolated from the core logic. The moment you mix these things, you're creating a recipe for a disaster. it is gonna be hard to reason about your code, hard to write tests for it, subtle bugs crawl into the application from different and unrelated places cause you have a bowl of mess.

You do not have a full understanding about those foreign dependencies and what is happening underneath them. How the lifecycle of events work there and how much of a ripple effect a change can have if you mix things up. And a more common situation, when you update to the next version of those libraries or frameworks so many different places in your application need to be modified and many different tests break since you did not put them under control. You need to **tame** those foreign dependencies with different techniques like **Facade Pattern**, **Layering Design**, **Singe Responsible Blocks Composed Together**, etc.

## Thunderstorm

Recently I've been working with [**Apache Storm**](https://storm.apache.org/) a lot. And *storm* is basically a library/framework which allows you to do realtime stream processing on a **massive and high traffic stream of data**.

It has different pieces and features involved including:

  - Spout Components
  - Bolt Components
  - Ack & Fail mechanism
  - Replay-ability of Tuples which failed in the middle of the process
  - Fault Tolerance capabilities
  - Different mechanism of Stream joining, subscription, etc.
  
Those are all great and powerful features and great tools in your hand. But none of them has **ANYTHING** to do with the core logic of the problem at your hand or the way you want to process the stream of data that you're dealing with. (this is true in most cases but not all BTW)

Yet many codebases that I've seen working on stream processing and using storm, had their logic and analysis of the data baked into the components of **storm** (Spouts and Bolts). As a result, it's harder to reason about the logic, it's tightly coupled to the elements of the library/framework being used and it's really hard to write tests for them. For instance components of storm, *Spout* and *Bolt* have some life cycle methods which are being called internally by storm and you do NOT have control over them (e.g `nextTuple`, `execute`, `prepare`, `close`, etc.) and this makes it harder to test your core logic when it's coupled with those components and tied to them. 


## Taming the Storm


Imagin a very simple *Storm Topology* with one spout and two bolts each having three executors/threads. Something like the following diagram:

![Simple-Storm-Topology](https://dl.dropboxusercontent.com/u/100502983/divorce_from_lib/Simple-Storm-Topology.png)

The *Spout* is reading the data off of a [**Kafka**](http://kafka.apache.org/) stream and produce the tuples which are being emitted to *Bolt-1* and then *Bolt-1* does some processing and transformation on the tuples and emit the new tuples to *Bolt-2*. Finally that bolt do some further processing and new information generation and publish them back to the *Kafka* stream.

If you read that paragraph carefully you can see that the logic of the application which includes:

 - Tuples processing
 - TUples transformation
 - New information generation based on existing data
 
can be easily separated from the framework/library which is *Storm* in this case and has nothing to do with it. Have the logic isolated, fully tested, easy to reason about since you have full control and understanding about it. Then plug that logic into the framework or wrap the logic building blocks with thin shell of the framework/library. So the building blocks can look like the following:

{% highlight java %}
public class TupleTransfomer {

  public static Tuple transformTuple(Tuple tuple) {
    // some logic for transforming input tuple here
    return transformedTuple;
  }
  
  ...
}

public class TupleProcessor {

  public static Tuple process(Tuple tuple) {
    // some logic for processing the tuple here
    return processedTuple;
  }
  
  ...
}

public class InfoGenerator {

  public static Info generate(Tuple tuple) {
    // some logic for generating info
    return info;
  }
  
  ...
}
{% endhighlight %}

Then you can wrap those building blocks which have the core logic the application in sotrm components. For instance is somewhat how the *Bole-1* `exectue` method will look like:

{% highlight java %}
@Override
public void execute(Tuple tuple) {
  // some boiler plate of storm here
  Tuple tuple = TupleTransfomer.transform(tuple);
  // some boiler place of stomr here
  outputCollector.emit(tuple);
  outputCollector.ack(tuple);
}

...
{% endhighlight %}

This is a happy **divorce** between the core logic of your application and the framework/library you're dealing with. In this case the framework/library is *Storm* but it could be anything like **Rails**, **Django** or something else. If there is a third party library you're dealing with (e.g Payment Processing Library) use an isolation/separation technique like [**Facade Pattern**](http://en.wikipedia.org/wiki/Facade_pattern) and hide that library behind the walls and block any kind of propagation or ripple effects changes in that library might bring to your application logic. These are all interesting ways of increasing the quality of your desing and not intermingling libraries and frameworks with the core logic of your application and the solution to the problem at hand.

**Happy Hacking :)**
