https://github.com/cujojs/when/blob/master/docs/api.md

[toc]

## 翻译术语

- Promise：诺言
- fulfill：兑现（诺言），应该等价于resolve
- settle：结果确定，即或者fulfill/resove，或者reject。

## Core

### when()

	var promise = when(x);

Get a trusted promise for `x`. If `x` is:

* 一个值，返回一个诺言，一`x`兑现
* 一个诺言，直接返回`x`.
* a foreign thenable, returns a promise that follows `x`

版本2：

	var transformedPromise = when(x, f);

Get a trusted promise by transforming `x` with `f`.  If `x` is

* a value, returns a promise fulfilled with `f(x)`
* a promise or thenable, returns a promise that
	* if `x` fulfills, will fulfill with the result of calling `f` with `x`'s fulfillment value.
	* if `x` rejects, will reject with the same reason as `x`

`when()` accepts any promise that provides a *thenable* promise--any object that provides a `.then()` method, even promises that aren't fully Promises/A+ compliant, such as jQuery's Deferred. It will assimilate such promises and make them behave like Promises/A+.

### when.try

**ALIAS:** `when.attempt()` for non-ES5 environments

	var promise = when.try(f /*, arg1, arg2, ...*/);

调用`f`，传入提供的实参。返回一个诺言作为结果。实参可以也是诺言，此时会先等它们兑现诺言后再调用`f`。返回的诺言`promise`兑现的值是`f`返回的结果。如果某个实参是一个被拒的诺言，或者`f`抛出异常，或`f`返回一个被拒的诺言，都将导致返回的诺言被拒。

当你想开始使用诺言、需要一个诺言对象时，可以使用该方法，这样就不必手工创建诺言了。例如：

	// 解析JSON，抛出异常返回一个拒绝的Promise
    return when.try(JSON.parse, jsonString);

See also:

