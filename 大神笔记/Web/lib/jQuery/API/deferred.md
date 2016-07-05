http://api.jquery.com/category/deferred-object/

[toc]

## Deferred Object

Deferred对象是一个可以链式调用的工具对象，由`jQuery.Deferred()`方法产生。

### 事件处理器

#### deferred.always()

添加一个处理器，当Deferred对象被解析（resolved）或拒绝（reject）后调用。

	deferred.always( alwaysCallbacks [, alwaysCallbacks ] )

返回值：Deferred

- `alwaysCallbacks`：类型：函数。一个函数，或一个函数数组，当Deferred对象被解析（resolved）或拒绝（reject）后调用。
- `alwaysCallbacks`：类型：函数。可选的附加函数，或函数的数组，当Deferred对象被解析（resolved）或拒绝（reject）后调用。

参数可以是单个函数，或一个函数的数组。当Deferred被解决或拒绝后，`alwaysCallbacks`被调用。因为`deferred.always()`返回Deferred对象，Deferred对象的方法可以被链式调用，包括再调用`.always()`方法{{此时方法的目标仍是同一个Deferred对象}}。Deferred解决或被拒后，回调按其添加的顺序调用，传给`resolve`、`reject`、`resolveWith`或`rejectWith`方法的参数会被传给这些处理器函数。

例子：Since the `jQuery.get()` method returns a `jqXHR` object, which is derived from a `Deferred` object, we can attach a callback for both success and error using the `deferred.always()` method.

```js
$.get( "test.php" ).always(function() {
  alert( "$.get completed with success or error callback arguments" );
});
```

#### deferred.done()

添加一个处理器，当Deferred对象被解析（resolved）后调用。

	deferred.done( doneCallbacks [, doneCallbacks ] )

返回值：Deferred

- `doneCallbacks`：类型：函数。一个函数，或函数数组，当Deferred对象被解析（resolved）后调用。
- `doneCallbacks`：类型：函数。可选的函数，或函数的数组，当Deferred对象被解析（resolved）后调用。

When the Deferred is resolved, the doneCallbacks are called. 回调按其添加的顺序调用。Since `deferred.done()` returns the deferred object, other methods of the deferred object can be chained to this one, including additional .done() methods. When the Deferred is resolved, doneCallbacks are executed using the arguments provided to the resolve or resolveWith method call in the order they were added.

例子：Since the `jQuery.get` method returns a jqXHR object, which is derived from a Deferred object, we can attach a success callback using the .done() method.

```js
$.get( "test.php" ).done(function() {
  alert( "$.get succeeded" );
});
```

Example: Resolve a Deferred object when the user clicks a button, triggering a number of callback functions:

```js
// 3 functions to call when the Deferred object is resolved
function fn1() {
  $( "p" ).append( " 1 " );
}
function fn2() {
  $( "p" ).append( " 2 " );
}
function fn3( n ) {
  $( "p" ).append( n + " 3 " + n );
}

// Create a deferred object
var dfd = $.Deferred();
// Add handlers to be called when dfd is resolved
dfd
// .done() can take any number of functions or arrays of functions
  .done( [ fn1, fn2 ], fn3, [ fn2, fn1 ] )
// We can chain done methods, too
  .done(function( n ) {
    $( "p" ).append( n + " we're done." );
  });
// Resolve the Deferred object when the button is clicked
$( "button" ).on( "click", function() {
  dfd.resolve( "and" );
});
```

#### deferred.fail()

deferred.fail( failCallbacks [, failCallbacks ] )
Returns: Deferred
Description: Add handlers to be called when the Deferred object is rejected.

- `failCallbacks`：类型：函数。A function, or array of functions, that are called when the Deferred is rejected.
- `failCallbacks`：类型：函数。Optional additional functions, or arrays of functions, that are called when the Deferred is rejected.

