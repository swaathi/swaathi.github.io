---
layout: post
title: If-ElseIf-Else-Ception
date: 2016-06-18 10:30:00 +5:30 GMT
share: y
---

I HATE when my code looks like a monstrosity of conditions.

{% highlight bash %}
If this, do that;
else if this + 5, do this;
else if this - 5, do this*5;
else don't do anything.
{% endhighlight %}

<!--break-->

There’s way too much logic involved and no effort to simplify it. It’s harder for someone else to understand this, let alone when you yourself look back at this later on. There’s got to be a quantified way to solve this code spaghetti!

There’s a great talk by Sandi Metz at Rails Conf 2014, [All the Little Things](https://www.youtube.com/watch?v=8bZh5LMaSmE), that addresses this.

![All The Little Things by Sandi Metz, Rails Conf 2014](https://cdn-images-1.medium.com/max/800/1*lPNcXEelEj3GP8zIPSRHoA.png){:class="img-responsive"}

Her approach is very simple — take your huge monolithic logic, and break it down to smaller, manageable, and reusable components. The rest will sort out over time.

Here are a few of my tricks when reducing complexity,

### Return From The Top

If you have an ultra simple function that compares a condition to return a value if it matches, and another if it doesn’t. Fairly straight forward scenario.

{% highlight rb %}
def pretty_time(time)
  if time
    time.strftime("%B %e, %Y")
  else
    nil.to_s
  end
end
{% endhighlight %}

Sure it’s not too bad, but it can look even better if you do this,

{% highlight rb %}
def pretty_time(time)
 return nil.to_s unless time
 time.strftime("%B %e, %Y")
end
{% endhighlight %}

You can bid good bye to your hanging else statements!

### Pipes Are Awesome

The pipe symbol, **\|\|** is a favorite among Ruby devs. You can quickly assign default values when a variable is nil.

{% highlight rb %}
> nil || 6
=> 6
{% endhighlight %}

But it’s not too friendly with strings.

{% highlight rb %}
> "" || "six"
=> ""
{% endhighlight %}

When you use this with the presence method, magic happens!

{% highlight rb %}
> "".presence || "six"
=> "six"
{% endhighlight %}

### Try, Try again

If I got a dollar for every time i ran into the _nil:NilClass error_, I’d be a millionaire! This made me paranoid to stick a if _obj.present?_ everywhere.

{% highlight rb %}
if user.present?
  return user.name
end
{% endhighlight %}

Yuck! There’s a better way to do this, you just got to keep trying.

{% highlight rb %}
user.try(:name)
{% endhighlight %}

Simple! Now when your user doesn’t exists, it will fail quietly and return a nil.

This works with methods too. If you have a method called fullname, try this out,

{% highlight rb %}
user.try(:fullname)
{% endhighlight %}

### Send Off

I have this horrible habit of organizing background workers as if they were logical classes. What I mean is, if I have a few tasks that need to be processed for the user in the background, I put them all in one worker (if it’s not too big). It’s easier to manage, and I don’t have a mess of workers when others collaborate.

Since I have different tasks to be processed for each call of the worker, I pass an “identifier”. Something that tells the worker class what function to perform.

{% highlight rb %}
class UserWorker
  include Sidekiq::Worker

  def perform(action, options={})
    if action == :welcome
      user.welcome_email(options)
    elsif action == :process
      user.process_profile_picture(options)
    elsif ....
    elsif ....
    end
  end
end
{% endhighlight %}

**UGLY!**

Fortunately, there’s a cleaner way of doing this. This is where Ruby’s powerful meta-programming comes handy. Use **send()**.

What is send?

> send( ) is an instance method of the Object class. The first argument to send( ) is the message that > you're sending to the object - that is, the name of a method. You can use a string or a symbol, but > symbols are preferred. Any remaining arguments are simply passed on to the method.

So from the above mess, this is how my reduced worker looks like.

{% highlight rb %}
class UserWorker
  include Sidekiq::Worker

  def perform(action, options={})
    send(action, options)
  end
  def welcome(options)
    user.welcome_email(options)
  end
  def process(options)
    user.process_profile_picture(options)
  end
end
{% endhighlight %}

So much cleaner, and so much easier to extend.

### Override

This particular tip needs to be taken with a pinch of salt. It’s not for everyone, and it might seriously damage your logic if not used carefully.

The great things about Ruby being OOPs complaint is it’s ease of extending base classes like String, TrueClass, and Integer. When used carefully you can do wonders.

I hate doing an if condition for booleans, it just feels wrong. Something like this,

{% highlight rb %}
if user.admin
  return "You are an admin"
else
  return "You are not an admin"
end
{% endhighlight %}

I’d have to create a specific function in the User class just to handle something as simple as printing out different messages for a boolean value. Here’s a neat trick,

{% highlight rb %}
class TrueClass
  def message(true_message, false_message)
    return true_message
  end
end
{% endhighlight %}

{% highlight rb %}
class FalseClass
  def message(true_message, false_message)
    return false_message
  end
end
{% endhighlight %}


This overrides **TrueClass** and **FalseClass** so you can do something as simple as this,

{% highlight rb %}
user.admin.message("You are an admin")
{% endhighlight %}

So simple! There’s so many tiny adjustments you can make with this little trick!
