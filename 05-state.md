# State

# Objectives

By the end of this chapter, you should be able to:

- Describe how props and state are different
- Initialize state in a React component
- Describe what a pure function is and how it applies to `setState`
- Pass state down from a parent component to a child component via props

# State

Props are great when you want to pass data to a component that isn't changing. However, when building dynamic web applications, we often want to be able to change our data. In React, we store dynamic data as state in the app. You may very often hear that "state is the root of all evil" and that is true to an extent, but it is often necessary when building applications. The reason why people are wary of including too much state in your application is that it adds complexity to your application. The more state you have to manage, the more difficult it becomes to reason about your application.

**Important State Concepts:**

- _State should be owned by only one component_. This is a crucial concept to keep in mind when you're designing your application. This simple rules makes React components much easier to debug and understand. When there is a problem, you typically know exactly which component is having an issue.

- _When state is changed inside of a component, the render method will eventually be invoked by React_. One of the key words here is eventually. Changes to state do not always show up right away (in practice they happen very quickly). This concept will lead to some rules we'll discuss about how state can change.

# Initializing state

Let's create some state!

```jsx
import React, { Component } from "react";
import { render } from "react-dom";

class App extends Component {
  constructor(props) {
    super(props);
    this.state = {
      name: "Michael"
    };
  }
  render() {
    return (
      <div>
        <h2>Our state is {this.state.name}</h2>
      </div>
    );
  }
}

render(<App />, document.getElementById("root"));
```

If you are new to ES 2015 classes, you may not have seen a `constructor` before. The constructor does the setup for your component (and more generally a constructor function sets up a class). In React, whenever you have a constructor, you will want to pass in `props` as a parameter and on the first line of the constructor, you should called `super(props);` - React will yell at you if you don't.

In the above example, after calling `super` on the props, we set up a `state` object for our `App` component. The `state` object is simply a normal JavaScript object.

Our example above doesn't really illustrate what state can do. So far the component won't actually change. Let's add a call to `setState`:

```jsx
import React, { Component } from "react";
import { render } from "react-dom";

class App extends Component {
  constructor(props) {
    super(props);
    this.state = {
      name: "Michael"
    };

    setTimeout(this.changeName.bind(this), 5000);
  }

  changeName = () => {
    this.setState({ name: "Whiskey" });
  };

  render() {
    return (
      <div>
        <h2>Our state is {this.state.name}</h2>
      </div>
    );
  }
}

render(<App />, document.getElementById("root"));
```

Now our example is dynamic! On our component class, we are creating a function, `changeName` that will call `setState` which will change the name property of our state object to `Whiskey`. In the constructor, we have a setTimeout that will invoke changeName for us in 5 seconds (5000ms). The call to `setState` will trigger another call to `render` which will return new JSX which has `this.state.name` equal to `Whiskey`.

Now that we have seen `setState`, we need to take a moment to back up and review some functional programming concepts.

# Functional Programming: Pure Functions

A _pure function_ is a foundational concept in functional programing. The idea is that a function when invoked should not have any side effects and it should not modify the parameters it was given. Let's look at an example:

```js
function doubleValues(arr) {
  for (let i = 0; i < arr.length; i++) {
    arr[i] = arr[i] * 2;
  }
}

const arr1 = [2, 4, 5];
doubleValues(arr1);
doubleValues(arr1);
```

The example above is an impure function. The input array is being modified which breaks one of our pure function rules. One rule of thumb when testing for a pure function is to ask yourself, can the function be run multiple times and will I still get the same result? In our example above `doubleValues` is invoked twice and the result changes each time because `arr1` is changing.

Here is an example of a pure function:

```js
function doubleValues(arr) {
  const result = [];
  for (let i = 0; i < arr.length; i++) {
    res.push(arr[i] * 2);
  }
  return result;
}

const arr1 = [2, 4, 5];
const result = doubleValues(arr1);
result = doubleValues(arr1);
```

Now if you look at the code above, we will always get the same result given the same input, no matter how many times the function is run. Since our input is not modified and the function has no other side effects, this is a pure function.

Let's look at one more example:

```js
const myObj = { id: 53, hobby: "hiking" };
function addName(name) {
  myObj.name = name;
}
addName("Michael");
```

Again, this is an impure function because there are side effects. We are modifying some global state which is not a parameter to the function. Let's refactor the code to pass in myObj as a parameter and we'll make sure not to modify the input:

```jsx
const myObj = { id: 53, hobby: "hiking" };
function addName(obj, name) {
  return { ...obj, name };
}
addName(myObj, "Michael");
```

Now we have a pure function because our function does not have side effects and the inputs are not being modified.

