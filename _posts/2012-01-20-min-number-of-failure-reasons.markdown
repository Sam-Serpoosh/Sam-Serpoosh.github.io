---
layout: post
title:  "Minimum NUmber of Failure Reasons"
date:   2012-01-20 21:00:00
categories: programming unit-test
---

Few weeks earlier one of my friends told me that he wrote a project for his File Structure course and likes to write unit-tests for it and put under high test coverage then Refactor it and makes its code as clean as possible. I was thrilled because this is one of my greatest hobbies and I like doing stuff like that.

Two days after that we sat together in a 2 hours session (I know it’s a long session and we should took a break after each 25 or 50 minutes according to Pomodoro technique but we were absorbed with the process an couldn’t help it) and wrote some tests and did a lot of Refactoring.

According to Michael Feathers [Working Effectively with Legacy Code](http://www.amazon.com/Working-Effectively-Legacy-Michael-Feathers/dp/0131177052/ref=sr_1_1?s=books&amp;ie=UTF8&amp;qid=1327066733&amp;sr=1-1) when you’re trying to Refactor a legacy code which is also not testable most of the times you shouldn’t write tests specifically for each line of code (and most of the times you can’t), you should write a test for a working package (don’t confuse this with Java package) which you know what are its input and output and then try to Refactor it until it become clean and testable and then you can write more detailed tests for different parts of it. That was exactly what we were doing in that project.

At a certain moment when we wanted to test the process of reading records with a specific format from a file, at first we wanted to use the write method of the production code for writing some records to the file and then read records from it and check the reading process. At this time it hit me that if we do something like that we’re violating the “Minimum Number of Failure Reasons” principle which says **each unit-test should fail because of ONE and only ONE reason**. It’s obvious that in this kind of test we were having at least two units of work a) writing records to file b) reading records from file. As a result when this test fails, there can be at least THREE different reasons for it. First of all it can be reading process, second of all it can be writing process and at last the interaction between those two. That unit-test should only check the reading process and if it fails it should only mean reading process has a problem which should be handled and nothing more than that.

For doing this we can have a separate unit-test for the interaction between writing and reading process and also a separate unit-test for the writing process. When those tests pass it means writing unit and the interaction unit work perfectly with no problems. At this moment we are sure about them. So now we write the unit test for reading unit and when it fails it only means that the reading process has a problem because we’ve already assured that writing unit and interaction unit work perfectly.

## Conclusion

In the process of unit-testing one of the most important things that we should be careful of is the principle of Minimum Number of Failure Reasons. Each unit-test should fail for ONE and only ONE reason. You should try to minimize the number of reasons that a unit-test fails because of them. When a test fails and you have two, three or more scenarios that could be the reason of failure you’re violating this principle. So when the number of reasons or scenarios in a failing test is ONE, it’s a good sign and you’ll have certainty about the reason. Don’t create a situation in which failure can have multiple reasons. A unit-test should only check one unit of work and break for only one reason because it is UNIT test for God’s sake.
