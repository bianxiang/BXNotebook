[toc]

http://davidwalsh.name/es6-generators

node 0.11+ with the --harmony flag)

通过 ES6 generators，允许我们拥有一种函数，可以在中途暂停、后续恢复。

产生器函数的暂停由**自己**控制：在函数内使用 `yield` 表达式让自己暂停。外部无法让函数暂停。
但产生器的恢复不能由自己控制，由外部控制。

> 总结：暂停由自己控制、恢复由外部控制。

### 基础

使用产生器，最基本的四个元素是：产生器、`yield` 表达式、产生器的迭代器、迭代器的 `next()` 函数。

例子，先定义一个产生器函数：

```js
function *foo() {
    yield 1;
    yield 2;
    yield 3;
    yield 4;
    yield 5;
}
```

星号的位置可以靠近 `function` 或函数名。即 `function* foo(){ }` 和 `function *foo(){ }` 两种写法是等效的。

要再外部控制产生器，需要一个**产生器的迭代器**（generator iterator）：

```js
var it = foo();
```

调用产生器函数得到的就是它的迭代器。有了迭代器，通过可以通过其 `next()` 函数获取当前值，并恢复执行：

```js
var message = it.next();
```

此时 `message` 的值是 `{ value:1, done:false }`。即 `next()` 返回一个对象，其 `value` 是 `yield` 后的值。`done` 表示迭代器函数是否完成。继续调用：

```js
console.log( it.next() ); // { value:2, done:false }
console.log( it.next() ); // { value:3, done:false }
console.log( it.next() ); // { value:4, done:false }
console.log( it.next() ); // { value:5, done:false }
console.log( it.next() ); // { value:undefined, done:true }
```

### 产生器函数的返回值

产生器函数可以有返回值。但这个值几乎没有什么用处。例子：

```js
function *foo() {
    yield 1;
    return 2;
}

var it = foo();

console.log( it.next() ); // { value:1, done:false }
console.log( it.next() ); // { value:2, done:true }
```

迭代器其实还是只产生一次值。

### 双向传递消息

通过暂停和恢复，还可以实现内外双向消息传递。普通函数只能在调用时传入参数，在结束时返回结果。但对于产生器，可以在暂停时向外传递信息，在恢复时，外部向内部产地信息。

`yield ___` 称为“yield**表达式**”，而不是“yield语句”，因为它有返回值。
一个 yield 表达式，如 `y = yield x`。第一次调用 `next()`，产生器送出x的值，**然后暂停**！！——此时对 `y` 的赋值尚未进行，表达式以下的语句也不会执行——**暂停点在赋值前**！！要想恢复执行，要再次调用 `next(a)`，并且 `a` 的值会赋给 `y`，然后继续执行后面的语句，直到遇到下一个 yield 或方法结束。

第一次调用 `next()`，不传任何值。传也没用，因为没有 `yield` 接收。但如果我们传了，没有害处，值会被忽略。（目前部分浏览器实现在这一点上可能有BUG，**因此最好还是不传**。）

例子：

```js
function *foo(x) {
    var y = 2 * (yield (x + 1));
    var z = yield (y / 3);
    return (x + y + z);
}

var it = foo( 5 );

console.log( it.next() );       // { value:6, done:false }
console.log( it.next( 12 ) );   // { value:8, done:false }
console.log( it.next( 13 ) );   // { value:42, done:true }
```

首先注意，产生器函数可以像普通函数那样，有函数参数（这里是 `x`）。

第一次 `next()` 拿到 `yield (x + 1)` 向外送出的值：6，**并暂停**！！第二次，`next(12)`，恢复执行，送12进入函数，`y` 得到 `2 * 12`。

例子2：

```js
var generator = function* () {
    console.log("before yield 1");
    var r = yield 1;
    console.log("after yield 1");
    console.log("yield 1 return: " + r);

    console.log("before yield 2");
    r = yield 2;
    console.log("after yield 2");
    console.log("yield 2 return: " + r);
};

console.log('before call generator');
var itr = generator();
console.log('after call generator');

console.log('before next 1');
var result = itr.next('a');
console.log('next 1 return ' + JSON.stringify(result));

console.log('before next 2');
result = itr.next('b');
console.log('next 2 return ' + JSON.stringify(result));

console.log('before next 3');
result = itr.next('c');
console.log('next 3 return ' + JSON.stringify(result));
```

输出：

```
before call generator
after call generator
before next 1
before yield 1
next 1 return {"value":1,"done":false}
before next 2
after yield 1
yield 1 return: b
before yield 2
next 2 return {"value":2,"done":false}
before next 3
after yield 2
yield 2 return: c
next 3 return {"done":true}
```

`yield` 可以不输出任何值，只是让函数暂停。`yield` 可以出现在任何表达式可以出现的地方。例子：

