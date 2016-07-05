[toc]

## Tutorial

http://facebook.github.io/react/docs/tutorial.html
https://github.com/facebook/react/blob/master/docs/docs/tutorial.md

以评论区做例子，包括评论列表、提交新评论。支持Markdown。别人的评论能够实时插入列表。

源码：https://github.com/reactjs/react-tutorial

其中包含各个语言版本的简单服务器，用于与前端交互。

To get started using the tutorial, just start editing `public/index.html`.

### 开始

创建HTML：

```html
    <!-- index.html -->
    <!DOCTYPE html>
    <html>
      <head>
        <meta charset="UTF-8" />
        <title>Hello React</title>
        <script src="https://cdnjs.cloudflare.com/ajax/libs/react/{{site.react_version}}/react.js"></script>
        <script src="https://cdnjs.cloudflare.com/ajax/libs/react/{{site.react_version}}/JSXTransformer.js"></script>
        <script src="https://cdnjs.cloudflare.com/ajax/libs/jquery/2.1.1/jquery.min.js"></script>
      </head>
      <body>
        <div id="content"></div>
        <script type="text/jsx">
          // Your code here
        </script>
      </body>
    </html>
```

jQuery 不是 React 必需的。

### 第一个组件

React都是模块化、可组合的组件。在我们的例子中，结构如下：

    - CommentBox
      - CommentList
        - Comment
      - CommentForm

先构建 `CommentBox` 组件：

```js
    // tutorial1.js
    var CommentBox = React.createClass({
      render: function() {
        return (
          <div className="commentBox">
            Hello, world! I am a CommentBox.
          </div>
        );
      }
    });
    React.render(
      <CommentBox />,
      document.getElementById('content')
    );
```

注意到原生的HTML元素小写字母开头，但React的组件大写字母开头。

通过**预编译**可以将 JSX 转换为 纯 Javascript，如下：

```javascript
    // tutorial1-raw.js
    var CommentBox = React.createClass({
      displayName: 'CommentBox',
      render: function() {
        return (
          React.createElement('div', {className: "commentBox"},
            "Hello, world! I am a CommentBox."
          )
        );
      }
    });
    React.render(
      React.createElement(CommentBox, null),
      document.getElementById('content')
    );
```

Read more on the [JSX Syntax article](/react/docs/jsx-in-depth.html).

使用 `React.createClass()` 创建一个新的 React 组件。`render` 方法返回一个 React 组件树。

`<div>`并不是真实的DOM节点；它们是React的 `div` 组件的实例。React is **safe**. We are not generating HTML strings so XSS protection is the default.

You do not have to return basic HTML. You can return a tree of components that you built. This is what makes React **composable**: a key tenet of maintainable frontends.

`React.render()` instantiates the root component, starts the framework, and injects the markup into a raw DOM element, provided as the second argument.

### Composing components

Let's build skeletons for `CommentList` and `CommentForm` which will, again, be simple `<div>`s. Add these two components to your file, keeping the existing `CommentBox` declaration and `React.render` call:

```javascript
    // tutorial2.js
    var CommentList = React.createClass({
      render: function() {
        return (
          <div className="commentList">
            Hello, world! I am a CommentList.
          </div>
        );
      }
    });

    var CommentForm = React.createClass({
      render: function() {
        return (
          <div className="commentForm">
            Hello, world! I am a CommentForm.
          </div>
        );
      }
    });
```

Next, update the `CommentBox` component to use these new components:

```javascript{6-8}
    // tutorial3.js
    var CommentBox = React.createClass({
      render: function() {
        return (
          <div className="commentBox">
            <h1>Comments</h1>
            <CommentList />
            <CommentForm />
          </div>
        );
      }
    });
```

注意到，我们混用了HTML标签和我们自定义的组件。HTML组件根我们自定义的都是 React 组件，没有什么特殊。The JSX compiler will automatically rewrite HTML tags to `React.createElement(tagName)` expressions and leave everything else alone. This is to prevent the pollution of the global namespace.

### props

