---
layout: post
title: Rails strategy for reusing common behavior in models
date: 2018-02-09 15:30:00 +5:30 GMT
share: y
---

A while ago I had an interesting problem to solve. Three of my modules reused the same functionality, they all had to parse address fields from a given lat and long.

<!--break-->

The following are the list of attributes each module needed to accommodate this,

```
lat
long
sublocality
city
state
country
pincode
```

They also had to implement the following methods,

```
extract_address_components!
area
full_address
```

As a developer your primary concern for maintainability should always be [DRY](https://en.wikipedia.org/wiki/Don't_repeat_yourself).

> Every piece of knowledge must have a single, unambiguous, authoritative representation within a system

The DRY principle was formulated by Andy Hunt and Dave Thomas in their book [The Pragmatic Programmer](https://en.wikipedia.org/wiki/The_Pragmatic_Programmer). Paraphrasing from Wikipedia; When the DRY principle is applied successfully, a modification of any single element of a system does not require a change in other logically unrelated elements. Additionally, elements that are logically related all change predictably and uniformly, and are thus kept in sync.

Rails 4 introduced a way of sharing code as modules between modules. They're called [concerns](https://richonrails.com/articles/rails-4-code-concerns-in-active-record-models) and they sit right in your models folder. You can then include these modules in any model you want! It would look something like this,

```
module AddressExtractor
  extend ActiveSupport::Concern

  def extract_address_components!
  end

  def area
  end

  def full_address
  end
end
```

And then include it in any of model file like this,

```
class Map < ApplicationRecord
  include Authentication
end
```

This works great.....for awhile.

Concerns lose their usefulness when you need to share common behavior across applications. A quick Google search would reveal that the 'popular' advice to do share code across applications is,

```
Whip up a microservice and expose the common behavior as an API
```

I don't know about you, but I don't have servers and domains available all the time! To share functionality across applications, just create a Gem! You can host it on Github and add it to your Rails applications like this,

```
gem 'address_extractor', git: 'https://github.com/skcript/address_extractor.git'
```

In your Gem, create a module that looks like,

```
module AddressExtractor
  REQUIRED_ATTRIBUTES = [
    'lat',
    'long',
    'sub',
    'city',
    'state',
    'country',
    'pincode',
    'address'
  ]

  def address_extractor
    extend ClassMethods
    include InstanceMethods

    self.new.has_all_required_attributes?
  end

  module ClassMethods
  end

  module InstanceMethods
    def has_all_required_attributes?
      missing = REQUIRED_ATTRIBUTES - attributes.keys

      if missing.any?
        raise "#{self.class} does not have the required attributes: #{missing.to_sentence}"
      end
    end

    def full_address
    end

    def area
    end

    def extract_address_components
    end
  end
end

ActiveRecord::Base.extend AddressExtractor
```

There's three things to watch out here,

1. **The `REQUIRED_ATTRIBUTES` constant**
This is a simple array of all attributes that your Gem requires to function.

2. **The `address_extractor` function**
This function 'extends' a class called ClassMethods and 'includes' a class called InstanceMethods. This allows you to write all the methods that your model class needs inside ClassMethods and all the methods that your model object needs inside InstanceMethods.
The `has_all_required_attributes?` simply checks if all the attributes are present in your model. Nifty right?

3. **Extending ActiveRecord**
In order for AddressExtractor to easily bind with your models, extend it to ActiveRecord itself.

**Pro Tip:**
If you don't want to create a Gem nor use Concerns, you can include the above module in your lib directory. Then create an initializer that requires AddressExtractor (`require 'address_extractor'`).

Finally, all you have to do is add `address_extractor` to any of your model.

```
class Map < ApplicationRecord
  address_extractor
end
```
