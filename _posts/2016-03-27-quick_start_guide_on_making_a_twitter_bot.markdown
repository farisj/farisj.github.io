---
layout: single
title: "Quick Start Guide On Making A Twitter Bot"
date: 2016-03-27T17:20:00-04:00
blurb: "How to set up a twitter bot quickly using Heroku"
---

# Quick Guide on How to Make A Twitter Bot

This post will highlight the important points needed to get a Twitter bot up and running using Ruby and Heroku.

### What's a Twitter Bot?

A Twitter bot is a script that someone writes to automatically tweet content out into the Twitter-verse. Some simply tweet predetermined content, as [@everyword](http://www.twitter.com/everyword) and its many spinoffs like [@CyberEveryword](http://www.twitter.com/cybereveryword) and [@everywordisgay](http://www.twitter.com/everywordisgay). Many others use language or twitter data to create content that *almost* feels real, like [@TwoHeadlines](http://www.twitter.com/twoheadlines) or [@ThinkpieceBot](http://www.twitter.com/thinkpiecebot). Some even try to pretend (or pretend to pretend) to be human users, like the angsty teenage bot [@oliviataters](http://www.twitter.com/oliviataters), or, in more topical news, [Microsoft's AI bot that turned into a racist, sexist Nazi within 24 hours of its creation](http://www.theverge.com/2016/3/24/11297050/tay-microsoft-chatbot-racist).

Some of my recent side projects have been Twitter bots. I've got two up and running at the moment: [@everyorb](http://www.twitter.com/everyorb) and [@realness_bot](http://www.twitter.com/realness_bot). While I'm only getting started, they are great little side projects that continue to tweet to this day (no maintentance needed!) and I'm really excited to share a quick start guide with you!

![](http://i.giphy.com/3M9X3Oft980eI.gif)

### So, how does it work?

This post will outline the steps needed to get your bot up and running. To begin, you will need:

1. An idea (can't help you with this!)
2. A phone number that is not connected with any current Twitter account
3. A Heroku account

### The Idea

This can really be anything - personally, mine came to me all of a sudden when I wasn't expecting them. However your creative process works, this is really the most important part.

### Registering a new Twitter Account

With your idea in mind, go ahead and register a new Twitter account with the handle that you want your bot to have.

### Register a phone number with a Twitter Developers account

In order for your bot to periodically make tweets, we're gonna have to use the Twitter API to send our messages out.

In order to use the API, we need to add a phone number to the twitter account (do this in Settings of the bot's account) as well as register with [dev.twitter.com](http://www.dev.twitter.com).

At the bottom of the above page, you'll see a variety of links. Click the "Manage Your Apps" button:

![](http://imgur.com/JXsTVOJ.png)

In the following page, in the top right corner, click "Create New App":

![](http://imgur.com/EfNIhKB.png)

On the following screen, you'll have to fill out the pertinent data to your Twitter "App." For the website field, I just put the Twitter URL for this bot.

![](http://imgur.com/IpfjhLO.png)

Scroll to the bottom of the page, read and agree to the developer agreement, and hit "Create Your Twitter Application."

Cool! Now we've got an app up and running. On this page, head to the "Keys and Access Tokens" tab to generate our keys to use with the API.

![](http://imgur.com/xtJ13o3.png)

Now, you'll see that Twitter has generated for you Consumer API Public and Secret Keys. These are half of the keys that we'll need for the bot. At the bottom of this page, you will see a prompt to "Create my access tokens." Let's hit it!

![](http://imgur.com/9BUBE6z.png)

Voila.

![](http://imgur.com/vjsAJau.png)

Great, now we have all of the authentication information that we need to use the Twitter API!

### Using the Twitter API

With all of our auth info, we can actually tweet via the bot scripts! In my experience, I've used the `twitter` gem in Ruby for my bots. To do this, I added a simple Gemfile that looked as such:

```ruby
source 'https://rubygems.org'

ruby '2.2.1'

gem 'twitter'

group :development, :test do
  gem 'pry'
end
```

Add whatever other gems you may be using for your bot.

After running `bundle install`, let's open up our bot script file. Mine is called `bot.rb`, but it can be named whatever you want!

At the top of your script, `require 'twitter'`. Then at the bottom, after your script has done its magic to craft the perfect tweet, you want to initialize the Twitter client and push your work to the Twittersphere.

Initialize the client as such:

```ruby
client = Twitter::REST::Client.new do |config|
  config.consumer_key = ENV['consumer_key']
  config.consumer_secret = ENV['consumer_secret']
  config.access_token = ENV['access_token']
  config.access_token_secret = ENV['access_token_secret']
end
```

In this context, the keys, tokens, and secrets are those which we registered in the Twitter Developers Console earlier. On your local machine, you can set these values directly, but don't commit them anywhere! These are the only way that the Twitter API recognizes your account. In the wrong hands, anyone could tweet anything from your bot with that data.

Now, assuming your tweet is stored in a variable named `tweet`, push your work to the internet by:

```ruby
client.update(tweet)
```

Now, let's commit our work and push to Github, so that we can push to Heroku and get this thing up and running!

**Note:** It is important that you add both the `Gemfile` and the `Gemfile.lock` to your repository, or Heroku will have trouble building later on.

(I'm not going to show you how to make a commit and push to a repo, this guide assumes you already know this. If you don't, there are plenty of gudes out there on the web which can explain! Go read one of those and then come back here if you need to.)

Hey look at that, we're on Github! So much progress has been made so far. Pat yourself on the back.

![](http://i.giphy.com/XreQmk7ETCak0.gif)

### Deploying to Heroku

Now that we have a script up on a Github repository, we want to deploy to Heroku, a web hosting service which we can use to run our bot script periodically and post tweets.

Log in to your Heroku account and in the top right corner you will see a `+` that turns into a drop down menu. Click that and then Create New App. Give it a name (if you want, it really doesn't matter since this application won't have a real interface/website) and Create That App!

![](http://imgur.com/gU25wI2.png)

Now you're on the dashboard. This is where you can see all the information about your app (or in our case, bot).

We now want to connect Heroku to Github so that it can read the code and host it on its servers.

On the Deploy page, select the Github Deployment method and search for the repository you previously pushed to. Connect that repository.

![](http://imgur.com/WBhgjNW.png)

At the bottom of this page now, you will see an option for Manual Deploy. Let's do that.

Without any problems, you should see a successful build and deployment!!

![](http://imgur.com/IclRu7L.png)

Go Team!

![](http://i.giphy.com/TEFplLVRDMWBi.gif)

### Adding the environment variables

Remember the Twitter API keys we generated earlier and didn't commit to our repository? We're going to add those in to the backend of the "app" on Heroku. Click on the Settings tab and you will notice a "Config Variables" section. Reveal those variables and add all four of the keys generated earlier with the same variable names as those you put in your bot script.

![](http://imgur.com/eZ6VsNh.png)

### Making it Run

Now that our script is on Heroku, we need to tell it to run our bot script periodically. To do this, we need to add a Resource to our application. Head over to the [Heroku Add-on Store](https://elements.heroku.com/addons). Since I'm content with my bot running once an hour, I used the free [Heroku Scheduler](https://elements.heroku.com/addons/scheduler) dyno. Install this dyno into your application.

| ![](http://imgur.com/13QUatK.png) | ![](http://imgur.com/YzzN5yY.png) |

Now you should see the Scheduler appear in your app's list of add-ons. Lets configure the scheduler to run our script.

![](http://imgur.com/QDeLWwa.png)

Add a new job and you will see a series of settings for executing commands! Fill these out (an example below) and hit save.

![](http://imgur.com/K51ScyM.png)

..and that's it! You'll need to wait a moment for the script to run to see that everything is going smoothly. I always start out with my Heroku Scheduler dyno running every ten minutes just to make sure things are good, and then change it to an hour later.

Congrats! You now have a Twitter bot! Now go forth and share your creation with the world.

![](http://i.giphy.com/wiNiBviTrV6ww.gif)
