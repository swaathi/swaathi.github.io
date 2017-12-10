---
layout: post
title: The Ruby Configure Block
date: 2017-12-08 14:30:00 +5:30 GMT
share: y
---

I've always liked the way certain Ruby gems allowed developers to configure parts of its workflow.

```
Airbrake.configure do |config|
  config.api_key = 'your_key_here'
end
```

This block just looks so clean and to the point. Almost like magic!

But magic is just a bit of zeros and ones. How'd they do it?

The answer lies in Ruby's singleton classes. Singleton is a design pattern that allows only one one instance of the class to be globally available. It provides a single point of access to some service in your application so that you don't have to pass around a reference to that service every time you need to use it. It is useful when you find yourself writing only class methods or when you don't need an object of the class to function.

However fair warning, singleton is also a [really hated programming pattern](https://blogs.msdn.microsoft.com/scottdensmore/2004/05/25/why-singletons-are-evil). A singleton is basically a global object - and everyone knows the more globals you have, the more problems you have!

A singleton class could look like this,

```
class Single
  def self.me
    'Forever Alone'
  end
  
  def self.me_in_the_future
    'Forever alone in the future'
  end
end
```

But appending a `self` to every method isn't very fun. So instead do this,

```
class Single
  class << self
    def me
      'Forever Alone'
    end
    
    def me_in_the_future
      'Forever alone in the future'
    end
  end
end
```

Just call `Single.me` it will return `Forever Alone`.

However classes aren't the only ones that can have singleton methods. A string can too!

```
a = "I am A."

def a.hello
  "Hello! It's me, A."
end
```

Now you can call `a.hello`. Wierd isn't it?

Not really. What we did here is we added a singleton method `hello` to an object `a`. The difference between class methods and singleton methods is that class methods are available to all instances of a class object while singleton methods are available only to that single instance. (- via [Toptal](https://www.toptal.com/ruby/ruby-metaprogramming-cooler-than-it-sounds))

Great. How do we add a configure block?

Here's the implementation,

```
module Idly
  class << self
    attr_accessor :configuration

    def configure
      self.configuration ||= Configuration.new
      yield(configuration)
    end
    
    def reset
      self.configuration = Configuration.new
    end
  end
end

class Configuration
  attr_accessor :chutney

  def initialize
    @chutney = 'white'
  end
end
```

Let's configure our Idly! (Yes, I'm hungry.)

```
Idly.configure do |config|
  config.chutney = 'tomato'
end
```

What this allows you to do is allow your gem to be configured from the outside, without modifying the contents of the gem. 

To use the configuration in your gem all you have to do is call,

```
Idly.configuration.chutney
```

If you want to reconfigure, simply call the `configure` block again!

That's it! Let's go get some piping hot idly and chutney now.

