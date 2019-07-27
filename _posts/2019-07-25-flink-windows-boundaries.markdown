---
layout: post
title:  Deep Dive Into Flink Windows Boundaries
date:   2019-07-25 18:00:00
categories: programming stream-processing flink
---

I've been trying to get a better and deeper understanding with regard to **window's boundaries** and **behavior** when it comes to *stateful stream processing* in [**Apache Flink**](https://flink.apache.org/). This led to a detailed and example-driven quest which did not leave me alone even during my sleep!

IMHO, one of the best ways to **really and deeply** understand a piece of functionality and verify that understanding, is writing **automated tests** for it. The reason being, traditional unit tests are extremely primitive and stupid; hence, you need to truly understand ins and outs of the target piece in order to properly lay out the ingredients of each test:

- Arrange
- Act
- Assert

Once you start writing such tests, you immediately realize that as usual, the devil is in the details and things can get a tad confusing. The purpose of this writing is to clarify some of those confusions and explain the relevant concepts in a more concrete and example-driven fashion.

In case you're interested, you can also find the entire code example in [this GitHub Gist](https://gist.github.com/Sam-Serpoosh/194068bd4e9fea9958bfae1cf618597b).

## Establishing Concepts, Terminology & Acronyms

Before getting into different scenarios and examples, let's establish some definitions and acronyms which will be used throughout this post.

Keep in mind that we'll be **entirely** dealing with **TimeCharacteristic.EventTime** in our execution environment here. This means that we're divorcing ourselves from **Processing Time** and everything is driven based on the **EVEN_TIME** of individual events arriving at the operator. For more information on **different kinds** of **times** you can be dealing with in a stream processing application, check out [this document](https://ci.apache.org/projects/flink/flink-docs-release-1.6/dev/event_time.html#event-time--processing-time--ingestion-time).

#### Terminology & Definitions

- [**Watermark**](https://ci.apache.org/projects/flink/flink-docs-release-1.6/dev/event_timestamps_watermarks.html): This determines the *advancement* of the stream your **operator** is consuming. In other words, Flink uses this concept in order to understand how much progress your operator has made with regard to the `EVENT_TIME`.
  - The **computation** of this value depends on a few different aspects of your operator's logic and its input stream. We'll get more into this later in our examples.
  - There are many good resources explaining this concept in more details. You can check out [this article](http://vishnuviswanath.com/flink_eventtime.html) to start with.
- [**Window**](https://ci.apache.org/projects/flink/flink-docs-stable/dev/stream/operators/windows.html): This allows you to hold on to multiple events as they arrive for **some period of time** and apply some logic to them as a group (e.g. **sum** them up).
  - There are different kinds of windows in stream processing and you can read all about them in [the official documents](https://ci.apache.org/projects/flink/flink-docs-stable/dev/stream/operators/windows.html) of Apache Flink.
  - In this post, we're mainly focusing on [**Tumbling Windows**](https://ci.apache.org/projects/flink/flink-docs-stable/dev/stream/operators/windows.html#tumbling-windows).
- [**Max Out Of Orderness**](https://ci.apache.org/projects/flink/flink-docs-release-1.6/dev/event_timestamp_extractors.html#assigners-allowing-a-fixed-amount-of-lateness): This can be used to **shift back** the watermark by a **fixed amount**. This also determines **when** the result of a window can be triggered/fired/emitted.
- [**Allowed Lateness**](https://ci.apache.org/projects/flink/flink-docs-stable/dev/stream/operators/windows.html#allowed-lateness): This is quite self explanatory and also determines how long a **window's state** should be kept before being **discarded**.

#### Acronyms & Symbols

Throughout this post I'm gonna use some acronyms and symbols, so let's quickly define and get them out of the way:

| Acronyms & Symbols | Meaning |
|:-:|:-:|
| CW | Current Watermark |
| EOW | End Of Window |
| MOO | Max Out of Orderness |
| AL | Allowed Lateness |
| [a, b) | Window: INCLUSIVE beg "a" & EXCLUSIVE end "b" |
| m | Minute (e.g. 2m means 2 minutes) |

## Current Watermark & The Behavior It Ensues

As mentioned before, the value of **CW** is computed according to some settings of the Flink operator and its logic. Imagine we have the **TumblingEventTimeWindows** of **[8:00, 8:05)** with size **5 minutes** (**EOW** would be **8:04:59:999**). Any **event** whose **timestamp** is in that range, belongs to this window and will be stored as such. Below you can find the role **CW** can play in a few important scenarios which we'll be dealing with in our examples later:

- **NO MOO & NO AL**
  - CW = max-event-time-seen-so-far
  - When `CW >= EOW`
      - Window's result is fired/triggered
      - Events belonging to this window will be **discarded**
- **With MOO**
  - CW = max-event-time-seen-so-far - MOO
  - When `CW >= EOW`
      - Window's result is fired/triggered
      - Events belonging to this window will be **discarded**
- **WITH AL**
  - CW = max-event-timestamp-seen-so-far
  - When `CW >= EOW + AL`
      - Window's result is fired/triggered **AND** the window is **discarded**
      - Events belonging to this window will be **discarded**

## Our Toy Example

For the purposes of this post, let's say we have a simple `Item` which has the following properties:

- `ts`
  - This represents our [**EVENT_TIME**](https://ci.apache.org/projects/flink/flink-docs-stable/dev/event_time.html) timestamp throughout all the examples; as opposed to `Processing Time` which we do **not** care about here. This is the timestamp that our [TimestampExtractor](https://ci.apache.org/projects/flink/flink-docs-release-1.6/dev/event_timestamp_extractors.html) will work with.
- `city`
  - The original **city** from which the item was shipped.
- `price`

A quick guideline on the images in the following sections:

- Yellow box means a **late** event which **won't** be **discarded** and still allocated to the window it belongs to
- Events that are further to the right on the **x-axis**/**processing time**, have arrived **later** than the ones on **their left**
- Red dot on the x-axis indicates the moment a window's result is **fired off**
- Yellow dot on the x-axis indicates the **arrival time** of a **late** event
- Finally, an event is represented as a box containing the following information:

| Row | Content |
|:-:|:-:|:-:|
| 1 | Event Time & The Window It Belongs To |
| 2 | EOW For That Window |
| 3 | CW Value |

### Case of Max Out Of Orderness & ZERO Allowed Lateness

Let's say we have the following simple **aggregation** logic in our *Operator*:

```java
final static Time WIN_SIZE = Time.minutes(5);

DataStream<Item> aggregateItem(DataStream<Item> items) {
  return items
    .assignTimestampsAndWatermarks(
      new WatermarkAssigner<>(MOO, Item::getTs)
    )
    .keyBy((KeySelector<Item, String>) Item::getCity)
    .window(TumblingEventTimeWindows.of(WIN_SIZE))
    .aggregate(new ItemAggregator())
    .uid(AGG_ID);
}
```

- [ItemAggregator](https://gist.github.com/Sam-Serpoosh/194068bd4e9fea9958bfae1cf618597b#file-tumblingeventtimewindowtest-java-L396-L439) is an implementation of `AggregateFunction<Item, Item, Item>` which knows how to aggregate *Item* instances/events
- [WatermarkAssigner](https://gist.github.com/Sam-Serpoosh/194068bd4e9fea9958bfae1cf618597b#file-tumblingeventtimewindowtest-java-L441-L457) is an implementation of `BoundedOutOfOrdernessTimestampExtractor<Item>`
- **MOO** determines the value for **max out of orderness** and is tweaked in different scenarios we'll be dealing with

#### MOO Is ZERO

<iframe align="center" src="https://drive.google.com/file/d/1S-F-7NuyHFiZdrlm4VQRzm4ANZYaXQeU/preview" width="736" height="197"></iframe>

At **t3**:

- **CW** becomes **8:06**
- Which makes it **>=** EOW of **[8:00, 8:05)**
- Hence, the result for **[8:00, 8:05)** containing **TWO elements** is fired off

#### MOO Is TWO Minutes

<iframe align="center" src="https://drive.google.com/file/d/1GluF3cINeXtIXfCT3PKBGXmHQWjZ-FWA/preview" width="756" height="177"></iframe>

At **t4**:

- **CW** becomes **8:06**
- Which makes it **>=** EOW of **[8:00, 8:05)**
- Hence, the result for **[8:00, 8:05)** containing **TWO elements** is fired off
- Key Differences:
  - In the **previous** scenario, the arrival of the event with timestamp **8:06** triggered the window's result since that made the **CW** to become **8:06** and **>= EOW** for **[8:00, 8:05)**
  - But in this scenario, due to **MOO** being **2m**, the value of **CW** gets shifted back **2 minutes**. Hence, the arrival of the event with timestmap **8:08** at **t4** advances the **CW** far enough (making it **8:06**) to trigger the result for our window.

### Case of Allowed Lateness & ZERO Max Out Of Orderness

Let's say we have the following simple **aggregation** logic in our *Operator*:

```java
final static Time WIN_SIZE = Time.minutes(5);

DataStream<Item> aggregateItem(DataStream<Item> items) {
  return items
    .assignTimestampsAndWatermarks(
      new WatermarkAssigner<>(MOO, Item::getTs)
    )
    .keyBy((KeySelector<Item, String>) Item::getCity)
    .window(TumblingEventTimeWindows.of(WIN_SIZE))
    .allowedLateness(AL)
    .aggregate(new ItemAggregator())
    .uid(AGG_ID);
}
```

- **AL** determines the value for **Allowed Lateness** and is tweaked in different scenarios we'll be dealing with

#### AL Is TWO Minutes

<iframe align="center" src="https://drive.google.com/file/d/1yEHRJwzsuug1iKowHNSCVGe9dKfMcelx/preview" width="760" height="188"></iframe>

At **t3**:

- **CW** has already been **8:06**
- Which is **>=** EOW of **[8:00, 8:05)**
- We have a **late arrival** whose event timestamp is **8:02**. You'd think it should be **discarded** given **CW >= EOW** of the window it belongs to, **[8:00, 8:05)**
- BUT, given our **AL** of **2 minutes**, we're practically **extending** our tolerance here by, well, **2 minutes**

At **t4**:

- **CW** becomes **8:08** due to the arrival of the event with timestamp **8:08**
- Now, **CW** has become **>=** to **EOW + 2m** which is **8:05:59:999**
- Hence, the window's result is fired off
- And from this point forward, any event whose event timestamp belongs to the window **[8:00, 8:05)** will be **discarded**

I'm going to leave the next natural test/scenario which is having **both** AL & MOO with **non-zero** values as an exercise to whoever is interested (e.g. MOO = **2m** & AL = **1m**)!

## A Word On Windows Boundaries

The key curiosity that motivated this whole quest, which now I've shared with you in excruciating details; was **how does Flink choose** the **windows' boundaries**?!

### Tumbling Event Time Window

Originally, I was under the impression that the boundaries for this type of window are determined based on the **arriving events' event timestamp**. But as you start to think about this more carefully, you'd see that it can result in a quagmire. Let's look at an example for a window of **size 5 minutes**:

- Event with **8:27** timestamp arrives which results in the window **[8:27, 8:32)**
- Event with **8:25** timestamp arrives and we have a few options here:
  - Open the window **[8:25, 8:30)**
      - This is **NOT** a tumbling window anymore due to the **overlap** which results in a contradiction **=> <=**
  - Open the window **[8:25, 8:27)**
      - This has the **wrong size** which also violates the tumbling window's definition and results in contraction **=> <=**
  - And any window-opening logic which **looks back in time** can result in quite a complex logic in Flink's state and window management

Hence, **Apache Flink** divides the [arrow of time](https://www.youtube.com/watch?v=GdTMuivYF30), **starting at Unix epoch** and based on the provided size of the **Tumbling Window**. And that's how we get boundaries like the following for the size **5 minutes**:
  
  - **[8:00, 8:05)**
  - **[8:45, 8:50)**
  - **[9:35, 9:40)**
  - Etc.

Needless to say, there are **no empty** windows! One of the rationales behind this design decision which is very reasonable is **SIMPLICITY**.

### Session Windows

In the case of **session windows**, event times drive the allocation of windows and their boundaries for the most part. But, **overlapping** windows get **merged** and the segregation between windows are determined by the configured **gap duration**.

## Acknowledgement

A **HUGE** thanks and a shout-out to [**Konstantin Knauf**](https://twitter.com/snntrable) who is a solution architect at [Ververica Data (Formerly Data Artisan)](https://twitter.com/VervericaData) and works on **Apache Flink**. He has been extremely helpful throughout this obsessive curiosity of mine and I cannot thank him enough.

After I banged my head against the wall for a while and couldn't get to the bottom of this, I posted [this question](https://stackoverflow.com/questions/57121018/flink-windows-boundaries-watermark-event-timestamp-processing-time) on **Stack Overflow** and pinged (most likely annoyed) folks on **Twitter**. Shortly afterwards, Konstantin responded to the question :)

<iframe align="center" src="https://drive.google.com/file/d/1XHDXGic2Cz-jkF1NI0JEZNdYOsU-xVsE/preview" width="640" height="588"></iframe>

Our conversation (a gist of which you can see below) continued on Twitter regarding some subtleties involved in these scenarios.

<iframe aligne="center" src="https://drive.google.com/file/d/1853sYQdz92kLOP3MwFBU3rCxqBs0Jf6N/preview" width="600" height="982"></iframe>

He also took the time to review the first draft of this post and I really appreciate that.