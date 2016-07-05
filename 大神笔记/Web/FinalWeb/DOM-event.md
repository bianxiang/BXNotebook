[toc]

## 事件处理

### 注册事件处理器

有两种注册方式。第一种是设置事件目标的一个属性。第二种是调用方法。每种方式又有两个版本。标准注册方法是 `addEventListener()`，但IE8不支持，需要使用 `attachEvent()`。

#### 设置事件处理器属性

属性一般以 `on` 开头，后跟事件名，如 `onclick`、 `onchange`、 `onload`、 `onmouseover`。注意这些属性名是全小写的。

```js
window.onload = function() {
    // Look up a <form> element
    var elt = document.getElementById("shipping_address");
    elt.onsubmit = function() { return validate(this); }
}
```

这种技术对于常见事件在所有浏览器下可用。属性注册的缺点是一个事件只能注册一个处理器。

#### 设置事件处理器HTML特性

```
	<button onclick="alert('Thank you');">Click Here</button>
```

在HTML事件处理器特性中如果包含多个 JavaScript 语句，需要用分号分隔，或跨多行。

一些整体性的事件，在 JavaScript 中注册到 Window 对象。但在 HTML 中，放到 `<body>`  标签。

When you specify a string of JavaScript code as the value of an HTML event handler attribute, the browser converts your string into a function that looks something like this:

```js
function(event) {
    with(document) {
        with(this.form || {}) {
            with(this) {
            /* your code here */
            }
        }
    }
}
```

#### addEventListener()

`addEventListener()` 有三个参数。第一个是事件类型，字符串。第二个是事件处理函数。最后一个是布尔值。一般为 false。true 表示这是一个捕获处理器，在事件分发的另一个阶段调用（§17.3.6）。

```js
var b = document.getElementById("mybutton");
b.onclick = function() { alert("Thanks for clicking me!"); };
b.addEventListener("click", function() { alert("Thanks again!"); }, false);
```

`addEventListener()` 不会覆盖通过事件属性指定的事件处理器。`addEventListener()`自己可以被调用多次。多个事件处理器按注册顺序被调用。使用同一组参数多次调用 `addEventListener()` 无效，后续调用不会改变执行顺序。

`removeEventListener()` 接收与 `addEventListener()` 相同的参数，解除注册。

#### attachEvent()

IE9 之前不支持 `addEventListener()` 和 `removeEventListener()`。只能用 `attachEvent()` 和 `detachEvent()`。二者特别的地方有：

- 由于IE的事件模型不支持事件捕获，`attachEvent()` 和 `detachEvent()` 只接两个参数：事件类型和处理器函数。
- 第一个参数不是事件名，是事件属性名，即要带 `on` 前缀。
- `attachEvent()` 允许同一个处理器函数被多次调用。触发时注册几次执行几次。

例子：

```js
var b = document.getElementById("mybutton");
var handler = function() { alert("Thanks!"); };
if (b.addEventListener)
    b.addEventListener("click", handler, false);
else if (b.attachEvent)
    b.attachEvent("onclick", handler);
```

### 事件处理器

执行顺序：

- 通过设置属性或HTML特性注册的事件处理器先被调用。
- 通过 `addEventListener()` 注册的事件处理器按注册顺序调用。
- 通过 `attachEvent()` 注册的事件处理器的调用顺序是任意的。

#### 事件处理器参数

事件处理器一般接受单个事件对象。事件对象的属性给出事件细节。例如 `type` 属性给出事件类型。

IE8和之前，通过设置属性注册的事件处理器被调用时不会传入事件对象。事件对象可以通过全局变量 `window.event` 获得。为可移植性，可以：

```
function handler(event) {
    event = event || window.event;
    // Handler code goes here
}
```

通过 `attachEvent()` 注册的事件处理器也会被传入一个事件对象，它们也可以使用 `window.event`。

#### 处理器返回值

通过设置属性或设置HTML特性注册的事件处理器的返回值一般来说是重要的。返回 `false` 一般表示请求浏览器不要执行默认的行为。例如输入框的 `onkeypress` 事件处理器可以通过返回 `false` 拒绝用户输入。

