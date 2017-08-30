---
layout: post
title: "Model Testing with RSpec"
date: 2015-08-04 08:48:37 -0400
blurb: "Thoughts on best practices for Rails Model testing in RSpec"
---

Models are a very important component of any MVC application. In Rails, our models are integrated with Activerecord in order to communicate with our backend database of choice to retrieve data that our controllers request.

This is all well and good, but if we want to be thoughtful and thorough developers, we need to design a system where we can guarantee that our models are designed in a way that 1) makes sense and 2) works with the many other moving parts of our application. How can we achieve this? Model testing in RSpec!

RSpec is a Ruby gem which provides myriad capabilities for testing differnet parts of a Rails application - one of these parts are the models.

So how does the RSpec gem work and what does it provide us to test different compnents of our models?

RSpec provides a rich DSL which gives programmers the tools to generate  model data and validate that the models are functioning in the manner that we expect them to. It uses several types of Ruby blocks to "describe" the various "contexts" that our models are used in (hint, hint). Let's see an example in action.

Consider the instance where you are checking the validations of your User model. It's important that Users have properly formatted names and emails, and you want to ensure that the validation that you wrote are working properly.

You might design a User spec as such:

``` ruby
Rspec.describe User do

  describe 'validations' do

    let(:user){
      User.new(name: name,
        email: email,
        password: "abc",
        password_confirmation: "abc")
    }

    let(:name) { 'jake'}
    let(:email) { 'jake@faris.com'}

    context 'when user has a valid name, email, and password' do
      it 'is valid' do
        expect(user).to be_valid
      end
    end

    context 'when user has a name with non-alphabetical character' do
      let(:name) { "JAKE!!!!" }

      it 'is invalid' do
        expect(user).to be_invalid
      end
    end

    context 'when user has an improperly formatted email' do
      let(:email) { "an email"}

      it 'is invalid' do
        expect(user).to be_invalid
      end
    end

  end
end
```

Let's break down what this means.

The outermost blocks in this piece of code are the `describe` blocks. These serve as semantic terms that indicate to the user what is being tested. The test is set for `User`s - specifically the `validation` components (and expressly not associations between other models, various custom methods, etc.). Read more about validations on [this blog post I wrote](http://farisj.github.io/blog/2015/07/07/validations-in-rails/).


Within a describe block, RSpec provides a `let` method, which takes a symbol and a block. `let` is a macro which generates a method definition whose name is the symbol argument and whose executed code is the block. But now the question is: the name symbol is mapped to a variable `name`, and the email symbol is mapped to the `email` variable. But where do these variables come from?

The `user` method depends on two other `let` blocks for name and email, which are defined immediately below as a default value. However, they are overwritten in the `context` blocks. A context is a semantic way of describing the situation in which your model is behaving. It answers the question of: What are the different ways I anticipate a User model to be used?

Within each context block, names and emails are overwritten to reflect the way that the model might be treated. So in each of the 3 context blocks above, when the `user` method is called, the method defined by the `let` block earlier is called and generated using the most closely scoped definition of `name` and `email`. So when we set up tests using the `it` blocks, those tests are "contextualized" to fit the scenario.

The final component of this example is the `it` block. This block is the actual test block that RSpec provides. Good design practice describes 3 steps of an test: the setup, the trigger, and the expectation. In this instance, we moved a lot of the setup and trigger to the `describe` and `context` blocks, respectively. (RSpec is very flexible in the sense that tests can be set up different ways based on the programmer's preferences and design practices.)

The "expectation" component of a test is created using the `expect` method. It takes either an argument or a block, and methods can be daisy chained on to the end to generate a human readable line of code that creates a test. Since we're dealing with validations, we `expect` that our `user` is going `to be_invalid` if it does not have a properly formatted email or name. See how those methods were essentially syntactically correct english? RSpec is not only super powerful, but aligns with the Ruby philosophy of programmer friendliness very well.

There are many other ways to use the `expect` method to test different properties of models. Some examples are listed below:

``` ruby
expect(user.name).to eq("Jake")
expect(user.age).to eq(22)
expect{user.nonexistent_method}.to raise_error
expect(user.posts).to include(@post)
expect(user.books).to_not include(@post)
expect(user.name).to match(/\A[a-zA-Z\s]+\z/)
expect(user.favorte_numbers).to contain_exactly(37,109,222)
```

These are just a few of the expectation methods that are provided with RSpec. [Read more about these here](https://www.relishapp.com/rspec/rspec-expectations/docs/built-in-matchers). I just made up a lot of these methods for the sake of example, but you get the point (hopefully).


Overall, RSpec provides a rich quantity of ways to set up and trigger various properties on your models, as well as a wide variety of ways to ensure that those properties behave as you expect. It's an invaluable tool in web development - with large code bases, it's reassuring to have a tool that can reduce the chance of improper data input or weird, seemingly unexplainable bugs that might appear weeks down the line in development. Obviously, with really complex models and architectures, it will be hard to think of every edge case from the start, but an RSpec framework provides an easy to use way to identify, reproduce, and eliminate bugs in your models as they appear. (They also have frameworks in place for feature tests and other sorts of tests as well!)

Now that I'm in round 2 of Project Mode at Flatiron School, I have been using some of these model tests to ensure that the models in the large, complex web app my team is building is working properly. It's an awesome tool and I am looking forward to becoming more and more familiar with its nuances. Peace!
