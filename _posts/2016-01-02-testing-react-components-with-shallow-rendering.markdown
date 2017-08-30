---
layout: post
title: "Testing React Components With Shallow Rendering"
date: 2016-01-02T21:44:54-05:00
blurb: "Using Shallow Rendering to only render the DOM in question"
---

Facebook's React provides developers with a lightweight frontend framework that creates views using a blend of Javascript and HTML in what are known as components. While React can be written using pure Javascript, our team constructs these components using JSX - our team prefers it because it dramatically improves readability and has no impact on performance since it transcompiles to pure Javascript on every build. While it will take time to transcompile, a modern web app will need a build step anyway.

Components are designed to be modular and reusable, which actually makes them great for unit testing! This post assumes you have a moderate understanding of React, but let's look at an instance of two React classes that interact with each other - a simple `TodoList` and some `Todo`s.

``` javascript
import React from "react";

import Todo from "components/todo";

class TodoList extends React.Component{
  render(){
    var { todoTexts } = this.props;
    return (
      <div className="todo-list">
        <span className="greeting">Welcome. Here are your todos.</span>
        {
          todoTexts.map( (todo, index) => {
            return (<Todo key={ `todo-${index}` } text={ todo } />);
          })
        }
      </div>
      );
  }
}

TodoList.displayName = "TodoList";
export default TodoList;
```

