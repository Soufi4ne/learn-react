# Forms and Refs

# Objectives

By the end of this chapter, you should be able to:

- Build forms in React
- Explain the difference between controlled and uncontrolled components
- Understand what refs are, how to use them, and when not to use them
- Compare and contrast using state versus using refs

# Forms

So far we've seen how to respond to user interaction primarily through mouse events. In particular, we haven't explored how to deal with things like forms and input fields using React. Let's change that!

One of the most common patterns you'll see in React when using input fields is that we need to make React aware of changes to the input when the user types. This means that the component containing the input typically needs to keep track of the text in the input using state. This is a very common pattern, and is explained in the React [docs](https://facebook.github.io/react/docs/forms.html) as follows:

In HTML, form elements such as `<input>`, `<textarea>`, and `<select>` typically maintain their own state and update it based on user input. In React, mutable state is typically kept in the state property of components, and only updated with setState().

We can combine the two by making the React state be the "single source of truth". Then the React component that renders a form also controls what happens in that form on subsequent user input. An input form element whose value is controlled by React in this way is called a "controlled component".

Here's a minimal working example:

```jsx
import React, { Component } from "react";
import { render } from "react-dom";

class App extends Component {
  constructor(props) {
    super(props);
    this.state = {
      firstName: "",
      lastName: ""
    };
    this.handleChange = this.handleChange.bind(this);
  }

  handleChange(e) {
    this.setState({
      [e.target.name]: e.target.value
    });
  }

  render() {
    let greeting =
      this.state.firstName && this.state.lastName
        ? `Hello, ${this.state.firstName} ${this.state.lastName}!`
        : "I don't know your full name :(";

    return (
      <div>
        <h1>Please tell me your name.</h1>
        <input
          onChange={this.handleChange}
          placeholder="What's your first name?"
          name="firstName"
          value={this.state.firstName}
        />
        <input
          onChange={this.handleChange}
          placeholder="What's your last name?"
          name="lastName"
          value={this.state.lastName}
        />
        <p>{greeting}</p>
      </div>
    );
  }
}

render(<App />, document.getElementById("root"));
```

Note that we are using the `handleChange` function to make React aware of changes to either input value by setting some state on 
our `App` component. In this way, our `inputs` become "controlled components," as they are essentially being controlled by React.

By setting name properties on the inputs, we only need one `handleChange` function, rather than creating separate handlers for each input. Note that if you remove the code inside of the `handleChange` function, the inputs will still update as the user types, but React won't be aware of any changes, so the greeting message will never update!

This pattern can seem a bit cumbersome at first, but you will quickly acclimate to it. Here's another example, using a few different controlled components inside of a form:

```jsx
import React, { Component } from "react";
import { render } from "react-dom";

class App extends Component {
  constructor(props) {
    super(props);
    this.state = {
      food: "",
      quantity: "1",
      ordered: false
    };
    this.handleChange = this.handleChange.bind(this);
    this.handleSubmit = this.handleSubmit.bind(this);
  }

  handleChange(e) {
    this.setState({
      [e.target.name]: e.target.value
    });
  }

  handleSubmit(e) {
    e.preventDefault();
    this.setState({
      ordered: true
    });
  }

  render() {
    let orderForm = (
      <form onSubmit={this.handleSubmit}>
        <div>
          <label>
            Food
            <input
              onChange={this.handleChange}
              placeholder="What food would you like?"
              name="food"
              value={this.state.food}
            />
          </label>
        </div>
        <div>
          <label>
            Quantity
            <select
              value={this.state.quantity}
              onChange={this.handleChange}
              name="quantity"
            >
              <option value="1">1</option>
              <option value="2">2</option>
              <option value="3">3</option>
              <option value="4">4</option>
              <option value="5">5</option>
            </select>
          </label>
        </div>
        <input type="submit" value="Place your order" />
      </form>
    );

    let orderReceived = (
      <div>
        <h3>Order received!</h3>
        <p>Food: {this.state.food}</p>
        <p>Quantity: {this.state.quantity}</p>
        <p>Coming right up!</p>
      </div>
    );

    let visibleElement = this.state.ordered ? orderReceived : orderForm;

    return (
      <div>
        <h1>What would you like to eat?</h1>
        {visibleElement}
      </div>
    );
  }
}

render(<App />, document.getElementById("root"));
```

Note that just like in vanilla JavaScript, you need to be sure to prevent the default behavior of a form submission if you want to avoid a page reload.

Let's look at one more example. In this component, we're using controlled components to automatically perform some validation and formatting of data that the user is providing:

```jsx
import React, { Component } from "react";
import { render } from "react-dom";

class App extends Component {
  constructor(props) {
    super(props);
    this.state = { username: "" };
    this.handleChange = this.handleChange.bind(this);
    this.handleSubmit = this.handleSubmit.bind(this);
  }

  handleChange(e) {
    this.setState({ username: e.target.value.slice(0, 8) });
  }

  handleSubmit(e) {
    e.preventDefault();
    alert(`Hello ${this.state.username}!`);
  }

  render() {
    let error = null;
    if (/[^A-Z0-9]/gi.test(this.state.username)) {
      error = (
        <p style={{ color: "red" }}>HEY! Only letters and numbers allowed.</p>
      );
    }

    return (
      <div>
        <h1>Create your username (max 8 characters).</h1>
        <form onSubmit={this.handleSubmit}>
          <input
            onChange={this.handleChange}
            placeholder="Enter your username"
            name="username"
            value={this.state.username}
          />
          {error}
          <input type="submit" value="Sign me up!" />
        </form>
      </div>
    );
  }
}

render(<App />, document.getElementById("root"));
```

In this example, we're using a controlled component to do two things. First, we're validating the value in the input, to make sure it consists of only letters and numbers. Second, we're formatting the value so that it doesn't exceed eight characters. This is one of the big advantages of using a controlled component instead of an uncontrolled one.

# Refs

So far, we have seen that the only way for a parent component to pass information to a child component is through `props`. We've also seen how you can pass an event handler on the parent down to the child as a prop, so that the child can effectively alert the parent that it should set some state and re-render the child.

Sometimes, however, you may want to re-render a child without having to set state on the parent. In almost all cases, the pattern we've discussed before is how you should think of data flowing through your application, but in exceptional cases you can use a tool called refs.

Be aware that `refs` are not something you should be using frequently. When coming from a library like `jQuery`, it is easy to think that using `refs` is the way to get things done with React, but anytime that you are adding a ref, ask yourself "Can I do this using state?"

