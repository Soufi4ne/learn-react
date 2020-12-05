# Webpack

# Objectives

By the end of this chapter, you should be able to:

- Explain what `webpack` and `babel` are useful for
- Create an application using `create-react-app`
- Configure applications using `webpack`

# Introduction To Webpack

So what is `webpack`? You will commonly hear it defined as a module bundler, or build tool. 
If you are familiar with tools like [Grunt](https://gruntjs.com/) or [Gulp](http://gulpjs.com/), `webpack` has similar features. Ultimately the goal of a build tool is to take all of your files (JavaScript, CSS, etc), then combine and minify those files into a single JavaScript file and a single CSS file. When files are combined and minified, the user of your application can download the website faster.

Another important consideration to keep in mind when using a build tool is the cross browser compatibility of new JavaScript language features (ES2015). Since React uses so many new JavaScript language features in addition to JSX, much of the code that we write won't work in the browser as is. So we need webpack to convert our code into something that will be supported by most browsers. For example, webpack can be configured to transpile our JSX code into plain JavaScript. The transpiled code would then be saved in a file (commonly called `bundle.js`), so that the client does not have to transpile the code in their browser each time the page loads.

ES2015 import syntax is used extensively throughout React code and we will be using `webpack` to transpile the import syntax into something the browser can understand. If you are not familiar with ES2015 import syntax, [MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import) has some examples and you will learn more about it in this lesson as well.

# Create React App

There are quite a few features and concepts to understand when you're diving into webpack. Before we get into how webpack is setup, let's pause and talk about a great tool called `create-react-app` (can be found on github [here](https://github.com/facebookincubator/create-react-app)). Create React app is a command line tool that will generate a very well setup React application, which includes all of the configuration for `webpack`.

If you are brand new to React or if you have never used a build tool before, we would strongly recommend that you get started with React by using `create-react-app`.

Note that `create-react-app` uses Node.js to manage JavaScript dependencies.

To build your first app, open up the terminal and type the following command:

```sh
# You will only need to run this one time on your machine
npm install -g create-react-app
```

The command installs the `create-react-app` command line tool on your computer. You will only need to do this install one time.

Next, in an empty folder, run:

```sh
create-react-app my-app
cd my-app/
npm start
```

Assuming everything went well, you should now have a React application running on your computer and taking advantage of `webpack`.

Let's explore the files that were created:

```sh
my-app/
  README.md
  node_modules/
  package.json
  .gitignore
  public/
    favicon.ico
    index.html
  src/
    App.css
    App.js
    App.test.js
    index.css
    index.js
    logo.svg
```

The most important files for our purposes are the `index.js`, `public/index.html` and `App.js`. If you need to change anything about your html, you would do it in `public/index.html`. If you would like to change the component you render, you would look in `index.js`. 
The `App.js `file is the component that is being rendered in `index.js`. If you would like to modify that component, you can, or if you would like to create other components, you can create a new javascript file. We'll talk more about how these files look in the next lesson.

Going forward, all of the exercises in this course can be accomplished using `create-react-app`. Please read on to get a better understanding of how webpack is working and how the ES2015 import syntax works. However, if you get stuck on a webpack example or concept, don't worry. You can always get started with `create-react-app` instead.

# Essential Files For Webpack

npm's `package.json`

Since we will be using `npm` which is the package manager for `node.js` to fetch packages (`react`, `webpack` etc). We will want to create a package.json. A `package.json` is a file that describes the application we are building and lists all of the dependencies and development dependencies (which are not installed in production). 

To create a package.json file you can type in npm init and then add whatever additional information you want or just keep pressing enter. Almost all of the time, you will not need to edit this file when initializing so you can pass in the -y flag to confirm everything.

Let's create our first` package.json` file using` npm init -y`

# Installing dependencies

Now that we have a `package.json` file configured, let's install all the necessary dependencies entering the following in your terminal:

```sh
npm install -g webpack
npm install --save-dev babel-core babel-loader \
                       babel-preset-es2015
```

The `--save-dev` flag will save the names and versions of these modules to the `package.json` file in a section called `devDependencies`. `devDependencies` are dependencies that you need for development, but not production code. Listing your dependencies in a package.json file is essential when working with other developers as they can easily install all dependencies by simply running `npm install` in the terminal.

# Creating our first webpack.config.js

Now that we have a `package.json` and required dependencies, let's create the configuration file for `webpack`. 
This file is called `webpack.config.js` - let's see what goes inside.

**context** - the absolute path to your project. Typically, `__dirname` is used in the path.

**entry** - the file where we will start bundling. Usually this file is the one that contains your initial `ReactDOM.render` call.

**output** - this object contains a few keys related to where the bundle will be output to

- path - the location for where the bundle should be saved
- filename - the name of the file you want to call your bundle

**devtool** - this is very useful for debugging purposes. We will be using `eval` to see where in our bundle errors are occurring.

**module** - this object can contain quite a few things, we are only concerned now with a key called `rules`

- rules - the value is an array of rules used for transpiling and bundling. Inside of each value in the array, we pass in an object with the following keys
  - use - the loader to load for this rule. In the example, the loader will transpile
  - test - what files to look for (regular expression)
  - exclude - what files to exclude from bundling

Here is our `webpack.config.js` file:

```js
module.exports = {
  // The absolute path to your project
  context: __dirname + "/",
  // the entry point for our app
  entry: "./main.js",
  // where to put the compiled output (what our script tag will link to)
  output: {
    // where does it go?
    path: __dirname + "/",
    // what is the file called?
    filename: "bundle.js"
  },
  // how can we debug our bundle? for production, we can use 'source-map'
  devtool: "eval",
  module: {
    rules: [
      {
        //Check for all js files
        test: /\.js$/,
        // Don't include node_modules directory in the search for js files
        exclude: /node_modules/,
        // Use the babel-loader plugin to transpile the javascript
        use: [
          {
            loader: "babel-loader",
            options: { presets: ["es2015"] }
          }
        ]
      }
    ]
  }
};
```

# Babel Configs in .babelrc

Since we have included `babel` as a loader, there is one more file that we need to configure. This file is called a `.babelrc` and it specifies which babel "presets" or plugins we want to use. The one we will be using is for ES2015. This allows us to use ES2015 modules, which we will see in the next section! Let's create a `.babelrc` file, it's quite small, but essential.

```json
{
  "presets": ["es2015"]
}
```

# Building With Webpack

Now that we have an initial `webpack.config.js` setup, let's trying building some JavaScript! Create a file called `index.html` that has the following code:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>First Build With Webpack</title>
</head>
<body>
    <h1>First Build With Webpack!</h1>
    <script src="bundle.js"></script>
</body>
</html>
```

Next, created a file called `main.js`. Notice that `script` tag in our HTML includes a file called `bundle.js`. 
Webpack will be responsible for producing the `bundle.js `file as output. Let's add the following code to `main.js`:

```js
alert("Successful build with Webpack");
```

Finally, in the root of your project, run the `webpack` command. It should produce a file called `bundle.js`. 
Next, open your `index.html` file in the browser. You should see the alert message.

# Webpack Dev Server

Another nice tool we can use is the `webpack-dev-server` which watches for changes in our files and starts a development server for us. To install it, use `npm install -g webpack-dev-server` . To run the server just type `webpack-dev-server`. Once the server is running, you should be able to visit your application by going to `http://localhost:8080` by default.

# ES2015 modules

Before we start learning about ES2015 modules, let's setup a few files that we can import. 
We will be using an entry file called `main.js`, a file called `index.html` to serve content, and a folder called helpers with some helper files. Here is what our folder structure should look like

```sh
├── helpers
│   └── functions.js
│   └── default.js
├── node_modules
├── index.html
├── main.js
├── package.json
└── webpack.config.js
```

Our index.html is the same as the previous example:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Webpack With Imports</title>
</head>
<body>
    <h1>Webpack With Imports</h1>
    <script src="bundle.js"></script>
</body>
</html>
```

Also create a file called `main.js` and leave it blank for now.

You should be able to build and run our application using `webpack-dev-server`. Open up `http://localhost:8080` in your browser to see our page. Now let's take a look at how ES2015 modules work!

## Named exports with export

In order to import something from a file (let's call the file `helpers/functions.js`), then the `functions.js` file must `export` something for other files to import. The `export` keyword exports a function, object, or other variable for another file to import. For example, add the following code to `helpers/functions.js`:

```js
export function sayHi() {
  console.log("hi!");
}

export function sayBye() {
  console.log("bye!");
}

const instructor = "Elie";
const instructor2 = "Tim";
const instructor3 = "Matt";
export { instructor, instructor2, instructor3 };
```

## Importing using import

To import one of the values form `helpers.js` to another file, try changing `main.js` to have the following code:

```js
import { sayHi } from "./helpers/functions.js";

// sayHi can now be invoked.
sayHi();
```

Assuming you still have your `webpack-dev-server` running at `http://localhost:8080`, refresh your browser and check the developer console to verify that your `bundle.js` file has been updated and that you see hi! in the console.

## Default Exports with default

If we want to export a single value or to have a fallback value for our module, we can use a default export. 
It is not possible to use the `var`, `let`, or `const` keywords on the same line as a export default.

Inside `helpers/default.js`

```js
export default class Person {
  constructor(firstName, lastName) {
    this.firstName = firstName;
    this.lastName = lastName;
  }
  static isPerson(person) {
    return person instanceof this;
  }
  fullName() {
    return `${this.firstName} ${this.lastName}`;
  }
}
```

You can also add the export statement after the class definition:

```js
class Person {
  constructor(firstName, lastName) {
    this.firstName = firstName;
    this.lastName = lastName;
  }
  static isPerson(person) {
    return person instanceof this;
  }
  fullName() {
    return `${this.firstName} ${this.lastName}`;
  }
}

export default Person;
```

If we were using `var`, `let`, or `const`, the export statement must be separate:

```js
const Person = function(firstName, lastName) {
  this.firstName = firstName;
  this.lastName = lastName;
};

export default Person;
```

# Importing A Default Export

When importing something that has been exported using `default`, the name actually does not matter and we do not need to use `{}`.
For example, we could import the `Person` class from above like this:

```js
import Person from "./helpers/default";

const p = new Person("Michael", "Hueter");
```

or we could import the class like this as well:

```js
import AnyName from "./helpers/default";

// AnyName is the same Person class, but we can name it anything
// we like since it is a default export.
const p = new AnyName("Michael", "Hueter");
```

# Import Example

Let's look at another example of using the `import` keyword. Here is what our` main.js` file looks like:

```js
// All imports from ./helpers/functions are NOT exported using default
import {
  sayHi,
  sayBye,
  instructor,
  instructor2,
  instructor3
} from "./helpers/functions";

// Person is exported using default, so we do not need {}.
import Person from "./helpers/default";

sayHi();
sayBye();

console.log(instructor);
console.log(instructor2);
console.log(instructor3);

const p = new Person("Elie", "Schoppik");

console.log(Person.isPerson(p));

console.log(p.fullName());
```

Notice that when you  `importing` a default export, you don't need to wrap what you're importing in curly braces (hence `import Person` and not `import { Person }`). Whenever you're importing a non-default export, however, you need to wrap what you're importing inside of curly braces.

There are quite a few things we can do with the `import` keyword; you can read more about it in this [MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import) article.

# Using webpack with React

Now that we understand how to use modules, let's add React to the mix! We can keep the same exact setup, but just add a few more modules for react and a babel preset!

```sh
npm install --save-dev babel-preset-react react react-dom
```

In our `webpack.config.j`s we have to make one small change. For the test property inside of our rules, 
we are changing the test to `/\.jsx?$/`. The ? in the regular expression means that the x is optional in `.jsx`, so this regular expression matches file names that end in `.js` or .`jsx`:

```js
module.exports = {
  // The absolute path to your project
  context: __dirname + "/",
  // the entry point for our app
  entry: "./main.js",
  // where to put the compiled output (what our script tag will link to)
  output: {
    // where does it go?
    path: __dirname + "/",
    // what is the file called?
    filename: "bundle.js"
  },
  // how can we debug our bundle? for production, we can use 'source-map'
  devtool: "eval",
  module: {
    rules: [
      {
        //Check for all js files
        test: /\.jsx?$/,
        // Don't include node_modules directory in the search for js files
        exclude: /node_modules/,
        // Use the babel-loader plugin to transpile the javascript
        use: [
          {
            loader: "babel-loader",
            options: { presets: ["es2015"] }
          }
        ]
      }
    ]
  }
};
```
We also need to add react to our .babelrc file:

```json
{
  "presets": ["es2015", "react"]
}
```

# Webpack for Production

The easiest way to minify and compress files for production is to run `webpack -p`. There is much more to explore with `webpack`. You can get more in depth explanations of all that `webpack` has to offer at the [webpack docs](https://webpack.js.org/concepts/).

# Exercise

## Webpack Exercises

### Part 1 

Answer the following questions:

- What is `webpack`? What is `babel`?
- What is a `loader`? Give three examples of different kinds of loaders.
- What is the difference between `path` and `publicPath` when used as keys in the `output` object?
- What is a babel `preset`?
- What does `test:/\.jsx?$/` mean inside of `webpack.config.js`?
- What is the `webpack-dev-server`?
- Research _tree shaking_ and _code splitting_. What are these two things and how do they help reduce bundle sizes?
- What does the `default` keyword do when exporting?
- What is object destructuring?
- How can you enable your `webpack.config.js` to do certain things in production versus development?

### Part 2

Refactor your `Person` example from the JSX exercises to use Webpack and run the application using the `webpack-dev-server`.

# Next

When you're ready, move on to [**Props**](./04-props.md)