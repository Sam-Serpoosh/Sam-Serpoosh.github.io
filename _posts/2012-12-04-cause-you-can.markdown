---
layout: post
title:  "Just because you can does NOT mean you should"
date:   2012-12-04 22:00:00
categories: programming ruby unit-test
---

I tweeted about this thing a couple times and now I want to elaborate a little more on it with an example! A lot of times I hear people talking about the fact that something is much easier to do in a specific language and we can do it that way! And a lot of times that makes sense and is a good way to do that specific task in that specific language! But sometimes it’s just a violation of a principle that we’ve learned the hard way that it matters and we should be careful about it in our code base! For example imagine we want to write a user authenticator which determines whether user’s credentials are correct or not! You can see something similar to the following in a lot of code bases! (Remember that it’s just a very simple thing just for the sake of this post and by no means this is a user authenticator :D)

{% highlight ruby %}
describe UserAuthenticator do
    it “does not authenticate with wrong password” do
        user = stub(:username => “bob”, :password => “pwd”)
        user_rep = stub(:get_user_by_username => user)
        DatabaseUserRepository.stub(:new => user_rep)
        authenticated = subject.authenticate(“bob”, “wrong password”)
        authenticated.should be_false
    end
end

class UserAuthenticator
    def initialize
        @user_rep = DatabaseUserRepository.new
    end

    def authenticate(username, password)
        user = @user_rep.get_user_by_username(username)
        password == user.password
    end
end
{% endhighlight %}

It’s not this easy to do such a thing in a static-type language like Java or C# but since ruby is a dynamic language (duck-type) so ruby and rspec let you do something like the above code! As long as it responds to “new” message you can stub it in any way you like so a lot of people just because they CAN do this they do something similar in their code! But we forgot the main point here! We are violating the Dependency Inversion Principle and instead of depending on abstraction we are tightly coupling ourselves to a concrete thing (DatabaseUserRepository)! I know that Dependency Inversion Principle in a dynamic language such as Ruby has a slightly different meaning and it’s not exactly as its meaning in Java or C#! But what matters for us here is something that can responds to the message “get_user_by_username”! That’s all that matters here! But we are making ourselves more dependent than what matters! Imagine we want to also support storing user information in flat files or some other mechanism! How are we gonna handle those parts of code for the same logic and functionality?! Are we gonna have another UserAuthenticator that instantiate FileUserRepository in its constructor and the rest will be the exact same?! Of course not and I can see you’re cringing right now! WTF?!

We have something which is exactly helpful in this kind of situation and it’s called Dependency Injection! (Of course there are other ways for handling this, e.g. Strategy Pattern, but I go with Dependency Injection for this post just for fun :D)

We inject the dependency and the only thing that matters to us is that it can respond to “get_user_by_username” message! So when it comes to support different kinds of storage mechanism and their appropriate repositories the code will be totally flexible in that scenario and all we need is a new instance of our UserAuthenticator with a FileUserRepository injected to it! So the code for such a design will be like the following:

{% highlight ruby %}
describe UserAuthenticator do
    let(:user_rep) { stub }
    let(:authenticator) { UserAuthenticator.new(user_rep) }
    it “does not authenticate user with wrong password” do
        user = stub(:username => “bob”, :password => “pwd”)
        user_rep.stub(:get_user_by_username => user)
        authenticated = authenticator.authenticate(“bob”, “wrong password”)
        authenticated.should be_false
    end
end

class UserAuthenticator
    def  initialize(user_rep)
        @user_rep = user_rep
    end

    def authenticate(username, password)
        user = @user_rep.get_user_by_username(username)
        password == user.password
    end
end
{% endhighlight %}

The point is not just being able to write test for the code! The whole point of TDD is driving a better design for us so in a static-type language it’s not easy to do something like the first example hence it’s hard to write tests for it this way so it will drive us towards depending to abstraction and also injecting that dependency through constructor so we can easily replace the actual collaborator with a stub and test the unit of work that we want at the moment completely isolation! But because we can write a test like the first example in ruby and rspec it doesn’t mean we should do that! The point is having a better design not being able to write tests for the code!

Remember again, just because you can does NOT mean you SHOULD! Power brings responsibility so use the power in a right way and at the right place!

Something that worth a try!

I hope you enjoyed it!
