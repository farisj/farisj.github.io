---
layout: post
title: "Validations in Active Record"
date: 2015-07-07 16:04:11 -0400
blurb: "Exploring how Validations work in ActiveRecord"
---

Imagine this: you're developing your very first Rails application, and need users to give you some data to manipulate in order to implement your intended functionality. How do you do this? Forms, obviously!

Forms are primary way that users can interact with and manipulate your program's data. How many privileges the user has varies from project to project, but you, the architect of this program, want to make sure that the user's work with your program the way they want you to, not however they please. So how can we manipulate our forms in order to force the user's input to adhere to a specific style? The answer... validations!


Validations are features in Rails which allow our Models to apply constraints to any potential datasets to ensure they adhere to a particular format to fit into your model/database properly. Let's look at a way that a model can "validate" it's data.

Consider an model where you have a database with a Person model, where every person needs a name:

``` ruby
class Person < ActiveRecord::Base
  validates :name, presence: true
end
```

Ok, so what does this mean for our Person model? Basically, ActiveRecord will manipulate those instances of Person who do not pass the validation. The validation seen above, `presence: true`, ensures that the `:name` property of our Person is not `nil`. There are a variety of methods that ActiveRecord  provides you.

``` ruby
@person = Person.create(name: nil)
@person.persisted? #=> false
@person.valid? #=> false
@person.errors #=> Instance of ActiveModel::Errors class
@person.errors.messages #=> {:name=>["can't be blank"]}
@person.errors.full_messages #=> ["Name can't be blank"]

```

Woah! ActiveRecord gives us a lot more than this, but these are some pretty useful and important methods. Let's talk about them. Firstly, note that even though `Person.create` was called, `person.persisted?` returned false, and thus the Model refused to update to the database. Great! We've given our Model control over what data it takes in. These validations also give us the `@person.valid?` method, which we can use elsewhere to pass on to other parts of our program for whatever we need. Note that there is an ActiveModel::Errors class attached to this model. This class does a lot, but it gives us a great string representation of what's wrong with our data. The `@person.errors.full_messages` can be used to tell the user exactly what's wrong. Cool! These are a lot of useful tools for working with our forms and getting users to input appropriate data.

Methods aside, let's look at a few of the more interesting (and common) validation methods.


Let's explore a few of the more important ones:

#### Presence

As said before, the `presence: true` validation checks to see that the specified property of a model is not `nil`. Pretty straightforward.

#### Uniqueness

Let's consider that you're maintaining a series of Users and they need to register usernames which are unique. We can implement this in our Model like this:

``` ruby
class User < ActiveRecord::Base
  validates :username, uniqueness: true
end
```

Now, when attempting to add rows to our User table in our database, ActiveRecord will check and see that the `:username` field in this potential new row must be unique in the table. Simple as that!

#### length

In the same example with the usernames, let's consider the idea that we want usernames to be between 3 and 8 characters.


``` ruby
class User < ActiveRecord::Base
  validates :username, length: {minimum: 3, maximum: 8}
end
```

We don't need to add in the uniqueness validator here, because if `:username` is nil, it's obviously less than 3 characters.

Let's look at what happens to our User instances:

``` ruby
@short_name = Person.create(username: "a")
@short_name.valid? #=> false
@short_name.errors.messages #=> {:name=>["is too short (minimum is 3 characters)"]}
@short_name.errors.full_messages #=> ["Name is too short (minimum is 3 characters)"]

@long_name = Person.create(username: "abcdefghijklmnop")
@long_name.valid? #=> false
@long_name.errors.messages #=> {:name=>["is too long (maximum is 8 characters)"]}
@long_name.errors.full_messages #=> ["Name is too long (maximum is 8 characters)"]
```

Cool stuff, eh? This can be very practical for things like usernames, passwords, Twitter handles, whatever you want.

#### inclusion