Window 对象 `onbeforeunload` 事件处理器的返回值也是关键的。当浏览器准备转到另一个页面时触发该事件。若该事件处理器返回一个字符串，它将显示在对话框中让用户确认。

注意通过 `addEventListener()` 或 `attachEvent()` 注册的事件处理器必须通过调用 `preventDefault()` 方法或设置事件对象的 `returnValue` 属性来阻止默认行为。不能通过返回值。

#### 事件处理器上下文

当通过设置属性注册事件处理器时，看上去好像在给文档元素定义一个新方法（不是函数，是方法）：

```
e.onclick = function() { /* handler code */ };
```

即在处理器函数内，`this` 关键字指向事件目标。

使用 `addEventListener()`，`this` 指向事件目标。但使用 `attachEvent()` 注册的处理器作为函数调用，它们的 `this` 值指向全局对象（Window对象）。

### 事件的传播

事件冒泡会冒泡到 Document 对象，最后到 Window 对象。
文档元素的 `load` 事件也冒泡，但在 Document 对象处停止，不会传给 Window 对象。

事件冒泡发生在事件传播的第三个阶段。调用目标对象自己的事件处理器是第二阶段。第一阶段是捕获阶段。事件冒泡IE也支持。但事件捕获IE不支持。

捕获阶段就像冒泡阶段的反转。Window 对象的捕获处理器先被调用，然后是 Document 对象的捕获处理器，然后是 body 对象……

使用事件捕获的常见常见是处理鼠标拖动：鼠标事件应由被拖动的对象处理，而不是鼠标下方的元素处理。

### 取消默认行为

在通过 `addEventListener()` 注册的事件处理器中，可以调用事件对象的 `preventDefault()` 方法取消默认行为。但在 IE9 之前，取消默认行为的方法是设置事件对象的 `returnValue` 为 false。下面展示三种取消技术：

```js
function cancelHandler(event) {
    var event = event || window.event; // For IE
    /* Do something to handle the event here */
    // Now cancel the default action associated with the event
    if (event.preventDefault) event.preventDefault(); // 标注
    if (event.returnValue) event.returnValue = false; // IE
    return false; // 如果事件处理器是通过对象属性注册的
}
```

### 停止事件传播

停掉事件传播。通过 `addEventListener()` 注册的事件监听器，可以通过 `stopPropagation()` 方法阻止事件继续传播。但注册在同一对象上的其他处理器仍会被调用。`stopPropagation()` 可以在事件传播的任意时刻调用，可以在捕获阶段、在事件目标上，或在冒泡阶段。
IE9 之前不支持 `stopPropagation()` 方法。IE 的事件对象有一个 `cancelBubble` 属性。设置该属性为 `true` 能阻止事件传播。

事件对象的 `stopImmediatePropagation()` 方法不仅起到 `stopPropagation()` 左右，还会立即停止事件传给注册到相同对象的其他事件处理器。


### 鼠标事件

鼠标事件有很多。除 `mouseenter` 和 `mouseleave` 外的事件都冒泡。一些事件有默认行为且可以取消。

- **click**：高层事件。
- **contextmenu**：可取消的事件，表示上下文菜单要弹出。目前浏览器在收到鼠标右击后显示上下文菜单，因此该事件可以像 `click` 事件一样使用。
- **dblclick**：双击鼠标
- **mousedown**：按下鼠标键
- **mouseup**：
- **mousemove**：鼠标移动
- **mouseover**：鼠标进入元素。`relatedTarget`（IE下是`fromElement`）指出鼠标来自哪个元素。
- **mouseout**：鼠标离开元素。`relatedTarget`（IE下是`toElement`）指出鼠标要去哪个元素。
- **mouseenter**：与“mouseover”一样，但不冒泡。HTML5引入。
- **mouseleave**：与“mouseout”一样，但不冒泡。HTML5引入。

事件对象的 `clientX` 和 `clientY` 给出鼠标位置。是窗口坐标，加上滚动条偏移转换为文档坐标。

