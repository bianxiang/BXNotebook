[toc]

## 事件

参考：

- https://mobiforge.com/design-development/html5-mobile-web-touch-events
- http://www.html5rocks.com/en/mobile/touchandmouse/
- http://www.html5rocks.com/zh/mobile/touch/

### 触屏事件

多数时候，触屏事件映射到传统事件，如`click`和`scroll`。本节简要描述`gesture`和`touch`事件（iPhone和iPad的Safari产生），以及`orientationchange`事件（用户旋转设备）。目前这些事件还没有规范。

在Safari中用两个手指缩放或旋转产生`gesture`事件。手势开始和结束时分别产生 `gesturestart`和`gestureend`事件。在这两个事件之间，是一系列`gesturechange`事件，追踪手势的进度。这些事件的事件对象带有`scale`和`rotation`属性。`scale`的值是两个手指当前距离与初始距离之比。“捏紧”手势的`scale`小于1.0，“捏开”手势的`scale`大于1.0。`rotation`是手指旋转的角度。It is reported in degrees, with positive values indicating clockwise rotation.

手势事件是高层事件，只有当侦测到手势才发生。若想实现自定义的手势，可以监听底层的touch事件。手指触摸屏幕触发`touchstart`事件，手指移动触发`touchmove`事件。手指抬起，触发`touchend`事件。

此外规范提供了另外三个触摸事件，但测试的浏览器都不支持它们：

- `touchenter`：移动的手指进入一个 DOM 元素。
- `touchleave`：移动手指离开一个 DOM 元素。
- `touchcancel`：触摸中断（实现规范）。

The `orientationchanged` event is triggered on the Window object by devices that allow the user to rotate the screen from portrait to landscape mode. The object passed with an `orientationchanged` event is not useful itself. In mobile Safari, however, the `orientation` property of the `Window` object gives the current orientation as one of the numbers 0, 90, 180, or -90.

#### 触屏事件

先看几个相关的API。

`Touch`对象描述一个触摸点，有以下属性：

- identifier：唯一的数字标示。多点触控时，追踪按下的是哪个点。
- target：事件目标，一个DOM元素，即使触摸已经移出了该元素。
- screenX、screenY：相对于屏幕的水平和垂直坐标
- clientX、clientY：相对于视口的水平和垂直坐标，excluding scroll offset
- pageX、pageY：相对于页面的水平和垂直坐标，including scroll offset
- 半径坐标和 rotationAngle：画出与手指形状类似的椭圆形。

下面是主要的touch事件：

- touchstart
- touchmove
- touchend
- touchcancel：例如，触摸移出了可触摸的区域

每个事件有三个TouchList：

- `touches`：all current touches on the screen
- `targetTouches`：all current touches that started on the target element of the event
- `changedTouches`：all touches involved in the current event

#### 最佳实践与注意事项

**禁止缩放**

浏览器对多点触控有一些默认的行为，如滚动和缩放。若不想触发这些默认行为，例如，要停用缩放，应使用下面的元标记设置视口，这样用户就无法扩展视口了：

	<meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">

**禁止滚动**

某些移动设备对于 touchmove 有默认的行为，例如经典的 iOS overscroll 特效，会在滚动超出内容的界限时引发视图反弹。这种做法在很多的多点触控应用程序中会带来混乱，不过很容易就能停用：

```js
document.body.addEventListener('touchmove', function(event) {
	event.preventDefault();
}, false);
```

**小心渲染**

当涉及复杂的多指手势时，可能要一次处理大量事件，此时应仔细考虑如何响应。可以在有触摸输入的时候就立刻进行绘制：

```js
canvas.addEventListener('touchmove', function(event) {
  renderTouches(event.touches);
}, false);
```

另一种方式是，跟踪所有手指，然后在一个循环中进行渲染，这样获得的性能要好得多：

```js
var touches = []
canvas.addEventListener('touchmove', function(event) {
  touches = event.touches;
}, false);

// Setup a 60fps timer
timer = setInterval(function() {
  renderTouches(touches);
}, 15);
```

