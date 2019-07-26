---
layout: post
title:  Dive Into Flink Windows Boundaries & Behavior
date:   2019-07-25 18:00:00
categories: programming stream-processing flink
---

I've been trying to get a better and deeper understanding with regard to **window's boundaries** when it comes to *stateful stream processing* in [**Apache Flink**](https://flink.apache.org/). This led to a detailed, concrete and example-driven quest which did not even leave me alone during my sleep!

IMHO, one of the best ways to **really and deeply** understand a piece of functionality and verify that understanding, is writing **automated tests** for it. Reason being, traditional unit tests are extremely primitive and stupid; hence, you need to truly understand ins and outs of the target piece in order to properly lay out the ingredients of each test:

- Arrange
- Act
- Assert

Once you start writing such tests, you immediatley realize that the devil is in the details and things can get a tad confusing.

### Establishing Concepts, Terminology & Acronyms

Before getting into different scenarios and examples, let's establish some definitions and acronyms which will be used throughout this post.

#### Terminolgy & Definitions

- [**Watermark**](https://ci.apache.org/projects/flink/flink-docs-release-1.6/dev/event_timestamps_watermarks.html): This determines the *advancement* of the stream your **operator** is consuming. In other words, Flink uses this concept in order to understand how much progress your operator has made with regard to the `EVENT_TIME`.
  - The **computation** of this value depends on a few different aspects of your operator's logic and its input stream. We'll get more into this later in our concrete examples.
  - There are many good resources explaining this concept in more details. You can check out [this article](http://vishnuviswanath.com/flink_eventtime.html) for instance.
- [**Window**](https://ci.apache.org/projects/flink/flink-docs-stable/dev/stream/operators/windows.html): This allows you to hold on to multiple events for **some period of time** and apply some logic to them as a group (e.g. **sum** them up).
  - There are different types of windows in stream processing and you can read all about them in [this document](https://ci.apache.org/projects/flink/flink-docs-stable/dev/stream/operators/windows.html).
  - In this post, we're mainly focusing on [**Tumbling Windows**](https://ci.apache.org/projects/flink/flink-docs-stable/dev/stream/operators/windows.html#tumbling-windows).
- [**Max Out Of Orderness**](https://ci.apache.org/projects/flink/flink-docs-release-1.6/dev/event_timestamp_extractors.html#assigners-allowing-a-fixed-amount-of-lateness): This can be used to **shift back** the `watermark` by a **fixed amount**. This also determines **when** the result of a `WINDOW` can be triggered/fired/emitted.
- [**Allowed Lateness**](https://ci.apache.org/projects/flink/flink-docs-stable/dev/stream/operators/windows.html#allowed-lateness): This is quite self explnatory and also determines how long a **window's state** should be kept before being **discarded**.

#### Acronyms & Symbols

Throughout this post I'm gonna use some acronyms and symbols, so let's quickly define them and get them out of the way:

| Acronum/Symbols | Meaning |
|:-:|:-:|
| CW | Current Watermark |
| EOW | End Of Window |
| MOO | Max Out of Orderness |
| AL | Allowed Lateness |
| [a, b) | Window: INCLUSIVE beg "a" & EXCLUSIVE end "b" |
| m | minute (e.g. 2m => 2 minutes) |

### Watarmark & The Behavior It Ensues

As mentioned before, the value of `CW` is computed according to some settings of the Flink operator and its logic. Imagine we have a window of `[8:00, 8:05)` which is the outcome of a `TumblingEventTimeWindows` with size **5 minutes** (`EOW` in this example would be `8:04:59:999`). Any `Item` whose `ts` is in that range, belongs to that window and will be stored as such. Below you can find the role `CW` can play in a few important scenarios which we'll be dealing with in our examples later:

- **NO MOO & NO AL**
  - `CW = max-event-time-seen-so-far`
  - When `CW >= EOW`
      - Window's result is fired/triggered
- **With MOO**
  - `CW = max-event-time-seen-so-far - MOO`
  - When `CW >= EOW`
      - Window's result is fired/triggered
- **WITH AL**
  - `CW = max-event-timestamp-seen-so-far`
  - When `CW >= EOW + AL`
      - Window's result is fired/triggered **AND** the window is **discarded**

### Our Toy Example

For the purposes of this post, let's say we have a simple `Item` which has the following properties:

- `ts`: This represents our [**EVENT_TIME**](https://ci.apache.org/projects/flink/flink-docs-stable/dev/event_time.html) timestamp throughout all the examples; as opposed to `Processing Time` which we do **not** care about here. This is the timestamp that our [TimestampExtractor](https://ci.apache.org/projects/flink/flink-docs-release-1.6/dev/event_timestamp_extractors.html) will work with.
- `city`: The original **city** from which the item was shipped.
- `price`

#### Case of Max Out Of Orderness & ZERO Allowed Lateness

We have this simple **aggregation** logic in *Operator*:

```java
final static Time WIN_SIZE = Time.minutes(5);

DataStream<Item> aggregateItem(DataStream<Item> items) {
  return items
    .assignTimestampsAndWatermarks(
      new WatermarkAssigner<>(MAX_OUT_OF_ORDERNESS, Item::getTs)
    )
    .keyBy((KeySelector<Item, String>) Item::getCity)
    .window(TumblingEventTimeWindows.of(WIN_SIZE))
    .aggregate(new ItemAggregator())
    .uid(AGG_ID);
}

// ItemAggregator is an implementation of AggregateFunction<Item, Item, Item>
// which knows how to aggregate Item instances/events

// WatermarkAssigner is an implementation of
// BoundedOutOfOrdernessTimestampExtractor<Item>

// MAX_OUT_OF_ORDERNESS is tweaked in different scenarios laid out below
```