The deferred.fail() method accepts one or more arguments, all of which can be either a single function or an array of functions. When the Deferred is rejected, the failCallbacks are called. Callbacks are executed in the order they were added. Since deferred.fail() returns the deferred object, other methods of the deferred object can be chained to this one, including additional deferred.fail() methods. The failCallbacks are executed using the arguments provided to the deferred.reject() or deferred.rejectWith() method call in the order they were added.

Example: Since the jQuery.get method returns a jqXHR object, which is derived from a Deferred, you can attach a success and failure callback using the deferred.done() and deferred.fail() methods.

```js
$.get( "test.php" )
  .done(function() {
    alert( "$.get succeeded" );
  })
  .fail(function() {
    alert( "$.get failed!" );
  });
```

#### deferred.progress()

deferred.progress( progressCallbacks [, progressCallbacks ] )
Returns: Deferred
Description: Add handlers to be called when the Deferred object generates progress notifications.

- `progressCallbacks` Type: Function() or Array
A function, or array of functions, to be called when the Deferred generates progress notifications.
- `progressCallbacks` Type: Function() or Array
Optional additional function, or array of functions, to be called when the Deferred generates progress notifications.

The deferred.progress() method accepts one or more arguments, all of which can be either a single function or an array of functions. When the Deferred generates progress notifications by calling `notify` or `notifyWith`, the progressCallbacks are called. Since deferred.progress() returns the Deferred object, other methods of the Deferred object can be chained to this one. When the Deferred is resolved or rejected, progress callbacks will no longer be called, with the exception that any progressCallbacks added after the Deferred enters the resolved or rejected state are executed immediately when they are added, using the arguments that were passed to the .notify() or notifyWith() call.


### 产生事件

#### deferred.resolve()

deferred.resolve( [args ] )
Returns: Deferred
Description: Resolve a Deferred object and call any doneCallbacks with the given args.

- args
Type: Anything
Optional arguments that are passed to the doneCallbacks.

Normally, only the creator of a Deferred should call this method; you can prevent other code from changing the Deferred's state by returning a restricted Promise object through `deferred.promise()`.

When the Deferred is resolved, any doneCallbacks added by `deferred.then()` or `deferred.done()` are called. Callbacks are executed in the order they were added. Each callback is passed the args from the deferred.resolve(). Any doneCallbacks added after the Deferred enters the resolved state are executed immediately when they are added, using the arguments that were passed to the deferred.resolve() call.

#### deferred.resolveWith()

deferred.resolveWith( context [, args ] )
Returns: Deferred
Description: Resolve a Deferred object and call any doneCallbacks with the given context and args.

- `context`：Type: Object。Context passed to the doneCallbacks as the this object.
- `args`：Type: Array。An optional array of arguments that are passed to the doneCallbacks.

Normally, only the creator of a Deferred should call this method; you can prevent other code from changing the Deferred's state by returning a restricted Promise object through `deferred.promise()`.

When the Deferred is resolved, any doneCallbacks added by `deferred.then` or `deferred.done` are called. Callbacks are executed in the order they were added. Each callback is passed the args from the .resolve(). Any doneCallbacks added after the Deferred enters the resolved state are executed immediately when they are added, using the arguments that were passed to the .resolve() call.

#### deferred.reject()

deferred.reject( [args ] )
Returns: Deferred
Description: Reject a Deferred object and call any failCallbacks with the given `args`.

- `args`：类型：任何。Optional arguments that are passed to the failCallbacks.

Normally, only the creator of a Deferred should call this method; you can prevent other code from changing the Deferred's state by returning a restricted Promise object through `deferred.promise()`.

When the Deferred is rejected, any failCallbacks added by `deferred.then()` or `deferred.fail()` are called. Callbacks are executed in the order they were added. Each callback is passed the args from the deferred.reject() call. Any failCallbacks added after the Deferred enters the rejected state are executed immediately when they are added, using the arguments that were passed to the deferred.reject() call.

#### deferred.rejectWith()

