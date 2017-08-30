---
layout: post
title: "10 Things They Don't Teach You in Bootcamp or College"
date: 2016-08-14T17:22:36-04:00
blurb: "Some thoughts on my first year on the job"
---

### 10 Things They Don't Teach You About Being A Software Engineer In College or At Bootcamp

One year ago this week, I graduated from the Flatiron School, enhancing the skills I obtained from my traditional degree in Computer Science the May before. While I learned a lot about Data Structures, Algorithms, Computer Organization and other things, these concepts were rather conceptual. To complement this type of education focused on developing critical thinking skills and probem solving, I augmented my studies with a stint at a web development bootcamp here in New York City. There, I learned about the fundamentals of Rails and MVC architecture, among other things.

While I have been very fortunate to have both of these types of education, there are a lot of things that one can't simply learn in the classroom, that only actual real-world work experience can teach. A lot of my first 9 months in the workforce has been spent understanding how the real world development process works, while building my technical and business skills as a Software Engineer. In that time, I've come to realize that there are several things that occur in the workforce that my traditional and nontraditional educations didn't teach me. Here are the top ten:


### 1. Working in Large Apps

This is something that bootcamps will warn you about: on your first day as a real programmer, you'll dive into a codebase that dozens of people have been developing over the years. You won't understand much of it at first. This idea is brought up a lot at bootcamp, but it's much more overwhelming than you may think.

My first time looking at the code of a real Rails application, I thought to myself, "Okay, yes, this is the `app` folder, we have `models`, `views`, and `controllers` subdirectories, I know what those do. Easy!"

Then you open up one of those subdirectories and see ten more subdirectories, divided into different parts and purposes of your application. It's a blow to the ego for sure: you know a lot less than you think you do when you're fresh out of college and bootcamp. Which leads me to my second point...


### 2. Accepting You Won't Know It All

Unless you're running `rails new new_startup_app` on the first day, you're going to be working with code that has been written in the past, possibly years earlier, possibly by previous employees that have since moved on from your current company. The code may have been written in a way that isn't intuitive at first. It could be designed to accomplish a specific business purpose. The reality is that very little code is as pristine perfect Ruby code as one is able to write at a bootcamp when working with only 3 or 4 resources. Things get complicated.

The truth is, if you've snagged a job a software company, they see the potential in you to wade through all of the code that has been developed and contribute to it in a meaningful way. This is where your peers and coworkers come in - they are there to provide you a wealth of knowledge that comes with having been at your company before you. It might take a new engineer 4 or 5 hours to understand a subset of business logic on their own, but with the help of a coworker who knows the material well, that time can be reduced significantly and get you moving on the project faster.

My conclusion to this is: ask questions! One individual engineer will never know as much as a group of engineers put together.

### 3. Accepting You Can't Know It All

Chances are, you'll be working with an external API or two (or ten) to abstract away the maintenance of certain parts of your technical stack. Very few companies have the time or resources to build *everything* in-house - and few would want to anyway (why reinvent the wheel, right?). You might find your company using the Google Calendar API to schedule meetings, or the Twitter API to share updates on your company's brand. There are a million different APIs out there to augment your work.

There have been a few moments where I personally have struggled with the idea that I will probably never understand fully the way these APIs work. Yes, the Twitter API is by and large RESTful, and I can say I'm pretty comfortable with REST as a concept. It's obvious there's much more going under the hood than a simple request/response lifecycle, and I probably won't get to know all of it since I don't have access to that code base.

Trying to know everything is unsustainable - one simply cannot keep all the nuances of every API one relies on in their head. Even Ruby on Rails is a beast that only a select few completely understand. It's okay to accept that it's impossible to understand the entire scope of every piece of technology that you work with - that's why there's documentation!

### 4. Race Conditions & Other Unexpected Problems

This one was definitely a huge reality check personally - in my work, dealing with race conditions of asynchronous methods has been a challenge that I was certainly not expecting. At Alphasights, we have historically used DelayedJobs in our Rails apps for a lot of asynchronous processing. In the time I've been here, we've had many discussions regarding this infrastructure and have decided to move a lot of business logic over to a RabbitMQ server and process that work with Sneakers. It's been great to be involved in the conversations that have guided that decision and better understand best practices for asynchronous work.

It's not that race conditions are a particularly difficult problem to understand - what was so surprising to me about encountering such a huge problem in real world engineering is that I hadn't really learned much about it in college or at bootcamp. This particular problem highlighted the distinction between "big" problems in Computer Science and the problems one faces on a daily basis in the real world.

### 5. User Behavior

Looking back, it's obvious now, but it does need to be said: users never behave as you expect them to.

Not only is there unintended behavior as one may anticipate (users not filling out a form in the correct format, for example), users will always suprise you in ways you didn't expect. When they may seem obvious in retrospect, it's hard to anticipate everything.

