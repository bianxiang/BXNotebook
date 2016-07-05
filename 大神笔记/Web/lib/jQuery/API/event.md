[toc]

### 绑定与解除绑定

#### .on()

Returns: jQuery

Attach an event handler function for one or more events to the selected elements.

**.on( events [, selector ] [, data ], handler )**
**.on( events [, selector ] [, data ] )**

- events：字符串。一个或多个空格分隔的事件类型和可选的命名空间，如`"click"`或` "keydown.myPlugin"`。
- selector：字符串。筛选触发事件的后代。若省略该参数或设为null，则事件直接在元素身上触发。
- data：任何。事件触发时，进入事件处理器参数的`event.data`字段。
- handler：函数`( Event eventObject [, Anything extraParameter ] [, ... ] )`。触发的函数。值`false`等价于定义一个函数，只返回`false`。

The `.on()` method attaches event handlers to the currently selected set of elements in the jQuery object. 从jQuery 1.7开始，`.on()`提供绑定事件处理器的所有功能。如果想事件处理器只执行一次，然后自动移除，参见方法`.one()`。

**事件名和命名空间**

事件名只能包含字母、下划线和冒号。

事件名可以加命名空间限定，以简化移除和触发事件。例如`"click.myPlugin.simple"`定义了`myPlugin`和`simple`两个命名空间。`.off("click.myPlugin")`或`.off("click.simple")`都能移除通过上述字符串绑定的事件处理器；不影响绑定到元素的其他click事件处理器。命名空间与CSS类一样不具有层级性，只需要匹配一个。以下划线开头的命名空间是jQuery专用的。

第二种形式的`.on()`，`events`参数是一个对象。键值对分别是，键与`events`参数一样是事件名或空格分隔的事件，可选命名空间。键的值是对应该事件的事件处理函数（或false）。

**直接和代理的事件**

多数浏览器中事件胡冒泡（到body或`document`元素）。在IE8和之前的浏览器，一些事件（如`change`和`submit`）不冒泡。jQuery对这些事件做了修补以保证跨浏览器兼容。

如果`selector`参数省略或为null，事件处理器被称作直接的或直接绑定的。The handler is called every time an event occurs on the selected elements, whether it occurs directly on the element or bubbles from a descendant (inner) element.

如果制定了`selector`参数，事件处理器被称作代理的。The handler is not called when the event occurs directly on the bound element, but only for descendants (inner elements) that match the selector. jQuery先让事件从事件目标冒泡到处理器绑定的元素，然后对路径与选择符匹配的的元素调用事件处理器。

事件处理器只绑定到当前选中的元素；在调用`.on()`时它们必须已经存在。否则可以用代理事件绑定处理器。

代理事件的优势是它们可以处理后续才添加到文档的后代元素的事件。绑定代理事件的元素可以是某个容器元素。或者如果想监控所有冒泡的事件可以用`document`。在文档头部就可以获取到有效的`document`元素，因此用都不必等待文档Body加载完毕。

代理事件的另一个优势时，可以减少需要监控大量元素时的开销。例如，如果一个表格有1,000行，一种方法是向1000个元素绑定一个处理器：

```js
$( "#dataTable tbody tr" ).on( "click", function() {
  console.log( $( this ).text() );
});
```

使用事件代理的方式，只需要向一个元素添加一个事件处理器。例如添加到tbody，此时事件也只需要冒泡一次：

```js
$( "#dataTable tbody" ).on( "click", "tr", function() {
  console.log( $( this ).text() );
});
```

> 注意：Delegated events do not work for SVG.

**事件处理器与它的环境**

