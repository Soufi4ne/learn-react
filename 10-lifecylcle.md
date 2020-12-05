# Component Life Cycle

# Objectives

By the end of this chapter, you should be able to:

- Identify life cycle methods related to initialization, changing state and props, and unmounting components
- Use the `componentDidMount` life cycle method for AJAX requests
- Use the `componentWillUnmount` method to clean up any state the component needs to remove
- Catch errors using React 16 Error Boundaries and `componentDidCatch`

# Introduction

So far we have seen how to create components, render components, and pass down props to child components.

Each component has several “lifecycle methods” that you can tap into to run code at particular times in the process.

The idea of lifecycle methods or hooks is not new to React, you might have seen "hooks" before with Mongoose (pre/post save/remove hooks), Mocha/Jasmine (beforeEach / afterEach hooks). All we are doing here is running some code at a certain period in the existance (or lifecycle) of the component.

As a hint, all methods prefixed with will are called right before something happens, and methods prefixed with did are called right after something happens. Since there are quite a few methods, let's first break down the lifecycle into few categories.

**Initialization / Mounting** - When a component is rendered for the first time
**Updating** - When new props are passed, `setState` is called or `forceUpdate` is invoked. forceUpdate is a method you should almost never use - you can read more about it here
**Removing a Component / Unmounting** - When a component is removed (also called unmounting)

# An important note

As of React 16.3, quite a few of these lifecycle methods have been renamed and will be deprecated in React 17, you will see more information when learning about that specific method. Memorizing all of these methods will not be beneficial, what's important to understand now is the lifecycle of a component and 2 or 3 of the methods you will be using. [Here is a fantastic diagram where you can see these in action](http://projects.wojtekmaj.pl/react-lifecycle-methods-diagram/)

# Initialization

When a component gets rendered for the first time, we can tap into quite a few life cycle hooks.

## `constructor`
Before the component is mounted, the constructor in the Component is run. Using ES2015 syntax, this is where initial state is set, and where any method binding occurs.

In your constructor there are few things to stay away from including:

- calling setState() inside your constructor (instead, set initial state using `this.state = {}`)
- initialize state based on props. Beware of this pattern, as state won’t be up-to-date with any props update. Instead of syncing props to state, you often want to lift the state up instead. You can read more about that idea [here](https://reactjs.org/docs/lifting-state-up.html)

## `static getDerivedStateFromProps(nextProps, prevState)`

`getDerivedStateFromProps` is invoked after a component is instantiated as well as when it receives new props. It should return an object to update state, or null to indicate that the new props do not require any state updates. Notice that this is a `static` method and must be prefixed with the `static` keyword.

Calling t`his.setState()` generally doesn’t trigger `getDerivedStateFromProps()`.

## `componentWillMount` (UNSAFE_componentWillMount())

The first life cycle method we haven't seen before is `componentWillMount`, which is called **before** the `render` method is executed the first time. Since this method is called before `render`, this means that the component has not been added to the DOM yet and we cannot use React refs. It is important to note that setting the state in this phase will not trigger a re-rendering (since nothing has been rendered!).

```jsx
componentWillMount(){
    console.log('In componentWillMount');
    // nothing has rendered yet

    // Props have been initialized
    console.log(this.props);

    // State has been initialized
    console.log(this.state);
}
```

The `componentWillMount` method is not used as frequently as the next method we will discuss, `componentDidMount`. Sometimes you will see it used for initial configuration of the component, but this can also often be done directly in the `constructor`. In both life cycle methods you have access to `props` and `state`, but not much else.

> This method will be deprecated in React 17 and methods like componentDidMount or getDerivedStateFromProps should be used instead. It will be renamed to UNSAFE_componentWillMount()

## `render`

Right after `componentWillMount` is invoked, the `render` method is invoked and the returned markup is added to the DOM.

## `componentDidMount`
After the component renders (also known as mounting), `componentDidMount` will be invoked. Since the component has rendered, we can access the DOM from this method.

```jsz
componentDidMount(){
    // the component mounted! Let's do some DOM stuff
}
```

Very commonly, when a component is rendered, an `AJAX` call is made to change state in the component. This is done in the `componentDidMount`. We will also make use of this lifecycle method when learning about asynchronous actions with redux and react.

Calling `setState()` in this method **will trigger an extra rendering**, but it will happen before the browser updates the screen. This guarantees that even though the render() will be called twice in this case, the user won’t see the intermediate state. Use this pattern with caution because it often causes performance issues.

It can, however, be necessary for cases like modals and tooltips when you need to measure a DOM node before rendering something that depends on its size or position.

# AJAX Example

You can’t guarantee the AJAX request won’t resolve before the component mounts. If it did, that would mean that you’d be trying to setState on an unmounted component, which not only won’t work, but React will yell at you for. Doing AJAX in componentDidMount will guarantee that there’s a component to update.

We are going to make a component that renders state which was received from an AJAX request. To make our AJAX request, we'll use an AJAX library called `axios`. You can check out the [axios documentation](https://github.com/mzabriskie/axios) to see all of the library's functionality. We will be using it to make a HTTP GET request.

First, add `axios` as a dependency to your applicaiton:

```sh
npm install --save axios
```

Next, try out the following code that uses the [icanhazdadjoke](https://www.icanhazdadjoke.com/) to get joke data:

```jsx
import React, { Component } from "react";
import axios from "axios";

class Jokes extends Component {
  constructor(props) {
    super(props);
    this.state = {
      jokes: []
    };
  }

  componentDidMount() {
    axios
      .get(`https://icanhazdadjoke.com/search?term=movie`, {
        headers: { Accept: "application/json" }
      })
      .then(response => {
        const jokes = response.data.results.map(joke => ({
          id: joke.id,
          text: joke.joke
        }));
        this.setState({ jokes });
      });
  }

  render() {
    let data = "Sorry, no joke results yet";
    if (this.state.jokes.length > 0) {
      data = this.state.jokes.map(joke => (
        <div key={joke.id}>
          <p>{joke.text}</p>
        </div>
      ));
    }
    return <div>{data}</div>;
  }
}

export default Jokes;
```

The above code makes a GET request to `https://icanhazdadjoke.com/search?term=movie`. The result of the request is all cheesy jokes with the word "movie" in them. Once the response is returned, the callback inside of `.then` is invoked and we use `map` to convert the results into an array that contains the joke `id` and the joke `text`.

# Component Life Cycle Initialization: Recap

To recap initialization, these are the steps that take place when a component is first created and rendered in the DOM:

- The `constructor` method is run and default state and props are set as well as any method binding (using `bind` to set the correct value of the keyword `this`)
- `static getDerivedStateFromProps` is invoked
- `UNSAFE_componentWillMount` is invoked
- `render` is invoked
- `componentDidMount` is invoked

# Changing State

Whenever we call `setState`, we can tap into quite a few life cycle hooks:

## `componentWillReceiveProps` (UNSAFE_componentWillReceiveProps)

This life cycle method is only called when the props have changed and when this is not an initial rendering. `componentWillReceiveProps` allows us to update the state depending on the existing and upcoming props before the `render` method has been called. This is essential when you want to intercept the passing down of props from a parent component to a child component. For example, maybe you want to set some state only when a particular prop is about to change, but not otherwise. 

> This method will be deprecated in React 17 and will be renamed to UNSAFE_componentWillReceiveProps()

```jsx
componentWillReceiveProps(nextProps) {
  if (this.props.someProp !== nextProps.someProp) {
    this.setState({
      // set some state
    });
  }
}
```

## `static getDerivedStateFromProps`

This method as described above is a more modern way to change state depending on changes to the component and should be used instead of `componentWillReceiveProps`.

## `shouldComponentUpdate`

By default, when a parent component updates, all children are re-rendered as well. But `shouldComponentUpdate` allows us to pass on a re-rendering if we know one is not needed. This function should return a boolean; if it's true, the component will render, and if it's false, the component won't.

```jsx
shouldComponentUpdate(nextProps, nextState) {
    // return a boolean value
    // true tells react that the component should be updated
    // after a call to setState
    return true;
}
```