`altKey`、 `ctrlKey`、 `metaKey`、 `shiftKey` 给出掩码的状态。

`button` 给出按下的是哪个键（如果有按下）。不同浏览器下返回值不同。

一些浏览器只有左键点击才会触发 `click` 事件。其他键的点击需要监听 `mousedown` 和 `mouseup` 事件。

例子，允许一个绝对定位元素被拖动。

```
    <img src="draggable.gif"
        style="position:absolute; left:100px; top:100px;"
        onmousedown="if (event.shiftKey) drag(this, event);">
```

`drag()` 内注册一个 `mousemove` 和 `mouseup` 事件处理器。`mousemove` 处理器负责移动元素据，`mouseup` 处理器负责让自己停止监听 `mousemove` 事件。

注意 `mousemove` 和 `mouseup` 处理器注册成了捕获事件处理器。原因是用户移动鼠标的速度可能比文档元素跟踪的速度慢，一些 `mousemove` 事件可能发生在原来的事件目标之外。不用捕获，这些事件不会被送到正确的处理器。
IE 事件默认不支持标准捕获方式。但它有一个 `setCapture()` 方法。

```js
function drag(elementToDrag, event) {
	// 转换为文档坐标
	var scroll = getScrollOffsets(); // A utility function from elsewhere
	var startX = event.clientX + scroll.x;
	var startY = event.clientY + scroll.y;
	// 元素初始位置（文档坐标）。因为绝对定位，假设 offsetParent 是body
    var origX = elementToDrag.offsetLeft;
    var origY = elementToDrag.offsetTop;
	// 计算鼠标与元素距离。鼠标移动过程中保持这个距离
    var deltaX = startX - origX;
    var deltaY = startY - origY;
    // 注册处理器
	if (document.addEventListener) { // Standard event model
    	// Register capturing event handlers on the document
	    document.addEventListener("mousemove", moveHandler, true);
    	document.addEventListener("mouseup", upHandler, true);
	} else if (document.attachEvent) { // IE Event Model for IE5-8
        // In the IE event model, we capture events by calling
        // setCapture() on the element to capture them.
        elementToDrag.setCapture();
        elementToDrag.attachEvent("onmousemove", moveHandler);
        elementToDrag.attachEvent("onmouseup", upHandler);
        // Treat loss of mouse capture as a mouseup event.
        elementToDrag.attachEvent("onlosecapture", upHandler);
    }
    // We've handled this event. Don't let anybody else see it.
    if (event.stopPropagation) event.stopPropagation(); // Standard model
	else event.cancelBubble = true; // IE
    // Now prevent any default action.
    if (event.preventDefault) event.preventDefault(); // Standard model
	else event.returnValue = false; // IE

    function moveHandler(e) {
        if (!e) e = window.event; // IE event Model
        // Move the element to the current mouse position, adjusted by the
        // position of the scrollbars and the offset of the initial click.
        var scroll = getScrollOffsets();
        elementToDrag.style.left = (e.clientX + scroll.x - deltaX) + "px";
        elementToDrag.style.top = (e.clientY + scroll.y - deltaY) + "px";
        // And don't let anyone else see this event.
        if (e.stopPropagation) e.stopPropagation(); // Standard
        else e.cancelBubble = true; // IE
    }

    function upHandler(e) {
        if (!e) e = window.event; // IE Event Model
        // Unregister the capturing event handlers.
        if (document.removeEventListener) { // DOM event model
            document.removeEventListener("mouseup", upHandler, true);
            document.removeEventListener("mousemove", moveHandler, true);
        } else if (document.detachEvent) { // IE 5+ Event Model
            elementToDrag.detachEvent("onlosecapture", upHandler);
            elementToDrag.detachEvent("onmouseup", upHandler);
            elementToDrag.detachEvent("onmousemove", moveHandler);
            elementToDrag.releaseCapture();
        }
        // And don't let the event propagate any further.
        if (e.stopPropagation) e.stopPropagation(); // Standard model
        else e.cancelBubble = true; // IE
    }
}
```