deferred.rejectWith( context [, args ] )
Returns: Deferred
Description: Reject a Deferred object and call any failCallbacks with the given context and args.

- `context`：类型：对象。Context passed to the failCallbacks as the this object.
- `args`：类型：数组。An optional array of arguments that are passed to the failCallbacks.

Normally, only the creator of a Deferred should call this method; you can prevent other code from changing the Deferred's state by returning a restricted Promise object through `deferred.promise()`.

When the Deferred is rejected, any failCallbacks added by `deferred.then` or `deferred.fail` are called. Callbacks are executed in the order they were added. Each callback is passed the args from the deferred.reject() call. Any failCallbacks added after the Deferred enters the rejected state are executed immediately when they are added, using the arguments that were passed to the .reject() call.

#### deferred.notify()

deferred.notify( args )
Returns: Deferred
Description: Call the progressCallbacks on a Deferred object with the given `args`.

- args：类型：对象。Optional arguments that are passed to the progressCallbacks.

Normally, only the creator of a Deferred should call this method; you can prevent other code from changing the Deferred's state or reporting status by returning a restricted Promise object through `deferred.promise()`.

When `deferred.notify` is called, any progressCallbacks added by `deferred.then` or `deferred.progress` are called. Callbacks are executed in the order they were added. Each callback is passed the args from the .notify(). Any calls to .notify() after a Deferred is resolved or rejected (or any progressCallbacks added after that) are ignored.

#### deferred.notifyWith()

deferred.notifyWith( context [, args ] )
Returns: Deferred
Description: Call the progressCallbacks on a Deferred object with the given context and args.

- context：类型：对象。Context passed to the progressCallbacks as the this object.
- args：类型：对象。An optional array of arguments that are passed to the progressCallbacks.

Normally, only the creator of a Deferred should call this method; you can prevent other code from changing the Deferred's state or reporting status by returning a restricted Promise object through `deferred.promise()`.

When `deferred.notifyWith` is called, any progressCallbacks added by deferred.then or deferred.progress are called. Callbacks are executed in the order they were added. Each callback is passed the args from the .notifyWith(). Any calls to .notifyWith() after a Deferred is resolved or rejected (or any progressCallbacks added after that) are ignored. For more information, see the documentation for Deferred object.

#### deferred.state()

deferred.state()
Returns: String
Description: Determine the current state of a Deferred object.

This method does not accept any arguments.

The deferred.state() method returns a string representing the current state of the Deferred object. The Deferred object can be in one of three states:

- "pending": The Deferred object is not yet in a completed state (neither "rejected" nor "resolved").
- "resolved": The Deferred object is in the resolved state, meaning that either deferred.resolve() or deferred.resolveWith() has been called for the object and the doneCallbacks have been called (or are in the process of being called).
- "rejected": The Deferred object is in the rejected state, meaning that either deferred.reject() or deferred.rejectWith() has been called for the object and the failCallbacks have been called (or are in the process of being called).

该方法主要用于调试，例如，whether a Deferred has already been resolved even though you are inside code that intended to reject it.

#### deferred.then()

Returns: Promise
Description: Add handlers to be called when the Deferred object is resolved, rejected, or still in progress.

deferred.then( doneFilter [, failFilter ] [, progressFilter ] )

- `doneFilter`：一个函数。当Deferred解决时调用。
- `failFilter`：一个函数。可选。当Deferred被拒时调用。
- `progressFilter`：一个函数。可选。当Deferred收到进度更新。

已被移除的两个方法：

- `deferred.then( doneCallbacks, failCallbacks )`
- `deferred.then( doneCallbacks, failCallbacks [, progressCallbacks ] )`

jQuery 1.8之前，参数可以是函数的数组。

For all signatures, the arguments can be `null` if no callback of that type is desired. Alternatively, use .done(), .fail() or .progress() to set only one type of callback without filtering status or values.

