[toc]

https://underthehood.myob.com/javascript-es6-for-the-win/
https://underthehood.myob.com/changing-of-the-guard-in-web-technologies/

ES6 has been approved and is now officially ES2015 !

通过一些“转换编译器”（Transpilers），让浏览器支持 ES6 或 ES7。现在这样哟uliangge：

- [Traceur](https://github.com/google/traceur-compiler)
- [Babel](https://babeljs.io/) (formerly known as 6to5)

这里选择Babel。因为它支持ES6，部分ES7，还支持React JSX。

### ES6、Babel的新特性举例

**类**

```js
// Before:
function Fruit(size, age, colour) {
  this.size = size;
  this.age = age;
  this.colour = colour;
}

Fruit.type = function() {
  return 'This is a fruit.';
}

function Banana(size, age, colour) {
  Fruit.call(this, size, age, colour);
  this.peel = true;
}

// Still requires a recent browser for the following line
Banana.prototype = Object.create(Fruit.prototype);

Banana.prototype.open = function open() {
  this.peel = false;
}
```

```js
// After:
class Fruit {
  contructor(size, age, colour) {
    this.size = size;
    this.age = age;
    this.colour = colour;
  }

  // Class methods - static methods
  static type() {
    return 'This is a fruit.';
  }
}

class Banana extends Fruit {
  contructor(size, age, colour) {
    super(size, age, colour);
    this.peel = true;
  }

  open() {
    this.peel = false;
  }
}
```

**Arrow functions**

```js
// Before:
something.on('start', function(bananas) {
  return bananas > 10;
});
```

```js
// After:
something.on('start', bananas => bananas > 10);
```

**常量**

```js
const MYCONSTANT = "After declaration no changing";
```

**默认参数**

```js
function myfunction(value, option = 'default') {
  // If you call myfunction(value), here option will be equal to 'default'
  // Replaces your good old: option = option || 'default';
}
```

**模板字面量**

```js
var company = 'MYOB', language = 'Javascript';
var messsage = `${company} uses cutting edge ${language}.`;
// message will be interpolated into 'MYOB uses cutting edge Javascript.'
// Replaces your good old: message = company + ' uses cutting edge ' + language;
```

**Let scoping (replacing var in many cases)**

```js
let x = 10;
{
  let x = 20;
  // Inside the block x can be redefined
}
// Outside of the scoped block x = 10
```

**模块**

```js
// In a fruit.js file
export default function eat(fruit) {
  return `You eat a ${fruit}.`;
}

// In a main.js file
import eat from 'fruit';
eat('Banana');
// The function will return 'You eat a Banana.'
```

**Class properties (ES7 - draft)**

```js
// Instead of creating properties in the class constructor like so:
// this.size = 0;
class Fruit {
  size = 0;
  age = 0;
  colour = 'white';
}
```

**Computed property names**

```js
    var namespace = 'MYOB';
    var something = {
      [namespace + '']: 'dynamic naming'
    };
    // something['MYOBid'] or something.MYOBid return 'dynamic naming'
```

**Decorators (ES7 - draft)**

```js
// Based on Yehuda Katz proposal (EmberJS Core team member)
@memoize
function calculate(x, y, z) { ... }

// Example of a computed property
class Person extends Model {
  @readonly
  name() {
    return `${this.firstName} ${this.lastName}`;
  }
}
```

Many other features are available - similar to constructs you might be used to in languages such as Python, like array comprehensions, generators, destructuring, object rest/spread...

### Webpack, Babel, React, Flux

In this post we will see how to setup Webpack to use Babel and will build a simple React + Flux page using some of the new features of Javascript like classes with the very latest React syntax (as of version 0.13.x).

文件结构：

    src/ +
         | actions/
         | components/
         | stores/
         index.html
         module.js

When writing Flux applications it is a common choice to separate the different layers into different directories. The entry point to our application will be **module.js** and **index.html** will be a simple HTML page using the components we will build.

The stores/ folder contains our model and stores our data. The components/ folder contain the React components responsible for interacting with this data and the actions/ folder will contain the actions your component can perform to mutate the data in the store.

Last but not least module.js will bootstrap our application, what Webpack calls the entry point.

**package.json**：

```json
{
  "name": "webpack-example",
  "scripts": {
    "build": "webpack --progress --profile --colors",
    "watch": "webpack-dev-server --hot --inline --progress --colors",
    "test": "echo \"No unit tests and e2e tests yet\" && exit 1"
  },
  "license": "MIT",
  "dependencies": {
    "alt": "0.17.1",
    "babel-runtime": "5.6.18",
    "react": "0.13.3",
    "react-router": "0.13.3"
  },
  "devDependencies": {
    "babel-core": "5.6.18",
    "babel-loader": "5.3.1",
    "html-webpack-plugin": "1.6.0",
    "webpack": "1.10.1",
    "webpack-dev-server": "1.10.1"
  }
}
```

The Babel runtime provides polyfills and utility functions for the ES6 and ES7 features your browser might not yet support.

`html-webpack-plugin` is a handy Webpack plugin that automatically generates the import scripts we need in our HTML page.

```html
    <!DOCTYPE html>
    <html>
      <head>
        <title>Webpack example</title>
      </head>
      <body>
        <div id="content"></div>
      </body>
    </html>
```

Next, we need our Javascript entry point `module.js` to bootstrap our application.

```js
    import React from 'react';

    React.render(
      <h1>Example</h1>,
      document.getElementById('content')
    );
```

在根文件夹下创建 webpack.config.js ：

```js
'use strict';

var webpack = require('webpack'),
  HtmlWebpackPlugin = require('html-webpack-plugin'),
  path = require('path'),
  srcPath = path.join(__dirname, 'src');

module.exports = {
  target: 'web',
  cache: true,
  entry: {
    module: path.join(srcPath, 'module.js'),
    common: ['react', 'react-router', 'alt']
  },
  resolve: {
    root: srcPath,
    extensions: ['', '.js'],
    modulesDirectories: ['node_modules', 'src']
  },
  output: {
    path: path.join(__dirname, 'tmp'),
    publicPath: '',
    filename: '[name].js',
    library: ['Example', '[name]'],
    pathInfo: true
  },

  module: {
    loaders: [
      {test: /\.js?$/, exclude: /node_modules/, loader: 'babel?cacheDirectory'}
    ]
  },
  plugins: [
    new webpack.optimize.CommonsChunkPlugin('common', 'common.js'),
    new HtmlWebpackPlugin({
      inject: true,
      template: 'src/index.html'
    }),
    new webpack.NoErrorsPlugin()
  ],

  debug: true,
  devtool: 'eval-cheap-module-source-map',
  devServer: {
    contentBase: './tmp',
    historyApiFallback: true
  }
};
```

`output`中，`library` can help you define some namespacing if you export this project and the other fields are not critical at this point.

`HtmlWebpackPlugin` will take our `index.html` as a template and automatically inject the scripts tags to load our `common.js` and `module.js` bundles.

`NoErrorsPlugin` is a little plugin used when running webpack-dev-server just to make sure it doesn't refresh your browser if your latest change throws an error when rebuilding.

**现在可以跑起来了**：

- To only build your project: `npm run build` - check your tmp/ folder, you should find common.js, module.js and index.html.
- To build and run webpack-dev-server: npm run watch - open http://localhost:8080 and voila ! Your React application is up and running. Try to make a change to the module.js file and enjoy the watching and hot-reloading features of Webpack. It will quickly rebuild, creating a diff and a patch of your changes and refresh your browser.

### 路由

In our src folder, we are going to create a new file called `routes.js` alongside the existing index.html and module.js files.

```js
    import React from 'react';
    import {Route} from 'react-router';

    import Main from 'components/main';
    import Example from 'components/example';

    const routes = (
      <Route handler={Main}>
        <Route name='example' handler={Example}/>
      </Route>
    );

    export default routes;
```

The `import {Route} from 'react-router'` syntax basically says to only import one of the exported objects/functions of the module, namely `Router`.

At the end of the file the export default routes is how you can define what gets exported from your module. In our case we are going to export the `routes` we constructed so they can be consumed elsewhere. The `default` keyword means that if you import that module without specifying anything in brackets (like we did for `react-router`), what you get by default will be that `routes` object.

Finally, the routing itself simply declares a shell component called `Main` with a child route represented by the component `Example`. Let me stress out here that contrary to the traditional concept of template pages found in most frameworks, in React, a page is in fact a component itself, and components can be composed of myriad of child components.

### 组件

We need 2 for our example. The first one below will be the *shell* component `Main` that we will store in `components/main.js`.

```js
import React from 'react';
import {RouteHandler, Link} from 'react-router';

class Main extends React.Component {
  render() {
    return (
      <div>
        <h1>Example</h1>
        <Link to='example'>Go to the Example page...</Link>
        <RouteHandler/>
      </div>
    );
  }
}

export default Main;
```

The `<RouteHandler/>` tag determine where the example component will be loaded in the page.

Brace yourself for a more exciting component with a lot more awesomeness: `Example`. We will also store it in `components/example.js`.

```js
import React from 'react';
import connectToStores from 'alt/utils/connectToStores';
import DummyStore from 'stores/dummyStore';
import DummyActions from 'actions/dummyActions';

@connectToStores
class Example extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      name: props.name
    }
  }

  static getStores(props) {
    return [DummyStore];
  }

  static getPropsFromStores(props) {
    return DummyStore.getState();
  }

  render() {
    return (
      <div>
        <input type="text" value={this.state.name} onChange={this.onChange}/>
        <h1>It works: {this.props.name}</h1>
      </div>
    );
  }

  onChange = evt => {
    this.setState({name: evt.target.value});
    DummyActions.updateName(evt.target.value);
  }
}

export default Example;
```

We have a good example here of a Flux connected component that uses `Alt.js`, imports a store and actions and plug everything together.

We import a Flux store we call `DummyStore` and actions we call `DummyActions` (more on this later). The only thing that matters is for the store to have a `name` property. We also use a nifty feature of Alt.js called `connectToStores`. This is a higher order function (the new big thing replacing mixins) that we will use to augment our component with store support plumbing.

`connectToStores` can be used as a function wrapper you call on your class or you can use another fantastic ES6 (ES7 actually) feature called `decorators` (like in Python). To decorate our Example component with store capabilities, we just add `@connectToStores` before the class declaration. Please note that like in many other languages, you can use decorators on class, functions and so on... This allows elegant and flexible composition strategies instead of abusing inheritance

Using `@connectToStores` means we need to override two functions (ES6 static functions): `getStores()` and `getPropsFromStores()`. The first one is to define the store(s) we are going to use, in that case `DummyStore`. The second one returns the current state of your store(s) properties. As a full introduction on Flux and Alt.js is out of scope here, you can learn more about how Alt.js makes your Flux life easier on their documentation website but I recommend looking at up to date documentation on their [Github README](https://github.com/goatslacker/alt).

So to propagate changes to data from a component back to a store you use actions. The `onChange()` member function role is exactly that one in this example. It will read the changed value from the React event object and update the state object, as well as calling the `updateName()` function from `DummyActions` to also notify the store(s) of the change - behind the scenes, the change goes through a `Dispatcher`, a fundamental piece of the flux architecture.

You might be surprised by how this function is declared, because it is an arrow function. Not only does the function keyword disappear but the important piece here is that arrow functions always inherit the context of their enclosing object (this is like having `.bind(this)` appended to your function call). Why is that useful here? Because we want to use the function in the rendered JSX content. Look at `render()`. We can write `onChange={this.onChange}` and be sure it will run our function in the right context.

More information about this and more tips from the Facebook people in [this post](http://babeljs.io/blog/2015/06/07/react-on-es6-plus/).

We now need to change our entry point file `module.js` so it renders our *shell* components, our routes and does any initialisation that we might need to do.

```js
    // Bootstrapping module
    import React from 'react';
    import Router from 'react-router';
    import routes from 'routes';

    Router.run(routes, Router.HistoryLocation, (Root, state) => {
      React.render(<Root {...state}/>, document.getElementById('content'));
    });
```

Notice that `{...state}` code? Thats another great ES6 time saver. It means unpack the state object properties into arguments applied to that component. The `...` is called the **spread operator**.

You might have noticed so far that we used Alt.js features in components but we never really instantiated anything. Well this is a good point, and time has come to create a new file to do that as we will need the store and actions to work together. Let's call it `control.js`.

```js
import Alt from 'alt';

const flux = new Alt();
export default flux;
```

Please note that Alt.js allows you to create several instances of Alt if you want to - useful for isomorphic apps for example, but this would be the purpose of a whole new article.

Armed with our flux instance we can now create the store and actions that deal with our `name` property.

First our actions, actions/dummyActions.js:

```js
import flux from 'control';
import {createActions} from 'alt/utils/decorators';

@createActions(flux)
class DummyActions {
  constructor() {
    this.generateActions('updateName');
  }
}

export default DummyActions;
```

An actions class is a simple ES6 class where you declare the actions you want to be able to perform. We are showing here another condensed version as Alt.js give us great boilerplate killers:

`generateActions()` is a great shortcut to create an action that simply takes a property value as a parameter and dispatches it to the store so it is notified of that properties new value.
`createActions` is a higher order function we use again in its decorator flavour to turn our POJC (Plain Old Javascript Class) into an actions class living under our flux instance of Alt.js.

Next our store, stores/dummyStore.js:

```js
import flux from 'control';
import {createStore, bind} from 'alt/utils/decorators';
import actions from 'actions/dummyActions';

@createStore(flux)
class DummyStore {
  name = 'awesome';

  @bind(actions.updateName)
  updateName(name) {
    this.name = name;
  }
}

export default DummyStore;
```

The store is also a simple class that gets decorated. It imports the action we just declared in dummyActions. By binding our member function updateName to the updateName function from our actions class the value that we get from the action replaces the value of our name property. Again we used the decorator version of bind for elegance and less boilerplate. An alternative would be to write equivalent code in the class constructor.

### 运行吧


