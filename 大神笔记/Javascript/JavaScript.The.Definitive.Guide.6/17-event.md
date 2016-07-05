[toc]

## 17. 事件

事件类型，也称事件名，是一个字符串，如 `mousemove`、 `keydown`、 `load`。

事件对象会传给事件处理方法（IE8及之前，只能通过全局变量 `event` 获取）。所有的事件对象都有 `type` 属性指出事件类型和 `target` 属性指出事件目标（在IE8及之前，需要用 `srcElement` 替代 `target`）。

**事件传播**指浏览器决定触发哪个对象的事件处理器。只有一个对象的事件（如 Window 对象的 load 事件）不需要传播。某些发生在文档元素上的事件会沿文档树向上冒泡。例如，移动鼠标到一个链接上，`mousemove` 事件先在 `<a>` 元素上触发，然后它在父容器上触发，最终在 Document 对象上触发。有时在 Document 对象或其他容器元素上注册一个事件处理器，比在各个独立的元素上注册独立的处理器更方便简单。事件处理器能够停止事件传播，停止事件继续冒泡，让事件不在父辈继续触发。

另一种形式的事件传播，称为事件捕获（capturing）：容器的事件处理器有机会在事件到达实际目标前拦截（捕获）事件。IE8 及之前的浏览器不支持事件捕获，因此没有被广泛使用。对于如处理鼠标拖动的事件，捕获鼠标事件是必须的。

有些事件有默认行为。如点击链接打开链接。事件处理器可以阻止默认行为。

### 17.1 事件类型

长期以来被大量浏览器支持的事件参见 §17.1.1。近年来又有大量新事件出现，它们来自：

- DOM Level 3 Events 规范，见 §17.1.2。
- HTML5 规范，如历史管理、拖放、跨文档消息、音视频播放。见 §17.1.3。
- 依靠触摸的移动设备，需要一些新的触摸和手势（gesture）事件，参见 §17.1.4。

#### 17.1.1 遗留事件类型

##### 17.1.1.1 表单事件

提交表单使 `<form>` 触发 `submit` 事件，重置触发 `reset` 事件。按钮类的元素点击触发 `click` 事件。表单元素在输入改变、选择改变、勾选选择框时触发 `change` 事件。对于文本框，触发 `change` 事件需要用户完成交互：将焦点移出。表单元素在焦点改变后触发 `focus` 事件和 `blur` 事件。

表单相关的事件在 §15.9.3 已详细介绍过。

`submit`、`reset` 和一些 `click` 事件可以被取消。`focus` 和 `blur` 事件不冒泡，但其他表单事件都冒泡。IE 定义了 `focusin` 和 `focusout` 事件，像其他事件一样冒泡，可以替代 `focus` 和 `blur`。jQuery允许在不支持的浏览器中使用 `focusin` 和 `focusout` 事件；DOM Level 3 Events规范也将此收入了规范。

用户在文本框中输入文字时（或粘贴）触发 `input` 事件（IE除外）。与 `change` 不同的是，每次插入时都会触发 `input` 事件。其事件对象没有给出输入内容。新的 `textinput` 事件可以替代此事件。

##### 17.1.1.2 Window 事件

`load` 事件在文档和所有外部资源（如图片）都加载完后触发。`load` 事件已在第13章详细讨论。**DOMContentLoaded 和 readystatechange**可以替代 `load` 事件：它们触发更早：当文档、元素准备好被操纵时，在外部资源被完全加载完之前。§17.4 会介绍这些加载事件的例子。

`unload` 事件在用户离开文档时触发。`unload` 事件处理器可以用户保存用户的状态，但不能取消导航。`beforeunload` 事件可以让用户确认是否要离开。若其事件处理器返回一个字符串，字符串将显示在一个确认对话框上，用户可以选择留在当前页面、取消导航。

一些文档元素（如 `<img>`）也可以触发 `load` 和 `error` 事件。它们在资源完成加载或加载错误时触发。一些浏览器（HTML5已规范）也支持 `abort` 事件，在用户停止加载资源时触发。

Window 对象也能触发 `focus` 和 `blur` 事件。它们在浏览器窗口从操作系统获得或失去键盘焦点时触发。

当用户调整窗口大小或滚动滚动条时，触发 Window 对象的 `resize` 和 `scroll` 事件。其他可滚动的元素（如设置了CSS属性 `overflow`）也支持 `scroll` 事件。`scroll` 事件不冒泡。

##### 17.1.1.3 鼠标事件