```js
// note: `foo(..)` here is NOT a generator!!
function foo(x) {
    console.log("x: " + x);
}

function *bar() {
    yield; // just pause
    foo( yield );
}
```

### for..of

ES6 提供了一种专门的语法处理产生器的迭代：即 for..of 循环。

```js
function *foo() {
    yield 1;
    yield 2;
    yield 3;
    yield 4;
    yield 5;
    return 6;
}

for (var v of foo()) {
    console.log( v );
}
// 1 2 3 4 5

console.log( v ); // still `5`, not `6`
```

### 错误处理

通过迭代器的 `throw` 方法，可以把一个异常扔进产生器函数内。例如：

```js
function *foo() {
    try {
        var x = yield 3;
        console.log( "x: " + x ); // may never get here!
    } catch (err) {
        console.log( "Error: " + err );
    }
}

var it = foo();
var res = it.next(); // { value:3, done:false }

it.throw( "Oops!" ); // Error: Oops!
```

在产生器函数内，围绕 `yield` 表达式，使用普通的 try-catch 即可捕获 `throw()` 抛入的异常。

另一方面，若再产生器函数内，不捕获异常，则就相当于在调用 `throw()` 处原地抛出该异常。如：

```js
function *foo() { }

var it = foo();
try {
    it.throw( "Oops!" );
} catch (err) {
    console.log( "Error: " + err ); // Error: Oops!
}
```

反过来，若在产生器函数内抛出异常，可以在调用 `next()` 时捕获：

```js
function *foo() {
    var x = yield 3;
    var y = x.toUpperCase(); // could be a TypeError error!
    yield y;
}

var it = foo();
it.next(); // { value:3, done:false }
try {
    it.next( 42 ); // `42` won't have `toUpperCase()`
} catch (err) {
    console.log( err ); // TypeError (from `toUpperCase()` call)
}
```

### 代理

在产生器函数内调用另一个产生器函数，将自己的循环控制代理给它，此时需要使用 `yield *` 关键字。例子：

```js
function *foo() {
    yield 3;
    yield 4;
}

function *bar() {
    yield 1;
    yield 2;
    yield *foo(); // `yield *` delegates iteration control to `foo()`
    yield 5;
}

for (var v of bar()) {
    console.log( v );
}
// 1 2 3 4 5
```

`yield *foo()` 与 `yield* foo()` 是等效的，即 `*` 位置不重要。

若用到双向通信，通过迭代器`next(..)`传入的消息，会传入`yield*`代理：

```js
function *foo() {
    var z = yield 3;
    var w = yield 4;
    console.log( "z: " + z + ", w: " + w );
}

function *bar() {
    var x = yield 1;
    var y = yield 2;
    yield *foo(); // `yield*` delegates iteration control to `foo()`
    var v = yield 5;
    console.log( "x: " + x + ", y: " + y + ", v: " + v );
}

var it = bar();

it.next();      // { value:1, done:false }
it.next( "X" ); // { value:2, done:false }
it.next( "Y" ); // { value:3, done:false }
it.next( "Z" ); // { value:4, done:false }
it.next( "W" ); // { value:5, done:false }
// z: Z, w: W

it.next( "V" ); // { value:undefined, done:true }
// x: X, y: Y, v: V
```

`yield* xxx()` 也是一个表达式，其返回值是代理产生器函数的返回值：

```js
function *foo() {
    yield 2;
    yield 3;
    return "foo"; // return value back to `yield*` expression
}

function *bar() {
    yield 1;
    var v = yield *foo();
    console.log( "v: " + v );
    yield 4;
}

var it = bar();

it.next(); // { value:1, done:false }
it.next(); // { value:2, done:false }
it.next(); // { value:3, done:false }
it.next(); // "v: foo"   { value:4, done:false }
it.next(); // { value:undefined, done:true }
```

同样可以在两个方向，跨 `yield*` 代理进行错误处理。

```js
function *foo() {
    try {
        yield 2;
    } catch (err) {
        console.log( "foo caught: " + err );
    }

    yield; // pause

    // now, throw another error
    throw "Oops!";
}

function *bar() {
    yield 1;
    try {
        yield *foo();
    } catch (err) {
        console.log( "bar caught: " + err );
    }
}

var it = bar();

it.next(); // { value:1, done:false }
it.next(); // { value:2, done:false }

it.throw( "Uh oh!" ); // will be caught inside `foo()`
// foo caught: Uh oh!

it.next(); // { value:undefined, done:true }  --> No error here!
// bar caught: Oops!
```

捕获 `throw("Uh oh!")` 的是代理 `*foo()`。异常被直接传到了最里面。

### 异步

产生器最强大的一点是，产生器内部的代码是同步的，外部控制可以是异步的。