下面是 `Comment` 组件。它从父母那里接收数据，这些数据可以通过孩子的**属性**获取：`this.props`。 通过`props`，我们可以读取`CommentList`传给`Comment`的数据。

```javascript
    // tutorial4.js
    var Comment = React.createClass({
      render: function() {
        return (
          <div className="comment">
            <h2 className="commentAuthor">
              {this.props.author}
            </h2>
            {this.props.children}
          </div>
        );
      }
    });
```

We access named attributes passed to the component as keys on `this.props` and any nested elements as `this.props.children`.

### 组件的属性

Now that we have defined the `Comment` component, we will want to pass it the author name and comment text. This allows us to reuse the same code for each unique comment. Now let's add some comments within our `CommentList`:

```javascript{6-7}
    // tutorial5.js
    var CommentList = React.createClass({
      render: function() {
        return (
          <div className="commentList">
            <Comment author="Pete Hunt">This is one comment</Comment>
            <Comment author="Jordan Walke">This is *another* comment</Comment>
          </div>
        );
      }
    });
```

Note that we have passed some data from the parent `CommentList` component to the child `Comment` components. For example, we passed *Pete Hunt* (via an attribute) and *This is one comment* (via an XML-like child node) to the first `Comment`. As noted above, the `Comment` component will access these 'properties' through `this.props.author`, and `this.props.children`.

### 添加Markdown

First, add the third-party library **marked** to your application. This is a JavaScript library which takes Markdown text and converts it to raw HTML. This requires a script tag in your head (which we have already included in the React playground):

```html{8}
    <!-- index.html -->
    <head>
      <meta charset="UTF-8" />
      <title>Hello React</title>
      <script src="https://cdnjs.cloudflare.com/ajax/libs/react/{{site.react_version}}/react.js"></script>
      <script src="https://cdnjs.cloudflare.com/ajax/libs/react/{{site.react_version}}/JSXTransformer.js"></script>
      <script src="https://cdnjs.cloudflare.com/ajax/libs/jquery/2.1.1/jquery.min.js"></script>
      <script src="https://cdnjs.cloudflare.com/ajax/libs/marked/0.3.2/marked.min.js"></script>
    </head>
```

Next, let's convert the comment text to Markdown and output it:

```javascript{9}
    // tutorial6.js
    var Comment = React.createClass({
      render: function() {
        return (
          <div className="comment">
            <h2 className="commentAuthor">
              {this.props.author}
            </h2>
            {marked(this.props.children.toString())}
          </div>
        );
      }
    });
```

All we're doing here is calling the marked library. We need to convert `this.props.children` from React's wrapped text to a raw string that marked will understand so we explicitly call `toString()`.

But there's a problem! Our rendered comments look like this in the browser: "`<p>`This is `<em>`another`</em>` comment`</p>`". We want those tags to actually render as HTML.

That's React protecting you from an [XSS attack](https://en.wikipedia.org/wiki/Cross-site_scripting). There's a way to get around it but the framework warns you not to use it:

```javascript{4,10}
    // tutorial7.js
    var Comment = React.createClass({
      render: function() {
        var rawMarkup = marked(this.props.children.toString(), {sanitize: true});
        return (
          <div className="comment">
            <h2 className="commentAuthor">
              {this.props.author}
            </h2>
            <span dangerouslySetInnerHTML={{"{{"}}__html: rawMarkup}} />
          </div>
        );
      }
    });
```

This is a special API that intentionally makes it difficult to insert raw HTML, but for marked we'll take advantage of this backdoor.

**Remember:** by using this feature you're relying on marked to be secure. In this case, we pass `sanitize: true` which tells marked to escape any HTML markup in the source instead of passing it through unchanged.

### Hook up the data model

So far we've been inserting the comments directly in the source code. Instead, let's render a blob of JSON data into the comment list. Eventually this will come from the server, but for now, write it in your source:

```javascript
    // tutorial8.js
    var data = [
      {author: "Pete Hunt", text: "This is one comment"},
      {author: "Jordan Walke", text: "This is *another* comment"}
    ];
```