This `TodoList` is pretty straightforward - it greets the user with a simple `span` and then lists as many `Todo`s as were given to it via its parent (let's assume the parent of the `Todo` is contacting a store or API or whatever to gather exactly what are the todos).

Now, like good developers, let's establish a test suite to ensure that our `TodoList` does exactly what we want it to.

### Enter: TestUtils!

React provides a nice set of test utilities that allow us to inspect and examine the components we build.


There are several different uses, but I want to discuss the use of __Shallow Rendering__. Shallow rendering allows us to inspect the results of a component's `render` function, and see check what HTML and React components that particular component returns.

So, just to be sure, let's make sure that the `TodoList` returns an element with class "todo-list", that it has a child element that greets the user (we can guess this by it having a class "greeting"), and that it will render `Todo`s for all of the Todos that are given to it.

A test suite for that might look like this:

``` javascript
import chai from "chai";
import jsxChai from "jsx-chai";

import React from "react";
import ReactDOM from "react-dom";
import TestUtils from "react-addons-test-utils";
import shallowTestUtils from "react-shallow-testutils";

import TodoList from "app/components/todolist";
import Todo from "app/components/todo";

chai.use(jsxChai);

var expect = chai.expect;

describe("TodoList", function(){

  var renderer, todolist;

  beforeEach(function(){
    var todoTexts = ["eat breakfast", "have lunch", "go out to dinner"];
    renderer = TestUtils.createRenderer();

    renderer.render(
      <TodoList todoTexts={ todoTexts } />
    );

    todolist = renderer.getRenderOutput();
  });

  it("renders a todolist", function(){
    expect(todolist.props.className).to.equal("todo-list");
  });

  it("greets the user", function(){
    var greetingSpan = todolist.props.children[0];

    expect(greetingSpan.props.className).to.equal("greeting");
  });

  it("will render three todos", function(){
    var todos = shallowTestUtils.findAllWithType(todolist, Todo);
    expect(todos.length).to.equal(3);

    var dinnerTodo = todos[2];
    expect(dinnerTodo).to.deep.equal(
      <Todo key="todo-2" text="go out to dinner" />
      );
  });
});
```

Woah, okay, that's a lot of information. Let's break it up piece by piece.

#### Loading Dependencies

The dependencies that I've loaded up are as follows:

``` javascript
import chai from "chai";
import jsxChai from "jsx-chai";

import React from "react";
import ReactDOM from "react-dom";
import TestUtils from "react-addons-test-utils";
import shallowTestUtils from "react-shallow-testutils";

import TodoList from "app/components/todolist";
import Todo from "app/components/todo";

chai.use(jsxChai);

var expect = chai.expect;
```

Let's go through these piece by piece.

To construct expect statements, I'm using Chai and jsxChai. These libraries will allow us to build expectations directly against JSX components, like `expect(component).to.equal(<Todo />)` for example.


As of React 0.14.0, the main functionality of React is split into two dependencies, `React` and `ReactDOM`. The former consists of all of the fundamental logic that React relies upon, and the latter gives us the virtual DOM into which we can render components and verify that they render the way we want to in the tests.

`TestUtils` and `shallowTestUtils` are two other optional React libraries provided by Facebook itself. They are going to be the primary libraries which will allow us to artificially render our components and inspect their inner workings.

Finally, we're testing the `TodoList`, so we need that, as well as its inner component, `Todo`, which we'll use in a test later on.


#### The Setup

``` javascript
  beforeEach(function(){
    var todoTexts = ["eat breakfast", "have lunch", "go out to dinner"];
    renderer = TestUtils.createRenderer();

    renderer.render(
      <TodoList todoTexts={ todoTexts } />
    );

    todolist = renderer.getRenderOutput();
  });
```

This `beforeEach` simply sets up a suitable test environment for each test. Using `TestUtils.createRenderer()`, it generates an object which can superficially render a React component and return that object which is rendered (in this case, `todolist`). Since we declared `todolist` and `renderer` beforehand, we now have access to them in the scopes of the test.

#### Test 1: The Todolist

``` javascript
it("renders a todolist", function(){
  expect(todolist.props.className).to.equal("todo-list");
});
```

As noted above, the `todolist` variable is the object which is to be rendered. We can treat it like other React components and inspect its props - here we're just making sure that it receives a `className` of "todo-list".

#### Test 2: The Greeting

``` javascript
it("greets the user", function(){
  var greetingSpan = todolist.props.children[0];

  expect(greetingSpan.props.className).to.equal("greeting");
});
```

As this test shows, not only can we inspect the rendered component itself, but its children as well. Since the children are delivered down through React components via props, we can obtain the children there. This test simply grabs the first child of the rendered component and ensures that it has a class of "greeting".

#### Test 3: The Todos

``` javascript
it("will render three todos", function(){
  var todos = shallowTestUtils.findAllWithType(todolist, Todo);
  expect(todos.length).to.equal(3);

  var dinnerTodo = todos[2];
  expect(dinnerTodo).to.deep.equal(
    <Todo key="todo-2" text="go out to dinner" />
   );
  });
```

The `findAllWithType` method comes in super clutch right here.

It traverses the semi-rendered `TodoList` and returns an array of all sub-components that are of the type passed in the second argument. This way, we can make sure that there are exactly as many `Todo`s as we expect (3, since we passed 3 in to the `TodoList`), and they look like exactly what we expect.

### But Wait, Jake! React Doesn't work like that, does it?

I know what you may be thinking right now: If the `renderer` renders a `TodoList` that contains `Todo`s, shouldn't the `Todo`s be rendered as well? Isn't this almost a recursive process of rendering down to the smallest, dumbest sub-component?

No, because we're using `shallowTestUtils`! This tool allows us to inspect *only* the rendered output of that component itself, leaving all React subcomponents as unrendered. In this way, we only need to check that the correct props are passed to subcomponents, and don't need to worry about their inner details here (we should definitely make a `Todo` test though!)

![asdf](http://i.giphy.com/vh6B8BQizPBPW.gif)

Like, totally cool, right?

In this way, React components are made to be very prone to unit testing - we can test a parent component to make sure it renders the subcomponents it needs to, while not having to look at the entire DOM structure that gets rendered when that component is loaded in the browser. It's really helpful, especially given the complicated nature of front-end and user-simulating tests. So, with your new knowledge of React's Shallow Rendering, go forth and test!
