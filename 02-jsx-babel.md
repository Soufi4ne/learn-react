# JSX and Babel

# Objectives

By the end of this chapter, you should be able to:

- Understand what JSX is and how it is used with React
- Define what a transpiler is and include babel-core in an application
- Conditionally create JSX based on props

# JSX Introduction

In the last section, we saw that writing HTML directly into our JavaScript doesn't work, because HTML isn't valid JavaScript syntax. 
But when building projects in React, we'll very often put HTML directly in our JavaScript files. 
In order to do this, we'll need to make use of two technologies: JSX and Babel.

JSX allows you to write HTML in your JavaScript. Writing HTML in your JavaScript is a very different approach when comparing React to other frameworks and libraries. The benefit, however, is that we can write our HTML in a way that's familiar, rather than having to pull functions from React to generate DOM elements. Here is another example of what a component using JSX looks like:

```js
class App extends React.Component {
  render() {
    return (
      <div>
        <h1>Hello World!</h1>
      </div>
    );
  }
}
```

As we've already seen, JSX is not understood by the browser. Instead, we need to "convert" our JSX into JavaScript. 
To do the conversion, we need a little bit of help from a transpiler. You can read more about transpilers in this [scotch.io](https://scotch.io/tutorials/javascript-transpilers-what-they-are-why-we-need-them) article, but the main idea is that a transpiler will take our JSX code and convert it into valid JavaScript that the browser will understand. Enter `babel`, a transpiling framework.

# Babel: Getting Set Up

## Simple Web Server For Babel

For our next code example, you won't be able to run the code if you open the HTML file in your browser through the file system! (If the URL in your browser bar starts with `file://`, the transpilation won't work.)

Instead, you must start an HTTP server to serve your code. One simple option for a server is `serve`, a npm package. Open up your Terminal and navigate to the directory where your HTML file is. Then type

```sh
serve .
```