We need to get this data into `CommentList` in a modular way. Modify `CommentBox` and the `React.render()` call to pass this data into the `CommentList` via `props`:

```javascript{7,15}
    // tutorial9.js
    var CommentBox = React.createClass({
      render: function() {
        return (
          <div className="commentBox">
            <h1>Comments</h1>
            <CommentList data={this.props.data} />
            <CommentForm />
          </div>
        );
      }
    });

    React.render(
      <CommentBox data={data} />,
      document.getElementById('content')
    );
```

Now that the data is available in the `CommentList`, let's render the comments dynamically:

```javascript{4-10,13}
    // tutorial10.js
    var CommentList = React.createClass({
      render: function() {
        var commentNodes = this.props.data.map(function (comment) {
          return (
            <Comment author={comment.author}>
              {comment.text}
            </Comment>
          );
        });
        return (
          <div className="commentList">
            {commentNodes}
          </div>
        );
      }
    });
```

That's it!

### Fetching from the server

Let's replace the hard-coded data with some dynamic data from the server. We will remove the data prop and replace it with a URL to fetch:

```javascript{3}
    // tutorial11.js
    React.render(
      <CommentBox url="comments.json" />,
      document.getElementById('content')
    );
```

This component is different from the prior components because it will have to re-render itself. The component won't have any data until the request from the server comes back, at which point the component may need to render some new comments.

### Reactive state

So far, based on its props, each component has rendered itself once. `props` are **immutable**: they are passed from the parent and are "owned" by the parent. To implement interactions, we introduce mutable **state** to the component. `this.state` is private to the component and can be changed by calling `this.setState()`. When the state updates, the component re-renders itself.

`render()` methods are written declaratively as functions of `this.props` and `this.state`. The framework guarantees the UI is always consistent with the inputs.

When the server fetches data, we will be changing the comment data we have. Let's add an array of comment data to the `CommentBox` component as its state:

```javascript{3-5,10}
    // tutorial12.js
    var CommentBox = React.createClass({
      getInitialState: function() {
        return {data: []};
      },
      render: function() {
        return (
          <div className="commentBox">
            <h1>Comments</h1>
            <CommentList data={this.state.data} />
            <CommentForm />
          </div>
        );
      }
    });
```

`getInitialState()` executes exactly once during the lifecycle of the component and sets up the initial state of the component.

#### 更新状态

When the component is first created, we want to GET some JSON from the server and update the state to reflect the latest data. In a real application this would be a dynamic endpoint, but for this example we will keep things simple by creating a static JSON file `public/comments.json` containing the array of comments:

```javascript
    // tutorial13.json
    [
      {"author": "Pete Hunt", "text": "This is one comment"},
      {"author": "Jordan Walke", "text": "This is *another* comment"}
    ]
```

We'll use jQuery to help make an asynchronous request to the server.

Note: because this is becoming an AJAX application you'll need to develop your app using a web server rather than as a file sitting on your file system. [As mentioned above](#running-a-server), we have provided several servers you can use [on GitHub](https://github.com/reactjs/react-tutorial/). They provide the functionality you need for the rest of this tutorial.

```javascript{6-18}
    // tutorial13.js
    var CommentBox = React.createClass({
      getInitialState: function() {
        return {data: []};
      },
      componentDidMount: function() {
        $.ajax({
          url: this.props.url,
          dataType: 'json',
          cache: false,
          success: function(data) {
            this.setState({data: data});
          }.bind(this),
          error: function(xhr, status, err) {
            console.error(this.props.url, status, err.toString());
          }.bind(this)
        });
      },
      render: function() {
        return (
          <div className="commentBox">
            <h1>Comments</h1>
            <CommentList data={this.state.data} />
            <CommentForm />
          </div>
        );
      }
    });
```

Here, `componentDidMount` is a method called automatically by React when a component is rendered. The key to dynamic updates is the call to `this.setState()`. We replace the old array of comments with the new one from the server and the UI automatically updates itself. Because of this reactivity, it is only a minor change to add live updates. We will use simple polling here but you could easily use **WebSockets** or other technologies.