To learn more about refs, including exploring some use cases, you can always head to the [docs](https://facebook.github.io/react/docs/refs-and-the-dom.html). For now, let's look at an example.

First, let's take a look at a component that doesn't use `refs`:

```jsx
import React from "react";
import { render } from "react-dom";

class SecurityForm extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      pin: "",
      error: false
    };
    this.handleSubmit = this.handleSubmit.bind(this);
    this.handleChange = this.handleChange.bind(this);
  }

  handleSubmit(e) {
    e.preventDefault();
    if (this.state.pin === "1234") {
      alert("Thanks for the pin!");
      this.setState({ pin: "" });
    } else {
      this.setState({ pin: "", error: true });
    }
  }

  handleChange(e) {
    this.setState({ pin: e.target.value, error: false });
  }

  render() {
    let error = this.state.error ? <p>Wrong. Please Try again.</p> : null;
    return (
      <form onSubmit={this.handleSubmit}>
        <label>Enter your pin:</label>
        <input
          type="text"
          value={this.state.pin}
          onChange={this.handleChange}
        />
        <button type="submit">Enter</button>
        {error}
      </form>
    );
  }
}

render(<SecurityForm />, document.getElementById("app"));
```

This app has a single component, a form that prompts the user to enter their pin code. If the pin is wrong, the user is prompted to try again.

However, after the form is submitted, the input field loses focus. It would be nice if we could automatically give focus to the input if the user supplies the wrong pin. Here's an example of where `refs` can be useful, as it can supply us with a reference to the DOM element that we can then apply focus to.

Here's a solution to the problem:

```jsx
import React from "react";
import { render } from "react-dom";

class SecurityForm extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      pin: "",
      error: false
    };
    this.handleSubmit = this.handleSubmit.bind(this);
    this.handleChange = this.handleChange.bind(this);
    this.addFocus = this.addFocus.bind(this);
  }

  addFocus() {
    this.input.focus();
  }

  handleSubmit(e) {
    e.preventDefault();
    if (this.state.pin === "1234") {
      alert("Thanks for the pin!");
      this.setState({ pin: "" });
    } else {
      this.setState({ pin: "", error: true });
      this.addFocus();
    }
  }

  handleChange(e) {
    this.setState({ pin: e.target.value, error: false });
  }

  render() {
    let error = this.state.error ? <p> Wrong. Please Try again.</p> : null;
    return (
      <form onSubmit={this.handleSubmit}>
        <label>Enter your pin:</label>
        <input
          type="text"
          value={this.state.pin}
          onChange={this.handleChange}
          ref={el => (this.input = el)}
        />
        <button type="submit">Enter</button>
        {error}
      </form>
    );
  }
}

render(<SecurityForm />, document.getElementById("app"));
```

Notice that we used a `ref` on the input. Refs take callback functions, and those callback functions have references to the current DOM element (or React component, depending on where the `ref` is placed). In this case, we're setting a property of `input` on `this` and assigning it to the input where the user is typing. This way, when the form is submitted, we can call this.addFocus if the pin code is incorrect and automatically focus the input again.

# Controlled vs. uncontrolled components

Earlier in the chapter we saw how to create controlled components by letting React control the state of form elements 
like `inputs` and `selects`.

The upside to controlled components is that all state on the page is managed by React, which is typically as it should be. However, if you'd like the DOM to manage some of that state, you can use `refs` to create what are called uncontrolled components. This might be useful if you're trying to integrate React code with non-React code, though as a general principle, if you're building something from scratch with React, you shouldn't be using uncontrolled components.

In case you're curious, though, here's how the food order example from above could be refactored to use uncontrolled components and refs:

```jsx
import React, { Component } from "react";
import { render } from "react-dom";

class App extends Component {
  constructor(props) {
    super(props);
    this.state = { ordered: false };
    this.food = { value: "" };
    this.quantity = { value: "" };
    this.handleSubmit = this.handleSubmit.bind(this);
  }

  handleSubmit(e) {
    e.preventDefault();
    this.setState({
      ordered: true
    });
  }

  render() {
    let orderForm = (
      <form onSubmit={this.handleSubmit}>
        <div>
          <label>
            Food
            <input
              ref={el => (this.food = el)}
              placeholder="What food would you like?"
              name="food"
            />
          </label>
        </div>
        <div>
          <label>
            Quantity
            <select ref={el => (this.quantity = el)} name="quantity">
              <option value="1">1</option>
              <option value="2">2</option>
              <option value="3">3</option>
              <option value="4">4</option>
              <option value="5">5</option>
            </select>
          </label>
        </div>
        <input type="submit" value="Place your order" />
      </form>
    );

    let orderReceived = (
      <div>
        <h3>Order received!</h3>
        <p>Food: {this.food.value}</p>
        <p>Quantity: {this.quantity.value}</p>
        <p>Coming right up!</p>
      </div>
    );

    let visibleElement = this.state.ordered ? orderReceived : orderForm;

    return (
      <div>
        <h1>What would you like to eat?</h1>
        {visibleElement}
      </div>
    );
  }
}

render(<App />, document.getElementById("root"));
```

Since the DOM retains control over the state of the form elements, we no longer need a `handleChange` function.

It's not necessarily true that you should always use controlled components, but in general they are a more flexible pattern, and align more closely with the "React" way to do things. If you'd like to read more about when you might want to use a controlled component versus an uncontrolled component, check out [this](https://goshakkk.name/controlled-vs-uncontrolled-inputs-react/) article.

# refs as strings

If you're reading tutorials, you may sometimes see a syntax like `ref='foo'`, where the `ref` is being assigned to a string rather than a callback. But this should not be used: the React [documentation](https://facebook.github.io/react/docs/refs-and-the-dom.html#legacy-api-string-refs) specifically discourages this syntax in current projects, and this syntax should be considered deprecated.

# getDOMNode()
Similarly, some tutorials you may see a method called `getDOMNode`. This is also deprecated and should **not** be used. You can learn more about the changes [here](https://stackoverflow.com/questions/30190608/react-js-the-difference-between-finddomnode-and-getdomnode). If you MUST get access to a DOM node, using the ref callback is the preferred way of doing so.

# When not to use refs

If we haven't yet convinced you that you should avoid using refs in your projects, don't take it from us. Here's what the React docs have to say:

Your first inclination may be to use refs to "make things happen" in your app. If this is the case, take a moment and think more critically about where state should be owned in the component hierarchy. Often, it becomes clear that the proper place to "own" that state is at a higher level in the hierarchy.

For more on refs vs. state, check out [this](https://stackoverflow.com/questions/29503213/use-state-or-refs-in-react-js-form-components) StackOverflow question.

# Additional Resources

- [http://reactkungfu.com/2015/09/common-react-dot-js-mistakes-unneeded-state/](http://reactkungfu.com/2015/09/common-react-dot-js-mistakes-unneeded-state/)

# Exercises

## Forms Exercises

## Part 1

Create an app with a form that lets you add `divs` to the page. Inside of the form you should be able to specify the `div`'s width, height, and background color. When you submit the form, the div should show up on the page with the specified style.

## Part 2

Let's revisit our `Todo` app from the events chapter. Create another component called `NewTodoForm` which should render a form that, when submitted, will create a new Todo. 

# Next

When you're ready, move on to [Component Life Cycle](./10-lifecycle.md)
