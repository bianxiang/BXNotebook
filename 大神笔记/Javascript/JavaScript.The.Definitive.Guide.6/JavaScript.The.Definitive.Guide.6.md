[toc]

## 13 JavaScript in Web Browsers

### 13.1 客户端JavaScript

Window对象是客户端Javascript API的主要入口。It represents a web browser window or frame, 可以通过标识符`window`获取。Window对象有一些属性，如`location`：

	// Set the location property to navigate to a new web page
    window.location = "http://www.oreilly.com/";

Window对象还还定义有一些方法，如`alert()`和`setTimeout()`：

    // Wait 2 seconds and then say hello
    setTimeout(function() { alert("hello world"); }, 2000);

注意到上述代码未显式使用`window`。因为Window对象也是全局对象，即它的属性和方法也是全局变量和函数。Window对象有一个`window`属性，总是指向自己。

Window对象最重要的属性`document`: it refers to a Document object that represents the content displayed in the window. Document对象有一些方法如`getElementById()`：

    // Find the element with id="timestamp"
    var timestamp = document.getElementById("timestamp");

`getElementById()`返回一个Element对象。每个Element对象都有`style`和`className`属性。设置这些CSS相关的属性以改变文档的显示：

    timestamp.style.backgroundColor = "yellow";
    timestamp.className = "highlight";

Window、Document和Element还有一些重要属性，是时间处理器属性。

	timestamp.onclick = function() { this.innerHTML = new Date().toString(); }

One of the most important event handlers is the `onload` handler of the Window object. It is triggered when the content of the document displayed in the window is stable and ready to be manipulated. JavaScript code is commonly wrapped within an `onload` event handler.

    <script>
        // Don't do anything until the entire document has loaded
        window.onload = function() {
            // Find all container elements with class "reveal"
            var elements = document.getElementsByClassName("reveal");
            for(var i = 0; i < elements.length; i++) { // For each one...
                var elt = elements[i];
                // Find the "handle" element with the container
                var title = elt.getElementsByClassName("handle")[0];
                // When that element is clicked, reveal the rest of the content
                title.onclick = function() {
                    if (elt.className == "reveal") elt.className = "revealed";
                    else if (elt.className == "revealed") elt.className = "reveal";
                }
            }
        };
    </script>

### （未）13.2 Embedding JavaScript in HTML

### （未）13.3 Execution of JavaScript Programs

### （未）13.4 Compatibility and Interoperability

### （未）13.5 Accessibility

### （未）13.6  Security

## 14 Window 对象

### 14.1 定时器

`setTimeout()`的返回值可以传入`clearTimeout()`取消执行。`setInterval()`的返回值可以传入`clearInterval()`以取消执行。

历史上，如果第一个参数是字符串，字符串会被`eval()`后执行。

HTML5规范（除了IE）允许提供更多的参数：前两个参数之外的参数会被作为附加参数传给被调用的函数。

If you call `setTimeout()` with a time of 0 ms, the function you specify is not invoked right away. Instead, it is placed on a queue to be invoked “**as soon as possible**” after any currently pending event handlers finish running.

### 14.2 浏览器位置和导航

Window对象和Document对象的`location`属性指向一个Location对象，表格当前显示的URL。

	window.location === document.location // always true

Document对象还有一个`URL`属性，是一个静态的字符串，保存文档第一次加载时的URL。如果在文档内的不同片段（如“#table-of-contents”）导航，Location对象会更新但`document.URL`不会。

#### 14.2.1 解析URL

Location的`href`属性是一个字符串，包含完整URL。Location对象的`toString()`方法返回`href`属性的值 。

其他属性`protocol`, `host`, `hostname`, `port`, `pathname`, `search`, and `hash`—specify the various individual parts of the URL. They are also supported by Link objects (created by `<a>` and `<area>` elements in HTML documents).  See the Location and Link entries in Part IV for further details.

