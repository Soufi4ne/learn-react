# Events in React

# Objectives

By the end of this chapter, you should be able to:

- Compare and contrast events and synthetic events
- Explain what "method binding" is and why it's important
- Access synthetic events from inside event handlers
- Pass event handlers from parent components to child components as props, and explain why this pattern is so commonly used

# Events with React

Events in React are not actually `DOM` events. Instead, they are called `synthetic` events. These synthetic events are just wrappers over native DOM events that allow for cross-browser compatibility. Should you ever need the built-in DOM event, each synthetic event in React has a nativeEvent property that points to the native event. Having said that, when you're just starting out with React, you should never really need to access the `nativeEvent`.

The React event API is almost identical to the DOM API in its methods, but they are not exactly the same. For a full list of React events, you can check out the [documentation](https://facebook.github.io/react/docs/events.html). There are many different types of synthetic events, including keyboard events, mouse events, animation events, and more. For example, here is a complete list of all the mouse events supported in React:

```sh
onClick
onContextMenu
onDoubleClick
onDrag
onDragEnd
onDragEnter
onDragExit
onDragLeave
onDragOver
onDragStart
onDrop
onMouseDown
onMouseEnter
onMouseLeave
onMouseMove
onMouseOut
onMouseOver
onMouseUp
```

Unlike what one typically does in vanilla JavaScript, you don't register event listeners to these events using `addEventListener`. Instead, you pass callbacks to these events down as props to your component. The name of the prop should be the type of event you're listening for. Here's a simple example using the `onClick` event:

```jsx
import React, { Component } from "react";
import { render } from "react-dom";

class App extends Component {
  handleClick() {
    alert("click!");
  }

  render() {
    return (
      <div>
        <button onClick={this.handleClick}>Click me!</button>
      </div>
    );
  }
}

render(<App />, document.getElementById("root"));
```

Event handlers are typically defined inside of the class, as in the example above. 
In this example, clicking on the button causes the `handleClick` function to execute.

# Method binding with `this`

Our first example using an event handler was relatively simple. Let's take the complexity up a degree by writing an event handler that relies on the keyword this:

```jsx
import React, { Component } from "react";
import { render } from "react-dom";

class App extends Component {
  handleName() {
    alert(this.props.name); // PROBLEM
  }

  render() {
    return (
      <div>
        <button onClick={this.handleName}>{this.props.name}</button>
      </div>
    );
  }
}

render(<App name="Matt" />, document.getElementById("root"));
```

The above example might look fine, but if you try to run this app, you'll soon find that clicking on the button throws an error! More specifically, you should be getting a `TypeError` with the message `Cannot read property 'props' of undefined`. 
Moreover, if you `console.log` the value of this inside of `handleName`, you'll see that its value is `undefined`. The lesson here is that you need to be careful when calling functions that rely on the keyword `this`, as it's very easy to lose the proper context of the keyword.

There are several ways to fix this problem.

## Binding on the Component

The first approach is to `bind` the method to the proper `this` value when you pass the method in as a prop:

```jsx
<button onClick={this.handleName.bind(this)}>{this.props.name}</button>
```

Another way to achieve the same thing that you might see is using an ES2015 arrow function, which "forwards" the current this instead of creating a new context:

```jsx
<button onClick={() => this.handleName()}>{this.props.name}</button>
```

The downside with using arrow function callbacks is that it creates a new callback function every time the component renders, but there is usually not a noticeable performance hit.

The binding-on-the-component approach works fine if you have a small number of methods that you're only passing once. But what if you have many functions you need to bind, or you need to bind multiple times? For instance, take a look at the following, slightly modified app:

```jsx
import React, { Component } from "react";
import { render } from "react-dom";

class App extends Component {
  handleName() {
    alert(this.props.name);
  }

  render() {
    return (
      <div>
        <button onClick={this.handleName.bind(this)}>{this.props.name}</button>
        <button onClick={this.handleName.bind(this)}>
          Also {this.props.name}
        </button>
      </div>
    );
  }
}

render(<App name="Matt" />, document.getElementById("root"));
```

In this example we're binding twice, which is a bit redundant. Fortunately, there's another way we can solve this problem.

## Binding in the Constructor

We can bind the keyword `this` inside of the constructor. This saves you from having to bind inside of the `render` method, and can be helpful if you need to bind many methods or bind a method that you are passing into several different components. Here's what that looks like:

```jsx
import React, { Component } from "react";
import { render } from "react-dom";

class App extends Component {
  constructor(props) {
    super(props);
    this.handleName = this.handleName.bind(this);
  }

  handleName() {
    alert(this.props.name);
  }

  render() {
    return (
      <div>
        <button onClick={this.handleName}>{this.props.name}</button>
        <button onClick={this.handleName}>Also {this.props.name}</button>
      </div>
    );
  }
}

render(<App name="Matt" />, document.getElementById("root"));
```

Notice that in this case we only had to method bind once in the constructor. When we later pass the function down as props, we don't need to bind again.

You can read more about the tradeoffs of different ways to bind components [here](https://reactjs.org/docs/handling-events.html).

# Accessing the Synthetic Event

Just like in vanilla JavaScript, you can also access the event object (in this case, a synthetic event) inside of your event handlers. Here's a quick example:

```jsx
import React, { Component } from "react";
import { render } from "react-dom";

class App extends Component {
  constructor(props) {
    super(props);
    this.state = { type: "" };
    this.handleEvent = this.handleEvent.bind(this);
  }

  handleEvent(e) {
    this.setState({
      type: e.type
    });
  }

  render() {
    let styles = {
      backgroundColor: "blue",
      width: "200px",
      height: "200px",
      color: "white",
      fontSize: "20px"
    };

    let innerText = this.state.type
      ? `I detect an event! Type: ${this.state.type}`
      : "No event detected";

    return (
      <div
        style={styles}
        onMouseOver={this.handleEvent}
        onMouseOut={this.handleEvent}
        onClick={this.handleEvent}
      >
        {innerText}
      </div>
    );
  }
}

render(<App name="Matt" />, document.getElementById("root"));
```

This application simply creates a single div and attaches an event handler to three different events. Note that the handler takes a parameter called `e` - regardless of the name we give it, this variable will refer to the synthetic event. As you interact with the blue div on the page, you should see that it detects the three types of events that the handler is attached to.

# Passing Down Event Handlers

Very commonly, when we want to change data in a child component, we can not do that directly from the child component, because props are immutable. In order to change this data, we need to re-render the component and pass down new state from a parent. Therefore, the typical pattern is as follows:

- On the parent, we define a function that calls setState on the parent.
- From the parent, we pass down this function as a prop, along with whatever other props the child needs.
- The child uses this function passed down as an event handler. When the event fires, the function gets called, which in turn triggers a `setState` call on the parent.

In other words, even though the child can't directly change its own props, it can invoke a function passed down from the parent as a prop. This invocation will trigger a re-rendering of the parent, and, in turn, the child.

The easiest way to understannd this pattern is to see a bunch of examples. So let's dig into one. We'll use `create-react-app `to create an application where we can show a list of instructors, and remove instructors from the list.

To begin, we'll need to create our app and some component files. In the terminal, type:

```sh
create-react-app instructor-app
cd instructor-app
touch src/Instructor{,List}.js # what does this one line do?
```

Next, let's write the code for our four main JavaScript files: App.js, InstructorList.js, Instructor.js, and index.js.

Here's our `index.js`:

```jsx
import React from "react";
import ReactDOM from "react-dom";
import App from "./App";
import registerServiceWorker from "./registerServiceWorker";

ReactDOM.render(<App />, document.getElementById("root"));
registerServiceWorker();
```

Next, here's our App.js:

```jsx
import React, { Component } from "react";
import InstructorList from "./InstructorList";

export default class App extends Component {
  render() {
    return (
      <div>
        <h1>Here are all the instructors!</h1>
        <InstructorList />
      </div>
    );
  }
}
```

So far, so good. Next up is our `InstructorList.js`:

```jsx
import React, { Component } from "react";
import Instructor from "./Instructor";

export default class InstructorList extends Component {
  constructor(props) {
    super(props);
    this.state = {
      instructors: ["Elie", "Matt", "Michael"]
    };
  }

  handleRemove(idx) {
    const { instructors } = this.state;
    const newInstructors = instructors
      .slice(0, idx)
      .concat(instructors.slice(idx + 1));
    this.setState({
      instructors: newInstructors
    });
  }

  render() {
    let instructors = this.state.instructors.map((name, idx) => (
      <Instructor
        removeInstructor={this.handleRemove.bind(this, idx)}
        name={name}
        key={idx}
      />
    ));
    return <div>{instructors}</div>;
  }
}
```

Above is where the complexity begins. In terms of the pattern we discussed earlier, this `InstructorList` component is serving as the parent, while the Instructor components are the children. When a user clicks on the delete button for an instructor, we want that Instructor component to be able to tell the `InstructorList` parent to remove that instructor. But since children can't update their own props, we have to define the event handler on the parent and pass it to each child as a prop.

In this example, our event handler is called `handleRemove`. Given an index, it creates a new array of instructors from an old array by removing the instructor at the given index from the old array. Then it calls setState on `instructorList`. Note that in this case we've chosen not to method bind inside of the constructor. Instead, we've bound inside of the map, so that we can pass the current instructor index into `handleRemove`. (Note that this isn't the only way we could've done this. It's also possible to method bind in the constructor, and pass the Instructor component the current index as a prop so that it knows what index to pass to handleRemove.)

Finally, let's look at the `Instructor` component:

```jsx
import React, { Component } from "react";

export default class Instructor extends Component {
  render() {
    return (
      <div>
        <h2>
          This instructor's name is {this.props.name}.
          <button onClick={this.props.removeInstructor}>X</button>
        </h2>
      </div>
    );
  }
}
```

Note here that when someone clicks on a button, t`his.props.removeInstructor` will get invoked. But `this.props.removeInstructor` points to `handleRemove` on the `InstructorList` component! So clicking on a button from the child will cause state to be set on the parent.

# Exercise

## Events Exercises

## Part 1

Create two components `CustomLink` and `App`. The `CustomLink` component should accept three props:

- `href` - a URL
- `text` - the text inside the link
- `handleClick` - a callback to run when the user clicks on the link.

The component should render a link tag with an appropriate `href` and `text` coming from the props. It should also open in a new window (set the `target` attribute to `"_blank"`).

The `App` component should show at least three `CustomLink` components, along with a button that, when clicked, disables all of the links. In other words, if you click on the button and then click on the link, nothing should happen. Clicking on the button again should re-enable the links.

## Part 2

Create three components, `TodoList`, `Todo`, and `App`. The `TodoList` component should contain an array called `todos`. The `TodoList` component should be responsible for listing all of the todos. Each `Todo` component should consist of a title and a description, along with buttons to mark a todo as complete and completely remove a todo from the list.

Much of this application should be similar to the `Instructor` application from the notes, but try to build this application from scratch as much as possible. As a bonus, add some styling!

# Next

When you're ready, move on to [**Forms and Refs**](./09-forms.md)