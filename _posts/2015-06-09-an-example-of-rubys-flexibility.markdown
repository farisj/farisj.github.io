---
layout: single
title: "An Example of Ruby's Flexibility"
last_modified_at: 2015-06-09 18:10:58 -0400
blurb: "My first post on the power of ruby"
---

Blocks are a powerful yet tricky tool in Ruby to help programmers iterate over data and transform it or analyze it how they like. But beware! Power can be a dangerous thing.

Let's look at a way that Ruby's enumerables can be so flexible, almost too flexible...

Consider the following array.

```ruby
people = {

  "Bob" => {
    gender: "man",
    age: 28
  },

    "Alice" => {
    gender: "woman",
    age: 23
  },

    "Susan" => {
    gender: "woman",
    age: 30
  }
}
```

Being a hash, one can iterate over the data within this hash using enumerators.

Let's consider iterating over `people` using `.each`.

~~~ruby
people.each do |name, person_hash|

  print "Data about #{name}:"; puts
  print person_hash; puts; puts

end
~~~
This gives us the output

~~~
Data about Bob:
{:gender=>"man", :age=>28}

Data about Alice:
{:gender=>"woman", :age=>23}

Data about Susan:
{:gender=>"woman", :age=>30}
~~~

So good so far. But what if you are parsing something complicated and lose track of what `people` contains? Let's see what happens if we accidentally assume that `people` just contains an array of strings of people's names (even though it is still the hash).

Consider this code, with the same hash `people`.

~~~ruby
people.each do |person|
  print person; puts
end
~~~

This gives us the output

~~~
["Bob", {:gender=>"man", :age=>28}]
["Alice", {:gender=>"woman", :age=>23}]
["Susan", {:gender=>"woman", :age=>30}]
~~~

Woah! Arrays containing keys and values? That's not what we were expecting at all!

The danger in Ruby is that even though our block one had one argument, Ruby converted our data to fit our argument. While this can be a very powerful processing tool, it serves as a reminder that Ruby's flexibility requires us, the programmers, to be hyper-aware of what our variables are and how we are using them - because Ruby will try its hardest to work regardless!

This is just one tiny example of how flexible Ruby can be. I'm sure as I get deeper into the language, there will be many more nuances like this one to talk about!
