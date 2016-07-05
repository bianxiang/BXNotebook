[toc]

## 大小与位置

相关规范：

- CSSOM-View Module (see http://www.w3.org/TR/cssom-view/)

每个HTML元素都有以下属性：

	offsetWidth   clientWidth   scrollWidth
	offsetHeight  clientHeight  scrollHeight
	offsetLeft    clientLeft    scrollLeft
	offsetTop     clientTop     scrollTop
	offsetParent

### 文档坐标与视口坐标

X轴向右，Y轴向下。坐标系统原点可以有两个：可以相对于文档的左上角，或文档所在的视口的左上角。For documents displayed in frames, the viewport is the `<iframe>` element that defines the frame. （有时视口坐标也被称为窗口（window）坐标）。

通过CSS指定元素位置时使用的是文档坐标。但查询元素位置最简单的方法返回的是视口坐标。在鼠标事件中，鼠标指针的坐标是视口坐标。

### 大小

#### 获取元素在视口中的大小（Border + Padding + Content）

`getBoundingClientRect()` 返回的对象还有 `height` 和 `width` 属性。大小包括内容、内补和边框。

```js
var div = document.querySelector('div').getBoundingClientRect();
console.log(div.height, div.width);
```

`offsetHeight` 和 `offsetWidth` 属性返回相同值。

W3C标准和多数浏览器，返回的对象包含 `width` 和 `height` 属性，但原来的IE实现不包含。为了可移植，兼容方法是：

```js
var box = e.getBoundingClientRect();
var w = box.width || (box.right - box.left);
var h = box.height || (box.bottom - box.top);
```

内联元素，如`<span>`、`<code>`，可能跨多行，因此可能包含多个矩形。如第一行末尾的矩形和第二个开头的矩形。调用内联元素的 `getBoundingClientRect()`，返回的是包围所有这些矩形的矩形。例如，对于换行的内联元素，bounding 矩形包含两个行的全部宽度。

If you want to query the individual rectangles of inline elements, call the `getClientRects()` method to obtain a read-only array-like object whose elements are rectangle objects like those returned by `getBoundingClientRect()`.

#### `clientWidth` 和 `clientHeight`；`clientLeft` 和 `clientTop`

`clientWidth` 和 `clientHeight` 返回元素内容加内补的大小，不包含边框。

如果浏览器在 padding 和边框之间添加了滚动条， `clientWidth`、 `clientHeight` 的值不包含滚动条。对于内联元素 `clientWidth`、 `clientHeight` 总是返回 `0`。

`clientLeft` 和 `clientTop` 属性不是特别有用：它们返回 padding 外缘和 border 外缘之间的距离，多数情况下就是左边和上边边框的宽度。如果元素有滚动条且滚动条在左面或上边（很少见）`clientLeft`、`clientTop` 也包含滚动条宽度。For inline elements, `clientLeft` and `clientTop` are always 0.

#### `scrollWidth` 和 `scrollHeight`

`Element.scrollWidth` 是只读的，返回内容宽度或元素自己宽度中较大的一个，单位像素。若元素比内容区域更宽（如有滚动条），`scrollWidth` 比 `clientWidth` 更大。

该属性会被取整。如果想要小数，使用 `element.getBoundingClientRect()`。

```
    <!DOCTYPE html>
    <html lang="en">
    <head>
    <style>
    *{margin:0;padding:0;}
    div{height:100px;width:100px; overflow:auto;}
    p{height:1000px;width:1000px;background-color:red;}
    </style>
    </head>
    <body>
    <div><p></p></div>
    <script>
    var div = document.querySelector('div');
    console.log(div.scrollHeight, div.scrollWidth); // logs '1000 1000'
    </script>
    </body>
    </html>
```

若一个节点在一个可滚动的区域内，且节点小于可滚动区域的视口，则要获取节点的宽度和高度，不要使用 `scrollHeight` 和 `scrollWidth`，**它们给出的是视口的大小**。应该使用 `clientHeight` 和 `clientWidth`。{{这段与实验不符或意思有误}}

#### 视口大小

与滚动条位置一样，IE8不支持标准做法，IE中的可行方法取决于浏览器处于quirks模式还是标准模式。下面给出获取视口大小的通用方法：

```js
// Return the viewport size as w and h properties of an object
function getViewportSize(w) {
    // Use the specified window or the current window if no argument
    w = w || window;
    // This works for all browsers except IE8 and before
    if (w.innerWidth != null) return {w: w.innerWidth, h:w.innerHeight};
    // For IE (or any browser) in Standards mode
    var d = w.document;
    if (document.compatMode == "CSS1Compat")
        return { w: d.documentElement.clientWidth,
            h: d.documentElement.clientHeight };
    // For browsers in Quirks mode
    return { w: d.body.clientWidth, h: d.body.clientWidth };
}
```

### 位置

#### 视口坐标

`getBoundingClientRect()` 获取元素的**外边**相对于视口的偏移。返回一个对象。`left` 和 `right` 的元素的外边相对于视口左边的偏移。`top` 和 `bottom` 相对于视口上边的变异。

```js
var divEdges = document.querySelector('div').getBoundingClientRect();
console.log(divEdges.top, divEdges.right, divEdges.bottom, divEdges.left);
```

#### 相对于定位祖先

`offsetTop` 和 `offsetLeft` 给出元素的最外边，相对于其 `offsetParent` 的内边偏移。`offsetParent` 是最近的定位祖先。若在寻找的过程中遇到 `<td>`、 `<th>` 或 `<table>` 元素，即使 `position` 为 `static`，仍作为 `offsetParent`。

如果 `offsetParent` 为 null，坐标是文档坐标。

当 `offsetParent` 是 `<body>` 且 `<body>` 或 `<html>` 有外补白、内补白或边框时，多数浏览器会把相对于“边框外”改成“边框内”。

`offsetParent`, `offsetTop`, and `offsetLeft` are extensions to the `HTMLElement` object.

#### 滚动位置

`scrollTop` 和 `scrollLeft` 属性是可读写的。分别给出上边和左边不可见部分的大小。

#### 主滚动条位置

```js
// Return the current scrollbar offsets as the x and y properties of an object
function getScrollOffsets(w) {
    // Use the specified window or the current window if no argument
    w = w || window;
    // 除了IE8
    if (w.pageXOffset != null) return {x: w.pageXOffset, y:w.pageYOffset};
    // For IE (or any browser) in Standards mode
    var d = w.document;
    if (document.compatMode == "CSS1Compat")
        return {x:d.documentElement.scrollLeft, y:d.documentElement.scrollTop};
    // For browsers in Quirks mode
    return { x: d.body.scrollLeft, y: d.body.scrollTop };
}
```

### 某个位置上是哪个元素

要查询视口上处于某个位置的（最上面的）元素，可以利用 Document 对象的 `elementFromPoint()` 方法，传入一个**视口坐标**，该方法返回一个 Element 对象。
如果指定点超出视口，`elementFromPoint()` 将返回 `null`，即使这个点转换成文档坐标后是有效的。

`elementFromPoint()` 看似很有用，如判断当前鼠标经过的元素。但实际鼠标事件已经通过 `target` 属性包含了这个信息。 `elementFromPoint()` 很少在实际中使用。

```js
console.log(document.elementFromPoint(50, 50));
```

### 利用 `scrollIntoView()` 将元素滚进视图

有时只是向让某个元素可见，最简单的方法是在此元素上调用 `scrollIntoView()`。默认尝试令元素上边接近视口上边。如果传入一个 `false`。则尝试令元素下边接近视口下边。如果需要浏览器还会水平滚动视口让元素可见。

The behavior of `scrollIntoView()` is similar to what the browser does when you set `window.location.hash` to the name of a named anchor (an `<a name="">` element).

### 精确滚动

> 注意 `scrollTo()` 和 `scrollBy()` 都是 `window` 对象的方法。

除了设置 `scrollTop` 或 `scrollLeft`，还可以使用 `scrollTo()`（或 `scroll()`）。传入一个**文档坐标**，令这个点滚到视口的左上角。如果位置太接近底边或右边，则尽可能滚的远。

例子，滚到文档底部：

```js
// 获取文档和视口的高度
var documentHeight = document.documentElement.offsetHeight;
var viewportHeight = window.innerHeight;
window.scrollTo(0, documentHeight - viewportHeight);
```

`scrollBy()` 则是增量的。例子：

```js
// 每200毫秒滚10像素
javascript:void setInterval(function() {scrollBy(0, 10)}, 200);
```