鼠标事件在**最里层的元素触发**。会向上冒泡。事件对象的属性 `clientX` 和 `clientY` 指出鼠标位置（window坐标{{视口坐标}}）。`button` 和 `which` 指出哪个鼠标键被按下（这些属性可移植性较差）。如果某个修饰键被按下，`altKey`、 `ctrlKey`、 `metaKey` 和 `shiftKey` 属性为true。对于 `click` 事件，`detail` 属性指出是单击、双击还是三击。

用户移动**或拖动**鼠标触发 `mousemove` 事件。由于此事件频繁触发，事件处理器不要做高负载运算。用户按下或释放一个鼠标键时触发 `mousedown` 和 `mouseup` 事件。若想侦测鼠标**拖动**操作，在 `mousedown` 事件的处理器中注册 `mousemove` 事件处理器。若要保证**鼠标移出起始元素后**仍能收到 `mousemove` 事件，需要捕获（拦截）鼠标事件，参见 §17.5。

经过 `mousedown` 和 `mouseup` 事件序列，浏览器触发 `click` 事件。任何文档元素都能触发 `click` 事件。连续两次点击，第二个 `click` 事件后会跟着一个 `dblclick` 事件。鼠标右击点击后浏览器一般会显示一个上下文菜单；显示菜单前一般会触发 `contextmenu` 事件；取消掉该事件能阻止显示菜单。

鼠标移入移出一个元素会触发 `mouseover` 和 `mouseout` 事件。这两个事件的事件对象有一个 `relatedTarget` 属性，指出参与转换的另一个元素。`mouseover` 和 `mouseout` 事件也冒泡。多数情况这样并不好，因为事件处理器触发时，不一定是鼠标移出了该元素还是事件传播到该元素。介于此，IE 支持不冒泡的版本： `mouseenter` 和 `mouseleave`。jQuery 对非 IE 浏览器模拟支持这两个事件。DOM Level 3 Events 规范已将其收入标准。

用户转动鼠标滚轮，浏览器触发 `mousewheel` 事件（在 Firefox 中是 `DOMMouseScroll` 事件）。事件对象会告诉你移动距离、方向等。The DOM Level 3 Events specification is standardizing a more general multidimensional **wheel event** that, if implemented, will supersede both `mousewheel` and `DOMMouseScroll`. §17.6 includes a mousewheel event example.

##### 17.1.1.4 按键事件

对 OS 或浏览器有意义的键盘快捷键常常被 OS 或浏览器吃掉，一般不会对 JavaScript 可见。键盘事件在有键盘焦点的文档元素上触发，并向上冒泡到 document 和 window。如果没有元素有焦点，事件直接在 document 上触发。事件对象的 `keyCode` 字段指出哪个键被按下或释放。此外事件对象还有 `altKey`、 `ctrlKey`、 `metaKey` 和 `shiftKey` 属性。

`keydown` 和 `keyup` 是底层事件。如果按下的是可打印的字符，在 down 之后 up 之前还会触发一个 `keypress` 事件。如果保持按下，可能在 `keyup` 事件前会有多次 `keypress` 事件{{为什么是keypress事件事件，应该是keydown事件吧}}。`keypress` 是高层事件，它的事件对象指出产生的字符，而不是按下的键。

`keyCode` 属性的值未被标准化。The DOM Level 3 Events specification, described below, attempts to addresses these interoperability problems, but has not yet been implemented. §17.9 includes an example of handling `keydown` events and §17.8 includes an example of processing `keypress` events.

#### 17.1.2 DOM 3 事件

DOM Level 3 Events 规范废弃了几个 Level 2 定义但几乎未被实现的事件。Browsers are still allowed to generate events like DOMActivate, DOMFocusIn, and DOMNodeInserted, but these are no longer required, and they are not documented in this book.

`wheel` 事件支持二维鼠标滚动事件。A handler for a `wheel` event receives an event object with all the usual mouse event properties, and also `deltaX`, `deltaY`, and `deltaZ` properties that report rotation around three different mouse wheel axes. See §17.6 for more on `mousewheel` events.

DOM Level 3 Events 不推荐使用 `keypress` 事件，推荐使用新事件 `textinput`。后者的事件对象有一个 `data` 属性，给出输入的字符串。`textinput` 事件可能被多个输入源触发，包括键盘、粘贴、拖放、输入法等。规范还定义了一个事件属性 `inputMethod` 和一些常量表示不同的输入（keyboard, paste or drop, handwriting or voice recognition等）。目前 Safari 和 Chrome 支持这个事件，但使用事件名 `textInput`。它的事件对象有 `data` 属性，没有 `inputMethod` 属性。§17.8 includes an example that makes use of this `textInput` event.