The `hash` property returns the “fragment identifier” portion of the URL, if there is one: a hash mark (#) followed by an element ID. The `search` property is similar. It returns the portion of the URL that starts with a question mark: often some sort of query string. 例子，解析URL，抽出查询参数：（注意使用`decodeURIComponent`）

    function urlArgs() {
        var args = {}; // Start with an empty object
        var query = location.search.substring(1); // minus '?'
        var pairs = query.split("&"); // Split at ampersands
        for(var i = 0; i < pairs.length; i++) { // For each fragment
            var pos = pairs[i].indexOf('='); // Look for "name=value"
            if (pos == -1) continue; // If not found, skip it
            var name = pairs[i].substring(0,pos); // Extract the name
            var value = pairs[i].substring(pos+1); // Extract the value
            value = decodeURIComponent(value); // Decode the value
            args[name] = value; // Store as a property
        }
        return args; // Return the parsed arguments
    }

#### 14.2.2 加载新文档

Location对象的`assign()`方法让窗口加载并显示指定URL。`replace()`功能类似，只是当前页面不会进入浏览器历史。

    if (!XMLHttpRequest) location.replace("staticpage.html");

Location对象的`reload()`方法重新加载文档。

传统上，切换页面更直接的方法是直接给`location`赋值：

	location = "http://www.oreilly.com";

也可以使用相对URL（相当于当前URL）：

	location = "page2.html"; // Load the next page

如果只有片段标识符，不会导致页面重载，只是让页面滚动到指定节。`#top`是一个特殊标识：如果没有这个ID，浏览器将回到文档开头：

	location = "#top"; // Jump to the top of the document

The URL decomposition properties of the Location object are writable, 改变它们将导致URL并会导致浏览器载入新文档（`hash`除外，导航到指定节）：

	location.search = "?page=" + (pagenum+1); // load the next page

### 14.3 浏览历史

Window对象的`history`属性指向History对象。History的`length`属性给出历史列表中条目数。处于安全目的，目前不能访问历史列表中项的URL。

History对象的`back()`和`forward()`方法向后向前。`go()`方法取一个整数向前或向后指定数量的页面：

	history.go(-2); // Go back 2, like clicking the Back button twice

If a window contains child windows (such as `<iframe>`elements—see §14.8.2), the browsing histories of the child windows are chronologically interleaved with the history of the main window. This means that calling `history.back()`(for example) on the main window may cause one of the child windows to navigate back to a previously displayed document but leave the main window in its current state.

现代Web一你敢用会动态的改变页面内容，不用加载新文档。这种应用也想让用户可以通过后退和前进按钮在动态创建的状态间转换。HTML5 standardizes two techniques for doing this, and they are described in §22.2.

History management before HTML5 is a more complex problem. An application that
manages its own history must be able to create a new entry in the window browsing history, associate its state information with that history entry, determine when the user has used the Back button to move to a different history entry, get the state information associated with that entry, and re-create the previous state of the application. One approach uses a hidden `<iframe>`to save state information and create entries in the browser’s history. In order to create a new history entry, you dynamically write a new document into this hidden frame using the open() and write() methods of the Document object (see §15.10.2). The document content should include whatever state information is required to re-create the application state. When the user clicks the Back button, the content of the hidden frame will change. Before HTML5, no events are generated to notify you of this change, however, so in order to detect that the user has clicked Back you might use `setInterval()`(§14.1) to check the hidden frame two or three times a second to see if it has changed. 当然有现成的库解决这个问题。

### 14.4 浏览器和屏幕信息

本节描述Window对象的`navigator`和`screen`属性，分别指向Navigator和Screen对象。

#### （未）14.4.1 The Navigator Object

#### 14.4.2 Screen对象

`width`和`height`属性指定显示器大小，单位像素。`availWidth`和`availHeight`属性指出可用大小，排除了任务栏等大小{{但貌似并未去掉浏览器状态栏、菜单栏、书签栏等部分}}。The `colorDepth` property specifies the bits-per-pixel value of the screen. Typical values are 16, 24, and 32.

`window.screen`属性和`Screen`对象都不是标准的但被广泛实现。

### （未）14.5 对话框

### 14.6 错误处理

The `onerror` property of a Window object is an event handler that is invoked when an uncaught exception propagates all the way up the call stack and an error message is about to be displayed in the browser’s JavaScript console. If you assign a function to this property, the function is invoked whenever a JavaScript error occurs in that window.

由于历史原因，`onerror`的回调函数有三个字符串参数，分别是：错误描述、出错的Javascript文件的URL，行号。(Other client-side objects have  onerrorhandlers to handle different error conditions, but these are all regular event handlers that are passed a single event object.)

返回值很关键，如果返回false，告诉浏览器错误已被处理，不需后续操作，即浏览器不必再打印错误消息。Unfortunately, for historical reasons, an error handler in Firefox must return **true** to indicate that it has handled the error.

`onerror`是早期时代的产物——没有**try/catch**时的解决方案，现代代码很少需要使用。

### （未）14.7 Document Elements As Window Properties

### （未）14.8 Multiple Windows and Frames


## 18. HTTP

xxx 18.1 XMLHttpRequest
xxx 18.1.1 指定请求
xxx 18.1.2 接收响应
xxx 18.1.2.2 解码响应
xxx 18.1.3 对请求体进行编码
xxx 18.1.4 HTTP进度事件
xxx 18.1.5 放弃请求与超时

##### （未）18.1.2.1 同步响应

##### （未）18.1.3.3 XML-encoded requests

#### （未）18.1.6 跨域HTTP请求

### （未）18.2 JSONP

### （未）18.3 Comet with Server-Sent Events

## 19. jQuery库

Part IV includes a jQuery quick reference.

### 19.1 jQuery基础

jQuery只向全局命名空间添加了两个名字：`jQuery()`函数和`$`变量。

	var divs = $("div");

返回值divs是一个jQuery对象。

#### 19.1.1 jQuery()函数

The second way to invoke $() is to pass it an Element or Document or Window object. Called like this, it simply wraps the element, document, or window in a jQuery object and returns that object.

第一个参数传入HTML片段。Or you can pass an object as the second argument. If you do this, the object properties are assumed to specify the names and values of HTML attributes to be set on the object. 但如果属性名是“css”, “html”, “text”, “width”, “height”, “offset”, “val”或“data”，或者与jQuery的事件处理器注册方法同名，jQuery will invoke the method of the same name on the newly created element and pass the property value to it. For example:

    var img = $("<img/>", // Create a new <img> element
    {   src:url, // with this HTML attribute,
    	css: {borderWidth:5}, // this CSS style,
    	click: handleClick // and this event handler.
    });

#### 19.1.2 查询与查询结果

In addition to the length property, jQuery objects have three other properties that are sometimes of interest. The selector property is the selector string (if any) that was used when the jQuery object was created. The context property is the context object that was passed as the second argument to $(), or the Document object otherwise. Finally, all jQuery objects have a property named jquery, and testing for the existence of this property is a simple way to distinguish jQuery objects from other array-like objects. The value of the jquery property is the jQuery version number as a string.

### （未）19.2 jQuery Getters and Setters

#### 19.2.6 读写元素的几何

读写元素位置使用`offset()`方法。该方法使用的位置相对于文档。返回的对象包含left和top两个属性表示横纵坐标。传入含有这两个属性的对象可以设置元素位置。It sets the CSS position attribute as necessary to make elements positionable:

    var elt = $("#sprite"); // The element we want to move
    var position = elt.offset(); // Get its current position
    position.top += 100; // Change its Y coordinate
    elt.offset(position); // Set the new position
    // Move all <h1> elements to the right by a distance that depends on their
    // position in the document
    $("h1").offset(function(index, curpos) {
		return {left: curpos.left + 25*index, top:curpos.top};
    });

`position()`方法是只读的，返回的位置相对于offset parent而不是文档。§15.8.5讲到每个元素都有一个`offsetParent`属性，即相对定位的祖先。Positioned elements always serve as the offset parents for their descendants, but some browsers also make other elements, such as table cells, into offset parents. jQuery only considers positioned elements to be offset parents, and the `offsetParent()` method of a jQuery object maps each element to the nearest positioned ancestor element or to the `<body>` element. Note the unfortunate naming mismatch for these methods: `offset()` returns the absolute position of an element, in document coordinates. And `position()` returns the offset of an element relative to its `offsetParent()`.

有三组查询元素宽度高度的方法。`width()`和`height()`返回内盒宽高（不包含padding, borders, margins）。`innerWidth(`)和`innerHeight()`返回的宽度和高度中含padding。`outerWidth()`和`outerHeight()`返回的大小中含有padding和边框。调用这两个方法时传入true，则同时包含元素的margins。

    var body = $("body");
    var contentWidth = body.width();
    var paddingWidth = body.innerWidth();
    var borderWidth = body.outerWidth();
    var marginWidth = body.outerWidth(true);
    var padding = paddingWidth-contentWidth; // sum of left and right padding
    var borders = borderWidth-paddingWidth; // sum of left and right border widths
    var margins = marginWidth-borderWidth; // sum of left and right margins

