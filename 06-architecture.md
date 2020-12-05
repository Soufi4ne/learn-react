# React Component Architecture.

# Objectives

By the end of this chapter, you should be able to:

- Understand how to architect an application using components
- Install the React developer tools in chrome
- Describe what the virtual DOM is

# Component Hierarchy

As you continue to build React applications, you will begin to better understand how much logic should go into a component before splitting into multiple components. One of the best tutorials on how to structure React applications and determine what should be a component is from the docs [here](https://facebook.github.io/react/docs/thinking-in-react.html).

When thinking about your components as a beginner, it is always best to start with larger components and break them down into smaller ones when necessary. There is no need to prematurely optimize or break out all your components in to tiny pieces when starting. As you continue building more applications and learn more, you will have an easier time figuring out when to refactor and break up components.

# Different Styles of React Components

## ES2015 Class Syntax

React also plays quite well with the ES2015 `class` keyword and the above component can be written like this.

```jsx
import React from "react";
import PropTypes from "prop-types";

class App extends React.Component {
  constructor(props) {
    // if we want props passed down to this component from a parent component
    super(props);
    // set initial state
    this.state = {
      name: "Elie"
    };
  }
  // render component - this looks familiar!
  render() {
    return (
      <div>
        <h1>
          Hello {this.state.name}! You are a {this.props.job}{" "}
        </h1>
      </div>
    );
  }
}

// outside of the class we set defaultProps and propTypes
App.defaultProps = {
  job: "Instructor"
};

App.propTypes = {
  name: PropTypes.string,
  job: PropTypes.string
};
```

## Stateless Functional Components

Along with the two components above, we can create a different kind of component when our component does not use state. We will discuss in far more depth in future sections when and when not to use state, but in a situation where you do not need to use it, you can use a stateless functional component. Before we see what one looks like, let's break down these words:

- **Stateless** - no state
- **Functional** - a function
- **Component** - a valid React component

So what does it look like? In the example above, we were using state, but we really do not need it so here is what we can do!

```jsx
const App = props => (
  <div>
    <h1>
      Hello {props.name}! You are a {props.job}{" "}
    </h1>
  </div>
);
```

We can even take this a bit further by using ES2015 destructuring!

```jsx
const App = ({ name, job }) => (
  <div>
    <h1>
      Hello {name}! You are a {job}{" "}
    </h1>
  </div>
);
```

But what about `defaultProps` and `propTypes`? They are done exactly like the ES2015 class syntax, we simply place them as properties on the component.

# React Dev Tools

Facebook offers a very helpful debugging tool for React called the React Developer Tools, which is an extension for the Chrome Dev Tools. You can install it [here](https://chrome.google.com/webstore/detail/react-developer-tools/fmkadmapgofadopljbjfkapdkoienihi).

This will give you access to a tab called "React" in the chrome dev tools where you can examine your components and see all essential information about them. When you are debugging React applications, make sure you keep this tab open to see if you are correctly modifying state, props and other information on your components or to see the Component Hierarchy.

# Declarative vs. Imperative

Another concept you will come across frequently is the distinguishing between declarative and imperative code. React frequently labels itself as a declarative library, but what does this actually mean?

- **Imperative** - focus on "how" something is done and the steps necessary to implement it

- **Declarative** - focus less on the implementation (or the "how") something is done (taking for granted an imperative process behind the scene) and more on "what" should be done.

For a fantastic and thorough explanation of this concept, check out this article by Tyler McGinnis and this StackOverflow post.

# How React Works
Now that you have worked with a few components and gotten the hang of what React and JSX code looks like, let's dive deeper into some of the "how" React works. One thing you may hear many times with React is the concept of a "Virtual DOM" - let's dive deeper into that.

# Virtual DOM

At this point in your JavaScript journey, you have most likely done a bit of DOM manipulation. This involves searching for elements (using the `document` object or `jQuery`) or adding and removing elements from the DOM. Although these are not easy to spot and are not seen by the user, there are times when small changes to the DOM will require larger portions of the DOM to be recreated. While this is not an issue in smaller applications, it can cause performance problems at scale.

However with React, things are a bit different. React builds a copy of the DOM, which is calls the "Virtual DOM". It is essentially a representation of what the DOM looks like, but without the ability to directly change what the user sees and this actually makes it much faster for lookup and modification. When a React component gets rendered, every single virtual DOM object gets updated.

Now that the virtual DOM has been updated, React will compare it with the previous version of the virtual DOM and figure out exactly which virtual DOM objects have changed. The algorithm to perform this is called "diffing" and this process is known as Reconciliation.

Now that React knows which virtual DOM objects have changed, React can update ONLY the changed objects on the real DOM.

When the DOM is updated in React, the following happens:

- The virtual DOM is updated
- The virtual DOM then gets compared to what it looked like before the update. To figure out what has changed, React runs its "diffing" algorithm
- Only the changed objects get updated on the real DOM

You can read much more about it [here](https://stackoverflow.com/questions/21109361/why-is-reacts-concept-of-virtual-dom-said-to-be-more-performant-than-dirty-mode) and [here](http://reactkungfu.com/2015/10/the-difference-between-virtual-dom-and-dom/).

# Reconciliation

React provides a declarative API so that you don't have to worry about exactly what changes on every update. This makes writing applications a lot easier, but it might not be obvious how this is implemented within React.

[This](https://facebook.github.io/react/docs/reconciliation.html) section on the React Docs explains React's "diffing" algorithm so that component updates are predictable while being fast enough for high-performance apps. It also explains why a `key` prop is necessary when iterating and displaying multiple child components.

# Next

When you're ready, move on to [React Components Exercise](./07-exercises.md)