Let's consider a case where you're managing the website for an online clothing store and you have a Model for shirts. Shirts can only be one of a few fixed sizes, and we want to ensure that the size that is entered into our model corresponds to one of these sizes. We can implement this as such:

``` ruby
class Shirt < ActiveRecord::Base
  validates :size, inclusion: { in: ["XS", "S", "M", "L", "XL"] }
end
```

What will this give us in practice?

``` ruby
@shirt = Shirt.create(size: "S/M")
@shirt.valid? #=> false
@shirt.errors.messages #=> {:size=>["is not included in the list"]}
@shirt.errors.full_messages #=> ["Size is not included in the list"]
```

Make sense? This validator is checking to ensure that the `:size` is one of the few predetermined sizes.

One thing of note here... This error message isn't great. "Size is not included in the list?" What list? Is there any way to make this error more specific?

Well it turns out, yes there is! We can specifiy a custom message in our validator, as such:

``` ruby
class Shirt < ActiveRecord::Base
  validates :size, inclusion: { in: ["XS", "S", "M", "L", "XL"], message: "%{value} is not a valid size (XS, S, M, L, or XL)"}
end
```

This produces output as such:

``` ruby
@shirt = Shirt.create(size: "S/M")
@shirt.valid? #=> false
@shirt.errors.messages #=> {:size=>["S/M is not a valid size (XS, S, M, L, or XL)"]}
@shirt.errors.full_messages #=> ["Size S/M is not a valid size (XS, S, M, L, or XL)"]
```

Cool! So there's a way for us to describe our errors to match our program's problems as we want. This is a pretty helpful tool. Note that unlike regular Ruby, to interpolate the user's data into our error, we use `%{value}` rather than `#{value}`.

#### format

Back to our Users and `:username` example. Let's consider that we want the usernames to be only letters,numbers, or underscores, and have the first character be a letter. We can do this as such:

``` ruby
class User < ActiveRecord::Base
  validates :username, format: {with: /\A[a-z]\w+\z/, message: "%{value} is invalid. Please use only letters, numbers, or underscores, and start with a letter."}
end
```

This would produce the following output:

``` ruby
@user = User.create(username: "_a")
@user.valid? #=> false
@user.errors.messages #=> {:name=>["_a is invalid. Please use only letters, numbers, or underscores, and start with a letter."]}
@user.errors.full_messages #=> ["Name _a is invalid. Please use only letters, numbers, or underscores, and start with a letter."]
```

Awesome! So we can make sure that the user's input matches perfectly any sort of formatting we want. This is extremely useful in ensuring that users adhere to the design patterns we implemented.

#### Chaining Validations

You can have multiple constraints on an attribute of a model. Let's combine two above examples and discuss the notion of ensuring that User's usernames are both the proper characters AND the proper length. We can daisy chain the validations as such:

``` ruby
class User < ActiveRecord::Base
  validates :username, format: {with: /\A[a-z]\w+\z/, message: "%{value} is invalid. Please use only letters or numbers."}, length: {minimum: 2, maximum: 8}
end
```

This Model will ensure that the `:username` is within our regex and our length constraints. It's errors will contain both problems if both arise, as such:

``` ruby
@user = User.create(username: "_a")
@user.valid? #=> false
@user.errors.messages #=> {:name=> ["_a is invalid. Please use only letters or numbers.", "is too short (minimum is 3 characters)"]}
@user.errors.full_messages #=> ["Name _a is invalid. Please use only letters or numbers.", "Name is too short (minimum is 3 characters)"]
```

Now the hash style of the errors makes a lot of sense! This structure can be helpful when you're validating a lot of constraints on your model as well as making complex constraints. Each validation that is violated will present an error within this hash. How helpful!

These are just a few of the validators that I found interesting. There are a lot more validators, as well as parameters and constraints on those validators, which you can read all about [on the Ruby on Rails guides](http://guides.rubyonrails.org/active_record_validations.html).

Hope that this post was interesting and that it clears up some of the mystery of Model validators!
