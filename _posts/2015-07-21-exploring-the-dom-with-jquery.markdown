---
layout: single
title: "Exploring the DOM with jQuery"
date: 2015-07-21 00:11:49 -0400
blurb: "Exploring how JQuery traverses the DOM"
---

We've been diving into Javascript and jQuery this past week at Flatiron. jQuery is a Javascript library which provides a large amount of increased functionality and ease in manipulating data presented to the user in a web browser.

One of the key elements of this functionality that jQuery provides is the ability to navigate the DOM with ease. The DOM, or Document Object Model, is a property of HTML which describes the relationships that different elements on your page have with one another. This results in a tree-like structure, where `<html>` is usually the head of the tree, its children are `<head>` and `<body>`, and its subchildren are things like `<meta>`, `<script>`, `<title>` for `<head>`, and all visible HTML elements for `<body>` (`<div>`, `<p>`, there are so so many), which are then nested so on and so forth. Let me demonstrate how jQuery can be used to traverse this tree with ease.

Consider the following HTML structure:

```html
<div id="results">
  <h3 id="header">American Idol Winners, by Season</h3>
  <p id="first">Kelly Clarkson</p>
  <p id="second">Ruben Studdard</p>
  <p id="third">Fantasia Barrino</p>
  <p id="fourth">Carrie Underwood
    <span id="bice">Runner up: Bo Bice</span>
  </p>
  <p id="fifth">Taylor Hicks
    <span id="mcphee">Runner up: Katherine McPhee</span>
  </p>
</div>
```

I'm going to give you a rundown of a few basic jQuery traversal methods, as well as ways to enhance those manipulations using filtering.

Let's say we just want to have access to the winner of season 1, Kelly Clarkson. We can "find" her paragraph tag by selecting the entire div and calling `.find` using her id.

`$('#results').find('#first')`

Okay, so we've got Kelly. What about Ruben? We know he's the winner of the *next* season. (hint, hint)

`$('#results').find('#first').next()`

`.next()` (and its sister `.prev()`) find the adjacent sibling to a given DOM object in the forward or backward direction, respectively.

Let's say that we want to figure out who won the season where Bo Bice was the runner up. We know Bo Bice is contained in in the `<span id="bice">` tag, but how do we raise ourselves up to the outer level of the DOM? Answer: `.parent()`.

`$('#bice').parent()`

Cool! The DOM provides us with a rich relationship structure that we can manipulate with ease with jQuery.

Let's say that we want to access all of the winners of the seasons at once. We can use `.siblings()` to do that, as such.

`$('#first').siblings()`

But wait... that also will contain the `<h3 id="header">` tag! That's not what we want...

Luckily, jQuery has a lot of filtering options that we can use to get more specific in our selections.

We can filter out the unwanted tag by using the `.not()` method.

`$('#first').siblings().not('h3')`

Awesome! (We could also have done this by using `$('#first').siblings('p')`.)

What if we want to find all of the result tags where we know the runner up? Assuming the runners up are all contained in a `<span>` within the `<p>` tag, we can say:

`$('p').has('span')`

This query will return all `<p>` tags that contain a child `<span>` tag.

One last one for this post - let's say that we want to get all the siblings of an object until the structure of our DOM changes. In our instance, lets consider another way to get all the first three season winners, by traversing down our DOM and stopping when we hit the fourth:

`$('#first').nextUntil('#fourth')`

What the above code does is it finds the `<p id="first">` element, then finds all siblings of that element "until" we reach an element that fits the parameter `'#fourth'`. A parallel function can be found in `.prevUntil()`. This method has a lot of powerful potential when we're using other parts of our web app (Rails ERB anyone?) to dynamically generate an arbitrary amount of HTML.

jQuery is an extremely powerful and flexible language. There's are so many different ways to traverse your page using these simple tools that navigate up, down, and along the DOM. I am really looking forward to integrating jQuery into a web application and create web pages that can dynamically evolve and look awesome.
