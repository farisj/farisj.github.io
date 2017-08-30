---
layout: post
title: "The Proper Way to Test Custom Controllers in Rspec"
date: 2016-07-10T21:24:03-04:00
blurb: "Using the #controller method provided by rspec"
---

# The Proper Way to Test Custom Controllers in Rails

Recently at work we spent some time refactoring one of our app's test suites to fix a few flaky tests and decrease the total amount of time the suite takes to run. That day, I ran `rspec --profile` to obtain a glimpse of what test and test groups were slowing the test suite down the most.

Most of the tests were acceptance or request specs that use Capybara or other tools to help simulate real requests from the app. However, two tests were actually controller tests! I was intrigued and dug into those files to find out more.

It turns out that both of the slow specs were testing behavior of controllers that inherit from a parent controller with some specific functionality. At work, our main app serves as a data store to some other applications that we develop. These different apps or contexts all hit specific controllers or namespaces around which we customize security and scope, among other things.

Both of these controller tests defined a new controller that inherited from the controller being tested, and then drew custom routes to hit during the test. After the test was completed, the spec redrew all of the routes to reset things back to the previous router state.

An example of how this looked is below:

```ruby
class TestController < CustomController
  def index
  end
end

describe TestController do
  before(:all) do
    Rails.application.routes.draw do
      resources :test
      end
    end

  after(:all) do    
    Rails.application.reload_routes!    
  end   

  describe "custom behavior" do
    # tests here
  end
end
```

Turns out that drawing and re-drawing routes is very costly in Rspec, as both of these tests were in the top ten slowest groups in our entire suite! (Note: A mystery still remains where when I ran these tests individually, they weren't that slow. Maybe if I figure that out you'll see it in a blog post in the future!)

Instead of defining a new controller to inherit from the one we want to test, creating new routes, and then erasing those routes once the test ends, we can test controller behavior via a built in Rspec method!

The `controller` block within an Rspec `describe` or `context` will generate an anonymous controller that inherits from the controller class given in the parent describe block. With this in mind, we can rewrite our tests like this!

```ruby
describe TestController do
  controller do
    def index
    end
  end

  describe "custom behavior" do
    # tests here
  end
end
```

Now how much cleaner is that! Pretty sweet if you ask me. It also clears up the issue of slowly redrawing routes with each test!

Go forth and test!