The spread operator in the `return` statement spreads out the keys of the old object `obj` into a new object. Then we also define the name key in our new object. To read more about the object spread operator, check out the [MDN docs](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_operator#Spread_in_object_literals). The old way to do this was by using [Object.assign](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/assign).

# Pure Functions and `setState`

Now we'll explore applying the pure function concept to how we will use setState. Let's imagine we have the following component:

```jsx
import React, { Component } from "react";
import { render } from "react-dom";

class App extends Component {
  constructor(props) {
    super(props);
    this.state = {
      people: [
        { name: "Michael" },
        { name: "Elie" },
        { name: "Matt" },
        { name: "Angelina" }
      ]
    };

    setTimeout(this.addName.bind(this), 5000);
  }

  addName() {
    const newPeople = this.state.people.slice();
    newPeople.push({ name: "Whiskey" });
    this.setState({ people: newPeople });
  }

  render() {
    var peeps = this.state.people.map(p => {
      return <p key={p.name}>{p.name}</p>;
    });
    return <div>{peeps}</div>;
  }
}

render(<App />, document.getElementById("root"));
```

In our above example, we are adding a new person, Whiskey, to our array of name objects. The important piece of code to look at is the addName function. It is a pure function because we are not modifying the `this.state.people` array. Instead, we are using the `slice` method to create a copy of the array, and then we are adding our new name to the copy of our data.

This leads us to a rule with React:

**Functions that call setState should be pure functions.**

In other words, we never want to modify the state object directly. You may be wondering at this point, why do we care that calls to setState are pure? We'll discuss that next.

# `setState` is Asynchronous

As we alluded to earlier `setState` does not change the state of our app immediately. In fact, `setState` could be called multiple times before the state is actually updated and the DOM is re-rendered.

Let's take a look at our example again. If you were to add `console.log(this.state.people)` right after the invocation of `setState`, you would not see `Whiskey` in the people array.

```jsx
class App extends Component {
  constructor(props) {
    super(props);
    this.state = {
      people: [
        { name: "Michael" },
        { name: "Elie" },
        { name: "Matt" },
        { name: "Angelina" }
      ]
    };

    setTimeout(this.addName.bind(this), 5000);
  }

  addName() {
    const newPeople = this.state.people.slice();
    newPeople.push({ name: "Whiskey" });
    this.setState({ people: newPeople });
    // Whiskey WILL NOT be part of people by the time this console.log runs!
    console.log(this.state.people);
  }

  render() {
    var peeps = this.state.people.map(p => {
      return <p key={p.name}>{p.name}</p>;
    });
    return <div>{peeps}</div>;
  }
}

render(<App />, document.getElementById("root"));
```

If for some reason you need to check the state after it has been updated, you can actually provide another parameter to `setState` which is a callback function that will be invoked once the state has been updated. Our `setState` code would now look like this:

```jsx
addName() {
  const newPeople = this.state.people.slice();
  newPeople.push({ name: 'Whiskey' });
  this.setState({ people: newPeople }, () => {
    // Whiskey will now be included in the array!
    console.log(this.state.people);
  });
};
```

So because `setState` is asynchronous, you can't be sure when a render will happen. In other words, if you modify state directly you may end up confusing react and causing weird bugs. As long as you make sure you do not modify the state (keep your functions pure), then you will not run into problems.

# Passing An Updater Function to `setState`

`setState` can also take an updater function rather than a new object for the state update. Whatever is returned from the function will be the new state. This is often useful when you want to know the previous version of the state:

```jsx
import React, { Component } from "react";

export default class Counter extends Component {
  constructor(props) {
    super(props);
    this.state = {
      counter: 1
    };

    setInterval(() => {
      this.setState(
        (prevState, props) => {
          // Updating based on the previous value
          return { counter: prevState.counter + 1 };
        },
        () => {
          // Printing the update to the console
          console.log(this.state.counter);
        }
      );
    }, 500);
  }

  render() {
    return <div>Counter: {this.state.counter}</div>;
  }
}
```

Notice in this example that we have access to `prevState` and props inside of the updater function to `setState`. We have also added a callback function after the state updater parameter that will console log the new state after it has been changed.

# Passing State Down To Child Components As Props

Recall the first important rule we discussed about state: a piece of state should only be owned by one component. But many times you will want to provide data that is in one component to a child component. To pass state down to a child component, we will give the data to the child as a prop. Here's an example:

```jsx
class Person extends React.Component {
  render() {
    var style = { color: "blue" };
    return <p style={style}>{this.props.name}</p>;
  }
}

class App extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      people: [
        { name: "Michael" },
        { name: "Elie" },
        { name: "Matt" },
        { name: "Angelina" }
      ]
    };

    setTimeout(this.addName.bind(this), 5000);
  }

  addName() {
    const newPeople = [...this.state.people];
    newPeople.push({ name: "Whiskey" });
    this.setState({ people: newPeople });
  }

  render() {
    const peeps = this.state.people.map(p => {
      return <Person key={p.name} name={p.name} />;
    });
    return <div>{peeps}</div>;
  }
}

ReactDOM.render(<App />, document.getElementById("root"));
```

[You can see this in CodePen here.](https://codepen.io/hueter/pen/OzNEGz)

As you can see in this example, the state is still owned by the `App` component, but the data in the state is being passed to the `Person` component via the `Person` component's prop, `name`.

# Additional reading
At this point you may be thinking to yourself "Okay, so when should I use props and when should I use state?" Developing an intuition for when to use each is one of the most important parts of becoming a successful React developer. [This](https://github.com/uberVU/react-guide/blob/master/props-vs-state.md) article (and the references therein) is a good place to start digging deeper.

# Exercises

## State Exercises

For this exercise, you will create 24 components (square shaped divs) that have a specific background color. Each time you click on one of the divs, it cycles to the next background color.

*BONUS* - Make the background color changing random, i.e. when I click a color, it selects randomly from the list of colors.

Your application should look like this:

![Random Colors](./assets/05-01.gif)

# Next

When you're ready, move on to [React Component Architecture](./06-architecture/md)