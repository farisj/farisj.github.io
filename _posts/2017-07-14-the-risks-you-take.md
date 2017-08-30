---
layout: post
title: "The Risks You Take When You Use Ember.computed.oneWay"
date: 2017-07-14T17:22:36-04:00
blurb: "Undersanding a few important computed properties"
---

(Note: this post originally appeared on my company AlphaSight's blog. You can view that version  [here](https://m.alphasights.com/the-risks-you-take-when-you-use-ember-computed-oneway-f2c008a14404).)

# The risks you take when you use Ember.computed.oneWay

[Ember](https://www.emberjs.com), the framework for building Ambitious Applications(tm), provides developers a number of tools to build apps quickly. One of the most used tools in this repertoire is `Ember.computed`, a [suite of functions that transform data](https://www.emberjs.com/api/classes/Ember.computed.html) so that it can be used and manipulated across your application.

Some of the functions provided by `Ember.computed` are pretty straightforward. `Ember.computed.empty` takes a stringified key for an array, and returns true or false depending on whether or not that dependent property contains any elements. `Ember.computed.gte` takes a stringified key that represents a number and an actual number, and returns true or false based on whether or not that dependent property is greater than or equal to that second number.

While the more mathematical computed functions are simple to understand, there are a few more tools provided by `computed` which need clarification.

## alias, readOnly, oneWay

There are several functions that provide a way of renaming or reassigning other values for use in an Ember Object. Don't feel like writing `get(this, 'todo.user.name')` over and over, and wish there was a faster way? Ember's got ya covered.

### alias

`Ember.computed.alias` provides a bi-directional data flow property in your Ember object that allows you to write a shortcut to another property. Referencing the above example, if you kept referencing `todo.user.name` repeatedly in your Object, you could write a `computed.alias` to make `get` and `set` actions a few characters shorter.

```js
import Ember from 'ember';

const { computed } = Ember;

export default Ember.Component.exend({
	todo: null,
	userName: computed.alias('todo.user.name'),
	//...
});

```

With a set up like this, you can reference `todo.user.name` elsewhere in your component (and its template!) without reaching through `todo` and `user` every single time.

With `computed.alias`, `get` and `set` actions onto the aliased property _behave exactly as if you were referencing the original property_. This means that if you're using a component to `set` the `userName`, that would be reflected on that User model in the store.

### readOnly

`Ember.computed.readOnly` is _almost_ the same thing, with one very important difference. The set up for a `readOnly` property is similar:

```js
import Ember from 'ember';

const { computed } = Ember;

export default Ember.Component.exend({
	todo: null,
	userName: computed.readOnly('todo.user.name')
	//...
});

```

In this setup, calling `get` to userName is exactly what you expect. However, trying to call `set` on this property actually raises an error! You would see this in your devTools if you called `set` on a `readOnly` property:

`Cannot set read-only property 'userName' on object: <twiddle@component:my-component::ember323>`

The `readOnly` function provides us a healthy reminder about what we should and should not be setting in our application.

### oneWay

The `Ember.computed.oneWay` function is the trickiest one to understand, and also one that is potentially dangerous! Let's explore why.

The setup is essentially the same...

```js
import Ember from 'ember';

const { computed } = Ember;

export default Ember.Component.exend({
	todo: null,
	userName: computed.readOnly('todo.user.name')
	//...
});

```

With `oneWay`, properties act as an alias to their dependent property _until they are set_. This means that the two properties, once representing the same data, _diverge_ and no longer represent the same value. Suddenly, the data flow has been broken. The original property and `oneWay` property no longer have a connection, as it's understood that the `oneWay` property has been updated in a way that isn't expected to affect the original object.

### What Does This Mean For My Application?

The `computed.oneWay` property can be great for things like forms, where you really don't want a new input to immediately persist a change to a model's field in the Ember Store. However, certain code practices can cause `oneWay` to unexpectedly create incorrect displays to the user. Let's explore how.

Consider you have a collection of `Todos` and are using computed properties to display the data about those todos to the user. Consider the following template:

```hbs
//templates/application.hbs
{% raw %}
<div class="todos">
  <div class="selected-todo">
    {{#if selectedTodo}}
      {{todo-display todo=selectedTodo}}
    {{else}}
      <div>Please View A Todo.</div>
    {{/if}}
  </div>
  <br>
  <div>
    {{#each todos as |todo|}}
      <button {{action (mut selectedTodo) todo}}>
      	View {{todo.title}}
      </button>
    {{/each}}
  </div>
</div>
{% endraw %}
```

and in the `todo-display` component/template:

```js
import Ember from 'ember';

const { computed, set } = Ember;

export default Ember.Component.extend({
  todo: null,
  title: computed.oneWay('todo.title'),
  body: computed.oneWay('todo.body'),

  actions: {
    updateBody(){
      set(this, 'body', 'jk, do this thing. haha! youre stuck doing this thing instead');
    }
  }
});
```

```hbs
{% raw %}
<div class="todo">
  <div class="todo__content">
    <h3>{{title}}</h3>
    <div>{{body}}</div>
  </div>
  <button {{action "updateBody"}}> Update body </button>
</div>
{% endraw %}
```


This example's `application/template.hbs` provides a few buttons to set a `selectedTodo` property, and pass that into the `todo-display` component.

In the `todo-display` component, we are using `computed.oneWay` properties to display the `todo`'s title and body. We also have an action that overwrites the component's `body` property.

The entirety of this example is below in an Ember Twiddle.

<div style="position: relative; height: 0px; overflow: hidden; max-width: 100%; padding-bottom: 56.25%;"><iframe src="https://ember-twiddle.com/12ca19ecb67bcc6b83ebd8d1368cb66a?fullScreen=true" style="position: absolute; top: 0px; left: 0px; width: 100%; height: 100%;"></iframe></div>

In this scenario, toggling between Todos at first updates all of the data in the `todo-display` component correctly. However, after the first time the `updateBody` action is called, the data flow from `todo.body` to `body` on the component is broken. Thus, every subsequent toggle to a different Todo does not update the `body` property in the component.

This can be dangerous because your application suddenly has mixed state. Now, your users are able to view Todo B with a different body that was actioned when viewing Todo A. If you were performing an action based on the `todo-display`'s `body` and `title` properties, your action may not have the state it was intended to!

This behavior can be useful in other cases. For example,`Ember.computed.oneWay` can be useful when building a form _for a single model_. As long as the component is properly destroyed, it would not have a chance to affect another model. If we're using the same component _for multiple models_, that's where the problem occurs.

### How Can I prevent this?

One of the problems with this setup is the fact that different `todos` are being passed into the same `todo-display` component, and the state of that component is not being reset correctly whenever data changes. One fix to this could be:

```js
//components/todo-display.js
import Ember from 'ember';

const { computed, get } = Ember;

export default Ember.Component.extend({
  todo: null,
  title: computed.readOnly('todo.title'),
  body: null,

  didReceiveAttrs(){
  	let body = get(this, 'todo.body');
  	set(this, 'body', body);
  }
  //...
});
```

By hooking into the lifecycle of a component, we ensure that we are setting up our component the same way every time a new `todo` is passed in. Now, our `todo-display` component is more robust and does not display our todos incorrectly!

These are but a few of the powerful tools provided by `Ember.computed`. Now that you've read this post and understood the differences between `alias`, `readOnly`, and `oneWay`, go forth and create some side-effect-free components! Cheers ;)