> 提示：setInterval 不太适于动画，因为它没有考虑到浏览器自身的渲染循环。现代的桌面浏览器提供了 `requestAnimationFrame`，从性能和电池寿命方面考虑，这是一个更好的选择。因此只要移动浏览器支持这一方式，就应优先选择。

#### 触摸事件与鼠标事件

Windows 8 IE10引入了新的模型，叫Pointer事件。指针事件是鼠标和触摸的统一，还包含其他输入如笔。W3C已开始标准化。目前有一些库[PointerEvents](https://github.com/toolkitchen/PointerEvents) 和[Hand.js](http://blogs.msdn.com/b/eternalcoding/archive/2013/01/16/hand-js-a-polyfill-for-supporting-pointer-events-on-every-browser.aspx)，用于模拟指针事件。
为了良好的体验，还是要分别处理鼠标和触摸事件。但也有很多场景，统一的事件就足够了。

目前，建议最好还是同时支持两种交互模型。同时支持有很多挑战，本文就是为解决它们。

**点击和按压的顺序问题**

第一个问题是，触摸接口会模拟鼠标点击。这是因为需要适应大量只支持鼠标的应用。但触摸模拟点击有一些问题。

首先，若用户点击屏幕，触摸和点击事件都会触发：

    touchstart
    touchmove
    touchend
    mouseover
    mousemove
    mousedown
    mouseup
    click

需要注意的是，处理了触摸事件，就不要再处理对应的鼠标和点击事件了。即一个重要规则是：在触摸事件的处理器中调用`preventDefault()`，就不会产生对应的鼠标事件。
但这样，同时也会阻止其他浏览器默认的行为（如滚动）。所以，要门处理并`preventDefault()`，要么干脆不要添加处理器。

第二，若应用未针对移动交互设计，用户触摸一个元素，从`touchstart`事件到后续的鼠标事件（`mousedown`）有300毫秒延迟！这个延迟是为确认用户是否在做其他动作，如缩放。

消除延迟最简单的方法是告诉浏览器页面不会被缩放：

	<meta name="viewport" content="width=device-width,user-scalable=no">

**触摸不会触发鼠标移动事件**

触摸模拟的鼠标事件中并不包括`mousemove`事件。

**Touchmove 和 MouseMove 并不是一回事**

一个陷阱是让 touchmove 和 mousemove 处理器共用一套代码。这两个事件的行为是接近的，但有一些不同。触摸事件的目标总是触摸开始的元素。打死你鼠标事件的目标是鼠标指针下的当前元素。

一个例子是移除一个用户已开始触摸的元素。你的处理器将停止接收事件，因为事件的目标已经不存在了。It'll look like the user is holding their finger in one place even though they may have moved and eventually removed it.

一个解决方法是，不要注册静态的 touchend/touchmove 处理器，而是在 touchstart 事件处理器中添加 touchmove/touchend/touchcancel 处理器；在end/cancel中移除。This way you'll continue to receive events for the touch even if the target element is moved/removed.

**Keep Touch Handlers Contained, or They’ll Jank Your Scroll**

It’s also important to keep touch handlers confined only to the elements where you need them; touch elements can be very high-bandwidth, so it’s important to avoid touch handlers on scrolling elements (as your processing may interfere with browser optimizations for fast jank-free touch scrolling - modern browsers try to scroll on a GPU thread, but this is impossible if they have to check with javascript first to see if each touch event is going to be handled by the app). You can check out [an example](http://www.rbyers.net/janky-touch-scroll.html) of this behavior.

One piece of guidance to follow to avoid this problem is to make sure that if you are only handling touch events in a small portion of your UI, you only attach touch handlers there (not, e.g., on the `<body>` of the page); in short, limit the scope of your touch handlers as much as possible.

#### 开发工具

对于单点触摸，触摸事件可以基于鼠标事件来模拟。

如果您想在桌面上模拟单点触控事件，请尝试使用 [Phantom Limb](http://www.vodori.com/blog/phantom-limb.html)，它可以在网页上模拟触摸事件并提供一只巨手进行引导。

另外还有 [Touchable](https://github.com/dotmaster/Touchable-jQuery-Plugin) 这一 jQuery 插件，可跨平台地统一触摸和鼠标事件。