* [when.lift](#whenlift)
* [when/node call](#nodecall)
* [when/node apply](#nodeapply)

### when.lift

	var g = when.lift(f);

把函数`f`包裹成一个诺言化的函数。`g`将返回诺言，不再会抛出任何异常；`g`将接受诺言类型的实参或普通实参。即调用`g(x, y z)`等价于调用`when.try(f, x, y, z)`，但好处是一旦有了`g`，可以反复调用，或将它传给其他函数。In addition, `g`'s thisArg will behave in a predictable way, like any other function (you can `.bind()` it, or use `.call()` or `.apply()`, etc.).

与`when.try`的作用类似：当你想开始使用Promise，需要一个Promise对象时，可以使用该方法，这样就不必手工创建Proimse了，如`new Promise`。

    // jsonParse返回一个Promise。它不会再抛出异常。错误通过返回的Promise捕获。
    var jsonParse = when.lift(JSON.parse);
    // 使用
    return jsonParse(jsonString);

`when.lift`能够正确的处理`this`，于是对象方法可以被提升（lifted）：

    var parser = {
        reviver: ...
        parse: when.lift(function(str) {
            return JSON.parse(str, this.reviver);
        });
    };

    // Now use it wherever you need
    return parser.parse(jsonString);

See also:

* [when.try](#whentry)
* [when/node lift](#nodelift)
* [when/node liftAll](#nodeliftall)

### when.join

	var joinedPromise = when.join(promiseOrValue1, promiseOrValue2, ...);

返回一个诺言，在所有输入兑现都兑现。返回的诺言的值是一个数组，包含每个数组的值。

如果任一个输入的诺言被拒，则返回的诺言将被拒，拒绝原因取第一个被拒的输入的拒绝原因。

    // largerPromise will resolve to the greater of two eventual values
    var largerPromise = when.join(promise1, promise2).then(function (values) {
        return values[0] > values[1] ? values[0] : values[1];
    });

See also:

* [when.all](#whenall) - 传入一个诺言构成的数组

### when.promise

	var promise = when.promise(resolver);

创建一个诺言，其命运由`resolver`函数决定。`resolver`函数会被同步调用，传入三个参数：

    var promise = when.promise(function(resolve, reject, notify) {
        // 一些工作，可能是异步的，然后调用resolve或reject。
        // 不赞成使用：如果需要可以通知进度

        resolve(awesomeResult);
        // or resolve(anotherPromise);
        // or reject(nastyError);
    });


* `resolve(promiseOrValue)`：决定返回的`promise`的命令的主要函数。接受一个普通的值，或另一个诺言。
	* 若传入普通值，使用该值兑现诺言。
	* 若传入另一个诺言，如`resolve(otherPromise)`，`promise`的命运取决于`otherPromise`的命运
* `reject(reason)`：用于拒绝`promise`的函数
* `notify(update)`：（已不推荐使用）。参见：[Refactoring progress](##refactoring-progress)。

### when.resolve

	var resolved = when.resolve(x);

Get a promise for the supplied `x`. If `x` is already a trusted promise, it is returned. 如果`x`是一个值，返回的诺言将兑现`x`。If `x` is a thenable, the returned promise will follow `x`, adopting its eventual state (fulfilled or rejected).

### when.reject

	var rejected = when.reject(error);

拒绝一个诺言，提供的`error`将作为拒绝原因。

**DEPRECATION WARNING:** In when.js 2.x, error is allowed to be a promise for an error.  In when.js 3.0, error will always be used verbatim as the rejection reason, even if it is a promise.

### when.defer

	var deferred = when.defer();

注意：不推荐使用`when.defer`。多数情况下，使用`when.promise`、`when.try`或`when.lift`，provides better separation of concerns.

Create a `{promise, resolve, reject, notify}` tuple. In certain (rare) scenarios it can be convenient to have access to both the `promise` and it's associated resolving functions.

The deferred API:

    var promise = deferred.promise;
    // Resolve the promise, x may be a promise or non-promise
    deferred.resolve(x)
    // Reject the promise with error as the reason
    deferred.reject(error)
    // DEPRECATED Notify promise consumers of a progress update
    deferred.notify(x)

Note that `resolve`, `reject`, and `notify` all become no-ops after either `resolve` or `reject` has been called the first time.

创建deferred的常用目的是将基于回调的函数转换为Promise。In those cases, it's preferable to use the [when/callbacks](##asynchronous-functions) module to [call](##callbackscall) or [lift](##callbackslift) the callback-based functions instead. **要适配Node风格的回调函数**，使用 [when/node](##node-style-asynchronous-functions) 模块。

### when.isPromiseLike

	var is = when.isPromiseLike(x);

若`x`是一个带有`then`方法的对象或函数，则返回true。It does not distinguish trusted when.js promises from other "thenables" (e.g. from some other promise implementation).

不推荐使用`isPromiseLike`。如果你不确定`x`是否可以用作Promise，最好的办法是用`var trustedPromise = when(x);`做类型转换，然后使用`trustedPromise`。

## Promise

### promise.done

	promise.done(handleValue); // returns undefined

如果诺言兑现，`done`消耗诺言的终值。如果诺言被拒，causes a fatal error with a loud stack trace。

	promise.done(handleValue, handleError); // returns undefined

如果诺言兑现消费诺言的终值，或处理最终的错误。It will cause a fatal error if either `handleValue` or `handleError` throw or return a rejected promise.

`done`的目的是消耗（consumption）而不是传递（transformation），因此`done`总是返回`undefined`。

诺言错误处理的黄金法是：

Either `return` the promise, thereby *passing the error-handling buck* to the caller, or call `done` and *assuming responsibility for errors*.

See also

* [promise.then vs. promise.done](##promisethen-vs-promisedone)
* [promise.then](##promisethen)

### promise.then

	var transformedPromise = promise.then(onFulfilled);

[Promises/A+ `then`](http://promisesaplus.com). 传递诺言的值。返回一个新的诺言。

	var transformedPromise = promise.then(onFulfilled, onRejected);

`then`还可以用户恢复中间错误。但[`promise.catch`](#promisecatch)是更好的、可读性更好的选择。

注意，`onRejected`只会处理`promise`的错误，不会处理`onFulfilled`抛出的异常。比较：

    // 只使用then(): onRejected WILL NOT handle errors thrown by onFulfilled
    var transformedPromise = promise
        .then(onFulfilled, onRejected);

    // 使用catch()：onRejected会处理onFulfilled抛出的错误
    var transformedPromise = promise
        .then(onFulfilled)
        .catch(onRejected);

    // 使用catch()等价于：
    var transformedPromise = promise
        .then(onFulfilled)
        .then(void 0, onRejected);

**DEPRECATED**: Progress events are deprecated and will be removed in a future release. Until that release. See [Refactoring progress](#refactoring-progress).

    // 不推荐使用then()或promise.progress()监听进度事件
    var transformedPromise = promise.then(onFulfilled, onRejected, onProgress);
    // or
    var transformedPromise = promise
        .progress(onProgress)
        .then(onFulfilled)
        .catch(onRejected)

`then` arranges for:

- `onFulfilled` to be called with the value after `promise` is fulfilled, or
- `onRejected` to be called with the rejection reason after `promise` is rejected.
- **DEPRECATED**: `onProgress` to be called with any progress updates issued by `promise`. See [Refactoring progress](#refactoring-progress).

A promise makes the following guarantees about handlers registered in the same call to `.then()`:

1. `onFulfilled`和`onRejected`只会被调用一个
1. `onFulfilled`和`onRejected`不会被调用多次
1. `onProgress`可能被调用零次或多次

See also

* [Promises/A+](http://promisesaplus.com) for extensive information on the behavior of `then`.
* [promise.done](#promisedone)
* [promise.spread](#promisespread)
* [promise.progress](#promiseprogress)

### promise.spread

	var transformedPromise = promise.spread(onFulfilledArray);

与[`then`](#promisethen)类似，只是假定诺言的值是一个数组，传给`onFulfilledArray`。就相当于：

    // Wrapping onFulfilledArray
    promise.then(function(array) {
        return onFulfilledArray.apply(undefined, array);
    });

### promise.fold

	var resultPromise = promise2.fold(combine, promise1)

组合`promise1`和`promise2`产生一个`resultPromise`。`combine`在`promise1`和`promise2`都兑现后调用：`combine(promise1, promise2)`{{应该是`combine(promise1value, promise2value)`}}。与`then`类似，它可以返回诺言或值。

`promise.then`使得我们可以重用单参数函数，对诺言进行变换。`promise.fold`使得我们可以重用两参数函数。It can also be useful when you need to thread one extra piece of information into a promise chain, *without* having to capture it in a closure or use `promise.with`.

例如，假如`sum`方法已存在，可以计算两个诺言的值的和：

    function sum(x, y) {
        return x + y;
    }

    var promiseFor3 = when(3);
    var promiseFor5 = promiseFor3.fold(sum, promiseFor2);
    // Of course, it accepts values as well:
    var promiseFor5 = promiseFor3.fold(sum, 2);

例二，获取对象属性，或数组元素：

    function get(key, object) {
        return object[key];
    }

    when({ name: 'Bob' }).fold(get, 'name')
        .done(console.log); // logs 'Bob'

    when(['a', 'b', 'c']).fold(get, 1)
        .done(console.log); // logs 'b'

两个例子中，`sum`和`get`都是通用的、可重用的函数，不需要使用闭包。

### promise.catch

**ALIAS:** otherwise() for non-ES5 environments

	var recoveredPromise = promise.catch(onRejected);

若诺言被拒，调用`onRejected`，传入拒绝原因。

	var recoveredPromise = promise.catch(predicate, onRejected);

若提供一个`predicate`，则只捕获与断言匹配的错误。`predicate`可以是一个`Error`构造器，如`TypeError`、`ReferenceError`，自定义的错误（`prototype`必须是`instanceof Error`），或一个返回布尔值的函数：

    promise.then(function() {
        throw new CustomError('oops!');
    }).catch(CustomError, function(e) {
        // Only catch CustomError instances
        // all other types of errors will propagate automatically
    }).catch(function(e) {
        // Catch other errors
    })

Doing this in synchronous code is more clumsy, requiring `instanceof` checks inside a `catch` block:

    try {
        throw new CustomError('oops!');
    } catch(e) {
        if(e instanceof CustomError) {
            // Handler CustomError instances
        } else {
            // Handle other errors
        }
    }

### promise.finally

**ALIAS:** `ensure()` for non-ES5 environments

	var promise2 = promise1.finally(cleanup);

类似于同步版本的`finally`语句，不论诺言`promise1`兑现或被拒都将执行`cleanup`。

- 如果`promise1`兑现，`cleanup`返回成功，`promise2`将使用`promise1`相同的值兑现。
- 如果`promise1`被拒，`cleanup`返回成功，`promise2`将以`promise1`的原因被拒。
- 如果`promise1`被拒，`cleanup`抛出异常或返回一个拒绝的诺言，`promise2`将被拒，原因取抛出的异常或返回拒绝的诺言的原因。

组合`promise.catch`和`promise.finally`实现`catch`/`finally`语句的效果。例如：

    try {
      return doSomething(x);
    } catch(e) {
        return handleError(e);
    } finally {
        cleanup();
    }

    // ->
    return doSomething()
        .catch(handleError)
        .finally(cleanup);

### promise.yield

	originalPromise.yield(promiseOrValue);

返回一个新的诺言：

1. 如果`originalPromise`被拒，返回的诺言以相同原因被拒。
2. 如果`originalPromise`兑现，返回的诺言的命运取决于`promiseOrValue`：
    1. 如果`promiseOrValue`是一个值，返回的诺言将兑现，值为`promiseOrValue`
    2. 如果`promiseOrValue`是一个诺言，返回的诺言将：
	    - fulfilled with the fulfillment value of `promiseOrValue`, or
	    - rejected with the rejection reason of `promiseOrValue`

即，它类似于：

    originalPromise.then(function() {
        return promiseOrValue;
    });

See also:

* [promise.else](#promiseelse) - return a default value when promise rejects

### promise.else

**ALIAS:** `orElse()` for non-ES5 environments

    var p1 = doAyncOperationThatMightFail();
    return p1.else(defaultValue);

如果诺言被拒，`else`捕获拒绝，使用默认值兑现返回的诺言。This is a shortcut for manually `catch`ing a promise and returning a different value, as such:

    var p1 = doAyncOperationThatMightFail();
    return p1.catch(function() {
        return defaultValue;
    });

See also:

* [promise.catch](#promisecatch) - intercept a rejected promise
* [promise.tap](#promisetap) - execute a side effect in a promise chain
* [promise.yield](#promiseyield) - execute a side effect in a promise chain

### promise.tap

	var promise2 = promise.tap(onFulfilledSideEffect);

Executes a function as a side effect when `promise` fulfills. 返回一个新的诺言：

1. 若`promise`兑现，`onFulfilledSideEffect`执行：
  - 如果`onFulfilledSideEffect`返回成功，`promise2`以`promise`原来的值兑现。即`onfulfilledSideEffect`的结果被丢弃。
  - 如果`onFulfilledSideEffect`抛出异常或返回一个拒绝的诺言，`promise2`以相同原因拒绝。
2. 如果`promise`拒绝，`onFulfilledSideEffect`不会被执行，`promise2`以与`promise1`相同的原因拒绝。

下面两个写法是等价的：

    // Using only .then()
    promise.then(function(x) {
        doSideEffectsHere(x);
        return x;
    });

    // Using .tap()
    promise.tap(doSideEffectsHere);

### promise.delay

	var delayedPromise = promise.delay(milliseconds);

创建一个新的诺言，在指定毫秒后，以`promise`的值对象。如果`promise`拒绝，`delayedPromise`立即拒绝。

    var delayed;

    // delayed is a pending promise that will become fulfilled
    // in 1 second with the value "hello"
    delayed = when('hello').delay(1000);

    // delayed is a pending promise that will become fulfilled
    // 1 second after anotherPromise resolves, or will become rejected
    // *immediately* after anotherPromise rejects.
    delayed = promise.delay(1000);

    // Do something 1 second after promise resolves
    promise.delay(1000).then(doSomething).catch(handleRejection);

### promise.timeout

	var timedPromise = promise.timeout(milliseconds, reason);

创建一个新的诺言，若未在指定时间兑现或拒绝，以[`TimeoutError`](#timeouterror)为原因拒绝，或者以指定的`reason`拒绝。

    var node = require('when/node');

    // Lift fs.readFile so it returns promises
    var readFile = node.lift(fs.readFile);

    // Try to read the file, but timeout if it takes too long
    function readWithTimeout(path) {
        return readFile(path).timeout(500);
    }

可以通过`catch`捕获处理`TimeoutError`：

    var when = require('when');

    var p = readWithTimeout('/etc/passwd')
        .catch(when.TimeoutError, handleTimeout) // handle only TimeoutError
        .catch(handleFailure) // handle other errors

### promise.inspect

	var status = promise.inspect();

返回`promise`当前状态的快照。快照是一个对象：

* pending: `{ state: 'pending' }`
* fulfilled: `{ state: 'fulfilled', value: <promise's fulfillment value> }`
* rejected: `{ state: 'rejected', reason: <promise's rejection reason> }`

不推荐使用此方法。

See also:

* [when.settle](#whensettle) - settling an Array of promises

### promise.with

**ALIAS:** `withThis()` for non-ES5 environments

	var boundPromise = promise.with(object);

Creates a new promise that follows `promise`, but which will invoke its handlers with their `this` set to `object`.  Normally, promise handlers are invoked with no specific `thisArg`, so `with` can be very useful when bridging promises to object-oriented patterns and libraries.

**NOTE:** Promises returned from `with`/`withThis` are NOT Promises/A+ compliant, specifically violating 2.2.5 (http://promisesaplus.com/#point-41)

For example:

    function Thing(value, message) {
        this.value = value;
        this.message = message;
    }

    Thing.prototype.doSomething = function(x) {
        var promise = doAsyncStuff(x);
        return promise.with(this) // Set thisArg to this thing instance
            .then(this.addValue)  // Works since addValue will have correct thisArg
            .then(this.format);   // all subsequent promises retain thisArg
    };

    Thing.prototype.addValue = function(y) {
        return this.value + y;
    };

    Thing.prototype.format = function(result) {
        return this.message + result;
    };

    // Using it
    var thing = new Thing(41, 'The answer is ');

    thing.doSomething(1)
        .with(console) // Re-bind thisArg now to console
        .then(console.log); // Logs 'The answer is 42'


All promises created from `boundPromise` will also be bound to the same thisArg until `with` is used to re-bind or *unbind* it.  In the previous example, the promise returned from `thing.doSomething` still has its thisArg bound to `thing`.  That may not be what you want, so you can *unbind* it just before returning:

```js
Thing.prototype.doSomething = function(x) {
	var promise = doAsyncStuff(x);
	return promise.with(this)
		.then(this.addValue)
		.then(this.format)
		.with(); // Unbind thisArg
};
```

### promise.progress

**DEPRECATED** Progress events are deprecated. See [Refactoring progress](#refactoring-progress)

	promise.progress(onProgress);

Registers a handler for progress updates from `promise`.  It is a shortcut for:

promise.then(void 0, void 0, onProgress);

#### Notes on Progress events

Progress events are not specified in Promises/A+ and are optional in Promises/A.  They have proven to be useful in practice, but unfortunately, they are also underspecified, and there is no current *de facto* or agreed-upon behavior in the promise implementor community.

In when.js, progress events will be propagated through a promise chain:

1. In the same way as resolution and rejection handlers, your progress handler *MUST* return a progress event to be propagated to the next link in the chain.  If you return nothing, *undefined will be propagated*.
1. Also in the same way as resolutions and rejections, if you don't register a progress handler (e.g. `.then(handleResolve, handleReject /* no progress handler */)`), the update will be propagated through.
1. **This behavior will likely change in future releases:** If your progress handler throws an exception, the exception will be propagated to the next link in the chain. The best thing to do is to ensure your progress handlers do not throw exceptions.
	1. **Known Issue:** If you allow an exception to propagate and there are no more progress handlers in the chain, the exception will be silently ignored. We're working on a solution to this.

This gives you the opportunity to *transform* progress events at each step in the chain so that they are meaningful to the next step.  It also allows you to choose *not* to transform them, and simply let them propagate untransformed, by not registering a progress handler.

Here is an example:

    function myProgressHandler(update) {
        logProgress(update);
        // Return a transformed progress update that is
        // useful for progress handlers of the next promise!
        return update + 1;
    }

    function myOtherProgressHandler(update) {
        logProgress(update);
    }

    var d = when.defer();
    d.promise.then(undefined, undefined, myProgressHandler);

    var chainedPromise = d.promise.then(doStuff);
    chainedPromise.then(undefined, undefined, myOtherProgressHandler);

    var update = 1;
    d.notify(update);

    // Results in:
    // logProgress(1);
    // logProgress(2);

## 数组

### when.all

	var promise = when.all(array)

`array`是一个数组，或是一个返回数组的诺言。数组可以包含诺言或值。

返回一个诺言，仅当数组中所有项都兑现后才兑现。返回的诺言的解析值是一个数组，包含`array`中各个项的解析值。

任一诺言被拒，返回的诺言会被拒，原因取第一个被拒的原因。

See also:

* [when.join](#whenjoin)
* [when.settle](#whensettle)

### when.map

	var promise = when.map(array, mapper)

`array`是一个数组，或返回数组的诺言。数组可以包含诺言或值。

类似于`Array.prototype.map()`，但允许输入包含诺言，`mapper`可以返回一个值或诺言。

如果任一诺言被拒，返回的诺言会被拒，原因取第一个被拒的原因。

`mapper`函数的前面应是：`mapFunc(value:*, index:Number):*`。其中`value`是诺言的解析值，`index`是数组下标。

### when.filter

	var promise = when.filter(array, predicate);

`array`是一个数组，或返回数组的诺言。数组可以包含诺言或值。

Filters the input array, returning a promise for the filtered array. `predicate`可以返回布尔值，或是一个返回布尔值的断言。

如果任一诺言被拒，返回的诺言会被拒，原因取第一个被拒的原因。

`predicate`签名应是：`mapFunc(value:*, index:Number):*`。其中`value`是诺言的解析值，`index`是数组下标。

### when.reduce、when.reduceRight

    var promise = when.reduce(array, reducer [, initialValue])
    var promise = when.reduceRight(array, reducer [, initialValue])

`array`是一个数组，或返回数组的诺言。数组可以包含诺言或其他值。

类似于`Array.prototype.reduce()`或`Array.prototype.reduceRight()`，但输入可以包含诺言或值，reducer可以返回一个值或诺言，`initialValue`可以是一个诺言。

reduce函数的签名应是：`reducer(currentResult, value, index)`。

* `currentResult` is the current accumulated reduce value
* `value` is the fully resolved value at `index` in `array`
* `index` is the *basis* of `value` ... practically speaking, this is the array index of the array corresponding to `value`

    // sum the eventual values of several promises
    var sumPromise = when.reduce(inputPromisesOrValues, function (sum, value) {
        return sum += value;
    }, 0);

如果任一诺言被拒，返回的诺言会被拒，原因取第一个被拒的原因。

### when.settle

	var promise = when.settle(array);

返回一个诺言，解析值是一个数组，与`array`含有相同数量的元素。每个元素是输入数组相应的描述对象。仅当`array`自己是一个被拒的诺言时，返回的诺言才会被拒。否则，返回的诺言兑现，提供一个描述符的数组。This is in contrast to `when.all`, which will reject if any element of `array` rejects.

If the corresponding input promise is:

- 兑现，描述符将是：`{ state: 'fulfilled', value: <fulfillmentValue> }`
- 被拒绝，描述符将是：`{ state: 'rejected', reason: <rejectionReason> }`

    // Process all successful results, and also log all errors
    // Input array
    var array = [when.reject(1), 2, when(3), when.reject(4)];
    // Settle all inputs
    var settled = when.settle(array);
    // Logs 1 & 4 and processes 2 & 3
    settled.then(function(descriptors) {
        descriptors.forEach(function(d) {
            if(d.state === 'rejected') {
                logError(d.reason);
            } else {
                processSuccessfulResult(d.value);
            }
        });
    });

## 对象

`when/keys`模块提供`all()`和`map()`，处理对象的键，因为有时把多个诺言放入Hash比放入数组更方便处理。

### when/keys all

	var promise = keys.all(object)

`object`是一个对象，或一个产生对象的诺言，whose keys represent promises and/or values.

返回一个诺言，仅当`object`中所有项都兑现才兑现。返回的诺言对象的值是一个对象，包含`object`中每一项解析后的键值对。

任一诺言被拒，返回的诺言将被拒，原因取第一个被拒的原因。

### when/keys map

	var promise = keys.map(object, mapper)

`object`是一个对象，或一个产生对象的诺言，whose keys represent promises and/or values.

与`when.map`类似。mapper负责将键的值转换为一个诺言。`mapper`可以返回诺言或值。

任一诺言被拒，返回的诺言将被拒，原因取第一个被拒的原因。

`mapper`函数的签名为：`mapFunc(value:*, key:String):*`。

## Array Races

The *competitive race* pattern may be used if one or more of the entire possible set of *eventual outcomes* are sufficient to resolve a promise.

### when.any

	var promise = when.any(array)

只需要一个胜出。一旦有一个输入的诺言兑现，返回的诺言立即对象，兑现的值取兑现的输入诺言的值。以下两种情况返回的诺言会被拒：

* 如果输入数组为空，以`RangeError`拒绝
* 数组中所有诺言都拒绝

### when.some

**已不推荐使用**

	var promise = when.some(array, n)

需要`n`个胜者。一旦有`n`个输入的诺言兑现，返回的诺言兑现，兑现的值是一个数组，对应输入诺言兑现的值。以下情况返回的诺言被拒：

- 如果数组元素小于`n`，以`RangeError`拒绝。
- 当输入的诺言拒绝数量超过`(array.length - n) + 1`，即已不可能有`n`的胜出者

    // ping all of the p2p servers and fail if at least two don't respond
    var remotes = [ping('p2p.cdn.com'), ping('p2p2.cdn.com'), ping('p2p3.cdn.com')];
    when.some(remotes, 2).done(itsAllOk, failGracefully);

### when.race

	var promise = when.race(array);

返回的诺言的命运由最早确定命运的诺言决定。如果最早完成的诺言兑现，则结果诺言以相同值兑现。如果最早完成的诺言拒绝，则结果诺言以相同原因拒绝。

**警告** 根据ES6规范，如果数组为空，返回的诺言将永远处于未决的状态。

## 无限的诺言序列

[when.reduce](#whenreduce), [when/sequence](#whensequence), and [when/pipeline](#whenpipeline) are great ways to process asynchronous arrays of promises and tasks. 但有时，无法提前确定数组，或不想、不必处理数组中所有元素。例如，下面一些情况下你无法知道数组边界：

1. 处理的队列，仍不断有项添加进去。
1. 需要反复执行一个人物知道特定的条件满足
1. 需要有选择的处理数组中的项，而不是处理全部

可以使用`when/iterate`或`when/unfold`循环并异步处理项目直到特定断言为真。甚至永远执行下去由不会阻塞其他代码。

### when.iterate

	var promise = when.iterate(f, predicate, handler, seed);

通过反复调用`f`产生几个近乎无限的诺言流，直到`predicate`为true。

* `f` - function that, given a seed, returns the next value or a promise for it.
* `predicate` - function that receives the current iteration value, and should return truthy when the unfold should stop
* `handler`：接收`f`产生的每个值。可以返回一个诺言以延迟下一次迭代。
* `seed`：提供给handler和第一次`f`调用的初始值，可以是一个诺言。

例子：Here is a trivial example of counting up to any arbitrary number using promises and delays. 注意这个循环是异步的，不会阻塞其他代码。It stores no intermediate arrays in memory, and will never blow the call stack.

    // Logs
    // 0
    // 1
    // 2
    // ...
    // 100000000000
    when.iterate(function(x) {
        return x+1;
    }, function(x) {
        // Stop when x >= 100000000000
        return x >= 100000000000;
    }, function(x) {
        console.log(x);
    }, 0).done();

Which becomes even nicer with [ES6 arrow functions](http://tc39wiki.calculist.org/es6/arrow-functions/):

	when.iterate(x => x+1, x => x >= 100000000000, x => console.log(x), 0).done();

### when.unfold

	var promise = when.unfold(unspool, predicate, handler, seed);

与`when/iterate`类似，`when.unfold` generates a potentially infinite stream of promises by repeatedly calling `unspool` until `predicate` becomes true. `when.unfold` allows you to thread additional state information through the iteration.

* `unspool` - function that, given a seed, returns a `[valueToSendToHandler, newSeed]` pair. May return an array, array of promises, promise for an array, or promise for an array of promises.
* `predicate` - function that receives the current seed, and should return truthy when the unfold should stop
* `handler` - function that receives the `valueToSendToHandler` of the current iteration. This function can process `valueToSendToHandler` in whatever way you need.  It may return a promise to delay the next iteration of the unfold.
* `seed` - initial value provided to the first `unspool` invocation. May be a promise.

例子。This example generates random numbers at random intervals for 10 seconds.

The `predicate` could easily be modified (to `return false;`) to generate random numbers *forever*.  This would not overflow the call stack, and would not starve application code since it is asynchronous.

    var end = Date.now() + 10000;
    var start = Date.now();

    // Generate random numbers at random intervals!
    // Note that we could generate these forever, and never
    // blow the call stack, nor would we starve the application
    function unspool(seed) {
        // Return a random number as the value, and the time it was generated
        // as the new seed
        var next = [Math.random(), Date.now()];

        // Introduce a delay, just for fun, to show that we can return a promise
        return when(next).delay(Math.random() * 1000);
    }

    // Stop after 10 seconds
    function predicate(time) {
        return time > end;
    }

    function log(value) {
        console.log(value);
    }

    when.unfold(unspool, predicate, log, start).then(function() {
        console.log('Ran for', Date.now() - start, 'ms');
    }).done();

Which again becomes quite compact with [ES6 arrow functions](http://tc39wiki.calculist.org/es6/arrow-functions/):

    when.unfold(unspool, time => time > end, x => console.log(x), start)
        .then(() => console.log('Ran for', Date.now() - start, 'ms'))
        .done();

This example iterates over files in a directory, mapping each file to the first line (or first 80 characters) of its content.  It uses a `predicate` to terminate early, which would not be possible with `when.map`.

Notice that, while the pair returned by `unspool` is an Array (not a promise), it does *contain* a promise as it's 0th element.  The promise will be resolved by the `unfold` machinery.

Notice also the use of `when/node`'s [`call()`](#node-style-asynchronous-functions) to call Node-style async functions (`fs.readdir` and `fs.readFile`), and return a promise instead of requiring a callback.  This allows node-style functions can be promisified and composed with other promise-aware functions.

    var when = require('when');
    var node = require('when/node');

    var fs = node.liftAll(require('fs'));

    // Lifted fs methods return promises
    var files = fs.readdir('.');

    function unspool(files) {
      // Return the pair [<*promise* for contents of first file>, <remaining files>]
        // the first file's contents will be handed to printFirstLine()
        // the remaining files will be handed to condition(), and then
        // to the next call to unspool.
        // So we are iteratively working our way through the files in
        // the dir, but allowing condition() to stop the iteration at
        // any point.
        var file, content;

        file = files[0];
        content = fs.readFile(file)
            .catch(function(e) {
                return '[Skipping dir ' + file + ']';
            });
        return [content, files.slice(1)];
    }

    function predicate(remaining) {
        // This could be any test we want.  For fun, stop when
        // the next file name starts with a 'p'.
        return remaining[0].charAt(0) === 'p';
    }

    function printFirstLine(content) {
        // Even though contents was a promise in unspool() above,
        // when/unfold ensures that it is fully resolved here, i.e. it is
        // not a promise any longer.
        // We can do any work, even asynchronous work, we need
        // here on the current file

        // Node fs returns buffers, convert to string
        content = String(content);

        // Print the first line, or only the first 80 chars if the fist line is longer
        console.log(content.slice(0, Math.min(80, content.indexOf('\n'))));
    }

    when.unfold(unspool, predicate, printFirstLine, files).done();

## 任务执行

此模块允许你并行/串行执行任务。每个模块接受一个数组输入（或一个产生数组的诺言），数组中是一个个任务函数。所有任务完成后返回的诺言兑现。

### when/sequence

```js
var sequence = require('when/sequence');
var resultsPromise = sequence(arrayOfTasks, arg1, arg2 /*, ... */);
```

串行执行。传给`when.sequence()`的参数会传给每个任务函数。每个任务可以返回一个诺言或值。

返回的诺言兑现一个数组，包含每个任务的返回值。过程中任一任务抛出异常或返回一个被拒的诺言会拒绝`resultsPromise`。

### when/pipeline

```js
var pipeline = require('when/pipeline');
var resultsPromise = pipeline(arrayOfTasks, arg1, arg2 /*, ... */);
```

也是串行执行。传给`when.pipeline()`的参数会传给第一个任务函数（`arrayOfTasks[0]`），其返回值传给下一个任务。每个任务可以返回诺言或值。如果返回诺言，诺言兑现的值会被传给下一个任务。

当所有任务完成后，最后一个任务的结果作为`resultsPromise`兑现的值。过程中任一任务抛出异常或返回一个被拒的诺言会拒绝`resultsPromise`。

### when/parallel

```js
var parallel = require('when/parallel');
var resultsPromise = parallel(arrayOfTasks, arg1, arg2 /*, ... */);
```

并行运行任务。传给`when.parallel()`的参数会传给每个任务函数。每个函数返回一个诺言或值。

返回的诺言兑现一个数组，包含每个任务的返回值。过程中任一任务抛出异常或返回一个被拒的诺言会拒绝`resultsPromise`。

### when/poll

```js
var poll = require('when/poll');
var resultPromise = poll(work, interval, condition /*, initialDelay */);
```

以时间间隔`interval`反复执行`work`直到`condition`返回true。The `resultPromise` will be resolved with the most recent value returned from `work`. If `work` fails (throws an exception or returns a rejected promise) before `condition` returns true, the `resultPromise` will be rejected.

其中：

* `work`：定时调用的函数
* `interval`：调用`work`的间隔。可以是数字，或返回诺言的一个函数。若是函数，下次polling循环会等待诺言兑现。
* `condition`：function that evaluates each result of `work`. Polling will continue until it returns a truthy value.
* `initialDelay` - if provided and truthy, the first execution of `work` will be delayed by `interval`.  If not provided, or falsey, the first execution of `work` will happen as soon as possible.

## 函数：与非诺言代码交互

分三种类型的函数：同步函数、异步函数、Node风格的函数。分别提供三个模块对这三类函数进行包裹、转换。
{{
异步函数模块（when/callbacks）只能处理有两个回调函数的函数：即函数最后必须至少有两个参数，倒数第二个是成功回调，第一个是错误回调。
Node函数模块（when/node）处理只有一个回调函数的函数。回调函数的第一个参数被当作err。即Node风格。若回调函数没有参数，如`net.listen`的回调，也可以用该模块处理。因为没有参数err相对于undefined。能正常执行。但万一函数有一个或多个参数，但第一个参数没有遵循是err的规则，则第一个实参不是假值，将导致模块把函数执行结果当作拒绝。
本节三个模块的函数貌似不能正确处理this值。例如

	nodeFn.call server.listen, 8080

listen的this的指向貌似是错的。改成显式绑定是可以的：

	nodeFn.call server.listen.bind(server), 8080
}}

### 同步函数

`when/function`用于调用和适配"normal"函数（调用-同步返回-抛异常）。使用`fn.call`或`fn.apply`调用这些函数，或使用`fn.lift`产生一个函数，返回值将是一个诺言，抛出的异常会被转换为拒绝。如果传入的参数是一个诺言，会先等此诺言对象再进行函数调用。

#### fn.lift

    var promiseFunction = fn.lift(normalFunction);
    // Deprecated: using lift to partially apply while lifting
    var promiseFunction = fn.lift(normalFunction, arg1, arg2/* ...more args */);

将函数`normalFunction`提升为一个诺言化的函数。

注意：如果不使用`partial application feature`，使用`when.lift`比`fn.lift`快一点。

    var when = require('when');
    var fn   = require('when/function');
    function setText(element, text) {
        element.text = text;
    }
    function getMessage() {
        // Async function that returns a promise
    }
    var element = {};
    // Resolving the promise ourselves
    getMessage().then(function(message) {
        setText(element, message);
    });
    // Using fn.call()
    fn.call(setText, element, getMessage());
    // Creating a lifted function using fn.lift()
    var promiseSetText = fn.lift(setText);
    promiseSetText(element, getMessage());
    // Partial application
    var setElementMessage = fn.lift(setText, element);
    setElementMessage(getMessage());

#### fn.liftAll

    var liftedApi = fn.liftAll(srcApi);
    var liftedApi = fn.liftAll(srcApi, transform);
    var destApi = fn.liftAll(srcApi, transform, destApi);

提供原对象的所有方法，返回一个新对象，包含全部提升后的方法。可选的`transform`函数用来重命名，or otherwise customize how the lifted functions are added to the returned object。If `destApi` is provided, lifted methods will be added to it, instead of to a new object, and `destApi` will be returned.

#### fn.call

	var promisedResult = fn.call(normalFunction, arg1, arg2/* ...more args */);

A parallel to the `Function.prototype.call` function, that gives promise-awareness to the function given as first argument.

```js
var when = require('when');
var fn   = require('when/function');

function divideNumbers(a, b) {
	if(b !== 0) {
		return a / b;
	} else {
		throw new Error("Can't divide by zero!");
	}
}

// Prints '2'
fn.call(divideNumbers, 10, 5).then(console.log);

// Prints '4'
var promiseForFive = when(5);
fn.call(divideNumbers, 20, promiseForFive).then(console.log);

// Prints "Can't divide by zero!"
fn.call(divideNumbers, 10, 0).then(console.log, console.error);
```

#### fn.apply

```js
var promisedResult = fn.apply(normalFunction, [arg1, arg2/* ...more args */]);
```

`fn.apply` is to [`fn.call`](#fncall) as `Function.prototype.apply` is to `Function.prototype.call`: what changes is the way the arguments are taken.  While `fn.call` takes the arguments separately, `fn.apply` takes them as an array.

```js
var when = require('when');
var fn   = require('when/function');

function sumMultipleNumbers() {
	return Array.prototype.reduce.call(arguments, function(prev, n) {
		return prev + n;
	}, 0);
}

// Prints '50'
fn.apply(sumMultipleNumbers, [10, 20, 20]).then(console.log, console.error);

// Prints 'something wrong happened', and the sum function never executes
var shortCircuit = when.reject("something wrong happened");
fn.apply(sumMultipleNumbers, [10, 20, shortCircuit]).then(console.log, console.error);
```

#### fn.compose

```js
var composedFunc = fn.compose(func1, func2 /* ...more functions */);
```

Composes multiple functions by piping their return values. It is transparent to whether the functions return 'regular' values or promises: the piped argument is always a resolved value. If one of the functions throws or returns a rejected promise, the promise returned by `composedFunc` will be rejected.

```js
// Reusing the same functions from the fn.lift() example

// Gets the message from the server every 1s, then sets it on the 'element'
var refreshMessage = fn.compose(getMessage, setElementMessage);
setInterval(refreshMessage, 1000);

// Which is equivalent to:
setInterval(function() {
	return fn.call(getMessage).then(setElementMessage);
}, 1000);
```

### Node风格的异步函数

Node.js APIs有一套自己的标准处理异步函数：错误作为回调函数的第一个参数传入。To use promises instead of callbacks with node-style asynchronous functions, you can use the `when/node` module, which is very similar to `when/callbacks`, but tuned to this convention.

注意：有些Node函数不遵循Node异步函数标准，如`http.get`。这些函数不能与`when/node`协作。

#### node.lift

    var promiseFunc = nodefn.lift(nodeStyleFunction);
    // Deprecated: using lift to partially apply while lifting
    var promiseFunc = nodefn.lift(nodeStyleFunction, arg1, arg2/*...more args*/);

Function based on the same principles from `fn.lift()` and `callbacks.lift()`, but tuned to handle nodejs-style async functions.

    var dns    = require('dns');
    var when   = require('when');
    var nodefn = require('when/node');

    var resolveAddress = nodefn.lift(dns.resolve);

    when.join(
        resolveAddress('twitter.com'),
        resolveAddress('facebook.com'),
        resolveAddress('google.com')
    ).then(function(addresses) {
        // All addresses resolved
    }).catch(function(reason) {
        // At least one of the lookups failed
    });

#### node.liftAll

    var liftedApi = fn.liftAll(srcApi);
    var liftedApi = fn.liftAll(srcApi, transform);
    var destApi = fn.liftAll(srcApi, transform, destApi);

Lifts all the methods of a source object, returning a new object with all the lifted methods. The optional `transform` function allows you to rename or otherwise customize how the lifted functions are added to the returned object. If `destApi` is provided, lifted methods will be added to it, instead of to a new object, and `destApi` will be returned.

    // Lift the entire dns API
    var dns = require('dns');
    var promisedDns = node.liftAll(dns);

    when.join(
        promisedDns.resolve("twitter.com"),
        promisedDns.resolveNs("facebook.com"),
        promisedDns.resolveMx("google.com")
    ).then(function(addresses) {
        // All addresses resolved
    }).catch(function(reason) {
        // At least one of the lookups failed
    });

For additional flexibility, you can use the optional `transform` function to do things like renaming:

    // Lift all of the fs methods, but name them with an 'Async' suffix
    var fs = require('fs');
    var promisedFs = node.liftAll(fs, function(promisedFs, liftedFunc, name) {
        promisedFs[name + 'Async'] = liftedFunc;
        return promisedFs;
    });

    promisedFs.readFileAsync('file.txt').done(console.log.bind(console));

You can also supply your own destination object onto which all of the lifted functions will be added:

    // Lift all of the fs methods, but name them with an 'Async' suffix
    // and add them back onto fs!
    var fs = require('fs');
    var promisedFs = node.liftAll(fs, function(promisedFs, liftedFunc, name) {
        promisedFs[name + 'Async'] = liftedFunc;
        return promisedFs;
    }, fs);

    fs.readFileAsync('file.txt').done(console.log.bind(console));

#### node.call

	var promisedResult = nodefn.call(nodeStyleFunction, arg1, arg2/*more args*/);

Analogous to `fn.call()` and `callbacks.call()`: Takes a function plus optional arguments to that function, and returns a promise for its final value. The promise will be resolved or rejected depending on whether the conventional error argument is passed or not.

    var fs     = require('fs');
    var nodefn = require('when/node');

    var loadPasswd = nodefn.call(fs.readFile, '/etc/passwd');

    loadPasswd.done(function(passwd) {
        console.log('Contents of /etc/passwd:\n' + passwd);
    }, function(error) {
        console.log('Something wrong happened: ' + error);
    });

#### node.apply

	var promisedResult = nodefn.apply(nodeStyleFunction, [arg1, arg2/*more args*/]);

Following the tradition from `when/function` and `when/callbacks`, `when/node` also provides a array-based alternative to `nodefn.call()`.

    var fs     = require('fs');
    var nodefn = require('when/node');

    var loadPasswd = nodefn.apply(fs.readFile, ['/etc/passwd']);

    loadPasswd.done(function(passwd) {
        console.log('Contents of /etc/passwd:\n' + passwd);
    }, function(error) {
        console.log('Something wrong happened: ' + error);
    });

#### node.liftCallback

	var promiseAcceptingFunction = nodefn.liftCallback(nodeback);

Transforms a node-style callback function into a function that accepts a promise.  This allows you to bridge promises and node-style in "the other direction". For example, if you have a node-style callback, and a function that returns promises, you can lift the former to allow the two functions to be composed.

The lifted function will always returns its input promise, and always executes `nodeback` in a future turn of the event loop. Thus, the outcome of `nodeback` has no bearing on the returned promise.

If `nodeback` throws an exception, it will propagate to the host environment, just as it would when using node-style callbacks with typical Node.js APIs.

    var nodefn = require('when/node');
    function fetchData(key) {
        // go get the data and,
        return promise;
    }
    function handleData(err, result) {
        if(err) {
            // handle the error
        } else {
            // Use the result
        }
    }
    // Lift handleData
    var handlePromisedData = nodefn.liftCallback(handleData);
    var dataPromise = fetchData(123);
    handlePromisedData(dataPromise);

#### node.bindCallback

	var resultPromise = nodefn.bindCallback(promise, nodeback);

Lifts and then calls the node-style callback on the provided promise. This is a one-shot version of `nodefn.liftCallback`, and the `resultPromise` will behave as described there.

    var nodefn = require('when/node');
    function fetchData(key) {
        // go get the data and,
        return promise;
    }
    function handleData(err, result) {
        if(err) {
            // handle the error
        } else {
            // Use the result
        }
    }
    // Lift handleData
    var dataPromise = fetchData(123);
    nodefn.bindCallback(dataPromise, handleData);

#### node.createCallback

	var nodeStyleCallback = nodefn.createCallback(resolver);

A helper function of the `when/node` implementation, which might be useful for cases that aren't covered by the higher level API. It takes an object that responds to the resolver interface (`{ resolve:function, reject:function }`) and returns a function that can be used with any node-style asynchronous function, and will call `resolve()` or `reject()` on the resolver depending on whether the conventional error argument is passed to it.

    var when   = require('when');
    var nodefn = require('when/node');

    function nodeStyleAsyncFunction(callback) {
        if(Math.random() * 2 > 1) {
          callback(new Error('Oh no!'));
        } else {
          callback(null, "Interesting value");
        }
    }

    var deferred = when.defer();
    nodeStyleAsyncFunction(nodefn.createCallback(deferred.resolver));

    deferred.promise.then(function(interestingValue) {
      console.log(interestingValue)
    },function(err) {
      console.error(err)
    });

### 异步函数

The `when/callbacks` module provides functions to interact with those APIs via promises in a transparent way, without having to write custom wrappers or change existing code. 此模块的所有函数（除了`callbacks.promisify()`）都假设回调和错误回调在标准位置上：倒数第二个和最后一个参数。

#### callbacks.lift

    var promiseFunc = callbacks.lift(callbackTakingFunc);
    // Deprecated: using lift to partially apply while lifting
    var promiseFunc = callbacks.lift(callbackTakingFunc, arg1, arg2/* ...more args */);

Much like `fn.lift()`, `callbacks.lift` creates a promise-friendly function, based on an existing function, but following the asynchronous resolution patters from `callbacks.call()` and `callbacks.apply()`. It can be useful when a particular function needs no be called on multiple places, or for creating an alternative API for a library.

Like `Function.prototype.bind`, additional arguments will be partially applied to the new function.

    // Fictional ajax library, because we don't have enough of those
    function traditionalAjax(method, url, callback, errback) {
        var xhr = new XMLHttpRequest();
        xhr.open(method, url);
        xhr.onload = callback;
        xhr.onerror = errback;
        xhr.send();
    }

    var myLib = {
        // Traditional browser API: Takes callback and errback
        ajax: traditionalAjax,
        // Promise API: 返回Promise，也可以接受Promise作为参数
        promiseAjax: callbacks.lift(traditionalAjax)
    };

#### callbacks.liftAll

    var liftedApi = callbacks.liftAll(srcApi);
    var liftedApi = callbacks.liftAll(srcApi, transform);
    var destApi = callbacks.liftAll(srcApi, transform, destApi);

Lifts all the methods of a source object, returning a new object with all the lifted methods.  The optional `transform` function allows you to rename or otherwise customize how the lifted functions are added to the returned object.  If `destApi` is provided, lifted methods will be added to it, instead of to a new object, and `destApi` will be returned.

#### callbacks.call

	var promisedResult = callbacks.call(callbackTakingFunc, arg1, arg2/* more args */);

Takes a callback-taking function and returns a promise for its final value, forwarding any additional arguments. 函数调用回调则诺言兑现，兑现的值取回调的第一个参数。如果回调有多个参数，兑现的值是一个数组。如果异常回调被调用，则诺言会被拒绝。

    var domIsLoaded = callbacks.call($);
    domIsLoaded.then(doMyDomStuff);

    var waitFiveSeconds = callbacks.call(setTimeout, 5000);
    waitFiveSeconds.then(function() {
        console.log("Five seconds have passed");
    });

#### callbacks.apply

	var promisedResult = callbacks.apply(callbackTakingFunc, [arg1, arg2/* more args */]);

类似于`callbacks.call`，相当于`Function.prototype.apply`和`Function.prototype.call`的关系。

    // This example simulates fading away an element, fading in a new one, fetching
    // two remote resources, and then waiting for all that to finish before going
    // forward. The APIs are all callback-based, but only promises are manipulated.

    // Function.prototype.bind is needed to preserve the context
    var oldHidden = callbacks.apply($old.fadeOut.bind($old), ["slow"]);

    var transitionedScreens = oldHidden.then(function() {
        return callbacks.apply($new.fadeIn.bind($new),  ["slow"]);
    });

    var venuesLoaded  = callbacks.apply($.getJSON, ["./venues.json"]);
    var artistsLoaded = callbacks.apply($.getJSON, ["./artists.json"]);

    // Leveraging when.join to combine promises
    when.join(venuesLoaded, artistsLoaded, transitionedScreens).then(function() {
        // Render next screen when everything is ready
    }, function() {
        // Catch-all error handler
    });

#### callbacks.promisify

**已不推荐使用**

    var promiseFunc = callbacks.promisify(nonStandardFunc, {
        callback: zeroBasedIndex,
        errback:  otherZeroBasedIndex,
    });

Almost all the functions on the `callbacks` module assume that the creators of the API were kind enough to follow the unspoken standard of taking the callback and errback as the last arguments on the function call; `callbacks.promisify()` is for when they weren't. In addition to the function to be adapted, `promisify` takes an object that describes what are the positions of the callback and errback arguments.

    function inverseStandard(errback, callback) {
        // ...
    }

    var promisified1 = callbacks.promisify(inverseStandard, {
        callback: 1,
        errback:  0, // indexes are zero-based
    });

    function firstAndThird(callback, someParam, errback) {
        // ...
    }

    var promisified2 = callbacks.promisify(firstAndThird, {
        callback: 0,
        errback:  2,
    });

    // The arguments to the promisified call are interleaved with the callback and
    // errback.
    promisified(10);

    function inverseVariadic(/* arg1, arg2, arg3... , */errback, callback) {
        // ...
    }

    var promisified3 = callbacks.promisify(inverseVariadic, {
        callback: -1, // Negative indexes represent positions relative to the end
        errback:  -2,
    });

#### 我该用哪个模块

|                            | fn | node | callback |
|----------------------------|----|------|----------|
|**The function looks like:**|```doStuff(x, y);```<br/> Synchronous, no callbacks.| ```doStuff(x,y, callback);```<br/>The last parameter is a callback like ```function(err, result)```. | ```doStuff(x, y, callback, errback);```<br/>The next-to-last is callback, last is errback.|
|**Use this:**               | ```require('when');```<br/>```when.lift(doStuff)(x,y).then(...);```|```nodefn = require('when/node');```<br/>```nodefn.lift(doStuff)(x,y).then(...);```|```callbacks = require('when/callbacks');```<br/>```callbacks.lift(doStuff)(x,y).then(...);```|


## ES6 generators

**Experimental**: Requires an environment that supports ES6 generators and the `yield` keyword.

### when/generator

The `when/generator` module provides APIs for using ES6 generators as coroutines.  You can `yield` promises to await their resolution while control is transferred back to the JS event loop.  You can write code that looks and acts like synchronous code, even using synchronous `try`, `catch` and `finally`.

The following example uses `generator.call` to fetch a list of todos for a user, `yield`ing control until the promise returned by `getTodosForUser` is resolved.  If the promise fulfills, execution will continue and show the todos.  If the promise rejects, the rejection will be translated to a synchronous exception (using ES6 generator `.throw()`).  As you'd expect, control will jump to the `catch` and show an error.

```js
var gen = require('when/generator');

gen.call(function*(todosFilter, userId) {
	var todos;
	try {
		todos = yield getTodosForUser(userId);
		showTodos(todos.filter(todosFilter));
	} catch(e) {
		showError(e);
	}
}, isRecentTodo, 123);

function getTodosForUser(userId) {
	// returns a promise for an array of the user's todos
}

```

### generator.lift

```js
var coroutine = generator.lift(es6generator*);

// Deprecated: using lift to partially apply while lifting
var coroutine = generator.lift(es6generator*, arg1, arg2/*...more args*/);
```

Lifts `es6generator` to a promise-aware coroutine, instead of calling it immediately.  Returns a function that, when called, can use `yield` to await promises.  This can be more convenient than using `generator.call` or `generator.apply` by allowing you to create the coroutine once, and call it repeatedly as a plain function.

Additional arguments provided to `generator.lift` will be partially applied to the lifted coroutine.

Here is a revised version of the above example using `generator.lift`.  Note that we're also partially applying the `isRecentTodos` filtering function.

```js
var gen = require('when/generator');

// Use generator.lift to create a function that acts as a coroutine
var getRecentTodosForUser = gen.lift(function*(userId) {
	var todos;
	try {
		todos = yield getTodosForUser(userId);
		showTodos(todos.filter(isRecentTodo));
	} catch(e) {
		showError(e);
	}
});

function getTodosForUser(userId) {
	// returns a promise for an array of the user's todos
}

// Get the recent todos for user 123.
getRecentTodosForUser(123);
```

In addition to `try`, `catch`, and `finally`, `return` also works as expected.  In this revised example, `yield` allows us to return a result and move error handling out to the caller.

```js
var gen = require('when/generator');

// Use generator.lift to create a function that acts as a coroutine
var getRecentTodosForUser = gen.lift(function*(userId) {
	var todos = yield getTodosForUser(userId);
	return todos.filter(isRecentTodo);
});

function getTodosForUser(userId) {
	// returns a promise for an array of the user's todos
}

// filteredTodos is a promise for the recent todos for user 123
var filteredTodos = getRecentTodosForUser(123);
```

### generator.call

```js
var resultPromise = generator.call(es6generator*, arg1, arg2/*...more args*/);
```

Immediately calls `es6generator` with the supplied args, and allows it use `yield` to await promises.

### generator.apply

```js
var resultPromise = generator.apply(es6generator*, [arg1, arg2/*...more args*/]);
```

Similar to `generator.call`, immediately calls `es6generator` with the supplied args array, and allows it use `yield` to await promises.

## Limiting Concurrency

### when/guard

```js
var guard = require('when/guard');

var guarded = guard(condition, function() {
	// .. Do important stuff
});
```

Where:

* `condition` is a concurrency limiting condition, such as [guard.n](#guardn)

Limit the concurrency of a function.  Creates a new function whose concurrency is limited by `condition`.  This can be useful with operations such as [when.map](#whenmap), [when/parallel](#whenparallel), etc. that allow tasks to execute in "parallel", to limit the number which can be inflight simultanously.

```js
// Using when/guard with when.map to limit concurrency
// of the mapFunc

var guard, guardedAsyncOperation, mapped;

guard = require('when/guard');

// Allow only 1 inflight execution of guarded
guardedAsyncOperation = guard(guard.n(1), asyncOperation);

mapped = when.map(array, guardedAsyncOperation);
mapped.then(function(results) {
	// Handle results as usual
});
```

```js
// Using when/guard with when/parallel to limit concurrency
// across *all tasks*

var guard, parallel, guardTask, tasks, taskResults;

guard = require('when/guard');
parallel = require('when/parallel');

tasks = [/* Array of async functions to execute as tasks */];

// Use bind() to create a guard that can be applied to any function
// Only 2 tasks may execute simultaneously
guardTask = guard.bind(null, guard.n(2));

// Use guardTask to guard all the tasks.
tasks = tasks.map(guardTask);

// Execute the tasks with concurrency/"parallelism" limited to 2
taskResults = parallel(tasks);
taskResults.then(function(results) {
	// Handle results as usual
});
```

### Guard conditions

#### guard.n

```js
var condition = guard.n(number);
```

Creates a condition that allows at most `number` of simultaneous executions inflight.

## 错误类型

### TimeoutError

    var TimeoutError = when.TimeoutError;
    // or
    var TimeoutError = require('when/lib/TimeoutError');

[Timeout promises](#promisetimeout) reject with `TimeoutError` unless a custom reason is provided.

## Debugging promises

Errors in an asynchronous operation always occur in a different call stack than the the one that initiated the operation.  Because of that, such errors cannot be caught using synchronous `try/catch`.  Promises help to manage that process by capturing the error and rejecting the associated promise, so that application code can handle the error using promise error handling features, such as [`promise.catch`](#promisecatch).  This is generally a good thing.  If promises *didn't* do this, *any* thrown exception would be uncatchable, even those errors that could have been handled by the application, would instead cause a crash.

However, this also means that errors captured in rejected promises often go silent until observed by calling `promise.catch`.  In nearly all promise implementations, if application code never calls `promise.catch`, the error will be silent.

By default, when.js logs *potentially unhandled rejections* to `console.error`, along with stack traces.  This works even if you don't call [`promise.done`](#promisedone), or if you never call `promise.catch`, and is much like uncaught synchronous exceptions.

Tracking down asynchronous failures can be tricky, so to get even richer debugging information, including long, asynchronous, stack traces, you can enable [`when/monitor/console`](#whenmonitorconsole).

### Potentially unhandled rejections

For example, the following error will be completely silent in most promise implementations.

```js
var when = require('when');

when.resolve(123).then(function(x) {
	// This code executes in a future call stack, and the
	// ReferenceError cannot be caught with try/catch
	oops(x);
});
```

In when.js, you will get an error stack trace to the console:

```
Potentially unhandled rejection [1] ReferenceError: oops is not defined
    at /Users/brian/Projects/cujojs/when/experiments/unhandled.js:4:2
    at tryCatchReject (/Users/brian/Projects/cujojs/when/lib/makePromise.js:806:14)
    at FulfilledHandler.when (/Users/brian/Projects/cujojs/when/lib/makePromise.js:602:9)
    at ContinuationTask.run (/Users/brian/Projects/cujojs/when/lib/makePromise.js:726:24)
    at Scheduler._drain (/Users/brian/Projects/cujojs/when/lib/scheduler.js:56:14)
    at Scheduler.drain (/Users/brian/Projects/cujojs/when/lib/scheduler.js:21:9)
    at process._tickCallback (node.js:419:13)
    at Function.Module.runMain (module.js:499:11)
    at startup (node.js:119:16)
    at node.js:906:3
```

The error is tagged with "Potentially unhandled rejection" and an *id*, `[1]` in this case, to visually call out that it is potentially unhandled.  If the rejection is handled at a later time, a second message including the same id will be logged to help correlate which rejection was handled.  Read on to find out why this might happen.

#### Rejections handled later

It's important to remember that potentially unhandled rejections are, well, *potentially* unhandled. Due to their asynchronous nature, rejected promises may be handled at a later time.  For example, a rejection could be handled after a call to `setTimeout`, even though this is very rare in practice.

Promise rejections fall into 3 categories:

##### Typical usage

In typical usage, rejections should be handled quickly (by calling `then`, `catch`, etc.) either by code higher in the current call stack as a promise is returned, or by code in the current promise chain.

In these most common cases, where all rejections are handled, no errors will be logged just as you expect in synchronous code where all exceptions are caught and handled using `try/catch`.

##### Developer errors

These cases typically represent coding mistakes, such as `ReferenceError`s.  In these cases, the errors will be logged as potentially unhandled rejections, again just as you expect in synchronous code where there is an uncaught exception.

##### Edge cases

In rare cases, application code may leave a rejected promise unobserved for a longer period of time, and then at some point later (for example, after a `setTimeout`), handle it.

In such cases, the rejection may be reported as being potentially unhandled.  When that rejection *is* handled, when.js will log a second message to let you know. For example:

```js
var when = require('when');

var p = when.resolve(123).then(function() {
	throw new Error('this rejection will be handled later');
});

setTimeout(function() {
	p.catch(function(e) {
		// ... handled ...
	});
}, 1000);
```

```
Potentially unhandled rejection [1] Error: this rejection will be handled later
    at /Users/brian/Projects/cujojs/when/experiments/unhandled.js:4:8
    at tryCatchReject (/Users/brian/Projects/cujojs/when/lib/makePromise.js:806:14)
    at FulfilledHandler.when (/Users/brian/Projects/cujojs/when/lib/makePromise.js:602:9)
    at ContinuationTask.run (/Users/brian/Projects/cujojs/when/lib/makePromise.js:726:24)
    at Scheduler._drain (/Users/brian/Projects/cujojs/when/lib/scheduler.js:56:14)
    at Scheduler.drain (/Users/brian/Projects/cujojs/when/lib/scheduler.js:21:9)
    at process._tickCallback (node.js:419:13)
    at Function.Module.runMain (module.js:499:11)
    at startup (node.js:119:16)
    at node.js:906:3
    
... one second later ...

Handled previous rejection [1] Error: this rejection will be handled later
```

In this case, the rejection was handled later.  As mentioned above, the second message includes the id and the original message to correlate with the original error.

### promise.then vs. promise.done

Remember the golden rule: either `return` your promise, or call `done` on it.

At first glance, `then`, and `done` seem very similar.  However, there are important distinctions:

1. The *intent*
2. The error handling characteristics

#### Intent

The intent of `then` is to *transform* a promise's value and to pass or return a new promise for the transformed value along to other parts of your application.

The intent of `done` is to *consume* a promise's value, transferring *responsibility* for the value to your code.

#### Errors

In addition to transforming a value, `then` allows you to recover from, or propagate, *intermediate* errors.  Any errors that are not handled will be caught by the promise machinery and used to reject the promise returned by `then`.

**Note:** [`catch`](#promisecatch) is almost always a better choice for handling errors than `then`. It is more readable, and accepts a `predicate` for matching particular error types. 

Calling `done` transfers all responsibility for errors to your code.  If an error (either a thrown exception or returned rejection) escapes the `handleValue`, or `handleError` you provide to `done`, it will be rethrown in an uncatchable way to the host environment, causing a loud stack trace or a crash.

This can be a big help with debugging, since most environments will then generate a loud stack trace.  In some environments, such as Node.js, the VM will also exit immediately, making it very obvious that a fatal error has escaped your promise chain.

#### A Note on JavaScript Errors

JavaScript allows `throw`ing and `catch`ing any value, not just the various builtin Error types (Error, TypeError, ReferenceError, etc).  However, in most VMs, *only Error types* will produce a usable stack trace.  If at all possible, you should always `throw` Error types, and likewise always reject promises with Error types.

To get good stack traces, do this:

```js
return when.promise(function(resolve, reject) {
	// ...
	reject(new Error('Oops!'));
});
```

And not this:

```js
return when.promise(function(resolve, reject) {
	// ...
	reject('Oops!');
});
```

Do this:

```js
return promise.then(function(x) {
	// ...
	throw new Error('Oops!');
})
```

And not this:

```js
return promise.then(function(x) {
	// ...
	throw 'Oops!';
})
```

### when/monitor/console

Experimental promise monitoring and debugging utilities for when.js.

### What does it do?

tl;dr Load `when/monitor/console` and get awesome async stack traces, even if you forget to return promises or forget to call `promise.done`:

```js
require('when/monitor/console');
var when = require('when');

when().then(function f1() {
	when().then(function f2() {
		when().then(function f3() {
			doh();
		});
	});
});
```

```
ReferenceError: doh is not defined
    at f3 (/Users/brian/Projects/cujojs/when/experiments/trace.js:7:4)
from execution context:
    at f2 (/Users/brian/Projects/cujojs/when/experiments/trace.js:6:21)
from execution context:
    at f1 (/Users/brian/Projects/cujojs/when/experiments/trace.js:5:10)
from execution context:
    at Object.<anonymous> (/Users/brian/Projects/cujojs/when/experiments/trace.js:4:9)
```

It monitors promise state transitions and then takes action, such as logging to the console, when certain criteria are met, such as when a promise has been rejected but has no `onRejected` handlers attached to it, and thus the rejection would have been silent.

Since promises are asynchronous and their execution may span multiple disjoint stacks, it will also attempt to stitch together a more complete stack trace.  This synthesized trace includes the point at which a promise chain was created, through other promises in the chain to the point where the rejection "escaped" the end of the chain without being handled.

### Using it

Load `when/monitor/console` in your environment as early as possible.  That's it.  If you have no unhandled rejections, it will be silent, but when you do have them, it will report them to the console, complete with synthetic stack traces.

It works in modern browsers (AMD), and in Node and RingoJS (CommonJS).

#### AMD

Load `when/monitor/console` early, such as using curl.js's `preloads`:

```js
curl.config({
	packages: [
		{ name: 'when', location: 'path/to/when', main: 'when' },
		// ... other packages
	],
	preloads: ['when/monitor/console']
});

curl(['my/app']);
```

#### Node/Ringo/CommonJS

```js
require('when/monitor/console');
```

#### Browserify

```js
browserify -s PromiseMonitor when/monitor/console.js -o PromiseMonitor.js
```

#### PrettyMonitor for when.js and Node

[PrettyMonitor](https://github.com/AriaMinaei/pretty-monitor) by [@AriaMinaei](https://github.com/AriaMinaei) is an alternative promise monitor on Node.  It's built using when.js's own monitoring apis and modules, and provides a very nice visual display of unhandled rejections in Node.

### Roll your own!

The monitor modules are building blocks.  The [when/monitor/console](../monitor/console.js) module is one particular, and fairly simple, monitor built using the monitoring APIs and tools (PrettyMonitor is another, prettier one!).  Using when/monitor/console as an example, you can build your own promise monitoring tools that look for specific types of errors, or patterns and log or display them in whatever way you need.

## Upgrading to 3.0 from 2.x

While there have been significant architectural changes in 3.0, it remains almost fully backward compatible.  There are a few things that were deprecated and have now been removed, and functionality that has moved to a new preferred spot.

### ES5 Required

As of version 3.0, when.js requires an ES5 environment.  In older environments, use an ES5 shim such as [poly](https://github.com/cujojs/poly) or [es5-shim](https://github.com/es-shims/es5-shim).  For more information, see the [installation docs](installation.md).

### Backward incompatible changes

Previously deprecated features that have been removed in 3.0:

* `promise.always` was removed. Use [`promise.finally(cleanup)`](#promisefinally) (or its ES3 alias [`promise.ensure`](#promisefinally)), or [`promise.then(cleanup, cleanup)`](#promisethen) instead.
* `deferred.resolve`, `deferred.reject`, `deferred.resolver.resolve`, and `deferred.resolver.reject` no longer return promises. They always return `undefined`.  You can simply return `deferred.promise` instead if you need.
* [`when.all`](#whenall), [`when.any`](#whenany), and [`when.some`](#whensome) no longer directly accept `onFulfilled`, `onRejected`, and `onProgress` callbacks.  Simply use the returned promise instead.
	* For example, do this: `when.all(array).then(handleResults)` instead of this: `when.all(array, handleResults)`
* `when.isPromise` was removed. Use [`when.isPromiseLike`](#whenispromiselike) instead.

### Moved functionality

Some functionality has moved to a new, preferred API.  The old APIs still work, and were left in place for backward compatibility, but will eventually be removed:

* `when/delay` module. Use [`promise.delay`](#promisedelay) instead.
* `when/timeout` module. Use [`promise.timeout`](#promisetimeout) instead.
* `when/node/function` module. Use the [`when/node`](#node-style-asynchronous-functions) module instead.
* `when/unfold` and `when/unfold/list` modules. Use [`when.unfold`](#whenunfold) instead
* `when/function` `lift` and `call`. Use [`when.lift`](#whenlift) and [`when.try`](#whentry) instead.
* In the [browserify build](installation.md), `when.node` is now the preferred alias over `when.nodefn`.

## Progress events are deprecated

Progress events are now deprecated, and will be removed in a future release.  They are problematic for several reasons, including:

1. They're implemented in inconsistent ways across promise libraries, making them unreliable when mixing promises.
1. They don't work in a predictable way when combining promises with `all`, `race`, `any`, etc.
1. Returning a promise from a progress handler doesn't have the expected effect of making a promise chain wait.

### Refactoring progress

There is a simple alternative using `promise.tap` that can replace many usages of progress.  Here's an example of the pattern using `tap` to issue progress updates.

```js
var progressBar = //...;

function showProgressUpdate(update) {
	progressBar.setValue(update);
}

functionThatUsesPromiseProgress(showProgressUpdate)
	.then(showCompletedMessage);

// Accept the progress update function as an argument and use
// tap() to call it with progress values
function functionThatUsesPromiseProgress(notify) {
	return doFirstTask()
		.tap(function() {
			// `return` is optional, depending on your needs.
			// Returning allows notify to delay subsequent steps if it returns
			// a promise.  If you don't want that, just call notify and discard
			// its return value.
			return notify(0.333);
		})
		.then(doSecondTask)
		.tap(function() {
			return notify(0.667);
		}
		.then(doThirdTask)
		.tap(function() {
			return notify(1.0);
		});
}
```