事件处理器接收到的[Event](http://api.jquery.com/category/events/event-object/)对象经过了jQuery规范化，是浏览器提供的信息的一个子集。浏览器原来原生的事件对象可以通过`event.originalEvent`获得。

若一个处理器想停止事件继续冒泡（于是阻止了绑定到那些元素的事件处理器执行），可以调用`event.stopPropagation()`。当绑定到当前元素的其他处理器仍会运行。要阻止这些事件处理器，调用`event.stopImmediatePropagation()`。（一个元素事件处理器的调用顺序与绑定顺序相同。）

处理器还可以通过`event.preventDefault()`取消浏览器所有默认的行为。不过不是所有的默认行为都可以被取消。

在事件处理器中返回`false`将自动调用`event.stopPropagation()`和`event.preventDefault()`。

事件处理器执行时，`this`关键字指向事件被递送的目标；对于直接绑定的事件，即事件被绑定到得元素；对于代理事件，`this`是与`selector`匹配的元素。(Note that `this` may not be equal to `event.target` if the event has bubbled from a descendant element.)

**想事件处理器传送数据**

若提供了`data`参数，则每次调用事件处理器时，可以通过`event.data`获取。`data`可以是任何类型。但如果是字符串，则前一个参数`selector`必须提供。

jQuery 1.4开始允许同一个事件处理器绑定到一个元素多次。This is especially useful when the event.data feature is being used, or when other unique data resides in a closure around the event handler function. For example:

```js
function greet( event ) {
  alert( "Hello " + event.data.name );
}
$( "button" ).on( "click", {
  name: "Karl"
}, greet );
$( "button" ).on( "click", {
  name: "Addy"
}, greet );
```

除了通过`data`参数传递数据。利用`.trigger()`或`.triggerHandler()`的第二个参数也可以传递数据。这些数据会被作为事件处理器的参数传入。如果`.trigger()`或`.triggerHandler()`的第二个参数是数组，则每个元素都会作为一个独立参数传给事件处理器。

**事件性能**

在靠近文档树顶部的地方绑定太多代理事件处理器会降低性能。Each time the event occurs, jQuery must compare all selectors of all attached events of that type to every element in the path from the event target up to the top of the document. 为了性能考虑，绑定代理事件的位置尽量接近目标元素。避免大量将代理事件绑定到绑定`document`或`document.body`。

当用于过滤代理事件时，简单选择符（如`tag#id.class`）的处理还是比较快得。So, "#myForm", "a.external", and "button" are all fast selectors. 复杂的选择符，特别是层级的，can be several times slower——尽管对于多数应用足够快了。

**附加说明**

1.9移除了伪事件名`"hover"`，它是`"mouseenter mouseleave"`的缩写。It attaches a single event handler for those two events, and the handler must examine event.type to determine whether the event is mouseenter or mouseleave. 不要把伪事件名与`.hover()`（接受两个函数）混淆。

jQuery's event system requires that a DOM element allow attaching data via a property on the element, so that events can be tracked and delivered. The `object`, `embed`, and `applet` elements cannot attach data, and therefore cannot have jQuery events bound to them.

规范规定`focus`和`blur`事件不冒泡。但jQuery定义了跨浏览器事件`focusin`和 `focusout`是冒泡的。When `focus` and `blur` are used to attach delegated event handlers, jQuery maps the names and delivers them as `focusin` and `focusout` respectively. 为了一致性和清晰，请使用冒泡的事件名。

在所有的浏览器中`load`、`scroll`和`error`事件不冒泡。In Internet Explorer 8 and lower, the `paste` and `reset` events do not bubble. 这些事件不支持代理绑定，但可以被直接绑定。

The `error` event on the `window` object uses nonstandard arguments and return value conventions, so it is not supported by jQuery. 请直接把错误处理器赋给`window.onerror`属性。

The handler list for an element is set when the event is first delivered. Adding or removing event handlers on the current element won't take effect until the next time the event is handled. To prevent any further event handlers from executing on an element within an event handler, call `event.stopImmediatePropagation()`. This behavior goes against the W3C events specification. To better understand this case, consider the following code:

```js
var $test = $( "#test" );
function handler1() {
  console.log( "handler1" );
  $test.off( "click", handler2 );
}
function handler2() {
  console.log( "handler2" );
}
$test.on( "click", handler1 );
$test.on( "click", handler2 );
```

In the code above, handler2 will be executed anyway the first time even if it's removed using `.off()`. However, the handler will not be executed the following times the click event is triggered.

Example: Display a paragraph's text in an alert when it is clicked:

```js
$( "p" ).on( "click", function() {
  alert( $( this ).text() );
});
```

Example: Pass data to the event handler, which is specified here by name:

```js
function myHandler( event ) {
  alert( event.data.foo );
}
$( "p" ).on( "click", { foo: "bar" }, myHandler );
```

Example: Cancel a form submit action and prevent the event from bubbling up by returning false:

```js
$( "form" ).on( "submit", false );
```

Example: Cancel only the default action by using `.preventDefault()`.

```js
$( "form" ).on( "submit", function( event ) {
  event.preventDefault();
});
```

Example: Pass data to the event handler using the second argument to `.trigger()`

```js
$( "div" ).on( "click", function( event, person ) {
  alert( "Hello, " + person.name );
});
$( "div" ).trigger( "click", { name: "Jim" } );
```

Example: Use the the second argument of `.trigger()` to pass an array of data to the event handler

```js
$( "div" ).on( "click", function( event, salutation, name ) {
  alert( salutation + ", " + name );
});
$( "div" ).trigger( "click", [ "Goodbye", "Jim" ] );
```

#### （未）.unbind()

http://api.jquery.com/unbind/

#### （未）.undelegate()

http://api.jquery.com/undelegate/

### 触发事件

#### .trigger()

Returns: jQuery

Execute all handlers and behaviors attached to the matched elements for the given event type.

**.trigger( eventType [, extraParameters ] )
.trigger( event [, extraParameters ] )**

- eventType：字符串。Javascript事件类型，如`click`或`submit`。
- extraParameters：数组或普通对象。传给事件处理器的附加参数。
- event：Event类型。一个`jQuery.Event`对象。

手工触发事件。事件处理器的执行顺序与用户触发时一样。

```js
$( "#foo" ).on( "click", function() {
  alert( $( this ).text() );
});
$( "#foo" ).trigger( "click" );
```

`.trigger()`触发的事件会冒泡。事件处理器可以通过返回false停止冒泡，或者通过调用事件对象的`.stopPropagation()`方法。Although `.trigger()` simulates an event activation, complete with a synthesized event object, it does not perfectly replicate a naturally-occurring event.

To trigger handlers bound via jQuery without also triggering the **native** event, use `.triggerHandler()` instead.

对于通过`.on()`定义的自定义事件，往往需要`.trigger()`的第二个参数。例如我们绑定了一个自定义事件：

```js
$( "#foo" ).on( "custom", function( event, param1, param2 ) {
  alert( param1 + "\n" + param2 );
});
$( "#foo").trigger( "custom", [ "Custom", "Event" ] );
```

事件对象总是作为第一个参数传给事件处理器。传给`.trigger()`的数组，会紧随事件对象传给事件处理器。从jQuery 1.6.2开始，可以传递单个字符串或数字参数，不必用数组包裹。

注意区别通过`.trigger()`传入的`extraParameters`和通过`.on()`传入的`eventData`。二者都是传给事件处理器的附加参数。但通过`.trigger()`传入的，是触发时确定的信息。但通过`.on()`传入的额外参数必须在绑定时就确定。

包裹**普通Javascript对象**的jQuery集合也可以使用`.trigger()`方法，实现发布/监听模式。绑定到该对象的事件处理器都会被触发。

对于普通对象或DOM对象（window除外），如果触发的事件名与对象的一个属性匹配，jQuery会尝试调用那个树形，但前提是没有事件处理器调用`event.preventDefault()`。若这不是你想要的，换成使用`.triggerHandler()`方法。

与`.triggerHandler()`一样，`.trigger()`触发的事件名与对象的一个属性加`on`前缀时匹配（如触发`click`，对象有一个`onclick`方法），jQuery会尝试将属性作为方法调用。

例子:To submit the first form without using the `submit()` function, try:

```js
$( "form:first" ).trigger( "submit" );
```

Example: To pass arbitrary data to an event:

```js
$( "p" ).click(function( event, a, b ) {
// When a normal click fires, a and b are undefined
// for a trigger like below a refers to "foo" and b refers to "bar"
}).trigger( "click", [ "foo", "bar" ] );
```

Example: To pass arbitrary data through an event object:

```js
var event = jQuery.Event( "logged" );
event.user = "foo";
event.pass = "bar";
$( "body" ).trigger( event );
```

Example: Alternative way to pass data through an event object:

```js
$( "body" ).trigger({
  type:"logged",
  user:"foo",
  pass:"bar"
});
```


#### .triggerHandler()

Returns: Object

执行附加到某个元素上某个事件的所有处理器。

**.triggerHandler( eventType [, extraParameters ] )**

- `eventType`：字符串。Javascript事件类型，如`click`或`submit`。
- `extraParameters`：数字或对象。传递给事件处理器的附加参数。

`.triggerHandler( eventType )`执行通过jQuery绑定的某个事件的所有处理器。It will also execute any method called `on{eventType}()` found on the element. 该方法与`.trigger()`的区别是：

- `.triggerHandler( "event" )`不会调用`.event()`。例如，对表单调用`.triggerHandler( "submit" )`不会调用表单的`.submit()`。
- `.trigger()`对匹配的所有元素生效。但`.triggerHandler()`只影响第一个匹配的元素。
- `.triggerHandler()`触发的事件不会冒泡。如果它们不会被目标元素直接处理，则什么也不回发生。
- 它不返回jQuery对象，返回最后一个处理器的返回值。若没有处理器被实际触发，返回`undefined`。

例子：

```html
    <button id="old">.trigger( "focus" )</button>
    <button id="new">.triggerHandler( "focus" )</button><br><br>

    <input type="text" value="To Be Focused">

    <script>
    $( "#old" ).click(function() {
      $( "input" ).trigger( "focus" );
    });
    $( "#new" ).click(function() {
      $( "input" ).triggerHandler( "focus" );
    });
    $( "input" ).focus(function() {
      $( "<span>Focused!</span>" ).appendTo( "body" ).fadeOut( 1000 );
    });
    </script>
```


### 各种事件

#### .unload()

Returns: jQuery
**version deprecated: 1.8**

**.unload( handler )
.unload( [eventData ], handler )**

- **handler**：类型：`Function( Event eventObject )`
- **eventData**：类型：任何。传给事件处理器的对象。

该方法是`.on( "unload", handler )`的缩写。

当用户离开页面时，`unload`事件会发给`window`。用户离开可能通过多种方式，如点击链接，输入新地址，点击向前向后按钮，关闭浏览器窗口，刷新页面。

`unload`事件的具体表现取决于浏览器。例如有些版本的Firefox在关闭窗口时不触发。实际使用时要测各个浏览器。并于私有事件`beforeunload`比较。

`unload`事件的处理器应绑定到`window`对象。

```js
$( window ).unload(function() {
  return "Handler for .unload() called.";
});
```

多数浏览器会忽略时间处理器中调用`alert()`、`confirm()`和`prompt()`。返回的字符串可能显示在一个确认对话框中，但不是所有浏览器都支持。无法通过`.preventDefault()`取消该事件。

