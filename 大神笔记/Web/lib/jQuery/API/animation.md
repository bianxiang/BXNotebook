[toc]

### .queue()

Show or manipulate the queue of functions to be executed on the matched elements.

#### .queue( [queueName ] )

Returns: Array

显示元素上**要**执行的函数的队列。

	.queue( [queueName ] )

- `queueName`：字符串。队列名。Defaults to `fx`, the standard effects queue.

```
    <!doctype html>
    <html lang="en">
    <head>
      <meta charset="utf-8">
      <title>queue demo</title>
      <style>
      div {
        margin: 3px;
        width: 40px;
        height: 40px;
        position: absolute;
        left: 0px;
        top: 60px;
        background: green;
        display: none;
      }
      div.newcolor {
        background: blue;
      }
      p {
        color: red;
      }
      </style>
      <script src="https://code.jquery.com/jquery-1.10.2.js"></script>
    </head>
    <body>

    <p>The queue length is: <span></span></p>
    <div></div>

    <script>
    var div = $( "div" );

    function runIt() {
      div
        .show( "slow" )
        .animate({ left: "+=200" }, 2000 )
        .slideToggle( 1000 )
        .slideToggle( "fast" )
        .animate({ left: "-=200" }, 1500 )
        .hide( "slow" )
        .show( 1200 )
        .slideUp( "normal", runIt );
    }

    function showIt() {
      var n = div.queue( "fx" );
      $( "span" ).text( n.length );
      setTimeout( showIt, 100 );
    }

    runIt();
    showIt();
    </script>

    </body>
    </html>
```

#### .queue( [queueName ], newQueue )

Returns: jQuery

Description: Manipulate the queue of functions to be executed, once for each matched element.

	.queue( [queueName ], newQueue )
	.queue( [queueName ], callback )

- `queueName`：字符串。队列名。Defaults to `fx`, the standard effects queue.
- `newQueue`：数组。一个新的函数的数组，替换当前队列内容。
- `callback`：函数`( Function next() )`。The new function to add to the queue, with a function to call that will dequeue the next item.

每个元素都可以有一个或多个函数队列。In most applications, only one queue (called `fx`) is used. Queues allow a sequence of actions to be called on an element asynchronously, without halting program execution. The typical example of this is calling multiple animation methods on an element. For example:

```js
$( "#foo" ).slideUp().fadeIn();
```

When this statement is executed, the element begins its sliding animation immediately, but the fading transition is placed on the `fx` queue to be called only once the sliding transition is complete.

The `.queue()` method allows us to directly manipulate this queue of functions. Calling `.queue()` with a callback is particularly useful; it allows us to place a new function at the end of the queue. The callback function is executed once for each element in the jQuery set.

This feature is similar to providing a callback function with an animation method, but does not require the callback to be given at the time the animation is performed.

```js
$( "#foo" ).slideUp();
$( "#foo" ).queue(function() {
  alert( "Animation complete." );
  $( this ).dequeue();
});
```

This is equivalent to:

```js
$( "#foo" ).slideUp(function() {
  alert( "Animation complete." );
});
```

Note that when adding a function with `.queue()`, we should ensure that `.dequeue()` is eventually called so that the next function in line executes.

As of jQuery 1.4, the function that's called is passed another function as the first argument. When called, this automatically **dequeues** the next item and keeps the queue moving. We use it as follows:

```js
$( "#test" ).queue(function( next ) {
    // Do some stuff...
    next();
});
```

例子：

```
    <!doctype html>
    <html lang="en">
    <head>
      <meta charset="utf-8">
      <title>queue demo</title>
      <style>
      div {
        margin: 3px;
        width: 40px;
        height: 40px;
        position: absolute;
        left: 0px;
        top: 30px;
        background: green;
        display: none;
      }
      div.newcolor {
        background: blue;
      }
      </style>
      <script src="https://code.jquery.com/jquery-1.10.2.js"></script>
    </head>
    <body>

    Click here...
    <div></div>

    <script>
    $( document.body ).click(function() {
      $( "div" )
        .show( "slow" )
        .animate({ left: "+=200" }, 2000 )
        .queue(function() {
          $( this ).addClass( "newcolor" ).dequeue();
        })
        .animate({ left: "-=200" }, 500 )
        .queue(function() {
          $( this ).removeClass( "newcolor" ).dequeue();
        })
        .slideUp();
    });
    </script>

    </body>
    </html>
```






### .clearQueue()

Returns: jQuery

从队列中移除所有尚未运行的项。

	.clearQueue( [queueName ] )

- queueName：字符串。队列名。默认为`fx`，即标准特效队列。

执行`.clearQueue()`，队列中所有尚未执行的函数被移除。

不带参数调用时，`.clearQueue()`移除`fx`（标准特效队列）中得函数。这种方式与`.stop(true)`类似。但`.stop()`只用于动画，但`.clearQueue()`可以移除通过`.queue()`添加的，通用jQuery队列中的函数。

例子：

```html
    <!doctype html>
    <html lang="en">
    <head>
      <meta charset="utf-8">
      <title>clearQueue demo</title>
      <style>
      div {
        margin: 3px;
        width: 40px;
        height: 40px;
        position: absolute;
        left: 0px;
        top: 30px;
        background: green;
        display: none;
      }
      div.newcolor {
        background: blue;
      }
      </style>
      <script src="https://code.jquery.com/jquery-1.10.2.js"></script>
    </head>
    <body>

    <button id="start">Start</button>
    <button id="stop">Stop</button>
    <div></div>

    <script>
    $( "#start" ).click(function() {
      var myDiv = $( "div" );
      myDiv.show( "slow" );
      myDiv.animate({
        left:"+=200"
      }, 5000 );

      myDiv.queue(function() {
        var that = $( this );
        that.addClass( "newcolor" );
        that.dequeue();
      });

      myDiv.animate({
        left:"-=200"
      }, 1500 );
      myDiv.queue(function() {
        var that = $( this );
        that.removeClass( "newcolor" );
        that.dequeue();
      });
      myDiv.slideUp();
    });

    $( "#stop" ).click(function() {
      var myDiv = $( "div" );
      myDiv.clearQueue();
      myDiv.stop();
    });
    </script>

    </body>
    </html>
```