This is really part of the lifecycle of a product - users will always find use cases or problems with your app. It's impossible to cover every case 100% of the time, but it's important to design products that can handle unexpected change.


### 6. No one Cares about the Rules

Both here at my current job and at my internship doing iOS development my last summer of college, my bosses and peers were way less concerned about the "rules" of engineering that I had expected from my experiences in college and bootcamp. The primary example of this was with respect to databases. In my Databases class in college, I was instructed to reduce redunancy in databases by removing unnecessary functional dependencies. We learned about Boyce-Codd Normal Form, 3rd Normal Form, and other ways to keep data as DRY and elegant as possible. When at my internship, I focused a lot on creating tables that adhered to these rules.

To my surprise, functional dependencies and minimal redundancy was lower on the priority list than other things - performance being at the top of the list. The price to pay for these types of normalization is too high with regards to speed. This was particularly difficult for me to wrestle with - I wanted in my heart for all code to be beautiful and elegant and *perfect*. The solutions that the team agreed upon ended up being very successful, yet muddled with real-world business requirements that make the work less elegant than I had imagined reality to be.


### 7. Code Reviews

At the end of my 3-month bootcamp, we broke out into teams of 3 or 4 and build small-scale web apps to demonstrate the skills we had learned throughout the bootcamp. We were encouraged to imitate a "real-world" workflow by checking out our own branches, committing our work there and submitting pull requests for our teammates to review before merging to the master branch.

While this workflow was a valuable warm up to the ways PRs are managed at my job currently, these Pull Requests were often smaller and we didn't totally know what we were doing. In the case of my 2 week sprint, the 4 of us pushed up and merged about 180 pull requests total. We moved fast and messy throughout that project, and the small commits we had to review didn't seem too complex on an individual level.

In the real world however, Pull Requests generally need to be much more purposeful than what I wrote in bootcamp. Of course, producing PRs that encapuslate the smallest cohesive unit of work as possible is still a goal. However, some features aren't always so small that the specs can be glossed over and given a "LGTM" and approval to merge.

Something that I still struggle with to this day as a Junior Engineer is how to review my peers' Pull Requests. It's important to ensure that only quality code is committed to our applications. Yet, when a Pull Request isn't be one commit of ~20 lines, much more detail is necessary. So what is the best way to go about reviewing a feature PR that's +300/-150? This is something I still am trying to understand - I think that this will come as I continue to gain experience as an engineer.

### 8. Allowing Features to Evolve

As a newer Engineer, I try my best to make sure that every line of code that I write is crafted with quality, setting a standard for myself that I hope to carry out for the rest of my career as a Software Engineer. As such, it's easy to get into the mentality of being proud of one's code and to think that it's the best code out there for solving your particular problem.

The issue with having a sense of pride about one's code is that in the real world, the problems that need to be solve rarely can be defined with full scope early on. This means that a commit you make today might be useless tomorrow.

As a new engineer who wants to be as productive as possible, it doesn't feel great to throw out code that I've written just to rewrite it to solve a slightly different problem.

Even though I might not get to use that perfectly abstracted implementation of a feature that I had spent so much time on, I still developed as an engineer because I had to think up a solution to that problem. It took me a few months to realize that: every opportunity to write code is an opportunity to improve your skills as a programmer.

### 9. Balancing Business vs Technical Needs

A quality that AlphaSights looks for in its Engineers is the ability to understand, appreciate and balance business requirements against technical requirements when discussing feature implementations. My bootcamp touched on this briefly, but at work it's something that I think about every day. I think this notion is what distinguishes a Software Engineer from just a Programmer or Developer - being able to focus on the end goal of a business or product while simultaneously being attentive to the code one writes daily.

### 10. Perfomance

My traditional Computer Science degree focused a lot on Data Structures and Algorithms. Those classes and discussions gave me the foundation for my critical thinking skills and ability to dissect a technical problem into minute parts and address them in efficient, practical ways.

Those studies came from understanding the way that Quicksort versus Binary Search work, or walking through each step in Djikstra's algorithm to find the maximum flow throug a graph. Working on a business product however, I don't really need those things on a regular basis.

Yet, learning those things did make me aware of the problems that come with writing inefficient and nonperformant code. The types of performance issues that I need to think about nowadays are things like ensuring I don't have N+1 queries when hitting the database, or optimizing my asynchronous Javascript calls for data when a user loads a front-end application. While I never learned those particular things in college, having a fundamental understanding of algorithms has made me keener to noticing this behavior.

---

I'm not at all saying that my CS degree and bootcamp crash course were not valuable - they equipped me with many skills that have helped me succeed as an engineer in my first job. I would definitely encourage anyone interested in tech to pursue those paths of knowledge to achieve their goal of becoming a programmer. However, the reality is that the only way to really understand what it takes to be a Software Engineer is to actually be one.