```javascript{3,15,20-21,35}
    // tutorial14.js
    var CommentBox = React.createClass({
      loadCommentsFromServer: function() {
        $.ajax({
          url: this.props.url,
          dataType: 'json',
          cache: false,
          success: function(data) {
            this.setState({data: data});
          }.bind(this),
          error: function(xhr, status, err) {
            console.error(this.props.url, status, err.toString());
          }.bind(this)
        });
      },
      getInitialState: function() {
        return {data: []};
      },
      componentDidMount: function() {
        this.loadCommentsFromServer();
        setInterval(this.loadCommentsFromServer, this.props.pollInterval);
      },
      render: function() {
        return (
          <div className="commentBox">
            <h1>Comments</h1>
            <CommentList data={this.state.data} />
            <CommentForm />
          </div>
        );
      }
    });

    React.render(
      <CommentBox url="comments.json" pollInterval={2000} />,
      document.getElementById('content')
    );
```

All we have done here is move the AJAX call to a separate method and call it when the component is first loaded and every 2 seconds after that. Try running this in your browser and changing the `comments.json` file; within 2 seconds, the changes will show!

### 添加新评论

Now it's time to build the form. Our `CommentForm` component should ask the user for their name and comment text and send a request to the server to save the comment.

```javascript{5-9}
    // tutorial15.js
    var CommentForm = React.createClass({
      render: function() {
        return (
          <form className="commentForm">
            <input type="text" placeholder="Your name" />
            <input type="text" placeholder="Say something..." />
            <input type="submit" value="Post" />
          </form>
        );
      }
    });
```

Let's make the form interactive. When the user submits the form, we should clear it, submit a request to the server, and refresh the list of comments. To start, let's listen for the form's `submit` event and clear it.

```javascript{3-14,16-19}
    // tutorial16.js
    var CommentForm = React.createClass({
      handleSubmit: function(e) {
        e.preventDefault();
        var author = React.findDOMNode(this.refs.author).value.trim();
        var text = React.findDOMNode(this.refs.text).value.trim();
        if (!text || !author) {
          return;
        }
        // TODO: send request to the server
        React.findDOMNode(this.refs.author).value = '';
        React.findDOMNode(this.refs.text).value = '';
        return;
      },
      render: function() {
        return (
          <form className="commentForm" onSubmit={this.handleSubmit}>
            <input type="text" placeholder="Your name" ref="author" />
            <input type="text" placeholder="Say something..." ref="text" />
            <input type="submit" value="Post" />
          </form>
        );
      }
    });
```

##### Events

React attaches event handlers to components using a camelCase naming convention. We attach an `onSubmit` handler to the **form** that clears the form fields when the form is submitted with valid input.

Call `preventDefault()` on the event to prevent the browser's default action of submitting the form.

##### Refs

We use the `ref` attribute to assign a name to a child component and `this.refs` to reference the component. We can call `React.findDOMNode(component)` on a component to get the **native** browser DOM element.

##### Callbacks as props

When a user submits a comment, we will need to refresh the list of comments to include the new one. It makes sense to do all of this logic in `CommentBox` since `CommentBox` owns the state that represents the list of comments.

We need to pass data from the child component back up to its parent. We do this in our parent's `render` method by passing a new callback (`handleCommentSubmit`) into the child, binding it to the child's `onCommentSubmit` event. Whenever the event is triggered, the callback will be invoked:

```javascript{16-18,31}
    // tutorial17.js
    var CommentBox = React.createClass({
      loadCommentsFromServer: function() {
        $.ajax({
          url: this.props.url,
          dataType: 'json',
          cache: false,
          success: function(data) {
            this.setState({data: data});
          }.bind(this),
          error: function(xhr, status, err) {
            console.error(this.props.url, status, err.toString());
          }.bind(this)
        });
      },
      handleCommentSubmit: function(comment) {
        // TODO: submit to the server and refresh the list
      },
      getInitialState: function() {
        return {data: []};
      },
      componentDidMount: function() {
        this.loadCommentsFromServer();
        setInterval(this.loadCommentsFromServer, this.props.pollInterval);
      },
      render: function() {
        return (
          <div className="commentBox">
            <h1>Comments</h1>
            <CommentList data={this.state.data} />
            <CommentForm onCommentSubmit={this.handleCommentSubmit} />
          </div>
        );
      }
    });
```

