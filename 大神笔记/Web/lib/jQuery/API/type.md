[toc]

## 类型

### Anything

jQuery文档中，Anything表示这里可以是任何类型。

### htmlString

本文的中，htmlString值得是表示HTML元素的字符串。向`jQuery()`传入一个字符串，如果该字符串以`<tag ... >`开头，则开头到最后一个`>`的部分则会被当作HTML解析。但jQuery 1.9之前，只要字符串中包含`<tag ... >`就会被当成HTML解析。

When a string as passed as an argument to a manipulation method such as `.append()`, it is always considered to be HTML since jQuery's other common interpretation of a string (CSS selectors) does not apply in those contexts.

若要显式的将字符串解析为HTML，调用`$.parseHTML()`（jQuery 1.8+）。

```js
    // Appends <b>hello</b>:
    $( "<b>hello</b>" ).appendTo( "body" );

    // Appends <b>hello</b>。bye会被忽略
    $( "<b>hello</b>bye" ).appendTo( "body" );

    // 语法错误！bye<b>hello</b>不可识别
    $( "bye<b>hello</b>" ).appendTo( "body" );

    // Appends bye<b>hello</b>:
    $( $.parseHTML( "bye<b>hello</b>" ) ).appendTo( "body" );

    // Appends <b>hello</b>wait<b>bye</b>:
    $( "<b>hello</b>wait<b>bye</b>" ).appendTo( "body" );
```

### 数字

Due to the implementation of numbers as double-precision values, the following result is not an error:

```js
	0.1 + 0.2 // 0.30000000000000004
```

### PlainObject

PlainObject对象就是简单的对象，不包括数组，浏览器document等。`jQuery.isPlainObject()`可以判断对象是否是普通对象：

```js
var a = [];
var d = document;
var o = {};

typeof a; // object
typeof d; // object
typeof o; // object

jQuery.isPlainObject( a ); // false
jQuery.isPlainObject( d ); // false
jQuery.isPlainObject( o ); // true
```

### 事件

jQuery的事件系统根据W3C标准对事件对象进行了规范。事件对象保证会被传给事件处理器（不需要检查`window.event`）。它规范了target, relatedTarget, which, metaKey和pageX/Y属性，提供`stopPropagation()`和`preventDefault()`两个方法。这些属性的解释参见[Event object](http://api.jquery.com/category/events/event-object/)。

DOM标准事件有：blur, focus, load, resize, scroll, unload, beforeunload, click, dblclick, mousedown, mouseup, mousemove, mouseover, mouseout, mouseenter, mouseleave, change, select, submit, keydown, keypress, and keyup。Since the DOM event names have predefined meanings for some elements, using them for other purposes is not recommended. jQuery's event model can trigger an event by any name on an element, and it is propagated up the DOM tree to which that element belongs, if any.

### DOM元素

调用jQuery的`.each()`方法，或对jQuery集合调用事件方法，回调方法的上下文——`this`——是一个DOM元素。一些属性在各个浏览器上都是相同的，此时可以直接使用这个DOM元素：

```js
$( "input[type='text']" ).on( "blur", function() {
  if( !this.value ) {
    alert( "Please enter some text!" );
  }
});
```

### jQuery对象

一个jQuery对象内含一组DOM元素。jQuery对象本身很像一个数组（但实际不是）；它有`length`属性；其中的元素可以通过下标访问（[0] ... [length-1]）。其他数组的方法是jQuery对象不具备的。

多数时候通过`jQuery()`函数创建一个jQuery对象。很多jQuery方法返回jQuery对象，以实现级联调用的风格。

```js
	$( "p" ).css( "color", "red" ).find( ".special" ).css( "color", "green" );
```

如果jQuery方法会改变jQuery对象中的元素，如`.filter()`或`.find()`，这些方法返回的是新的jQuery对象。要返回之前的jQuery对象，使用`.end()`方法。

jQuery对象可能为空：指内含元素为空（`length === 0`）。

### jqXHR

从jQuery 1.5开始，`$.ajax()`方法返回`jqXHR`对象，它是`XMLHTTPRequest`的超集。For more information, see the [jqXHR section of the $.ajax entry](http://api.jquery.com/jQuery.ajax/#jqXHR).