如果jQuery对象中第一个元素是Window或Document，`width()`和`height()`返回窗口的视口大小或文档大小。其他方法只能对元素调用。`width()`和`height()`还可以设置元素的宽高。若只传入数字，单位取像素。如果传入字符串，会被当成CSS宽度高度值，可以任意单位。Finally, as with other setters, you can pass a function that will be called to compute the width or height.

`width()`和`height()`返回的大小，总是只包含内容盒子，不包含padding, borders, margins。但用于修改元素尺寸时，它们直接设置CSS的`width`和`height`属性。如果元素使用了`box-sizing: border-box`，则设置的值中含边框和内补。

`scrollTop()`和`scrollLeft()`用于读取、设置元素滚动条位置。两个方法能用于Window对象或文档元素。如果对Document调用，则返回或设置包含文档的Window对象的滚动条位置。Unlike with other setters, you cannot pass a function to scrollTop() or scrollLeft().

We can use scrollTop() as a getter and a setter, along with the height() method to define a method that scrolls the window up or down by the number of pages you specify:

    // Scroll the window by n pages. n can be fractional or negative
    function page(n) {
        var w = $(window); // Wrap the window in a jQuery object
        var pagesize = w.height(); // Get the size of a page
        var current = w.scrollTop(); // Get the current scrollbar position
        w.scrollTop(current + n*pagesize); // Set new scrollbar position
    }

### 19.4 事件处理

## 22. HTML5 API

### xxx 22.6 Blobs

### xxx 22.7 Filesystem