Let's call the callback from the `CommentForm` when the user submits the form:

```javascript{10}
    // tutorial18.js
    var CommentForm = React.createClass({
      handleSubmit: function(e) {
        e.preventDefault();
        var author = React.findDOMNode(this.refs.author).value.trim();
        var text = React.findDOMNode(this.refs.text).value.trim();
        if (!text || !author) {
          return;
        }
        this.props.onCommentSubmit({author: author, text: text});
        React.findDOMNode(this.refs.author).value = '';
        React.findDOMNode(this.refs.text).value = '';
        return;
      },
      render: function() {
        return (
          <form className="commentForm" onSubmit={this.handleSubmit}>
            <input type="text" placeholder="Your name" ref="author" />
            <input type="text" placeholder="Say something..." ref="text" />
            <input type="submit" value="Post" />
          </form>
        );
      }
    });
```

Now that the callbacks are in place, all we have to do is submit to the server and refresh the list:

```javascript{17-28}
    // tutorial19.js
    var CommentBox = React.createClass({
      loadCommentsFromServer: function() {
        $.ajax({
          url: this.props.url,
          dataType: 'json',
          cache: false,
          success: function(data) {
            this.setState({data: data});
          }.bind(this),
          error: function(xhr, status, err) {
            console.error(this.props.url, status, err.toString());
          }.bind(this)
        });
      },
      handleCommentSubmit: function(comment) {
        $.ajax({
          url: this.props.url,
          dataType: 'json',
          type: 'POST',
          data: comment,
          success: function(data) {
            this.setState({data: data});
          }.bind(this),
          error: function(xhr, status, err) {
            console.error(this.props.url, status, err.toString());
          }.bind(this)
        });
      },
      getInitialState: function() {
        return {data: []};
      },
      componentDidMount: function() {
        this.loadCommentsFromServer();
        setInterval(this.loadCommentsFromServer, this.props.pollInterval);
      },
      render: function() {
        return (
          <div className="commentBox">
            <h1>Comments</h1>
            <CommentList data={this.state.data} />
            <CommentForm onCommentSubmit={this.handleCommentSubmit} />
          </div>
        );
      }
    });
```

### 优化更新

Our application is now feature complete but it feels slow to have to wait for the request to complete before your comment appears in the list. We can optimistically add this comment to the list to make the app feel faster.

```javascript{17-19}
    // tutorial20.js
    var CommentBox = React.createClass({
      loadCommentsFromServer: function() {
        $.ajax({
          url: this.props.url,
          dataType: 'json',
          cache: false,
          success: function(data) {
            this.setState({data: data});
          }.bind(this),
          error: function(xhr, status, err) {
            console.error(this.props.url, status, err.toString());
          }.bind(this)
        });
      },
      handleCommentSubmit: function(comment) {
        var comments = this.state.data;
        var newComments = comments.concat([comment]);
        this.setState({data: newComments});
        $.ajax({
          url: this.props.url,
          dataType: 'json',
          type: 'POST',
          data: comment,
          success: function(data) {
            this.setState({data: data});
          }.bind(this),
          error: function(xhr, status, err) {
            console.error(this.props.url, status, err.toString());
          }.bind(this)
        });
      },
      getInitialState: function() {
        return {data: []};
      },
      componentDidMount: function() {
        this.loadCommentsFromServer();
        setInterval(this.loadCommentsFromServer, this.props.pollInterval);
      },
      render: function() {
        return (
          <div className="commentBox">
            <h1>Comments</h1>
            <CommentList data={this.state.data} />
            <CommentForm onCommentSubmit={this.handleCommentSubmit} />
          </div>
        );
      }
    });
```

### Congrats!

You have just built a comment box in a few simple steps. Learn more about [why to use React](/react/docs/why-react.html), or dive into the [API reference](/react/docs/top-level-api.html) and start hacking! Good luck!