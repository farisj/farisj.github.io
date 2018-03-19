---
layout: single
title: "Dotenvious, or, my first foray into Open Source"
date: 2017-07-14T17:22:36-04:00
blurb: Sharing my first journey developing my own open source project
---

## Dotenvious; or, my first foray into Open Source

As a software engineer, I rely on Open Source Software each and every day. Open Source Software (herein referred to as OSS) describes a piece of technology that is often community-driven and free to use. Common examples of Open Source Software are Ruby on Rails or ReactJS, but OSS casts a wide net - even Operating Systems (i.e. Linux, Ubuntu) can be open source.

Since I'm using software written by thousands of people from across the globe every day, I thought I'd dive further into understanding how some of this software works in hopes that one day I might be able to contribute. However - this is a nice intention but I also need an idea for an open source project too!

Remembering to keep my eyes out for any problems/projects I might be able to solve with an open source project of my own, I went back to work as a software engineer. At my job, I build web applications primarily with Ruby on Rails and EmberJS. My best bet for creating my own open source project was to find a problem that I encounter regularly and try to come up with a custom solution for my needs.

### My Open Source Origin Story

In our biggest Rails Application, we rely on a `.env` file that contains a variety of sensitive environment variables. In a production environment, these are commonly authentication credentials or variables that point to other applications across our domain. Since we're a fast paced company with constantly evolving business needs, engineers might add variables to this file at any given time as they're developing features. Generally, if a new environment variable is required, engineers will commit a fake example into a file called `.example-env`. This file is used to share domain knowledge across our team and help ensure that everyone's development environment is consistent.

With this `.example-env` file constantly changing, I found myself I always am double checking my `.env` file to make sure it's up to date with `.example-env`. This was a manual task that I noticed people on my team often forget to do, and find themselves with a broken development environment without any reason why.

Thus my idea for an open source project was born: creating a CLI to help manage the differences between our various environment variable setups.

I decided to call this project Dotenvious because it served as an extension to manage our current `.env` process - a gem called `Dotenv`. Get it? My `.env` file is 'jealous' of its fellow git-committed example files that always have the necessary variables.

I realize that there are a few other open source gems that can do this, but I loved the name and I wanted to further my understanding of gems and open source, so I decided to continue with this process for my own benefit.


### How Gems Work

To create a new gem, the easiest way is to install [Bundler](http://bundler.io/) and run `bundle gem my-new-gem`. This will create a directory called `my-new-gem` and create a few subdirectories and important files. Your directory may look something like:

```
bin/
lib/
spec/
.gitignore
Gemfile
LICENSE.txt
my-new-gem.gemspec
```

There will be a few other files in there as well. A key file I'd like to mention is the `gemspec` file. This file contains a lot of metadata and information about how your gem should be compiled among other things. You can view more information about how to develop a gem using Bundler [here](https://bundler.io/v1.15/guides/creating_gem.html) and more generally [here](http://guides.rubygems.org/make-your-own-gem/).

The typical development process involves making changes to your gem's `lib` or `bin` folders (or whatever files you're including via the `.gemspec`). As the gem is developed, one will have to recompile and reinstall the gem into your console, a la:

```
gem build my-new-gem.gemspec //=> compiles my-new-gem-0.0.x.gem
gem install my-new-gem-0.0.x.gem //=> Installs updated gem
```

From my personal experience with my company's large Rails Applications, I've found that there are two real things that I was doing when checking the `.example-env` file: adding new variables and updating variables as conventions changed.

Given these two use cases, I wanted to create a CLI where one can iterate over different variables in the `.example-env` file.

### Working with Dotenvious

When you run `dotenvious` in the terminal, Dotenvious will compare the `.env` and `.example-env` (by default) and evaluate whether or not they have any differences. If there are any differences, Dotenvious will prompt you to remedy any differences:

```
You have missing ENV variables. Examime them? [y/n]
```

As the CLI takes the user through the differences it has discovered, it will either prompt the user to add a new variable or overwrite one that has a different value:

```
MY_SECRET_KEY=secret
Add to .env? [y/n/q]
```
or

```
ENV[MY_AUTH_TOKEN] is set to: the0dev8token
Example [MY_AUTH_TOKEN] is set to: example-dev-token
Replace with the example value? [y/n/q]
```

As Dotenvious takes the user through every variable in their example file, they will see a variation of one of the above messages. After this is done, Dotenvious will sort the collection of variables by key, keeping things organized in case the user wants to edit the file manually.

It also will read a `.envious` file in one's repository, which looks similar to:

```
Dotenvious::Configuration.new do |config|
	config.example_file = '.env.example'
	config.custom_variables = ['MY_LOCAL_VAR']
	config.optional_variables = ['GOOGLE_AUTH_TOKEN','OTHER_AUTH_TOKEN']
end
```

On load, Dotenvious will check to see if this file exists and if so, load whatever customizations are present. If `example_file=` is set, Dotenvious will check that file as the example file to compare to `.env` (it is `.example-env` by default). Additionally, `custom_variables` will permit whitelisted variables to differ from what is set in the example file (it won't prompt the user if they are different). Thirdly, it won't prompt the user for `optional_variables` either.


### My First Feature Request

I announced the Dotenvious project to my coworkers and it was pretty well received! As far as a first iteration of an open source project goes, the general feedback I received was positive - it solved the problem my coworkers were having where they had to scan through the `.example-env` file for a newly added/changed variable and manually copy it over to their `.env` file. Dotenvious provided a more automated solution to this while still giving the user full control over the relationship between these two files.

One day, a coworker on a different team approached me with a question. Did Dotenvious work with YML files? They were working on a project in a repository I wasn't using, trying to wrangle their CircleCI setup to work correctly. As an experiment, he was trying to import some variables in a `.yml` file to his development environment.

At the time, no, Dotenvious did not work with `.yml` files. This meant that I had received my first feature request!

Due to the nature of Dotenvious' internals, this was actually a pretty easy addition to make! Simply moving a few lines of code around, I was able to add this feature pretty quickly.

### Future Work

I have some hopes for the future of Dotenvious. I hope to expand the CLI's functionality to include the ability to add variables to the optional or custom whitelists stored in `.envious` on the fly. Another feature request that has been asked of me is to export variables from `.env` directly into the terminal. (This might be dangerous, so I've been hesitant to do this.)

At any rate, my experience building Dotenvious has been hugely helpful in providing insight into the development process of other open source projects I appreciate or use frequently. Here's hoping that another open source project will be near in my future! :D