新的DOM标准还简化了 `keydown`、 `keyup`、 `keypress` 事件：添加了两个新的事件属性 `key` 和 `char`。二者的值都是字符串。如果键产生可打印的字符，`key` 和 `char` 等于产生的文本。对于控制键，`key` 属性将是一个字符串，如“Enter”, “Delete”, “Left”。The `char` property will either be null, or, for control keys like Tab that have a character code, it will be the string generated by the key. At the time of this writing, no browsers support these `key` and `char` properties, but Example 17-8 will use the `key` property if and when it is implemented.

#### 17.1.3 HTML5 事件

本节简要介绍 HTML5 事件。

`<audio>`和`<video>`有一堆相关的事件，负责通知网络事件、缓存状态、播放状态等：

    canplay loadeddata playing stalled
    canplaythrough loadedmetadata progress suspend
    durationchange loadstart ratechange timeupdate
    emptied pause seeked volumechange
    ended play seeking waiting

These media events are passed an ordinary event object with no special properties. The `target` property identifies the `<audio>` or `<video>` element, however, and that element has many relevant properties and methods. See §21.2 for more details on these elements, their properties, and their events.

HTML5 的拖拉 API 允许 Javascript 应用参与基于操作系统的拖拉操作，在 Web 应用和本地应用直接传递数据。其 API 定义了下面7个事件：

    dragstart drag dragend
    dragenter dragover dragleave
    drop

These **drag-and-drop** events are triggered with an event object like those sent with mouse events. One additional property, `dataTransfer`, holds a `DataTransfer` object that contains information about the data being transferred and the formats in which it is available. The HTML5 drag-and-drop API is explained and demonstrated in §17.7.

HTML5定义了一个历史管理机制（§22.2）。于此相关的事件是 `hashchange` 和 `popstate`。 这两个事件与 `load` 和 `unload` 一样在 `Window` 对象上触发。

HTML5 defines a lot of new features for HTML forms. In addition to standardizing the form input event described earlier, HTML5 also defines a form validation mechanism, which includes an invalid event fired on form elements that have failed validation. Browser vendors other than Opera have been slow to implement HTML5’s new form features and events, however, and this book does not cover them.

HTML5 includes support for offline web applications (see §20.4) that can be installed locally in an application cache so that they can run even when the browser is offline (as when a mobile device is out of network range). The two most important events associated with this are the `offline` and `online` events: they are triggered on the `Window` object whenever the browser loses or gains a network connection. A number of additional events are defined to provide notification of application download progress and application cache updates:

    cached checking downloading error
    noupdate obsolete progress updateready

A number of new web application APIs use a `message` event for asynchronous communication. The Cross-Document Messaging API (§22.3) allows scripts in a document from one server to exchange messages with scripts in a document from another server. This works around the limitations of the same-origin policy (§13.6.2) in a secure way. Each message that is sent triggers a `message` event on the Window of the receiving document. The event object passed to the handler includes a `data` property that holds the content of the message as well as  source and origin policies that identify the sender of the message. The `message` event is used in similar ways for communication with Web Workers (§22.4) and for network communication via Server-Sent Events (§18.3) and WebSockets (§22.9).

HTML5 and related standards define some events that are triggered on objects other than windows, documents, and document elements. Version 2 of the **XMLHttpRequest** specification, as well as the File API specification, define a series of events that track the progress of asynchronous I/O. They trigger events on an **XMLHttpRequest** or **FileReader** object. Each read operation begins with a `loadstart` event, followed by `progress` events and a `loadend` event. Additionally, each operation ends with a `load`, `error`, or `abort` event just before the final `loadend` event. See §18.1.4 and §22.6.5 for details.

Finally, HTML5 and related standards define a few miscellaneous event types. The Web Storage (§20.1) API defines a `storage` event (on the Window object) that provides notification of changes to stored data. HTML5 also standardizes the `beforeprint` and `afterprint` events that were originally introduced by Microsoft in IE. As their names imply, these events are triggered on a Window immediately before and immediately after its document is printed and provide an opportunity to add or remove content such as the date and time that the document was printed. (These events should not be used to change the presentation of a document for printing because CSS media types already exist for that purpose.)

#### xxx 17.1.4 触屏和手机事件

### xxx 17.2 注册事件处理器

### xxx 17.3 事件处理器调用

xxx 17.3.1 事件处理器参数
xxx 17.3.2 事件处理器上下文
xxx 17.3.3 事件处理器作用域（Scope）
xxx 17.3.4 处理器返回值
xxx 17.3.5 执行顺序
xxx 17.3.6 事件传播
xxx 17.3.7 事件取消

### （未）17.4 文档加载事件

### xxx 17.5 鼠标事件

### （未）17.6 鼠标滚轮事件

### （未）17.7 拖放事件

