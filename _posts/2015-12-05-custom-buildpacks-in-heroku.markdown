---
layout: single
title: "Custom Buildpacks in Heroku"
date: 2015-12-05T19:00:09-05:00
blurb: "Discussing the structure of custom Heroku Buildpacks"
---

So picture this - you're building a web app and hosting it on [Heroku](https://www.heroku.com/). The frameworks and tech stack you're using, however, has some configuration problems and don't quite push to Heroku in the way that you would like - maybe your repo doesn't quite have the structure you need on site, something that was abstracted away locally by something or other. What should you do?

### Enter: Custom Heroku Buildpacks!

Heroku allows for applications to take any buildpack that the admin specifies and runs that buildpack before deployment. To add a new buildpack to your app, simply run

~~~
heroku buildpacks:set https://github.com/some/buildpack.git -a myapp
~~~

in the terminal and you should be good to go.

##### So what do these buildpacks look like, exactly?

According to the [Heroku docs](https://devcenter.heroku.com/articles/buildpack-api), buildpacks contain three main files which are executed at runtime:

* `bin/detect`: Determines whether to apply this buildpack to an app.
* `bin/compile`: Used to perform the transformation steps on the app.
* `bin/release`: Provides metadata back to the runtime.

#### `bin/detect`

The `detect` file will determine if the application this buildpack is mounted on is suitable for the buildpack itself. It's essentially a safeguard to ensure that you're working within the right frameworks for whatever your buildpack is about to do.

It receives one argument, `BUILD_DIR`, which is the root of the app itself. From there `detect` should analyze the file structure to see if it is compatible (for instance, if you need it to be a Rails app, make sure there's a Gemfile and config.ru).

#### `bin/compile`

This file actually executes what you set out to do with this buildpack. This can be whatever you want!

In addition to `BUILD_DIR`, it also receives a `CACHE_DIR` argument, which represents a directory that the buildpack can use to cache files between builds.

Additionally, it receives a third argument, `ENV_DIR`, which is a file directory that contains all of the environment variables provided from the Heroku setup. In this directory, the file name is the variable name, and the file contents are the variable's value.

#### `bin/release`

An optional file to be run, this program will return a list of `addons` and `default_process_types` for the application to use, in YAML format. `addons` is only executed on the first deployment. If you're trying to add additional technologies in the app based on the buildpack, this is a good place to do that.

---

With this information, it's easy to do a variety of things before deployment to Heroku. A variety of custom buildpacks are listed on Heroku's website [here](https://elements.heroku.com/buildpacks), but go ahead and make your own if nothing quite fits your use case!
