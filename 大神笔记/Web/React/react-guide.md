[toc]

## Guide

### Why React?

React是一个用于创建UI的Javascript库。很多人选择将 React 看做 MVC 中的V。

我们创造React是为了解决一个问题：数据不断变化的大型应用。为此，React有两个主要思想。

**Simple**
Simply express how your app should look at any given point in time, and React will automatically manage all UI updates when your underlying data changes.
**Declarative**
When the data changes, React conceptually hits the "refresh" button, 并且知道只跟新修改了得部分。

React的中心是构建可重用的组件。或者说，用React，你唯一要做的事是构建组件。

You can learn more about our motivations behind building React in [this blog post](http://facebook.github.io/react/blog/2013/06/05/why-react.html).

### 显示数据

React makes it easy to display data and automatically keeps the interface up-to-date when the data changes.

Let's look at a really simple example. Create a `hello-react.html` file with the following code:

```html
    <!DOCTYPE html>
    <html>
      <head>
        <meta charset="UTF-8" />
        <title>Hello React</title>
        <script src="https://fb.me/react-{{site.react_version}}.js"></script>
        <script src="https://fb.me/JSXTransformer-{{site.react_version}}.js"></script>
      </head>
      <body>
        <div id="example"></div>
        <script type="text/jsx">

          // ** Your code goes here! **

        </script>
      </body>
    </html>
```

Replace the placeholder comment above with the following JSX:

```javascript
    var HelloWorld = React.createClass({
      render: function() {
        return (
          <p>
            Hello, <input type="text" placeholder="Your name here" />!
            It is {this.props.date.toTimeString()}
          </p>
        );
      }
    });

    setInterval(function() {
      React.render(
        <HelloWorld date={new Date()} />,
        document.getElementById('example')
      );
    }, 500);
```

**更新**

The way we are able to figure this out is that React does not manipulate the DOM unless it needs to. **It uses a fast, internal mock DOM to perform diffs and computes the most efficient DOM mutation for you.**

The inputs to this component are called `props`. They're passed as attributes in JSX syntax. You should think of these as **immutable** within the component, that is, never write to `this.props`.

**组件就像函数**

React组件非常简单：你可以将它们看做简单的函数，接收`props`和`state`，渲染HTML。

> 一个限制：React组件只能渲染单个根节点。如果你要返回对个节点，需要在外面保卫一个根节点。

**JSX语法**

JSX让你可以用HTML的语法创建Javascript对象。在React中使用纯Javascript创建链接，需要：

`React.createElement('a', {href: 'https://facebook.github.io/react/'}, 'Hello!')`

但通过JSX：

`<a href="https://facebook.github.io/react/">Hello!</a>`

The easiest way to get started with JSX is to use the in-browser `JSXTransformer`. We strongly recommend that you don't use this in production. You can precompile your code using our command-line [react-tools](https://www.npmjs.com/package/react-tools) package.

**没有JSX的React**

JSX不是必需使用的。可以通过纯Javascript`React.createElement`创建元素。

```javascript
    var child1 = React.createElement('li', null, 'First Text Content');
    var child2 = React.createElement('li', null, 'Second Text Content');
    var root = React.createElement('ul', { className: 'my-list' }, child1, child2);
    React.render(root, document.getElementById('example'));
```

For convenience, you can create short-hand factory functions to create elements from custom components.

```javascript
    var Factory = React.createFactory(ComponentClass);
    ...
    var root = Factory({ custom: 'prop' });
    React.render(root, document.getElementById('example'));
```

React already has built-in factories for common HTML tags:

```javascript
    var root = React.DOM.ul({ className: 'my-list' },
                 React.DOM.li(null, 'Text Content')
               );
```

#### JSX深入

[JSX](https://facebook.github.io/jsx/) is a JavaScript syntax extension that looks similar to XML. You can use a simple JSX syntactic transform with React.

##### HTML标签 vs. React组件

React可以渲染HTML标签（字符串）或React组件（类）。

To render a HTML tag, just use lower-case tag names in JSX:

```javascript
    var myDivElement = <div className="foo" />;
    React.render(myDivElement, document.getElementById('example'));
```

要渲染React组件，先创建一个**局部变量**，注意约定变量名大写开头：

```javascript
    var MyComponent = React.createClass({/*...*/});
    var myElement = <MyComponent someProperty={true} />;
    React.render(myElement, document.getElementById('example'));
```

> 因为JSX是JavaScript，不要将`class`、`for`等标示符作为XML特性名。替换成`className` 、`htmlFor`等。

##### 转换成Javascript

XML风格的语法转换为普通Javascript。XML元素、特性和孩子都转换成传给`React.createElement`的实参。

```javascript
    var Nav;
    // Input (JSX):
    var app = <Nav color="blue" />;
    // Output (JS):
    var app = React.createElement(Nav, {color:"blue"});
```

要使用`<Nav />`，`Nav`变量必须在当前作用于可见。

JSX允许嵌套：

```javascript
    var Nav, Profile;
    // Input (JSX):
    var app = <Nav color="blue"><Profile>click</Profile></Nav>;
    // Output (JS):
    var app = React.createElement(
      Nav,
      {color:"blue"},
      React.createElement(Profile, null, "click")
    );
```

JSX will infer the class's [displayName](/react/docs/component-specs.html#displayname) from the variable assignment when the displayName is undefined:

```javascript
    // Input (JSX):
    var Nav = React.createClass({ });
    // Output (JS):
    var Nav = React.createClass({displayName: "Nav", });
```

利用 [JSX Compiler](/react/jsx-compiler.html) 将JSX转换成Javascript。利用 [HTML to JSX converter](/react/html-jsx.html) 将存在的HTML转换成JSX。

> The JSX expression always evaluates to a ReactElement. The actual
> implementation details may vary. An optimized mode could inline the
> ReactElement as an object literal to bypass the validation code in
> `React.createElement`.

##### Namespaced Components

If you are building a component that has many children, like a form, you might end up with something with a lot of variable declarations:

```javascript
    // Awkward block of variable declarations
    var Form = MyFormComponent;
    var FormRow = Form.Row;
    var FormLabel = Form.Label;
    var FormInput = Form.Input;

    var App = (
      <Form>
        <FormRow>
          <FormLabel />
          <FormInput />
        </FormRow>
      </Form>
    );
```

To make it simpler and easier, *namespaced components* let you use one component that has other components as attributes:

```javascript
    var Form = MyFormComponent;

    var App = (
      <Form>
        <Form.Row>
          <Form.Label />
          <Form.Input />
        </Form.Row>
      </Form>
    );
```

为此，子组件要作为主组件的属性创建：

```javascript
    var MyFormComponent = React.createClass({ ... });

    MyFormComponent.Row = React.createClass({ ... });
    MyFormComponent.Label = React.createClass({ ... });
    MyFormComponent.Input = React.createClass({ ... });
```

JSX will handle this properly when compiling your code.

```javascript
    var App = (
      React.createElement(Form, null,
        React.createElement(Form.Row, null,
          React.createElement(Form.Label, null),
          React.createElement(Form.Input, null)
        )
      )
    );
```

> This feature is available in [v0.11](/react/blog/2014/07/17/react-v0.11.html#jsx) and above.

##### JavaScript表达式

**特性（Attribute）表达式**

Javascript表达式做特性的值，要在表达式外包围`{}`，而不是用`""`。

```javascript
    // Input (JSX):
    var person = <Person name={window.isLoggedIn ? window.name : ''} />;
    // Output (JS):
    var person = React.createElement(
      Person,
      {name: window.isLoggedIn ? window.name : ''}
    );
```

**布尔特性**

省略特性的值，JSX会默认值为`true`。要制定值为`false`必须用表达式。相关的特性如：`disabled`, `required`, `checked` 和 `readOnly`。

```javascript
    // These two are equivalent in JSX for disabling a button
    <input type="button" disabled />;
    <input type="button" disabled={true} />;

    // And these two are equivalent in JSX for not disabling a button
    <input type="button" />;
    <input type="button" disabled={false} />;
```

**孩子表达式**

JavaScript表达式还可以用于表述孩子：

```javascript
    // Input (JSX):
    var content = <Container>{window.isLoggedIn ? <Nav /> : <Login />}</Container>;
    // Output (JS):
    var content = React.createElement(
      Container,
      null,
      window.isLoggedIn ? React.createElement(Nav) : React.createElement(Login)
    );
```

##### 注释

JSX中注册很简单：它们都是JS表达式。You just need to be careful to put `{}` around the comments when you are within the children section of a tag.

```javascript
    var content = (
      <Nav>
        {/* child comment, put {} around */}
        <Person
          /* multi
             line
             comment */
          name={window.isLoggedIn ? window.name : ''} // end of line comment
        />
      </Nav>
    );
```

#### 展开（Spread）特性

If you know all the properties that you want to place on a component ahead of time, it is easy to use JSX:

```javascript
	var component = <Component foo={x} bar={y} />;
```

**Mutating Props is Bad, mkay**

If you don't know which properties you want to set, you might be tempted to add them onto the object later:

```javascript
    var component = <Component />;
    component.props.foo = x; // bad
    component.props.bar = y; // also bad
```

This is an anti-pattern because it means that we can't help you check the right propTypes until way later. This means that your propTypes errors end up with a cryptic stack trace.

The props should be considered **immutable**. 修改`props`对象可能造成无法估计的结果。

**展开（Spread）特性**

Now you can use a new feature of JSX called spread attributes:

```javascript
    var props = {};
    props.foo = x;
    props.bar = y;
    var component = <Component {...props} />;
```

The properties of the object that you pass in are **copied** onto the component's `props`.

You can use this multiple times or combine it with other attributes. The specification order is important. Later attributes override previous ones.

```javascript
    var props = { foo: 'default' };
    var component = <Component {...props} foo={'override'} />;
    console.log(component.props.foo); // 'override'
```

The `...` operator (or spread operator) is already supported for [arrays in ES6](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_operator). There is also an ES7 proposal for [Object Rest and Spread Properties](https://github.com/sebmarkbage/ecmascript-rest-spread). We're taking advantage of these supported and developing standards in order to provide a cleaner syntax in JSX.

#### JSX陷阱

JSX类似HTML，但有一些很重要的不同。

> For DOM differences, such as the inline `style` attribute, check [here](/react/docs/dom-differences.html).

**HTML Entities**

You can insert HTML entities within literal text in JSX:

```javascript
	<div>First &middot; Second</div>
```

If you want to display an HTML entity within dynamic content, you will run into double escaping issues as React escapes all the strings you are displaying in order to prevent a wide range of XSS attacks by default.

```javascript
    // Bad: It displays "First &middot; Second"
    <div>{'First &middot; Second'}</div>
```

There are various ways to work-around this issue. 最简单的方法是直接在Javascript中写Unicode字符。此时要保证文件以UTF-8保存，且设置相应UTF-8指令，让浏览器正确显示。

```javascript
	<div>{'First · Second'}</div>
```

A safer alternative is to find the [unicode number corresponding to the entity](http://www.fileformat.info/info/unicode/char/b7/index.htm) and use it inside of a JavaScript string.

```javascript
    <div>{'First \u00b7 Second'}</div>
    <div>{'First ' + String.fromCharCode(183) + ' Second'}</div>
```

数组中可以混合使用字符串和JSX元素。

```javascript
	<div>{['First ', <span>&middot;</span>, ' Second']}</div>
```

As a last resort, you always have the ability to [insert raw HTML](/react/tips/dangerously-set-inner-html.html).

```javascript
	<div dangerouslySetInnerHTML={{'{{'}}__html: 'First &middot; Second'}} />
```


**Custom HTML Attributes**

若你向一个**原生**HTML元素设置的属性不属于HTML规范，React将不会渲染它们。要使用自定义属性，必须使用前缀`data-`。

```javascript
	<div data-custom-attribute="foo" />
```

[Web Accessibility](http://www.w3.org/WAI/intro/aria) attributes starting with `aria-` will be rendered properly.

```javascript
	<div aria-hidden={true} />
```

### 交互与动态UI

一个简单的例子：

```javascript
    var LikeButton = React.createClass({
      getInitialState: function() {
        return {liked: false};
      },
      handleClick: function(event) {
        this.setState({liked: !this.state.liked});
      },
      render: function() {
        var text = this.state.liked ? 'like' : 'haven\'t liked';
        return (
          <p onClick={this.handleClick}>
            You {text} this. Click to toggle.
          </p>
        );
      }
    });

    React.render(
      <LikeButton />,
      document.getElementById('example')
    );
```

**事件处理与合成（Synthetic）事件**

With React you simply pass your event handler as a camelCased prop similar to how you'd do it in normal HTML. React ensures that all events behave identically in **IE8** and above by implementing a synthetic event system. That is, React knows how to bubble and capture events according to the spec, and the events passed to your event handler are guaranteed to be consistent with [the W3C spec](http://www.w3.org/TR/DOM-Level-3-Events/), regardless of which browser you're using.

**自动绑定与事件代理**

Under the hood, React does a few things to keep your code performant and easy to understand.

- Autobinding：When creating callbacks in JavaScript, you usually need to explicitly bind a method to its instance such that the value of `this` is correct. 在React中，方法自动绑定到组件实例。React caches the bound method such that it's extremely CPU and memory efficient. It's also less typing!
- 事件代理：实际React并不会将实际处理器绑定到节点。当React启动时，它在最上层使用一个监听器监听所有事件。组件挂载或移除后，事件处理器简单的从内部映射中添加或移除。事件发生后，React直到如何使用映射分发它。若在映射中没有事件处理器，React忽略事件。To learn more about why this is fast, see [David Walsh's excellent blog post](http://davidwalsh.name/event-delegate).

**组件是状态机**

React将UI看做简单的状态机：UI会处于各种状态，根据状态做渲染。

In React, you simply update a component's state, and then render a new UI based on this new state. React takes care of updating the DOM for you in the most efficient way.

**状态如何工作**

通知React数据改变，通过`setState(data, callback)`。This method merges `data` into `this.state` and re-renders the component. 当组件完成渲染后，`callback`（可选）回调。Most of the time you'll never need to provide a `callback` since React will take care of keeping your UI up-to-date for you.

**哪些组件应该有状态？**

多数组件只需要从`props`取数据并渲染。但对于需要响应用户输入、服务器请求等，需要用到状态。

**尽量减少有状态的组件**。By doing this you'll isolate the state to its most logical place and minimize redundancy, making it easier to reason about your application.

常见模式是，创建几个无状态的组件，用于渲染数据；然后在它们的层级之上，创建一个有状态的组件，吧状态通过`props`传给孩子。有状态的组件封装所有交互逻辑，无状态的组件负责渲染数据。

**状态应包括什么？**

**State should contain data that a component's event handlers may change to trigger a UI update.** 实际应用中，这些数据应该很少，且可以JSON序列化。When building a stateful component, think about the minimal possible representation of its state, and only store those properties in `this.state`. 在`render()`中根据状态计算其他派生的信息。不要将冗余或派生的值加入状态，否则你得自己负责同步它们，而不是让React帮你计算。

**状态不应包含什么？**

`this.state` should only contain the minimal amount of data needed to represent your UI's state. 它不应包含：

- 派生数据：Don't worry about precomputing values based on state — it's easier to ensure that your UI is consistent if you do all computation within `render()`. 例如，数据中元素个数，应在`render()`中计算：`this.state.listItems.length + ' list items'`。
- React组件：Build them in `render()` based on underlying props and state.
- **Duplicated data from props**: Try to use `props` as the source of truth where possible. One valid use to store props in state is to be able to know its previous values, because props can change over time.

### 多个组件

So far, we've looked at how to write a single component to display data and handle user input. Next let's examine one of React's finest features: composability.

**动机：分离关注**

By building modular components that reuse other components with well-defined interfaces, you get much of the same benefits that you get by using functions or classes. Specifically you can *separate the different concerns* of your app however you please simply by building new components. By building a custom component library for your application, you are expressing your UI in a way that best fits your domain.

**合成的例子**

Let's create a simple Avatar component which shows a profile picture and username using the Facebook Graph API.

```javascript
    var Avatar = React.createClass({
      render: function() {
        return (
          <div>
            <ProfilePic username={this.props.username} />
            <ProfileLink username={this.props.username} />
          </div>
        );
      }
    });

    var ProfilePic = React.createClass({
      render: function() {
        return (
          <img src={'https://graph.facebook.com/' + this.props.username + '/picture'} />
        );
      }
    });

    var ProfileLink = React.createClass({
      render: function() {
        return (
          <a href={'https://www.facebook.com/' + this.props.username}>
            {this.props.username}
          </a>
        );
      }
    });

    React.render(
      <Avatar username="pwh" />,
      document.getElementById('example')
    );
```

**所有权**

在上面的例子中，实例`Avatar`拥有实例`ProfilePic`和`ProfileLink`。在React中，组件A是拥有者，意味着它负责设置其他组件的`props`。若组件`X`在组件`Y`的`render()`方法中创建，则`X`被`Y`拥有。按照之前的讨论，组件不能设置其`props`——`props`由其拥有者设置。这是保证UI一致性的关键。

区别拥有关系和父子关系。

**孩子**

When you create a React component instance, you can include additional React components or JavaScript expressions between the opening and closing tags like this:

```javascript
	<Parent><Child /></Parent>
```

`Parent` can read its children by accessing the special `this.props.children` prop. `this.props.children` is an opaque data structure: use the [React.Children utilities](/react/docs/top-level-api.html#react.children) to manipulate them.

**Child Reconciliation**

Reconciliation is the process by which React updates the DOM with each new render pass. In general, children are reconciled according to the order in which they are rendered. For example, suppose two render passes generate the following respective markup:

```html
    // Render Pass 1
    <Card>
      <p>Paragraph 1</p>
      <p>Paragraph 2</p>
    </Card>
    // Render Pass 2
    <Card>
      <p>Paragraph 2</p>
    </Card>
```

Intuitively, `<p>Paragraph 1</p>` was removed. Instead, React will reconcile the DOM by changing the text content of the first child and destroying the last child. React reconciles according to the *order* of the children.

**Stateful Children**

For most components, this is not a big deal. However, for stateful components that maintain data in `this.state` across render passes, this can be very problematic.

In most cases, this can be sidestepped by hiding elements instead of destroying them:

```html
    // Render Pass 1
    <Card>
      <p>Paragraph 1</p>
      <p>Paragraph 2</p>
    </Card>
    // Render Pass 2
    <Card>
      <p style={{'{{'}}display: 'none'}}>Paragraph 1</p>
      <p>Paragraph 2</p>
    </Card>
```

**Dynamic Children**

The situation gets more complicated when the children are shuffled around (as in search results) or if new components are added onto the front of the list (as in streams). In these cases where the identity and state of each child must be maintained across render passes, you can uniquely identify each child by assigning it a `key`:

```javascript
    render: function() {
    var results = this.props.results;
    return (
      <ol>
        {results.map(function(result) {
          return <li key={result.id}>{result.text}</li>;
        })}
      </ol>
    );
    }
```

When React reconciles the keyed children, it will ensure that any child with `key` will be reordered (instead of clobbered) or destroyed (instead of reused).

The `key` should *always* be supplied directly to the components in the array, not to the container HTML child of each component in the array:

```javascript
    // WRONG!
    var ListItemWrapper = React.createClass({
      render: function() {
        return <li key={this.props.data.id}>{this.props.data.text}</li>;
      }
    });
    var MyComponent = React.createClass({
      render: function() {
        return (
          <ul>
            {this.props.results.map(function(result) {
              return <ListItemWrapper data={result}/>;
            })}
          </ul>
        );
      }
    });
```
```javascript
    // Correct :)
    var ListItemWrapper = React.createClass({
      render: function() {
        return <li>{this.props.data.text}</li>;
      }
    });
    var MyComponent = React.createClass({
      render: function() {
        return (
          <ul>
            {this.props.results.map(function(result) {
               return <ListItemWrapper key={result.id} data={result}/>;
            })}
          </ul>
        );
      }
    });
```

You can also key children by passing a ReactFragment object. See [Keyed Fragments](create-fragment.html) for more details.

**数据流**

In React, data flows from owner to owned component through `props` as discussed above. This is effectively one-way data binding: owners bind their owned component's props to some value the owner has computed based on its `props` or `state`. Since this process happens recursively, data changes are automatically reflected everywhere they are used.

**A Note on Performance**

You may be thinking that it's expensive to change data if there are a large number of nodes under an owner. The good news is that JavaScript is fast and `render()` methods tend to be quite simple, so in most applications this is extremely fast. Additionally, the bottleneck is almost always the DOM mutation and not JS execution. React will optimize this for you using batching and change detection.

However, sometimes you really want to have fine-grained control over your performance. In that case, simply override `shouldComponentUpdate()` to return false when you want React to skip processing of a subtree. See [the React reference docs](/react/docs/component-specs.html) for more information.

> If `shouldComponentUpdate()` returns false when data has actually changed, React can't keep your UI in sync. Be sure you know what you're doing while using it, and only use this function when you have a noticeable performance problem. Don't underestimate how fast JavaScript is relative to the DOM.

### 可重用的组件

When designing interfaces, break down the common design elements (buttons, form fields, layout components, etc.) into reusable components with well-defined interfaces. That way, the next time you need to build some UI, you can write much less code. This means faster development time, fewer bugs, and fewer bytes down the wire.

#### Prop Validation

As your app grows it's helpful to ensure that your components are used correctly. We do this by allowing you to specify `propTypes`. `React.PropTypes` exports a range of validators that can be used to make sure the data you receive is valid. When an invalid value is provided for a prop, a warning will be shown in the JavaScript console. Note that for performance reasons `propTypes` is only checked in development mode. Here is an example documenting the different validators provided:

```javascript
    React.createClass({
      propTypes: {
        // You can declare that a prop is a specific JS primitive. By default, these
        // are all optional.
        optionalArray: React.PropTypes.array,
        optionalBool: React.PropTypes.bool,
        optionalFunc: React.PropTypes.func,
        optionalNumber: React.PropTypes.number,
        optionalObject: React.PropTypes.object,
        optionalString: React.PropTypes.string,

        // Anything that can be rendered: numbers, strings, elements or an array
        // (or fragment) containing these types.
        optionalNode: React.PropTypes.node,

        // A React element.
        optionalElement: React.PropTypes.element,

        // You can also declare that a prop is an instance of a class. This uses
        // JS's instanceof operator.
        optionalMessage: React.PropTypes.instanceOf(Message),

        // You can ensure that your prop is limited to specific values by treating
        // it as an enum.
        optionalEnum: React.PropTypes.oneOf(['News', 'Photos']),

        // An object that could be one of many types
        optionalUnion: React.PropTypes.oneOfType([
          React.PropTypes.string,
          React.PropTypes.number,
          React.PropTypes.instanceOf(Message)
        ]),

        // An array of a certain type
        optionalArrayOf: React.PropTypes.arrayOf(React.PropTypes.number),

        // An object with property values of a certain type
        optionalObjectOf: React.PropTypes.objectOf(React.PropTypes.number),

        // An object taking on a particular shape
        optionalObjectWithShape: React.PropTypes.shape({
          color: React.PropTypes.string,
          fontSize: React.PropTypes.number
        }),

        // You can chain any of the above with `isRequired` to make sure a warning
        // is shown if the prop isn't provided.
        requiredFunc: React.PropTypes.func.isRequired,

        // A value of any data type
        requiredAny: React.PropTypes.any.isRequired,

        // You can also specify a custom validator. It should return an Error
        // object if the validation fails. Don't `console.warn` or throw, as this
        // won't work inside `oneOfType`.
        customProp: function(props, propName, componentName) {
          if (!/matchme/.test(props[propName])) {
            return new Error('Validation failed!');
          }
        }
      },
      /* ... */
    });
```


#### Default Prop Values

React lets you define default values for your `props` in a very declarative way:

```javascript
    var ComponentWithDefaultProps = React.createClass({
      getDefaultProps: function() {
        return {
          value: 'default value'
        };
      }
      /* ... */
    });
```

The result of `getDefaultProps()` will be cached and used to ensure that `this.props.value` will have a value if it was not specified by the parent component. This allows you to safely just use your `props` without having to write repetitive and fragile code to handle that yourself.

#### 转移属性：快捷方式

React中一类常见组件是简单扩展基础HTML元素的组件。此时你或许向把传给组件的HTML属性直接拷贝到HTML元素特性，省去一个个打。You can use the JSX `_spread_` syntax to achieve this:

```javascript
    var CheckLink = React.createClass({
      render: function() {
        // This takes any props passed to CheckLink and copies them to <a>
        return <a {...this.props}>{'√ '}{this.props.children}</a>;
      }
    });

    React.render(
      <CheckLink href="/checked.html">
        Click here!
      </CheckLink>,
      document.getElementById('example')
    );
```

#### Single Child

With `React.PropTypes.element` you can specify that only a single child can be passed to a component as children.

```javascript
    var MyComponent = React.createClass({
      propTypes: {
        children: React.PropTypes.element.isRequired
      },

      render: function() {
        return (
          <div>
            {this.props.children} // This must be exactly one element or it will throw.
          </div>
        );
      }

    });
```

#### Mixins

Components are the best way to reuse code in React, but sometimes very different components may share some common functionality. These are sometimes called [cross-cutting concerns](https://en.wikipedia.org/wiki/Cross-cutting_concern). React provides `mixins` to solve this problem.

一种常见的组件是，需要定期更新自己。使用`setInterval()`要记得用完清理。React提供一些生命周期方法，让你知道组件何时需要被创建和销毁。于是你可以创建一个mixin，在组件创建时启动`setInterval()`，在组件销毁时停止定时器。

```javascript
    var SetIntervalMixin = {
      componentWillMount: function() {
        this.intervals = [];
      },
      setInterval: function() {
        this.intervals.push(setInterval.apply(null, arguments));
      },
      componentWillUnmount: function() {
        this.intervals.forEach(clearInterval);
      }
    };

    var TickTock = React.createClass({
      mixins: [SetIntervalMixin], // Use the mixin
      getInitialState: function() {
        return {seconds: 0};
      },
      componentDidMount: function() {
        this.setInterval(this.tick, 1000); // Call a method on the mixin
      },
      tick: function() {
        this.setState({seconds: this.state.seconds + 1});
      },
      render: function() {
        return (
          <p>
            React has been running for {this.state.seconds} seconds.
          </p>
        );
      }
    });

    React.render(
      <TickTock />,
      document.getElementById('example')
    );
```

A nice feature of mixins is that if a component is using multiple mixins and several mixins define the same lifecycle method (i.e. several mixins want to do some cleanup when the component is destroyed), all of the lifecycle methods are guaranteed to be called. Methods defined on mixins run in the order mixins were listed, followed by a method call on the component.

#### ES6类

You may also define your React classes as a plain JavaScript class. For example using ES6 class syntax:

```javascript
    class HelloMessage extends React.Component {
      render() {
        return <div>Hello {this.props.name}</div>;
      }
    }
    React.render(<HelloMessage name="Sebastian" />, mountNode);
```

The API is similar to `React.createClass` with the exception of `getInitialState`. Instead of providing a separate `getInitialState` method, you set up your own `state` property in the constructor.

Another difference is that `propTypes` and `defaultProps` are defined as properties on the constructor instead of in the class body.

```javascript
    export class Counter extends React.Component {
      constructor(props) {
        super(props);
        this.state = {count: props.initialCount};
      }
      tick() {
        this.setState({count: this.state.count + 1});
      }
      render() {
        return (
          <div onClick={this.tick.bind(this)}>
            Clicks: {this.state.count}
          </div>
        );
      }
    }
    Counter.propTypes = { initialCount: React.PropTypes.number };
    Counter.defaultProps = { initialCount: 0 };
```

**No Autobinding**

Methods follow the same semantics as regular ES6 classes, meaning that they don't automatically bind `this` to the instance. You'll have to explicitly use `.bind(this)` or [arrow functions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions) `=>`.

**No Mixins**

Unfortunately ES6 launched without any mixin support. Therefore, there is no support for mixins when you use React with ES6 classes. Instead, we're working on making it easier to support such use cases without resorting to mixins.

### Transferring Props

It's a common pattern in React to wrap a component in an abstraction. The outer component exposes a simple property to do something that might have more complex implementation details.

You can use [JSX spread attributes](/react/docs/jsx-spread.html) to merge the old props with additional values:

```javascript
	<Component {...this.props} more="values" />
```

If you don't use JSX, you can use any object helper such as ES6 `Object.assign` or Underscore `_.extend`:

```javascript
	React.createElement(Component, Object.assign({}, this.props, { more: 'values' }));
```

The rest of this tutorial explains best practices. It uses JSX and experimental ES7 syntax.

**Manual Transfer**

Most of the time you should explicitly pass the properties down. That ensures that you only expose a subset of the inner API, one that you know will work.

```javascript
    var FancyCheckbox = React.createClass({
      render: function() {
        var fancyClass = this.props.checked ? 'FancyChecked' : 'FancyUnchecked';
        return (
          <div className={fancyClass} onClick={this.props.onClick}>
            {this.props.children}
          </div>
        );
      }
    });
    React.render(
      <FancyCheckbox checked={true} onClick={console.log.bind(console)}>
        Hello world!
      </FancyCheckbox>,
      document.getElementById('example')
    );
```

But what about the `name` prop? Or the `title` prop? Or `onMouseOver`?

**Transferring with `...` in JSX**

> In the example below, the `--harmony ` flag is required as this syntax is an experimental ES7 syntax. If using the in-browser JSX transformer, simply open your script with `<script type="text/jsx;harmony=true">`. See the [Rest and Spread Properties ...](/react/docs/transferring-props.html#rest-and-spread-properties-...) section below for more details.

Sometimes it's fragile and tedious to pass every property along. In that case you can use [destructuring assignment](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment) with rest properties to extract a set of unknown properties.

List out all the properties that you would like to consume, followed by `...other`.

```javascript
	var { checked, ...other } = this.props;
```

This ensures that you pass down all the props EXCEPT the ones you're consuming yourself.

```javascript
    var FancyCheckbox = React.createClass({
      render: function() {
        var { checked, ...other } = this.props;
        var fancyClass = checked ? 'FancyChecked' : 'FancyUnchecked';
        // `other` contains { onClick: console.log } but not the checked property
        return (
          <div {...other} className={fancyClass} />
        );
      }
    });
    React.render(
      <FancyCheckbox checked={true} onClick={console.log.bind(console)}>
        Hello world!
      </FancyCheckbox>,
      document.getElementById('example')
    );
```

> In the example above, the `checked` prop is also a valid DOM attribute. If you didn't use destructuring in this way you might inadvertently pass it along.

Always use the destructuring pattern when transferring unknown `other` props.

```javascript
    var FancyCheckbox = React.createClass({
      render: function() {
        var fancyClass = this.props.checked ? 'FancyChecked' : 'FancyUnchecked';
        // ANTI-PATTERN: `checked` would be passed down to the inner component
        return (
          <div {...this.props} className={fancyClass} />
        );
      }
    });
```

**Consuming and Transferring the Same Prop**

If your component wants to consume a property but also wants to pass it along, you can repass it explicitly with `checked={checked}`. This is preferable to passing the full `this.props` object since it's easier to refactor and lint.

```javascript
    var FancyCheckbox = React.createClass({
      render: function() {
        var { checked, title, ...other } = this.props;
        var fancyClass = checked ? 'FancyChecked' : 'FancyUnchecked';
        var fancyTitle = checked ? 'X ' + title : 'O ' + title;
        return (
          <label>
            <input {...other}
              checked={checked}
              className={fancyClass}
              type="checkbox"
            />
            {fancyTitle}
          </label>
        );
      }
    });
```

> Order matters. By putting the `{...other}` before your JSX props you ensure that the consumer of your component can't override them. In the example above we have guaranteed that the input will be of type `"checkbox"`.

**Rest and Spread Properties `...`**

Rest properties allow you to extract the remaining properties from an object into a new object. It excludes every other property listed in the destructuring pattern.

This is an experimental implementation of an [ES7 proposal](https://github.com/sebmarkbage/ecmascript-rest-spread).

```javascript
    var { x, y, ...z } = { x: 1, y: 2, a: 3, b: 4 };
    x; // 1
    y; // 2
    z; // { a: 3, b: 4 }
```

> Use the [JSX command-line tool](https://www.npmjs.com/package/react-tools) with the `--harmony` flag to activate the experimental ES7 syntax.

**Transferring with Underscore**

If you don't use JSX, you can use a library to achieve the same pattern. Underscore supports `_.omit` to filter out properties and `_.extend` to copy properties onto a new object.

```javascript
    var FancyCheckbox = React.createClass({
      render: function() {
        var checked = this.props.checked;
        var other = _.omit(this.props, 'checked');
        var fancyClass = checked ? 'FancyChecked' : 'FancyUnchecked';
        return (
          React.DOM.div(_.extend({}, other, { className: fancyClass }))
        );
      }
    });
```

### Forms

Form components such as `<input>`, `<textarea>`, and `<option>` differ from other native components because they can be mutated via user interactions. These components provide interfaces that make it easier to manage forms in response to user interactions.

For information on events on `<form>` see [Form Events](/react/docs/events.html#form-events).

**Interactive Props**

Form components support a few props that are affected via user interactions:

- `value`, supported by `<input>` and `<textarea>` components.
- `checked`, supported by `<input>` components of type `checkbox` or `radio`.
- `selected`, supported by `<option>` components.

In HTML, the value of `<textarea>` is set via children. In React, you should use `value` instead.

Form components allow listening for changes by setting a callback to the `onChange` prop. The `onChange` prop works across browsers to fire in response to user interactions when:

- The `value` of `<input>` or `<textarea>` changes.
- The `checked` state of `<input>` changes.
- The `selected` state of `<option>` changes.

Like all DOM events, the `onChange` prop is supported on all native components and can be used to listen to bubbled change events.

> For `<input>` and `<textarea>`, `onChange` supersedes — and should generally be used instead of — the DOM's built-in [`oninput`](https://developer.mozilla.org/en-US/docs/Web/API/GlobalEventHandlers/oninput) event handler.

**Controlled Components**

An `<input>` with `value` set is a *controlled* component. In a controlled `<input>`, the value of the rendered element will always reflect the `value` prop. For example:

```javascript
    render: function() {
    	return <input type="text" value="Hello!" />;
    }
```

This will render an input that always has a value of `Hello!`. Any user input will have no effect on the rendered element because React has declared the value to be `Hello!`. If you wanted to update the value in response to user input, you could use the `onChange` event:

```javascript
    getInitialState: function() {
    	return {value: 'Hello!'};
    },
    handleChange: function(event) {
    	this.setState({value: event.target.value});
    },
    render: function() {
    	var value = this.state.value;
    	return <input type="text" value={value} onChange={this.handleChange} />;
    }
```

In this example, we are simply accepting the newest value provided by the user and updating the `value` prop of the `<input>` component. This pattern makes it easy to implement interfaces that respond to or validate user interactions. For example:

```javascript
    handleChange: function(event) {
    	this.setState({value: event.target.value.substr(0, 140)});
    }
```

This would accept user input but truncate the value to the first 140 characters.

**Uncontrolled Components**

An `<input>` that does not supply a `value` (or sets it to `null`) is an *uncontrolled* component. In an uncontrolled `<input>`, the value of the rendered element will reflect the user's input. For example:

```javascript
    render: function() {
    	return <input type="text" />;
    }
```

This will render an input that starts off with an empty value. Any user input will be immediately reflected by the rendered element. If you wanted to listen to updates to the value, you could use the `onChange` event just like you can with controlled components.

**默认值**

If you want to initialize the component with a non-empty value, you can supply a `defaultValue` prop. For example:

```javascript
    render: function() {
    	return <input type="text" defaultValue="Hello!" />;
    }
```

This example will function much like the **Controlled Components** example above.

Likewise, `<input>` supports `defaultChecked` and `<select>` supports `defaultValue`.

> The `defaultValue` and `defaultChecked` props are only used during initial render. If you need to update the value in a subsequent render, you will need to use a [controlled component](#controlled-components).

**Why Controlled Components?**

Using form components such as `<input>` in React presents a challenge that is absent when writing traditional form HTML. For example, in HTML:

```html
	<input type="text" name="title" value="Untitled" />
```

This renders an input *initialized* with the value, `Untitled`. When the user updates the input, the node's `value` *property* will change. However, `node.getAttribute('value')` will still return the value used at initialization time, `Untitled`.

Unlike HTML, React components must represent the state of the view at any point in time and not only at initialization time. For example, in React:

```javascript
    render: function() {
    	return <input type="text" name="title" value="Untitled" />;
    }
```

Since this method describes the view at any point in time, the value of the text input should *always* be `Untitled`.

**Why Textarea Value?**

In HTML, the value of `<textarea>` is usually set using its children:

```html
    <!-- antipattern: DO NOT DO THIS! -->
    <textarea name="description">This is the description.</textarea>
```

For HTML, this easily allows developers to supply multiline values. However, since React is JavaScript, we do not have string limitations and can use `\n` if we want newlines. In a world where we have `value` and `defaultValue`, it is ambiguous what role children play. For this reason, you should not use children when setting `<textarea>` values:

```javascript
	<textarea name="description" value="This is a description." />
```

If you *do* decide to use children, they will behave like `defaultValue`.

**Why Select Value?**

The selected `<option>` in an HTML `<select>` is normally specified through that option's `selected` attribute. In React, in order to make components easier to manipulate, the following format is adopted instead:

```javascript
    <select value="B">
        <option value="A">Apple</option>
        <option value="B">Banana</option>
        <option value="C">Cranberry</option>
    </select>
```

To make an uncontrolled component, `defaultValue` is used instead.

> You can pass an array into the `value` attribute, allowing you to select multiple options in a `select` tag: `<select multiple={true} value={['B', 'C']}>`.