IE早期实现了一种拖放接口。HTML5后来规范了它。**但目前尚未有浏览器实现HTML5规范。因此本节讲得时IE的API，并兼顾HTML5。**

拖放是一种UI，用于在源与目标直接传递数据。有可能会跨应用交互数据。DnD是一种复杂的人机交互：

- 必须与底层OS结合，才能跨应用有效。
- 有移动、拷贝、链接三种传输操作。源或目标要有能力限制可用的操作。用户可以通过键盘修饰符等方式选择一种操作。
- 源要能够制定拖放的图标或图片。
- 提供事件通知，告诉源或目标拖放的进度。



### 17.8 文本事件

之前谈到键盘输入事件有三个：keydown、keyup和keypress。

The DOM Level 3 Events draft specification定义了更通用的textinput事件，但目前没有浏览器支持。Webkit支持一个类似的事件：textInput（大写I）。

textinput和textInput事件的事件对象都有一个`data`属性，值是用户输入的文本。

The event object passed with keypress events is more confusing. A keypress event represents a single character of input. The event object specifies that character as a numeric Unicode codepoint, and you must use `String.fromCharCode()` to convert it to a string.

In most browsers, the `keyCode` property of the event object specifies the codepoint of the input character. For historical reasons, however, Firefox uses the `charCode` property instead. Most browser only fire keypress events when a printable character is generated. Firefox, however, also fires “keypress” for nonprinting characters. To detect this case (so you can ignore the nonprinting characters), you can look for an event object with a `charCode` property that is defined but set to 0.

取消textinput, textInput和keypress事件可以阻止字符被输入，即过滤输入。

    // This is the keypress and textInput handler that filters the user's input
    function filter(event) {
        // Get the event object and the target element target
        var e = event || window.event; // Standard or IE model
        var target = e.target || e.srcElement; // Standard or IE model
        var text = null; // The text that was entered
        // Get the character or text that was entered
        if (e.type === "textinput" || e.type === "textInput") text = e.data;
        else { // This was a legacy keypress event
            // Firefox uses charCode for printable key press events
            var code = e.charCode || e.keyCode;
            // If this keystroke is a function key of any kind, do not filter it
            if (code < 32 || // ASCII control character
            	e.charCode == 0 || // Function key (Firefox only)
            	e.ctrlKey || e.altKey) // Modifier key held down
            	return; // Don't filter this event
            // Convert character code into a string
            var text = String.fromCharCode(code);
        }
        // Now look up information we need from this input element
        var allowed = target.getAttribute("data-allowed-chars"); // Legal chars
        var messageid = target.getAttribute("data-messageid"); // Message id
        if (messageid) // If there is a message id, get the element
        	var messageElement = document.getElementById(messageid);
        // Loop through the characters of the input text
        for(var i = 0; i < text.length; i++) {
            var c = text.charAt(i);
            if (allowed.indexOf(c) == -1) { // Is this a disallowed character?
                // Display the message element, if there is one
                if (messageElement) messageElement.style.visibility = "visible";
                // Cancel the default action so the text isn't inserted
                if (e.preventDefault) e.preventDefault();
                if (e.returnValue) e.returnValue = false;
                return false;
            }
        }
        // If all the characters were legal, hide the message if there is one.
        if (messageElement) messageElement.style.visibility = "hidden";
    }

keypress和textinput事件在文本插入到文档元素前触发，因此可以实现过滤。**input事件**在文本插入到元素后才触发。这个事件不能被取消。If you wanted to ensure that any text entered into an input field was in uppercase, for example, you might use the input event like this:

	SURNAME: <input type="text" oninput="this.value = this.value.toUpperCase();">

input事件是HTML 5标准事件，IE之外的所有现代浏览器都支持。IE可以利用非标准的propertychange事件侦测文本输入元素`value`属性的变化。Example 17-7 shows how you might force all input to uppercase in a cross-platform way.

    function forceToUpperCase(element) {
        if (typeof element === "string")
        	element = document.getElementById(element);
        element.oninput = upcase;
        element.onpropertychange = upcaseOnPropertyChange;
        // Easy case: the handler for the input event
        function upcase(event) { this.value = this.value.toUpperCase(); }
        // Hard case: the handler for the propertychange event
        function upcaseOnPropertyChange(event) {
            var e = event || window.event;
            // If the value property changed
            if (e.propertyName === "value") {
                // Remove onpropertychange handler to avoid recursion
                this.onpropertychange = null;
                // Change the value to all uppercase
                this.value = this.value.toUpperCase();
                // And restore the original propertychange handler
                this.onpropertychange = upcaseOnPropertyChange;
            }
        }
    }

### （未）17.9 键盘事件