(If you're more comfortable with `http-server`.) Next, open up your browser and go to `http://localhost:8000`. From there, you should be able to navigate to your HTML file. Once you have a web server running, you can move on to the babel examples.

# Babel

In order to convert our JSX to JavaScript, we need to transpile our code. The tool we will use to transpile will be `babel`. 
To include `babel`, we can add the following script tag:

```html
<script src="https://unpkg.com/babel-standalone@6/babel.js"></script>
```

In order to alert the browser that we want `babel` to transpile our code, we also need to modify the `type` on any `script` tag linking to jsx code that we write. Specifically, we need to set type to have a value of `text/babel`.

Let's revisit our broken example from the last section. Inside the `index.html`, let's modify our last `script` tag to use a `jsx` extension and a `type` of `text/babel:`

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <link rel="stylesheet" type="text/css" href="app.css">
  <title>React + Babel</title>
</head>
<body>
  <div id="app"></div>
  <script crossorigin src="https://unpkg.com/react@16/umd/react.development.js"></script>
  <script crossorigin src="https://unpkg.com/react-dom@16/umd/react-dom.development.js"></script>
  <script src="https://unpkg.com/babel-standalone@6/babel.js"></script>
  <script type="text/babel" src="app.jsx"></script>
</body>
</html>
```

Here, again, is the code inside of our `app.jsx`:

```jsx
class App extends React.Component {
  render() {
    return (
      <div>
        <h1>Hello JSX and the World!</h1>
      </div>
    );
  }
}
ReactDOM.render(<App />, document.getElementById('app'));

```

With Babel responsible for transpiling, you should now find that the page loads appropriately!

[You can see this on CodePen here](https://codepen.io/eschoppik/pen/JroWMJ)

So what exactly is going on? To see the result of Babel's transpilation process, check out this [link](https://babeljs.io/repl/#?babili=false&browsers=&build=&builtIns=false&code_lz=MYGwhgzhAECCAO9oFMAeAXZA7AJjASsmMOgHQDCA9gLbyVbbrQDeAUNNAE7Y7KcAUAShbsOXZOgCunLNH6ixHADw4AlgDcAfAsXKAFgEZNACWQgQlaACkAygA1oYXNHR7k0AOqVOIHAEIlAHpDbV1lQLUtHUEAblEAX1ZE1kJidAARAHkAWVJuXD55RSUEJEDNABodHEpgSWpGUgBzCQBREGQGrHQAIQBPAEkcfgByMEQRwVZYoA&debug=false&circleciRepo=&evaluate=false&lineWrap=true&presets=es2015%2Creact%2Cstage-2&prettier=false&targets=&version=6.26.0). While the output JavaScript is not optimized for human readability, you should be able to see that the JSX has been converted into browser compatible JavaScript. For example, Babel translates JSX like <div> into the result of a function call to `React.createElement`.

# Creating Custom Components

We've said that React is all about "components," but so far our single `App` component has just been built using existing HTML elements. Let's change things up by writing our first bona-fide React component inside of our `App`!

Try modifying your `app.jsx` to the following:

```jsx
class MyComponent extends React.Component {
  render() {
    return (
      <div>
        <h2>Here's a component!</h2>
        <p>Aww yeah.</p>
      </div>
    );
  }
}

class App extends React.Component {
  render() {
    return (
      <div>
        <h1>Here's my second React App!</h1>
        <MyComponent />
      </div>
    );
  }
}

ReactDOM.render(<App />, document.getElementById('app'));
```

Note that we used `MyComponent` inside of the `App` just as though it were an HTML element! This is one helpful way to think of components in a React app: they're a way to start building your own custom elements.

A couple other points are worth mentioning:

First, JSX is a little stricter than HTML. All tags must be closing, which is why we wrote `<MyComponent />` and not `<MyComponent >` in the example above. Every opening tag must have a closing tag, or be self closing: otherwise, the transpiler will break.

Second, note that when we created `MyComponent`, we did it using ES2015 class syntax. It used to be possible to create components without ES2015 using a method called `React.createClass`, but this was deprecated in version 15 of React and was removed from version 16. We'll add more ES2015 syntax to our React code over time, but if you're curious about using React without ES2015, you can head over to the [docs](https://reactjs.org/docs/react-without-es6.html) to learn more.

Finally, note that the HTML we want to tie to the component is passed as JSX to the component's `render` method. `render` is a method that our component class expects to receive because it extends from `React.Component`. There are several other methods that you can access; we'll learn more about them later.

# An Introduction to Props

It's cool that we've built our first custom component in React, but our components are still fairly rigid. For example, our `App` component will have the same structure and content no matter where we put it. It would be better if we could add some flexibility to it; for instance, it might be nice if we could modify the text inside of the component when it renders.

To achieve this goal, let's introduce a concept called props. We can think of these as "properties" for our components. We'll talk about props in much more detail later on, but for now, let's just look at a small example.

```jsx
class App extends React.Component {
  render() {
    return (
      <div>
        <h1>Hello {this.props.name}!</h1>
      </div>
    );
  }
}

ReactDOM.render(<App name="student" />, document.getElementById('app'));
```

Notice that in order to specify the name property, we just passed it in to the component using syntax similar to HTML attributes.

Also, as you can see from the above example, if we want to use the value of `this.props.name` in the JSX, then we must wrap it in curly braces. In fact, any code wrapped in curly braces will evaluate JavaScript. `this.props.name` is simply a JavaScript variable that we are looking up inside of the curly braces. Here is another example in which we give our `h1` tag some styling and also comment out some code we're not using:

```jsx
class App extends React.Component {
  render() {
    var style = { color: 'red' };
    return (
      <div>
        <h1 style={style}>Hello {this.props.name}!</h1>
        {/* <p>
                    This JSX will not be rendered because it is commented out using JavaScript
                 </p> */}
      </div>
    );
  }
}

ReactDOM.render(<App name="student" />, document.getElementById('app'));
```

Notice that the `p` tag is commented out because it is wrapped is a JavaScript comment using the curly braces. Another interesting thing to point out here is that we are using a JavaScript object to create a style for our `h1` tag. Because the style variable is a JavaScript object, we must be sure to put the `red` value in quotes.

# Adding CSS Classes to JSX

Let's say we want to add some CSS to a file called `app.css`. Adding a CSS class to your JSX is also possible. 
Add the following CSS to the `app.css` file:

```css
.primary-text {
  color: blue;
  font-size: 2em;
}
```

If we want to add the `primary-text` class to an element in our component, we may be tempted to write the JSX like this:

```html
<p class="primary-text">This should be large blue text</p>
```

However, since `class` is a reserved keyword in JavaScript, if you want to set the class of an element using the attribute object in React, you should use the `className` attribute instead. The full example looks like this:

```jsx
class App extends React.Component {
  render() {
    var style = { color: 'red' };
    return (
      <div>
        <h1 style={style}>Hello {this.props.name}!</h1>
        <p className="primary-text">This should be large blue text</p>
      </div>
    );
  }
}

ReactDOM.render(<App name="student" />, document.getElementById('app'));
```

[You can see what this looks like on CodePen here](https://codepen.io/eschoppik/pen/EwaWQZ)

# Conditional JSX

Very commonly we want to add conditional logic to our JSX. One common way to conditionally produce the JSX we want is to put our conditional logic above our return statement. Additionally, since we have the full power of JavaScript inside of JSX, we can add ternary logic to the JSX. Here's an example:

```jsx
class App extends React.Component {
  render() {
    var name = this.props.name;
    if (name === 'Tim') {
      name = 'favorite instructor';
    } else if (name === 'Matt' || name === 'Elie') {
      name = 'very solid instructor';
    }
    return (
      <div>
        <p>{this.props.name}</p>
        <p>{name}</p>
        {name === 'student' ? (
          <h1>Good job on the course so far!</h1>
        ) : (
          <h1>Hello, {name}!</h1>
        )}
      </div>
    );
  }
}

ReactDOM.render(<App name="Moxie" />, document.getElementById('app'));
```

There's a lot of conditional logic going on in the example above; try to guess what will render on the page before opening it up. At the bottom, try changing the value you pass in to `name` to some other value. How can you get `Good job on the course so far!` to show? How can you get the name to be `favorite instructor`?

# Using Babel In Production

The examples we have seen so far work perfectly fine, but you may have noticed that the websites are rather slow. The reason for the slow down is that we are transpiling our code on the client side (the browser is downloading babel and doing the work of transpiling from JSX to plain JavaScript). We need a better tool for transpiling our code. And we need to make sure that transpiling doesn't happen on the client side.

`Webpack` is going to help us transpile our JSX. `Webpack` is a very helpful tool, but it also has a very steep learning curve. Do not worry though! With a little bit a practice you will get the hang of it. Not only does it allow us to easily include `babel`, 
it also gives us access to some of the best and latest features in JavaScript, specifically `modules`. We'll talk about `webpack` and JavaScript `modules` in the next chapter.

# Exercise

## JSX Exercises

### Part 1

For this assignment you will be creating three components:

1. `FirstComponent`, which is an `h1` with the text "My very first component."
2. `SecondComponent`, which is an `h2` with the text "My second component."
3. `NamedComponent`, which is a `p` that should accept a property of "name" and display the text "My name is " + name.

### Part 2

1. Define a `Tweet` component which takes as props the username of the user who wrote the tweet, the name of the user who wrote the tweet, the date of the tweet, and the message being tweeted.
2. Create an `App` component that renders at least three tweets.
3. Style your `Tweet` component!

### Part 3

Create a component called `Person`. Inside of this component, render a `p` tag which displays "Learn some information about this person". Each person should have name and age properties. 

If the person is over 21 years old, display an additional `h3` that says "have a drink!". Otherwise, display an `h3` that says "you must be 21". If the person's name is longer than 8 characters, only display the first six characters of their name.

Add a prop called hobbies to your `Person` component that accepts an array of hobbies (an array of strings).  Your Person component should list each one of these hobbies as an `li`. 

Finally, render at least three copies of the `Person` component on the page.

# Next

When you're ready, move on to [**Webpack**](./03-webpack.md)