As of jQuery 1.8, the `deferred.then()` method returns a new promise that can filter the status and values of a deferred through a function, replacing the now-deprecated `deferred.pipe()` method. The `doneFilter` and `failFilter` functions filter the original deferred's resolved / rejected status and values. The `progressFilter` function filters any calls to the original deferred's `notify` or `notifyWith` methods. These filter functions can return a new value to be passed along to the promise's `.done()` or `.fail()` callbacks, or they can return another **observable** object (**Deferred**, **Promise**, etc) which will pass its resolved / rejected status and values to the promise's callbacks. If the filter function used is null, or not specified, the promise will be resolved or rejected with the same values as the original.

Callbacks are executed in the order they were added. Since `deferred.then` returns a Promise, other methods of the Promise object can be chained to this one, including additional `.then()` methods.

例一：

```js
$.get( "test.php" ).then(
  function() {
    alert( "$.get succeeded" );
  }, function() {
    alert( "$.get failed!" );
  }
);
```

Example: Filter the resolve value:

```js
var filterResolve = function() {
  var defer = $.Deferred(),
    filtered = defer.then(function( value ) {
      return value * 2;
    });

  defer.resolve( 5 );
  filtered.done(function( value ) {
    $( "p" ).html( "Value is ( 2*5 = ) 10: " + value );
  });
};

$( "button" ).on( "click", filterResolve );
```


Example: Filter reject value:

```js
var defer = $.Deferred(),
  filtered = defer.then( null, function( value ) {
    return value * 3;
  });

defer.reject( 6 );
filtered.fail(function( value ) {
  alert( "Value is ( 3*6 = ) 18: " + value );
});
```

Example: Chain tasks:

```js
var request = $.ajax( url, { dataType: "json" } ),
  chained = request.then(function( data ) {
    return $.ajax( url2, { data: { user: data.userId } } );
  });

chained.done(function( data ) {
  // data retrieved from url2 as provided by the first request
});
```

### deferred.promise()

deferred.promise( [target ] )
Returns: Promise
Description: Return a Deferred's Promise object.

- target：类型：对象。Object onto which the promise methods have to be attached

The `deferred.promise()` method allows an asynchronous function to prevent other code from interfering with the progress or status of its internal request. The Promise exposes only the Deferred methods needed to attach additional handlers or determine the state (`then`, `done`, `fail`, `always`, `pipe`, `progress`, and `state`), but not ones that change the state (`resolve`, `reject`, `notify`, `resolveWith`, `rejectWith`, and `notifyWith`).

If target is provided, deferred.promise() will attach the methods onto it and then return this object rather than create a new one. This can be useful to attach the Promise behavior to an object that already exists.

If you are creating a Deferred, keep a reference to the Deferred so that it can be resolved or rejected at some point. Return only the Promise object via `deferred.promise()` so other code can register callbacks or inspect the current state.

例1：

```js
function asyncEvent() {
  var dfd = jQuery.Deferred();

  // Resolve after a random interval
  setTimeout(function() {
    dfd.resolve( "hurray" );
  }, Math.floor( 400 + Math.random() * 2000 ) );

  // Reject after a random interval
  setTimeout(function() {
    dfd.reject( "sorry" );
  }, Math.floor( 400 + Math.random() * 2000 ) );

  // Show a "working..." message every half-second
  setTimeout(function working() {
    if ( dfd.state() === "pending" ) {
      dfd.notify( "working... " );
      setTimeout( working, 500 );
    }
  }, 1 );

  // Return the Promise so caller can't change the Deferred
  return dfd.promise();
}

// Attach a done, fail, and progress handler for the asyncEvent
$.when( asyncEvent() ).then(
  function( status ) {
    alert( status + ", things are going well" );
  },
  function( status ) {
    alert( status + ", you fail this time" );
  },
  function( status ) {
    $( "body" ).append( status );
  }
);
```

Example: Use the target argument to promote an existing object to a Promise:

```
// Existing object
var obj = {
    hello: function( name ) {
      alert( "Hello " + name );
    }
  },
  // Create a Deferred
  defer = $.Deferred();

// Set object as a promise
defer.promise( obj );

// Resolve the deferred
defer.resolve( "John" );

// Use the object as a Promise
obj.done(function( name ) {
  obj.hello( name ); // Will alert "Hello John"
}).hello( "Karl" ); // Will alert "Hello Karl"
```


### 构建Deferred/Promise对象

#### jQuery.Deferred()

jQuery.Deferred( [beforeStart ] )
Returns: Deferred

一个工厂函数，返回一个可以链式调用的工具对象，含有注册回调、触发回调等方法。

- `beforeStart`：Function( Deferred deferred )。构造器返回前调用的方法。构造器创建的deferred对象，会传给`beforeStart`方法，并且作为`beforeStart`方法的`this`对象。

Deferred对象开始处于pending状态。回调可以通过`deferred.then()`、 `deferred.always()`、 `deferred.done()`或`deferred.fail()`排队。调用`deferred.resolve()` 或 `deferred.resolveWith()`可以使Deferred进入已解析状态。调用`deferred.reject()`或`deferred.rejectWith()`使Deferred进入拒绝状态。对象进入已解析或拒绝状态后，会停在那个状态。后续调用改变状态的方法，如`deferred.resolve()`，会被忽略。此时仍可以向Deferred绑定回调，它们会立即执行。

jQuery Deferred is based on the CommonJS Promises/A design.

In most cases where a jQuery API call returns a Deferred or Promise-compatible object, such as jQuery.ajax() or jQuery.when(), you will only want to use the `deferred.then()`, `deferred.done()`, and `deferred.fail()` methods to add callbacks to the Deferred's queues. The internals of the API call or code that created the Deferred will invoke deferred.resolve() or deferred.reject() on the deferred at some point, causing the appropriate callbacks to run.

#### .promise()

	.promise( [type ] [, target ] )

Returns: Promise
Description: Return a Promise object to observe when all actions of a certain type bound to the collection, queued or not, have finished.

- `type` (default: fx) Type: String
The type of queue that needs to be observed.
- `target` Type: PlainObject
Object onto which the promise methods have to be attached

The .promise() method returns a dynamically generated Promise that is resolved once all actions of a certain type bound to the collection, queued or not, have ended.

By default, type is "fx", which means the returned Promise is resolved when all animations of the selected elements have completed.

Resolve context and sole argument is the collection onto which .promise() has been called.

If target is provided, .promise() will attach the methods onto it and then return this object rather than create a new one. This can be useful to attach the Promise behavior to an object that already exists.

Note: The returned Promise is linked to a Deferred object stored on the .data() for an element. Since the.remove() method removes the element's data as well as the element itself, it will prevent any of the element's unresolved Promises from resolving. If it is necessary to remove an element from the DOM before its Promise is resolved, use .detach() instead and follow with .removeData() after resolution.

Examples:

Example: Using .promise() on a collection with no active animation returns a resolved Promise:

```js
div.promise().done(function( arg1 ) {
  // Will fire right away and alert "true"
  alert( this === div && arg1 === div );
});
```

Example: Resolve the returned Promise when all animations have ended (including those initiated in the animation callback or added later on):

```js
$( "button" ).on( "click", function() {
  $( "p" ).append( "Started..." );

  $( "div" ).each(function( i ) {
    $( this ).fadeIn().fadeOut( 1000 * ( i + 1 ) );
  });

  $( "div" ).promise().done(function() {
    $( "p" ).append( " Finished! " );
  });
});
```

#### jQuery.when()

jQuery.when( deferreds )
Returns: Promise
Description: Provides a way to execute callback functions based on one or more objects, usually Deferred objects that represent asynchronous events.

- `deferreds`：Type: Deferred
One or more Deferred objects, or plain JavaScript objects.

