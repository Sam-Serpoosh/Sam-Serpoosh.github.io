---
layout: post
title:  "Details are Evil"
date:   2012-06-25 17:00:00
categories: programming rails cucumber test
---

Recently one of my friends asked me to help him for refactoring and improving the design of his rails application and its unit and integration tests. It was a really great experience because working on other people’s codes always teaches you great stuff. You get to see how someone with different point of view and different skills writes a piece of code and solves a problem. I personally learned a lot from reading other’s codes and watching people writing code and explaining their thoughts while they were doing it. I believe it was Picasso who once said: “the best thing that inspires me is watching other painters’ works.” If you want to see how important this is and how it can affect you, I recommend you to watch [this great talk by Geoffrey Grosenbach](http://av.vimeo.com/62506/261/102206764.mp4?aksessionid=1e645d48aca263133ef8a1cf47aa2583&amp;token=1339523148_9dca61edb9d37a2757c36902a595f9c5) about watching people code.

My friend told me one of the nightmares in the application was maintenance of the scenarios. Apparently they were too fragile and whenever they changed something in the application code they had to change the scenarios accordingly. And because of the fact that those scenarios were the live documentation of the application (which they should be) this frequency of changing them was driving them crazy. So I was reading some of the cucumber scenarios and I saw the following in the signup feature of the application:

{% highlight ruby %}
Given I am on the homepage
When I follow “Sign up”
And I fill in “Email” with “john@exmaple.com”
And I fill in “Password” with “foobar”
And I fill in “Confirmation” with “foobar “
And I press “Submit”
Then I should see “User has been created.”
And new user should be added.
{% endhighlight %}

Evil was found. As you can see in this scenario, there is too much information in it. Not useful information about the domain of the application but details of how to accomplish something in it. (signing up for this specific case)

A scenario should talk about the domain not about moving from one page to another and clicking some links or filling in some text boxes in the page. Imagine customers are talking to you about this kind of feature in the application they want you to produce for them. How would they describe it to you? They would say: “users can sign up through the website” or they would say: “a user should click on the Sign up link and then fill in text boxes with the appropriate information and after that click on the button. And when he/she successfully signed up there should be a message about it”? Of course it’s going to be the former when they are describing the feature. Most of the times they will tell you something like the latter expression when they see the system functioning. Then maybe they will tell you about some modifications they want you to make in the details of the feature. But normally they don’t do it when they are describing the feature. And this is the job for scenarios. They should DESCRIBE the feature not the details of it.

This is one of the numerous cases that testers can be really helpful. Testers have a different point of view from programmers and they tend more to look at the “WHAT” not the “HOW”. When a tester sees a scenario like this she will immediately say: “this is not a good scenario. It contains too much detail and distracts the reader from the essence of the feature”. This is why Liz Keough emphasizes that BDD scenarios should be born from conversations between user, tester, programmer, etc. and the conversation is the most important ingredient cause it’ll embrace different viewpoints.

Cucumber perfectly supports the right way of doing this. What you see here should be in the step definitions not in the feature and corresponding scenarios. So we changed this scenario to something like this:

{% highlight ruby %}
Given a user is on the homepage
When she signs up
Then she should be included in the users of the website
{% endhighlight %}

That’s it. All those detail information moved into the step definitions of signing up. You should not talk about HOW in the scenario. Scenario is a place for a WHAT of a feature. HOW should be handled in the step definitions (but as little as possible).

Specially in Rails

You can see this kind of problem more in cucumber scenarios for rails applications. The reason for that is because when you run the generator command for cucumber (rails generate cucumber:install) it creates a web_steps.rb file which contains some pre-defined steps in it. People tend to use them and don’t introduce their own steps as much as possible. So they write their scenarios in a way that they can use from the steps already existed the in web_steps. And that is the main reason my friend and his team wrote their scenarios in that way too. (e.g I follow “link” or I press “button text” etc.)

Fortunately new versions of cucumber-rails gem and cucumber generator removed that pre-defined web_steps file completely for this reason. But at the same time unfortunately some people are used to the old version style and still define their scenarios and steps like that. You can find more about this removal [here](https://github.com/cucumber/cucumber-rails/issues/174).

Remember, just because something exists it doesn’t mean you have to use it or you have to adapt yourself to it. You should try to write high quality, maintainable, flexible code and test and anything that impedes this, should be eliminated and removed from your arsenal.

I hope you like this and I appreciate your feedback.

Notes

Thanks a lot to Lisa Crispin for motivating me to share this.

I know that the example here is really classic and educational but I could get permission to share only this one scenario from my friend and his team.