核心思想：调用异步函数并暂停。在异步函数的回调中，通过 `next(response)` 的参数 `response` 把异步结果最终送到 `yield` 表达式的结果。

假如原来的代码是：

```js
function makeAjaxCall(url,cb) {
    // do some ajax fun
    // call `cb(result)` when complete
}

makeAjaxCall( "http://some.url.1", function(result1){
    var data = JSON.parse( result1 );
    makeAjaxCall( "http://some.url.2/?id=" + data.id, function(result2){
        var resp = JSON.parse( result2 );
        console.log( "The value you asked for: " + resp.value );
    });
} );
```

用产生器表达相同功能：

```js
function request(url) {
    // this is where we're hiding the asynchronicity,
    // away from the main code of our generator
    // `it.next(..)` is the generator's iterator-resume
    // call
    makeAjaxCall( url, function(response){
        it.next( response );
    } );
    // 注意：没有返回值！
}

function *main() {
    var result1 = yield request( "http://some.url.1" );
    var data = JSON.parse( result1 );

    var result2 = yield request( "http://some.url.2?id=" + data.id );
    var resp = JSON.parse( result2 );
    console.log( "The value you asked for: " + resp.value );
}

var it = main();
it.next(); // get it all started
```

`request(..)` 函数封装 `makeAjaxCall(..)` 函数，关键点是在回调方法中调用产生器迭代器的 `next(..)` 方法。

如果一种情况是，有时会从服务器取值，但有时可以从缓存中直接钠拿值。这时代码要略微修改避免一个问题：

```js
var cache = {};
function request(url) {
    if (cache[url]) {
        // "defer" cached response long enough for current
        // execution thread to complete
        setTimeout( function(){
            it.next( cache[url] );
        }, 0 );
    } else {
        makeAjaxCall( url, function(resp){
            cache[url] = resp;
            it.next( resp );
        } );
    }
}
```

关键点是 `setTimeout(..0)`。若直接调用 `it.next(..)` 将导致错误，因为此时产生器尚未处于暂停状态：`request(..)` 先执行，然后 `yield` 执行并暂停。后面会讲一种更好的解决方法。

### 更好的异步

要更好的处理异步，还需要用到**Promises**。

The earlier Ajax code examples here suffer from all the same Inversion of Control issues (aka "callback hell") as our initial nested-callback example. Some observations of where things are lacking for us so far:

1. There's no clear path for error handling. 我们可以侦测到Ajax调用的错误，通过`it.throw(..)`将其传回产生器，在产生器中通过try..catch处理。But that's just more manual work to wire up in the "back-end" (the code handling our generator iterator), and it may not be code we can re-use if we're doing lots of generators in our program.
1. 若`makeAjaxCall(..)`不受我们控制，可能会多次调用回调，或其他错误，此时我们的产生器会出错！处理这类问题又需要大量手工工作，同时也是难以移植的。
1. 有时我们需要并行做一些任务（如同时发出两个Ajax调用）。但由于一个产生器yield只是一个暂停点，多个不同同时运行。

解决办法就是让`yield`放出promises（即`yield xxx`表达式，xxx是一个promise）。

之前的`yield request(..)`，由于`request(..)`未返回任何值，实际`yield undefined`。
现在我们让`request(..)`返回一个promise：

```js
    function request(url) {
        // Note: returning a promise now!
        return new Promise( function(resolve, reject){
            makeAjaxCall( url, resolve );
        } );
    }
```

我们需要一个工具，控制产生器的迭代器，接收promise，恢复产生器。这里称这个工具为`runGenerator(..)`：

```js
    // run (async) a generator to completion
    // Note: simplified approach: no error handling here
    function runGenerator(g) {
        var it = g(), ret;
        // asynchronously iterate over generator
        (function iterate(val){
            ret = it.next( val );

            if (!ret.done) {
                // 检查返回值是否为promise
                if ("then" in ret.value) {
                    // wait on the promise
                    ret.value.then( iterate );
                }
                // immediate value: just send right back in
                else {
                    // avoid synchronous recursion
                    setTimeout( function(){
                        iterate( ret.value );
                    }, 0 );
                }
            }
        })();
    }
```

使用：

```js
    runGenerator( function *main(){
        var result1 = yield request( "http://some.url.1" );
        var data = JSON.parse( result1 );

        var result2 = yield request( "http://some.url.2?id=" + data.id );
        var resp = JSON.parse( result2 );
        console.log( "The value you asked for: " + resp.value );
    } );
···

可以看到产生器内部跟上一节几乎一样。

对于任务的并行执行，我们可以用`yield Promise.all([ .. ])`，合成单个promise。

下面我们来看错误处理：

```js
    // assume: `runGenerator(..)` now also handles error handling (omitted for brevity)
    function request(url) {
        return new Promise( function(resolve, reject){
            // pass an error-first style callback
            makeAjaxCall( url, function(err, text){
                if (err) reject( err );
                else resolve( text );
            } );
        } );
    }

    runGenerator( function *main(){
        try {
            var result1 = yield request( "http://some.url.1" );
        } catch (err) {
            console.log( "Error: " + err );
            return;
        }
        var data = JSON.parse( result1 );

        try {
            var result2 = yield request( "http://some.url.2?id=" + data.id );
        } catch (err) {
            console.log( "Error: " + err );
            return;
        }
        var resp = JSON.parse( result2 );
        console.log( "The value you asked for: " + resp.value );
    } );
