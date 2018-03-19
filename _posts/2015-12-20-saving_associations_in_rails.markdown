---
layout: single
title: "How Associations Are Built In Rails"
date: 2015-12-20T17:55:43-05:00
blurb: "A deeper dive into Rails Associations"
---

As web developers, one of the core components of our job is figuring out efficient ways to manage user's data.


In Rails, Active Record gives us many ways for us as web developers to maintain and manage relationships between data points in Rails using its handy dandy ORM. It has a variety of endpoints when it comes to dealing with data, wrapping this functionality into objects that serve our purposes.

However, we need to really understand these tools before we can use them with the finesse that web apps need from time to time. A lot of the time, it's easy to let Rails abstract away the magic of ORM and creating relationships between data, but what if we want to really understand what Rails is doing under the hood?

![Even Steve from Blue's Clues needs to decide the right tool for the job.](http://i.imgur.com/63AnIA6.jpg)


This post will address `ActiveRecord::Association` and outline how relationship methods are generated in this framework.

There are several different relationships that datapoints can have with each other. The most common of these relationships are:

* `belongs_to`
* `has_one`
* `has_many`
* `has_and_belongs_to_many`

This post will assume you have a basic knowledge of these relationships.

Most of us are familiar with these methods in some way shape or form in Rails. But what really happens under the hood when we add these lines into our Activerecord classes?

When Rails is booted up, ActiveRecord Models are loaded into memory and a little metaprogramming magic begins. When a Model contains one of the above methods, a series of methods are fired to construct methods that address these relationships. Consider we have two models, `User` and `Post`.

``` ruby
class User < ActiveRecord::Base
  has_many :posts
end
```

``` ruby
class Post < ActiveRecord::Base
  belongs_to :post
end
```

The main way we manipulate the relationship between these two models is using the accessor methods that are given to us by these models. These accessor methods are built in Rails through a chain of events - let's look at some of the methods in that call stack.

We start in the `ActiveRecord::Associations::Builder::Association` class, a parent class that is inherited by `CollectionAssociation` (for `has_many` and `has_and_belongs_to_many`) and `SingularAssociation` (for `belongs_to` and `has_one`). It contains a method `self.build`. The source code is below:

``` ruby
def self.build(model, name, scope, options, &block)
  if model.dangerous_attribute_method?(name)
    raise ArgumentError, "You tried to define an association named #{name} on the model #{model.name}, but " \
                         "this will conflict with a method #{name} already defined by Active Record. " \
                         "Please choose a different association name."
  end

  extension = define_extensions model, name, &block
  reflection = create_reflection model, name, scope, options, extension
  define_accessors model, reflection
  define_callbacks model, reflection
  define_validations model, reflection
  reflection
end
```

So, a lot is going on here - it's ensuring the method names are okay, defining some things, general rails shennanigans. Let's focus on `define_accessors`. A reflection in Rails is an object that allows the system to determine what type of associations that object should have (it's not a huge focus for the purpose of this post). This will generate things like `post.user` and `user.posts=`. Let's check out that method, also in this Association class:

``` ruby
def self.define_accessors(model, reflection)
  mixin = model.generated_association_methods
  name = reflection.name
  define_readers(mixin, name)
  define_writers(mixin, name)
end
```

As we can see, we break down the defining of readers and writers here. These methods look like this (we're still in the same class):

``` ruby
def self.define_readers(mixin, name)
  mixin.class_eval <<-CODE, __FILE__, __LINE__ + 1
    def #{name}(*args)
      association(:#{name}).reader(*args)
    end
  CODE
end

def self.define_writers(mixin, name)
  mixin.class_eval <<-CODE, __FILE__, __LINE__ + 1
    def #{name}=(value)
      association(:#{name}).writer(value)
    end
  CODE
end
```

A little bit of metaprogramming here - these methods define a new method on the class in question. The `reader` method will return either a `CollectionProxy` which contains data about a set of objects (for non-singular associations), or the object itself. The writer method that is being called actually delegates to a replace method, as seen below:

``` ruby
def writer(records)
  replace(records)
end
```

Now, the `replace` method and the rest of the call stack gets pretty complicated from here (and isn't the purpose of this post), but just know that the `replace` method eventually delivers a query to your database to update ids and reflect the proposed changes you're making in the database.

At its core, it's some relatively simple metaprogramming which is being used to create all of these methods. Of course, the inner details can get quite complicated (what does a Reflection do, exactly? how are these things differently handled between singular and collection Associtations? ), but the main idea of this metaprogramming is easy to follow.

Hope this clears up a little bit of the mystery from the magic of Rails!