Be very careful with your use of the `shouldComponentUpdate` life cycle method. This is an advanced optimization that should only be used after you understand React well. Typically React is very good at optimizing changes to the DOM.

## `componentWillUpdate` (UNSAFE_componentWillUpdate)

`componentWillUpdate` gets called as soon as the the shouldComponentUpdate returns true. This is sort of analogous to the `componentWillMount` hook; the difference now is that we're updating an existing component, not mounting a new one. You **cannot** call `setState` here. If you need to update state in response to a change in props, use getDerivedStateFromPRops instead.

> This method will be deprecated in React 17 and methods like `getDerivedStateFromProps` should be used instead. It will be renamed to UNSAFE_componentWillUpdate()

```jsx
componentWillUpdate(nextProps, nextState){
    // perform any preparations for an upcoming update
}
```

## `render`

We've seen this one before! Just **always** remember - changing state triggers a re-render unless you modify `shouldComponentUpdate`.

## `getSnapshotBeforeUpdate`

```jsx
getSnapshotBeforeUpdate(prevProps, prevState){
    // perform any preparations for an upcoming update
}
```

`getSnapshotBeforeUpdate()` is invoked right before the most recently rendered output is committed to e.g. the DOM. It enables your component to capture current values (e.g. scroll position) before they are potentially changed. Any value returned by this lifecycle will be passed as a parameter to `componentDidUpdate()`. 

This method is a bit more advanced as well and is necessary to support a newer feature of React called async rendering.

## `componentDidUpdate`

Finally `componentDidUpdate` is called after the render method, and is similar to the `componentDidMount`.

```jsx
componentDidUpdate(prevProps, prevState){
    // component just updated!
}
```

As with `componentDidMount`, `componentDidUpdate` is a good place to make any AJAX requests for data after an update. An example of this could be an "autosave" feature. `componentDidUpdate` is also useful if you need to update anything outside of React (external libraries) with new data.

To recap what happens when `setState` runs:

- `componentWillReceiveProps` runs
- `getDerivedStateFromProps` runs
- `shouldComponentUpdate` runs - if it returns false, render does not run
- `componentWillUpdate` runs
- `render` runs
- `getSnapshotBeforeUpdate` runs
- `componentDidUpdate` runs

# Removing a component (unmounting)

## `componentWillUnmount`

The only method we haven't examined yet is the `componentWillUnmount` which gets called before the component is removed from the DOM. This method can be beneficial when needing to perform cleanup operations, i.e. removing any timers defined in `componentDidMount`.

```jsx
componentWillUnmount(){
    // remove an event listener
    // clear a timeout
    // remove any reference to variables you will not be using to ensure there are no memory leaks
}
```

Let's look at an example of a component that that has a `setInterval` that needs to be cleared before the component is unmounted.

```jsx
import React, { Component } from "react";

class ColoredBox extends Component {
  constructor(props) {
    super(props);
    this.state = {
      backgroundColor: "yellow",
      intervalId: null
    };
  }

  componentDidMount() {
    const intervalId = setInterval(() => {
      this.setState((prevState, props) => {
        const backgroundColor =
          prevState.backgroundColor === "yellow" ? "red" : "yellow";
        console.log(`Changing color to ${backgroundColor}`);
        return { backgroundColor };
      });
    }, 1000);

    this.setState({ intervalId });
  }

  componentWillUnmount() {
    clearInterval(this.state.intervalId);
    console.log(`Interval ${this.state.intervalId} has been cleared`);
  }

  render() {
    const style = {
      width: "100px",
      height: "100px",
      backgroundColor: this.state.backgroundColor
    };
    return <div style={style} />;
  }
}

export default ColoredBox;
```

You can see in the example that when the component is unmounted, the interval will be cleared in the `componentWillUnmount` life cycle method. To see the unmount life cycle event being invoked, set up the following `BoxContainer` component that uses the `ColoredBox` component:

```jsx
import React, { Component } from "react";
import ColoredBox from "./ColoredBox";

export default class BoxContainer extends Component {
  constructor(props) {
    super(props);
    this.state = {
      showBox: true
    };
  }

  componentDidMount() {
    setTimeout(() => {
      this.setState({ showBox: false });
    }, 7500);
  }

  render() {
    let markup = <div>Nothing to see here</div>;
    if (this.state.showBox) {
      markup = <ColoredBox />;
    }
    return markup;
  }
}
```

The high level component will show the `ColoredBox` component until 7500ms have passed. After that time, the `showBox` state will be set to false, and the `ColoredBox` component will be unmounted. If you open up the console after loading the page, you can verify by the messages being logged that the timer does indeed eventually get cleared.

## `componentDidCatch`

In React 16, **Error Boundaries**, a more friendly way to catch errors were introduced. A class component can become an error boundary if it defines a lifecycle method called `componentDidCatch`. So what's the idea here?

Error boundaries are components that can catch errors anywhere in component (including any children component they render), and log errors and/or display a fallback UI. The `componentDidCatch` lifecycle hook is meant to catch errors during mounting, rendering and in other lifecycle methods.

Here is a simple example from the React Docs

```jsx
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  componentDidCatch(error, info) {
    // Display fallback UI
    this.setState({ hasError: true });
    // You can also log the error to an error reporting service
    logErrorToMyService(error, info);
  }

  render() {
    if (this.state.hasError) {
      // You can render any custom fallback UI
      return <h1>Something went wrong.</h1>;
    }
    // render anything inside of this component
    return this.props.children;
  }
}
```

It could then be used like this:

```jsx
<ErrorBoundary>
  <SomeComponent />
</ErrorBoundary>
```

In short, error boundaries are almost identical to `try/catch` blocks (but instead of the imperative try/catch, we use a far more declarative syntax), but for your entire component and any children they render.

> Only use error boundaries for recovering from unexpected exceptions; don’t try to use them for control flow.

# Recap

There are quite a few component life cycle methods and with some of them becoming deprecated as of React 17 - here are 
the **most important ones** to keep in mind along with their use cases:

**static getDerivedStateFromProps(nextProps, prevState)** - Useful for acting on changes in props to trigger changes in state (compare new and previous values if you want to handle changes). Instead of calling setState, you either return an object with new state or null if you do not want a change in state. If a parent component causes a component to re-render, this method will be called even if props have not changed.

**componentDidMount()** - This is where you will almost always load data so that you can work with a component and DOM. You can call setState in this lifecycle method and will most commonly use this method for AJAX calls.

**shouldComponentUpdate(nextProps, nextState)** - Useful for having very fine control over when your component will re-render. You cannot call setState in this lifecycle method and will use this most in cases where performance is a concern.

**componentDidUpdate(prevProps, prevState)** - Useful for dpdating the DOM in response to any changes in props or state or for making an AJAX call after a component updates. You can call setState in this lifecycle method.

**componentWillUnmount()** - Useful for any cleanup in your component (timers / event listeners). You cannot call setState in this lifecycle method.

# Additional Resources

- [https://facebook.github.io/react/docs/react-component.html](https://facebook.github.io/react/docs/react-component.html)
- [https://reactjs.org/blog/2018/03/27/update-on-async-rendering.html](https://reactjs.org/blog/2018/03/27/update-on-async-rendering.html)

# Exercises

## Component Lifecycle Exercises

### Part 1 

Rebuild the [Giphy API exercise](https://www.rithmschool.com/courses/intermediate-javascript-part-2/ajax-exercises) with React. 

Once you finished rebuilding this application, include a "gif of the day", which should be a random gif generated, when the page loads. You **will** need to use a component life cycle method for this.

### Part 2

Add another component to your Todo App called `EditTodoForm`, which renders a form to edit a todo. When the form is submitted, it should rename the todo's task. 

Depending on the UI which you choose, you **may** have to use a few lifecycle hooks to ensure that your application works as expected.

As a bonus, store your todos in localStorage! When using localStorage, you **will** need to use a life cycle method in order for this to work properly.