```

### `runGenerator(..)`：工具库

如 `Q.spawn(..)` 、 `co(..)` 等。

I will however briefly cover my own library's utility: [asynquence](http://github.com/getify/asynquence)'s `runner(..)` plugin, as I think it offers some unique capabilities over the others out there. I wrote an [in-depth 2-part blog post series on asynquence](http://davidwalsh.name/asynquence-part-1/) if you're interested in learning more than the brief exploration here.

First off, **asynquence** provides utilities for automatically handling the "error-first style" callbacks from the above snippets:

```js
    function request(url) {
        return ASQ( function(done){
            // pass an error-first style callback
            makeAjaxCall( url, done.errfcb );
        } );
    }
```

That's much nicer, isn't it!?

Next, **asynquence**'s `runner(..)` plugin consumes a generator right in the middle of an asynquence sequence (asynchronous series of steps), so you can pass message(s) in from the preceding step, and your generator can pass message(s) out, onto the next step, and all errors automatically propagate as you'd expect:

```js
    // first call `getSomeValues()` which produces a sequence/promise,
    // then chain off that sequence for more async steps
    getSomeValues()

    // now use a generator to process the retrieved values
    .runner( function*(token){
        // token.messages will be prefilled with any messages
        // from the previous step
        var value1 = token.messages[0];
        var value2 = token.messages[1];
        var value3 = token.messages[2];

        // make all 3 Ajax requests in parallel, wait for
        // all of them to finish (in whatever order)
        // Note: `ASQ().all(..)` is like `Promise.all(..)`
        var msgs = yield ASQ().all(
            request( "http://some.url.1?v=" + value1 ),
            request( "http://some.url.2?v=" + value2 ),
            request( "http://some.url.3?v=" + value3 )
        );

        // send this message onto the next step
        yield (msgs[0] + msgs[1] + msgs[2]);
    } )

    // now, send the final result of previous generator
    // off to another request
    .seq( function(msg){
        return request( "http://some.url.4?msg=" + msg );
    } )

    // now we're finally all done!
    .val( function(result){
        console.log( result ); // success, all done!
    } )

    // or, we had some error!
    .or( function(err) {
        console.log( "Error: " + err );
    } );
```

The asynquence runner(..) utility receives (optional) messages to start the generator, which come from the previous step of the sequence, and are accessible in the generator in the token.messages array.

Then, similar to what we demonstrated above with the runGenerator(..) utility, runner(..) listens for either a yielded promise or yielded asynquence sequence (in this case, an `ASQ().all(..)` sequence of "parallel" steps), and waits for it to complete before resuming the generator.

When the generator finishes, the final value it yields out passes along to the next step in the sequence.

Moreover, if any error happens anywhere in this sequence, even inside the generator, it will bubble out to the single or(..) error handler registered.

asynquence tries to make mixing and matching promises and generators as dead-simple as it could possibly be. You have the freedom to wire up any generator flows alongside promise-based sequence step flows, as you see fit.

### ES7 async

There is a proposal for the ES7 timeline, which looks fairly likely to be accepted, to create still yet another kind of function: an **async function**, which is like a generator that's automatically wrapped in a utility like runGenerator(..) (or asynquence's' runner(..)). That way, you can send out promises and the async function automatically wires them up to resume itself on completion (no need even for messing around with iterators!).

It will probably look something like this:

```js
    async function main() {
        var result1 = await request( "http://some.url.1" );
        var data = JSON.parse( result1 );

        var result2 = await request( "http://some.url.2?id=" + data.id );
        var resp = JSON.parse( result2 );
        console.log( "The value you asked for: " + resp.value );
    }

    main();
```

As you can see, an async function can be called directly (like main()), with no need for a wrapper utility like runGenerator(..) or ASQ().runner(..) to wrap it. Inside, instead of using yield, you'll use await (another new keyword) that tells the async function to wait for the promise to complete before proceeding.

Basically, we'll have most of the capability of library-wrapped generators, but directly supported by native syntax.

In the meantime, libraries like asynquence give us these runner utilities to make it pretty darn easy to get the most out of our asynchronous generators!

### （未）Getting Concurrent With ES6 Generators

http://davidwalsh.name/concurrent-generators