If a single Deferred is passed to jQuery.when(), its Promise object (a subset of the Deferred methods) is returned by the method. Additional methods of the Promise object can be called to attach callbacks, such as `deferred.then`. When the Deferred is resolved or rejected, usually by the code that created the Deferred originally, the appropriate callbacks will be called. For example, the jqXHR object returned by jQuery.ajax() is a Promise-compatible object and can be used this way:

```js
$.when( $.ajax( "test.aspx" ) ).then(function( data, textStatus, jqXHR ) {
  alert( jqXHR.status ); // Alerts 200
});
```

If a single argument is passed to jQuery.when() and it is not a Deferred or a Promise, it will be treated as a resolved Deferred and any doneCallbacks attached will be executed immediately. The doneCallbacks are passed the original argument. In this case any failCallbacks you might set are never called since the Deferred is never rejected. For example:

```js
$.when( { testing: 123 } ).done(function( x ) {
  alert( x.testing ); // Alerts "123"
});
```

If you don't pass it any arguments at all, jQuery.when() will return a resolved promise.

```js
$.when().then(function( x ) {
  alert( "I fired immediately" );
});
```

In the case where multiple Deferred objects are passed to jQuery.when(), the method returns the Promise from a new "master" Deferred object that tracks the aggregate state of all the Deferreds it has been passed. The method will resolve its master Deferred as soon as all the Deferreds resolve, or reject the master Deferred as soon as one of the Deferreds is rejected. If the master Deferred is resolved, the `doneCallbacks` for the master Deferred are executed. The arguments passed to the `doneCallbacks` provide the resolved values for each of the Deferreds, and matches the order the Deferreds were passed to jQuery.when(). For example:

```js
var d1 = $.Deferred();
var d2 = $.Deferred();
$.when( d1, d2 ).done(function ( v1, v2 ) {
    console.log( v1 ); // "Fish"
    console.log( v2 ); // "Pizza"
});
d1.resolve( "Fish" );
d2.resolve( "Pizza" );
```

In the event a Deferred was resolved with no value, the corresponding doneCallback argument will be `undefined`. If a Deferred resolved to a single value, the corresponding argument will hold that value. In the case where a Deferred resolved to multiple values, the corresponding argument will be an array of those values. For example:

```js
var d1 = $.Deferred();
var d2 = $.Deferred();
var d3 = $.Deferred();

$.when( d1, d2, d3 ).done(function ( v1, v2, v3 ) {
    console.log( v1 ); // v1 is undefined
    console.log( v2 ); // v2 is "abc"
    console.log( v3 ); // v3 is an array [ 1, 2, 3, 4, 5 ]
});

d1.resolve();
d2.resolve( "abc" );
d3.resolve( 1, 2, 3, 4, 5 );
```

In the multiple-Deferreds case where one of the Deferreds is rejected, jQuery.when() immediately fires the failCallbacks for its master Deferred. Note that some of the Deferreds may still be unresolved at that point. The arguments passed to the failCallbacks match the signature of the failCallback for the Deferred that was rejected. If you need to perform additional processing for this case, such as canceling any unfinished ajax requests, you can keep references to the underlying jqXHR objects in a closure and inspect/cancel them in the failCallback.

Examples:
Example: Execute a function after two ajax requests are successful. (See the jQuery.ajax() documentation for a complete description of success and error cases for an ajax request).

```js
$.when( $.ajax( "/page1.php" ), $.ajax( "/page2.php" ) ).done(function( a1, a2 ) {
  // a1 and a2 are arguments resolved for the page1 and page2 ajax requests, respectively.
  // Each argument is an array with the following structure: [ data, statusText, jqXHR ]
  var data = a1[ 0 ] + a2[ 0 ]; // a1[ 0 ] = "Whip", a2[ 0 ] = " It"
  if ( /Whip It/.test( data ) ) {
    alert( "We got what we came for!" );
  }
});
```

### 已移除：deferred.isRejected()

### 已移除：deferred.isResolved()

### 已作废（1.8）：deferred.pipe()

