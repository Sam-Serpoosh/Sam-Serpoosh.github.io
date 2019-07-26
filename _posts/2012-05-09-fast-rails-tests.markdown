---
layout: post
title:  "Fast Rails Tests, Smoother Development"
date:   2012-05-09 19:00:00
categories: programming rails refactoring technology-issues unit-test
---

If you're a rails developer or had an experience of writing a rails application you know that running your tests is a really painful and long process because it loads the rails environment at first. I’m assuming you write unit-tests for your rails application because it’s really offensive not to write tests in kind of an application that creates a test/spec folder for you from the very first beginning of its life. It assumes that you write tests for your app because you care about your code and software. Once again I remind you Uncle Bob’s famous quote about writing tests and TDD: "you can’t call yourself a professional developer if you’re not doing TDD." Anyway, running tests in rails is really painful and too slow. When you are doing your Red-Green-Refactor cycle whenever you run your tests you have to wait several seconds until you get a feedback from your tests. This is not the way how it should be done. This latency and delay is breaking the cycle and is really overwhelming. Pretty soon you won’t run your tests as frequent as you should, therefore you won’t be able to get quick feedback about what you’ve done and you miss a lot of TDD and unit-testing benefits in general.

I personally write my code with TDD approach and the quick feedback is really critical during my development cycle. So this kind of slow test won’t let me to be as productive as I should. Whenever I run my tests I am like: "Son of a Gun, why is this happening?!"

Recently I watched [Corey Haines talk at Arrrrr Camp](http://arrrrcamp.be/videos/2011/corey-haines---fast-rails-tests/) about fast-rails-tests. I can’t emphasize on watching this talk enough if you care about your craft and productivity as a developer. Note that I didn’t say rails developer because there are lots of good ideas that you can take from that talk as a developer and software craftsman in general. In that talk Corey mentions about separating your domain and business logic modules in a way that don’t depend on rails. Not surprisingly, isolation and separation of concerns is a golden solution here, AGAIN. According to Corey’s example imagine you have a ShoppingCart model which has many products in it (ActiveRecord model of course) and at some point you need to calculate the total price of the products in your ShoppingCart. By default when people face this kind of problem they write a method in the ShoppingCart model which calculates the total price of the products like this:

{% highlight ruby %}
def total_price
    products.map(&amp;:price).inject(0, &amp;:+)
end
{% endhighlight %}

BTW this is one of the reasons we will end up with “Fat Models”. Let’s ask a question from ourselves at this point: “does this really have anything to do with rails and ActiveRecord?” No, it’s obvious. So, why we put it in the model? Why we made this logic depend on the ActiveRecord and rails in general? We shouldn’t do this. This is a calculation and logic and it should be separated. There are different ways for doing this according to Corey’s talk and I definitely recommend you to watch it and learn about them because in each situation you should pick the best solution. For instance, we can separate this logic and include it in the model so it has this functionality, but the functionality implementation itself is in the right place and separated rather than being mixed with the model. Here’s the test and code for it:

{% highlight ruby %}
it “returns sum of price for multiple items” do
    TotalPrice.of(stub(:price =&gt; 10), stub(:price =&gt; 20)).should == 30
end

module TotalPrice
    def of(items)
        items.map(&amp;:price).inject(0, &amp;:+)
    end
end
{% endhighlight %}

Then we can use it like the following:

{% highlight ruby %}
class ShoppingCart < ActiveRecord::Base
    has_many :products

    def total_price
        TotalPrice.of(products)
    end
end
{% endhighlight %}

Now you can have your tests for the total price calculation separately from the model and it has no dependency to rails which is the right thing to do. Your tests for this logic will run extremely fast with no latency and delay because of good separation of concerns. I love this idea and ever since I watched this talk by Corey I’m looking for potential Refactorings in my rails apps for separating my domain and business logics and calculations from my models. One of the perfect candidates for this was in my User model. I have some password machinery in my User model which encrypts the password before the user is saved in the database using the before_save ActiveRecord call back. It kind of looks like this: (I made it so much simpler for focusing on the purpose of this post):

{% highlight ruby %}
class User < ActiveRecord::Base
    attr_accessible :password
    before_save :encrypt_password

    def encrypt_passowrd
        #some password encryption machinery here
    end
end
{% endhighlight %}

My tests for this encryption are really painful like all the rails dependent tests because they take too long to run. But then it hit me that this machinery has nothing to do with rails, it’s an encryption algorithm that I use for my password security issues. So I can SEPARATE this logic from the model perfectly. So I ended up writing the following module:

{% highlight ruby %}
module PasswordEncryption
    def encrypt(str)
        #some encryption machinery here
    end
end
{% endhighlight %}

Then I changed my model accordingly:

{% highlight ruby %}
class User < ActiveRecord::Base
    include PasswordEncryption

    attr_accessible :password
    before_save :encrypt_password

    def encrypt_password
        encrypt password
    end
end
{% endhighlight %}

Now I can run my tests during my development cycle very quickly and get fast feedback which is one of the main purposes of TDD and unit-testing in general. My development process for the business logic and calculations part of the application which BTW are most important parts, became so much smoother and faster and I can completely see the improvement in my productivity when I’m working on a rails app. Here are screenshots from running password encryption related tests in my app before and after this Refactoring and design change:

<iframe src="https://drive.google.com/file/d/1BEygUfbyoTb_w8gz5uyhmO2kn7kusQqK/preview" width="640" height="480"></iframe>

<iframe src="https://drive.google.com/file/d/1hqSJhNqcx3jdwrHLTyCAlGhA3TjPqb74/preview" width="640" height="480"></iframe>

Look how much difference that design change made. I loved this point by Corey during his talk: "most of the times when people have troubles testing something in their code either about running time or testing the functionality itself they change their tests but it’s Test-DRIVEN-Development and it means that we should change our design because that difficulty in testing shouts about design problem which needs to be solved. So, we should change our design NOT our tests."

I found this really simple trick which helps me a lot while I’m hunting for this kind of Refactoring and design candidates in my rails apps, whenever you face something in your code ask yourself this simple question: "does this have anything to do with rails?" when the answer is "No" go ahead and separate that logic. Try this and you’ll be surprised and thrilled about the fact that you will develop your app so much SMOOTHER and more FLUENT!

Hope that helps, it helped me a lot.

Related Posts:

- [Running Rails Rspec Tests - Without Rails](http://www.adomokos.com/2011/04/running-rails-rspec-tests-without-rails.html?showComment=1302880085636#c8190505485489994604)
- [Making ActiveRecord Models Thin](http://solnic.eu/2011/08/01/making-activerecord-models-thin.html)
- [Architecture The Lost Years (Uncle Bob)](http://www.youtube.com/watch?v=WpkDN78P884)
