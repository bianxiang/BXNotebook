[toc]

http://julian.com/research/velocity

Velocity支持的CSS属性参见http://julian.com/research/velocity/#cssSupport。
这是一个网页上的示例。支持的CSS在下拉列表框中。

### 基础：参数

[Demo](http://codepen.io/julianshapiro/pen/BjwtC)

第一个参数是一个Map，包含CSS属性/值。第二个参数是选项对象，可选。

```js
$element.velocity({
    width: "500px",
    property2: value2
}, {
    /* Velocity's default options */
    duration: 400,
    easing: "swing",
    queue: "",
    begin: undefined,
    progress: undefined,
    complete: undefined,
    display: undefined,
    visibility: undefined,
    loop: false,
    delay: false,
    mobileHA: true
});
```

修改`$.Velocity.defaults`可以全局覆盖选项的默认值。

为了支持CoffeeScript，Velocity提供单参数语法。传入单个对象，包含`properties`和`options`两个成员：

```js
$element.velocity({
    properties: { opacity: 1 },
    options: { duration: 500 }
});
```

Velocity还支持jQuery风格的、逗号分隔的选项，但仅支持duration, easing, complete三个属性：`$element.velocity(propertyMap [, duration] [, easing] [, complete])`。三个选项可以以任何顺序出现：

```js
$element.velocity({ top: 50 }, 1000);
$element.velocity({ top: 50 }, 1000, "swing");
$element.velocity({ top: 50 }, "swing");
$element.velocity({ top: 50 }, 1000, function() { alert("Hi"); });
```

### 基础：属性Map

**属性**

[Demo](http://codepen.io/julianshapiro/pen/fjbct)

Velocity会自动为属性添加前缀（如`transform`变成`webkit-transform`）；我们自己不要加前缀。一个属性只接受一个值，因此可以传入：`{ padding: 1 }`或`{ paddingLeft: 1, paddingRight: 1 }`。但不能传入`{ padding: "1 1 1 1" }`。

However, Velocity includes hooks to break down the following properties that require multiple values: `textShadow`, `boxShadow`, `clip`, `backgroundPosition`, and `transformOrigin`. 参见后面“CSS支持”一节。

**值**

[Demo](http://codepen.io/julianshapiro/pen/fjbct)

**Velocity 1.1.0新特性。**

Velocity支持以下单位：px, em, rem, %, deg, vw/vh。如果你不提供单位，会自动选择一个合适的单位。一般是px，有时是deg。

Velocity支持四个值运算符：+, -, *, /。这四个运算符后加等号表示相对运算。

```js
$element.velocity({
    top: 50, // Defaults to the px unit type
    left: "50%",
    width: "+=5rem", // Add 5rem to the current rem value
    height: "*=2" // 当前高度加倍
});
```

Browser support: rem units are not supported below IE 9. vh/vw units are not supported below IE 9 or below Android 4.4.

### 基础：链

[Demo](http://codepen.io/julianshapiro/pen/hyqlF)

对一个元素（或一组）调用多次Velocity方法，它们将**自动排队**：前一个执行完后后一个才执行：

```js
$element.velocity({ width: 75 }).velocity({ height: 0 });
```

### 选项：持续时间

Velocity支持毫秒时间和jQuery的关键字："slow", "normal", "fast"

```js
$element.velocity({ opacity: 1 }, { duration: 1000 });
$element.velocity({ opacity: 1 }, { duration: "slow" });
```

### 选项：缓动

Velocity默认支持以下缓动类型：

- Velocity自带[jQuery UI](http://easings.net/)的缓动（除了back, bounce, elastic）。还额外支持"spring"（参见后面“CSS支持”）。[Demo](http://codepen.io/julianshapiro/pen/bAiIt)
- CSS3的关键字："ease", "ease-in", "ease-out", "ease-in-out"。
- CSS3的贝叶斯曲线：Pass a four-item array of bezier points. (Refer to [Cubic-Bezier.com](http://cubic-bezier.com/) for crafing custom bezier curves.) [Demo](http://codepen.io/julianshapiro/pen/xBJcu)
- Spring physics: Pass a two-item array in the form of `[ tension, friction ]`. A higher tension (default: 500) increases total speed and bounciness. A lower friction (default: 20) increases ending vibration speed. Refer to [Spring Physics Tester](http://codepen.io/julianshapiro/pen/hyeDg) for testing values. [Demo](http://codepen.io/julianshapiro/pen/fgjaF)
- Step easing: Pass a one-item array in the form of `[ steps ]`. The animation will jump toward its end values using the specified number of steps. See [this article](http://css-tricks.com/clever-uses-step-easing/) to learn more about step easing. [Demo](http://codepen.io/julianshapiro/pen/ylvuh)

上面所有的缓动类型都兼容IE8+。

```js
/* Use one of the jQuery UI easings. */
$element.velocity({ width: 50 }, "easeInSine");
/* Use a custom bezier curve. */
$element.velocity({ width: 50 }, [ 0.17, 0.67, 0.83, 0.67 ]);
/* Use spring physics. */
$element.velocity({ width: 50 }, [ 250, 15 ]);
/* Use step easing. */
$element.velocity({ width: 50 }, [ 8 ]);
```

可以为每个属性分别指定一个缓动类型：传入一个两项的数组，第一项是属性值，第二项是缓动类型：

```js
$element.velocity({
    borderBottomWidth: [ "2px", "spring" ],
    width: [ "100px", [ 250, 15 ] ],
    height: "100px" // 默认easeInSine, the call's default easing
}, {
    easing: "easeInSine" // 默认的缓动
});
```

与jQuery一样，Velocity默认的缓动类型是"swing"。

### 选项：队列

设置`queue`为false可以强制新动画被立即调用，如果当前有动画正在执行，与它们并行执行。[Demo](http://codepen.io/julianshapiro/pen/Ioeqy)

```js
$element.velocity({ width: "50px" }, { duration: 3000 });
setTimeout(function() {
    /* 1500毫秒后并行 */
    $element.velocity({ height: "50px" }, { queue: false });
}, 1500);
```

或者可以创建一个自定义队列（不立即执行），这时需要传入自定义的队列名。可以并行构建多个自定义队列，后续可以独立的启动它们（通过`$element.dequeue("queueName")`，如果没有jQuery，使用`Velocity.Utilities.dequeue(element(s), "queueName")`。 [Demo](http://codepen.io/julianshapiro/pen/bIBGa)

Note that the `loop` option and `reverse` command do not work with custom and parallel queues.

Velocity使用与jQuery相同的队列逻辑，and thus interoperates seamlessly with `$.animate()`, `$.fade()`, and `$.delay()`. To learn more about Velocity's queueing behavior, read [this primer](http://stackoverflow.com/questions/1058158/can-somebody-explain-jquery-queue-to-me).

### 选项：开始和结束

#### Begin

[Demo](http://codepen.io/julianshapiro/pen/zCJgp)

指定动画开始前调用的函数。

起始和结束函数一样，每次调用只会执行一次，即使有多个元素需要动画。如果调用是循环的，开始回调只会执行一次——在第一次循环开始前。

元素数组（原始的，非jQuery对象）会被传入函数，同时作为函数上下文。

```js
$element.velocity({
    opacity: 0
}, {
    /* Log all the animated divs. */
    begin: function(elements) { console.log(elements); }
});
```

#### Complete

[Demo](http://codepen.io/julianshapiro/pen/DCLja)

完成函数在动画结束后调用。一次调用只会执行一次，即使有多个元素需要动画。如果调用是循环的，完成回调只会执行一次，在循环结束时执行那个。

元素数组（原始的，非jQuery对象）会被传入函数，同时作为函数上下文。

```js
$element.velocity({
    opacity: 0
}, {
    /* Log all the animated divs. */
    complete: function(elements) { console.log(elements); }
});
```
### 选项：进度

[Demo](http://codepen.io/julianshapiro/pen/Jktjq)

传入一个回调函数，在动画期间定期调用。

元素数组（原始的，非jQuery对象）会被传入函数，同时作为函数上下文。此外，还会传入percentComplete（数字）、timeRemaining（毫秒）和timeStart (Unix time)：

```js
$element.velocity({
    opacity: 0
}, {
    progress: function(elements, percentComplete, timeRemaining, timeStart) {
        console.log((percentComplete * 100) + "%");
        console.log(timeRemaining + "ms remaining!");
    }
});
```
### 选项：mobileHA

`mobileHA`表示mobile hardware acceleration。默认为true。

Use mobileHA instead of sprinkling [null transform hacks](http://aerotwist.com/blog/on-translate3d-and-layer-creation-hacks/) throughout your code. Velocity's manipulation of HA is highly streamlined.

Mobile browsers benefit hugely from HA, whereas desktop browsers do not. (JavaScript-powered desktop animations actually perform worse when hardware acceleration is applied.) Accordingly, mobileHA has no effect on desktop browsers.

As with Velocity's transform support — transform values set outside of Velocity (including their default values) are overridden when mobileHA is on. If this is an issue, turn mobileHA off by setting it to false:

```js
$element.velocity(propertiesMap, { mobileHA: false });
```

### 选项：循环

**Velocity 1.1.0新特性。**

[Demo](http://codepen.io/julianshapiro/pen/KgvyC)

将`loop`设为一个整数，指定动画要交替的次数。属性值在指定值和调用前的值之间交替。

```js
$element.velocity({ height: "10em" }, { loop: 2 });
```

例如，若元素原来的高度是`5em`，它的高度会在`5em`和`10em`之间交替两次。

开始和结束回调只会在循环开始前和结束后调用一次。

传入`true`触发无限循环：

```js
$element.velocity({ height: "10em" }, { loop: true });
```

Infinite loops never return promises, ignore the complete callback, and cannot be used with `queue: false`. 在对同一个元素启动另一个无限循环前，请先停止前一个无限循环。（循环可以通过Stop命令停止。）

### 选项：延迟

[Demo](http://codepen.io/julianshapiro/pen/GICev)

动画开始前暂停指定毫秒。如果`delay`和`loop`连用，每次循环都会暂停。

```js
$element.velocity({
    height: "+=10em"
}, {
    loop: 4,
    delay: 100 /* Wait 100ms before alternating back. */
});
```

### 选项：Display & Visibility

[Demo](http://codepen.io/julianshapiro/pen/kJlKB)

**Velocity 1.1.0新特性。**

Velocity的`display`和`visibility`选项与相应的CSS属性对应,取值也相同。`display`的值可以是`"inline", "inline-block", "block", "flex", ""`(empty quotes to remove the property altogether)或其他浏览器支持的值。`visibility`接受`"hidden", "visible", "collapse", and ""` (empty quotes to remove the property altogether)。

Velocity还支持一个特别的**display**值`"auto"`。表示将元素设为原生的`display`值。例如链接元素默认是`"inline"`，div元素的默认值是`"block"`。

Velocity's incorporation of CSS display/visibility changing allows for all of your animation logic to be contained within Velocity, 即你不再需要在代码中到处使用jQuery的`$.hide()`和`$.show()`方法。

**display**和**visibility**选项在动画完成后设置。若与`loop`选项连用，在最终循环结束后再设置可见性。例如，下面的元素在淡出后，从页面流中移除。此方法可以替换jQuery的`$.fadeOut()`函数。

```js
$element.velocity({ opacity: 0 }, { display: "none" });
```

但如果**display/visibility**的值不是`"none"`/`"hidden"`，如`"block"`/`"visible"`，相应的值在动画开始前设置，于是元素在动画的过程中是可见的。
(Further, if `opacity` is simultaneously animated, it'll be forced to a start value of 0. This is useful for fading an element back into view.)

下面的例子，`display`在元素淡出前被设为`"block"`：

```js
$element.velocity({ opacity: 1 }, { display: "block" });
```

The `display` and `visibility` options are ignored when used with the **Reverse** command.

### 命令：Fade & Slide

**fade**和**slide**命令自动设置目标元素的`display`属性，以显示或隐藏元素。`display`的值默认为元素原生的显示值（如内联元素是`inline`）。可以通过`display`选项显式制定一个值：

```js
$element.velocity("fadeIn", { display: "table" });
```

**fadeIn和fadeOut**

要淡入淡出一个元素，"fadeIn"或"fadeOut"作为Velocity的第一个参数。fade命令与标准Velocity调用一样，可以带选项，可以排队。

```js
$element
    .velocity("fadeIn", { duration: 1500 })
    .velocity("fadeOut", { delay: 500, duration: 1500 });
```

淡入元素，持续1500ms；暂停500ms；然后淡出。

**slideUp和slideDown**

要将元素高度变换到0或从0开始增加，向Velocity传入"slideDown"或"slideUp"。Slide命令与标准Velocity调用一样，可以带选项，可以排队。

```js
$element
    .velocity("slideDown", { duration: 1500 })
    .velocity("slideUp", { delay: 500, duration: 1500 });
```

上述动画，在1500毫秒内元素下滑，然后暂停500毫秒，再收起。

If you're looking for a highly performant mobile-first accordion powered by Velocity's slide functions, check out [Bellows](https://github.com/mobify/bellows).

### 命令：Scroll

[Demo](http://codepen.io/julianshapiro/pen/kBuEi)

要将浏览器滚动到元素顶部，向Velocity传入"scroll"。Scroll命令与标准Velocity调用一样，可以带选项，可以排队。

```js
$element
    .velocity("scroll", { duration: 1500, easing: "spring" })
    .velocity({ opacity: 1 });
```

如果元素的容器带滚动条，scroll命令有一个特殊的选项`container`用于指定容器。可以传入一个jQuery对象或原生DOM元素。容器元素必须是已定位的，静态元素无法工作：

```js
$element.velocity("scroll", { container: $("#container") });
```

默认滚动发生在Y轴。传入`axis: "x"`选项让滚动水平行进：

```js
/* Scroll the browser to the LEFT edge of the targeted div. */
$element.velocity("scroll", { axis: "x" });
```

Scroll还接受一个`offset`选项，单位像素，指定目标滚动的偏移位置：

```js
$element
    /* 滚动到的位置位于div以下250个像素 */
    .velocity("scroll", { duration: 750, offset: 250 })
    /* 滚动到的位置位于div以上50个像素 */
    .velocity("scroll", { duration: 750, offset: -50 });
```

若要滚动浏览器窗口到一定偏移，选择`html`元素，并通过`offset`指定偏移。在iOS下，通过`mobileHA: false`避免弹动的问题：

```js
$("html").velocity("scroll", { offset: "750px", mobileHA: false });
```

### 命令：Stop

[Demo](http://codepen.io/julianshapiro/pen/xLAfs)

要停止一个元素上所有当前的Velocity调用，将"stop"作为Velocity的第一个参数。在元素的动画队列中得下一个Velocity调用将立即得到执行。若要停止一个自定义队列，将对队列名作为第二个参数。

```js
$element.velocity("stop"); // Normal stop
$element.velocity("stop", "myQueue"); // Stop a custom queue
```

若调用是被停止的，它的完成回调和`display: none`会被跳过。

停止一个调用并动画回起始值可以用到Velocity的`reverse`命令：

```js
/* Prior Velocity call. */
$element.velocity({ opacity: 0 });
/* Later, midway through the call... */
$element.velocity("stop")
    /* Animate opacity back to 1. */
    .velocity("reverse");
```

> 一个循环的Velocity调用实际是一系列Velocity调用链接到一起。Accordingly, to completely stop a looping animation, you must pass in the second parameter to clear the element's remaining calls. Similarly, in order to fully stop a UI pack effect (which can consist of multiple Velocity calls), 你需要清掉元素整个动画队列。See [Clearing the Animation Queue] below for details on how to do so.

**Stopping Multi-Element Calls**

Because Velocity uses call-based tweening, when stop is called on an element, it is specifically the call responsible for that element's current animation that is stopped. Thus, if other elements were also targeted by that same call, they will also be stopped:

```js
/* Prior Velocity call. */
$allElements.velocity({ opacity: 0 });
/* Stop the above call. */
$allElements.velocity("stop");
```

或

```js
/* Behaves like above since it's ending a multi-element call. */
$firstElement.velocity("stop");
```

If you want per-element stop control in a multi-element animation, perform the animation by calling Velocity once on each element.

**清除动画队列**

当一个调用被停止，元素队列中下一个Velocity调用立即执行。传入true（或自定义队列的名称）可以清掉元素剩余所有排队的调用：

```js
/* Prior Velocity calls. */
$element.velocity({ width: 100 }, 1000).velocity({ height: 200 }, 1000);
/* Called immediately after. */
$element.velocity("stop", true);
```

Above, the initial `{ width: 100 }` call will be instantly stopped, and the ensuing `{ height: 200 }` will be removed and skipped entirely.

> If you're clearing the remaining queue entries on a call that targeted more than one element, be sure that your stop command is applied to the full set of elements that the call initially targeted. Otherwise, some elements will have stuck queues and will ignore further Velocity calls until they are manually dequeued.

### 命令：Reverse

http://codepen.io/julianshapiro/pen/hBFbc

To animate an element back to the values prior to its **last** Velocity call, pass in "reverse" as Velocity's first argument.

Reverse默认使用之前Velocity调用的选项（duration、easing等）。但也可以传入新的选项对象：

```js
$element.velocity("reverse");
// or
$element.velocity("reverse", { duration: 2000 });
```

之前调用的回调方法（开始和结束回调）会被reverse忽略。

> reverse命令只能用于默认的效果队列，不能用于自定义队列或并行队列（`queue: false`）。

### 特性：Transforms

http://codepen.io/julianshapiro/pen/FIwfv

为照顾CSS习惯，Velocity的平移使用`translateX`和`translateY`属性：

```js
$element.velocity({
    translateX: "200px",
    rotateZ: "45deg"
});
```

因为Velocity动画的属性只接受单个值，因此transform属性必须带坐标轴。例如`scale`不能是`"1.5, 2"`，要分别使用`scaleX()`和`scaleY()`。(Refer to the CSS Support dropdown for Velocity's full transform property support.)

Velocity的性能优化导致对transform值的外部修改（包括定义在样式表中得初始值，but this is remedied via **Forcefeeding**）。（利用**Hook**函数可以在Velocity中手工设置transform值）。

所有的浏览器，在动画三维transform属性时都会触发硬件加速。硬件加速的好处是增加平滑度，缺点是造成文字锯齿和增加内存消耗。If you want to also force HA for 2D transforms in order to benefit from the increased performance it can provide (at the expense of possible text blurriness), then trigger HA by animating a 3D transform property to a bogus value (such as 0) during your animation:

```js
$element.velocity({
    translateZ: 0, // Force HA by animating a 3D property
    translateX: "200px",
    rotateZ: "45deg"
});
```

(This section applies to desktop browsers only — by default, Velocity automatically turns HA on for all animations on mobile.)

Browser support: Remember that 3D transforms are not supported below IE 10 and below Android 3.0, and that even 2D transforms aren't supported below IE 9. Also, remember that a perspective must be set on a parent element for 3D transforms to take effect.

### 特性：颜色

http://codepen.io/julianshapiro/pen/wlEtB

Velocity颜色动画支持以下属性：color, backgroundColor, borderColor, outlineColor。还可以单独动画RGBA颜色的一部分。Red, Green, Blue 取值0到255,Alpha取值0到1。

```js
$element.velocity({
    /* Animate a color property to a hex value of red... */
    backgroundColor: "#ff0000",
    /* ... with a 50% opacity. */
    backgroundColorAlpha: 0.5,
    /* Animate the red RGB component to 50% (0.5 * 255). */
    colorRed: "50%",
    /* Concurrently animate to a stronger blue. */
    colorBlue: "+=50",
    /* Fade the text down to 85% opacity. */
    colorAlpha: 0.85
});
```
In IE 9 and below, where RGBA is not supported, Velocity reverts to plain RGB by ignoring the Alpha component.

### （未）Feature: SVG

http://julian.com/research/velocity/#svg

### 特性：Hook

http://codepen.io/julianshapiro/pen/LFeDB

Hooks是多值CSS属性的子值。例如CSS属性`textShadow`是多值的，取值如`"0px 0px 0px black"`。Velocity允许你独立的动画这子值，如`textShadowX`、`textShadowY`和`textShadowBlur`:

```js
$element.velocity({ textShadowBlur: "10px" });
```

Similarly, Velocity allows you to animate the subvalues of `boxShadow`, `clip`, and other multi-value properties. Refer to the CSS Property Support pane for the full list of Velocity's hooks.

Since it's not possible to individually set these hook values via jQuery's `$.css()`, Velocity provides a `hook()` helper function. It features the same API as `$.css()`, with the modification that it takes an element as its first argument (either a raw DOM node or a jQuery object): `$.Velocity.hook(element, property [, value])`:

设置一个hook值：

```js
$.Velocity.hook($element, "translateX", "500px"); // Must provide unit type
$.Velocity.hook(elementNode, "textShadowBlur", "10px"); // Must provide unit type
```

读取一个hook值：

```js
$.Velocity.hook($element, "translateX");
$.Velocity.hook(elementNode, "textShadowBlur");
```

注意，使用hook函数时，如果CSS属性需要单位，必须提供。

### Feature: Promises

http://julian.com/research/velocity/#promises

If you're new to promises, read [this article](http://www.html5rocks.com/en/tutorials/es6/promises/).

满足两个条件，一个Velocity调用自动返回一个promise对象：1、通过工具函数调用；2、侦测到浏览器支持promise。Conversely, promises are never returned when using jQuery's object syntax (e.g. `$element.velocity(...)`).

```js
/* Using Velocity's utility function... */
$.Velocity.animate(element, { opacity: 0.5 })
    /* Callback to fire once the animation is complete. */
    .then(function(elements) { console.log("Resolved."); })
    /* Callback to fire if an error occurs. */
    .catch(function(error) { console.log("Rejected."); });
```

The returned promise is resolved when the call's animation completes, regardless of whether it completed on its own or prematurely due to the user calling `$element.velocity("stop"[, true])`. The resolve function is passed the entire raw DOM (not jQuery) element array as both its context and its first argument. To access these elements individually, you must iterate over the array using jQuery's `$.each()` or JavaScript's native `.forEach()`.

Conversely, the promise is rejected when an invalid first argument is passed into a Velocity call (e.g. an empty properties object or a Velocity command that does not exist). The reject callback is passed the triggered error object.

Promises also work with effects from the UI pack (including custom effects). As usual, however, ensure that you're calling the UI pack effect using Velocity's utility function. Further, ensure you're using the latest version of the UI pack, since promise support was added only recently.

Browser support: Only Chrome desktop and Firefox desktop have native support for promises. For all other browsers, you must install a third-party promises library, such as Bluebird or When's ES6 Promises shim. (Do not use Q, since it doesn't automatically install itself like ES6 promises do.) When using Bluebird or [When](https://github.com/cujojs/when/tree/master/es6-shim), ensure that the promise library is loaded before Velocity. Both Bluebird and When work back to Android 2.3 and IE8.

### Feature: Mock

http://codepen.io/julianshapiro/pen/KmlCv

在做UI测试时，可以设置`$.Velocity.mock = true;`命令所有Velocity动画在0秒内完成，无延迟。(In essence, values are fully applied on the next animation tick.) This is incredibly helpful when performing repetitious UI testing for which testing end values is more important than testing animation tweening.

Alternatively, you can also set `$.Velocity.mock` to an arbitrary multiplier to speed up or slow down all animations on the page:

```js
/* Slow down all animations by a factor of 10. */
$.Velocity.mock = 10;
```

Slowing down animations in this way is useful when you're trying to fine tune the timing of multi-element animation sequences.

### 特性：工具函数

http://julian.com/research/velocity/#utilityFunction

Instead of using jQuery's plugin syntax to target jQuery objects, you can use Velocity's utility function to target raw DOM elements:

```js
/* Standard multi-argument syntax. */
var divs = document.getElementsByTagName("div");
$.Velocity(divs, { opacity: 0 }, { duration: 1500 });
/* Alternative single-argument syntax (ideal for CoffeeScript). <i>e</i> stands for elements, <i>p</i> for properties, and <i>o</i> for options: */
var divs = document.getElementsByTagName("div");
$.Velocity({ e: divs, p: { opacity: 0 }, o: { duration: 1500 });
```

The syntax is identical to Velocity's standard syntax, except for all of the arguments are shifted one place to the right so that you can pass in elements at position zero.

Using the utility function is useful when you're generating elements in real-time and cannot afford the overhead of jQuery object creation, which triggers DOM queries.

### 高级：值函数

http://codepen.io/julianshapiro/pen/Ecsoh

属性值可以是一个函数。These functions are called once per element — immediately before the element begins animating. 因此在循环或翻转过程中，函数不会被重复调用。函数返回值做属性值。

```js
$element.velocity({
    opacity: function() { return Math.random() }
});
```

Value functions are passed the iterating element as their context, plus a first argument containing the element's index within the set and a second argument containing the total length of the set. 利用这些值可以实现视觉上的平移：

```js
$element.velocity({
    translateX: function(i, total) {
        /* Generate translateX's end value. */
        return i * 10;
    }
});
```

对于一组元素的动画，值函数是最佳性能的选择。如果采用遍历元素，依次单独调用Velocity函数，将不能利用Velocity的元素集优化（element set optimization）。

### 高级：Forcefeeding

http://codepen.io/julianshapiro/pen/rkgyH

传统上，动画引擎通过查询DOM决定动画的初始值。Velocity使用另一种技术，称为**forcefeeding**：用户显式定义初始值，不必再查询DOM，避免了[布局的晃动](http://www.kellegous.com/j/2013/01/26/layout-performance/)。

Forcefeed start values are passed as a second or third item in an array that takes the place of a property's value. (As described in the Easing pane, the second item can optionally be a per-property easing).

```js
$element.velocity({
    /* Two-item array format. */
    translateX: [ 500, 0 ],
    /* Three-item array format with a per-property easing. */
    opacity: [ 0, "easeInSine", 1 ]
});
```

与标准属性值一样，forcefeed值也接受值函数。(You can take advantage of this feature to seed an element set differentiated start values. See Velocity's [3D demo codecast](https://www.youtube.com/watch?v=MDLiVB6g2NY&hd=1) for an example of this.)

Be sure to forcefeed only at the start of an animation, not between chained animations (where Velocity already does value caching internally):

```js
    $element
        /* Optionally forcefeed here. */
        .velocity({ translateX: [ 500, 0 ] })
        /* Do not forcefeed here; 500 is internally cached. */
        .velocity({ translateX: 1000 });
```

Forcefeeding对于高压条件很有用（大量DOM查询对性能冲击很大）。但在一些低强度的UI动画中，forcefeeding并不必要。

Note: Forcefeeding a hook's subproperty will default that hook's non-animated subproperties to their zero-values.

### 特性：Sequences

Sequence registration is deprecated. See the Custom Effects section of Velocity's UI Pack (below) for a better solution.

### 插件：UI Pack

At only 3Kb zipped, the UI pack is a must-have for improving your animation workflow. 它暴露两个函数：`$.Velocity.RegisterEffect()`和`$.Velocity.RunSequence()`。前者可以让你组合多个Velocity调用成一个命名的特效。The latter helps make nested animation sequences much more manageable.

此外UI pack还包含十多个预定义的特效。Refer to the Effects: Pre-Registered section below.

[Download](https://raw.githubusercontent.com/julianshapiro/velocity/master/velocity.ui.js) the pack (or visit [GitHub](https://github.com/julianshapiro/velocity)) then load it after Velocity. There's also a plugin for Angular users. The UI pack does not require jQuery to be loaded on your page.

#### 顺序执行

http://codepen.io/julianshapiro/pen/xnGDC/

顺序执行是UI包解决动画代码嵌套的方案。例如，若没有UI包，只能：

```js
$element1.velocity({ translateX: 100 }, 1000, function() {
    $element2.velocity({ translateX: 200 }, 1000, function() {
        $element3.velocity({ translateX: 300 }, 1000);
    });
});
```

如果不用上面逗号分隔的语法，使用选项对象的语法将更麻烦。

利用UI包得顺序执行功能，利用与Velocity工具函数的单对象参数语法创建一个数组：

```js
var mySequence = [
    { elements: $element1, properties: { translateX: 100 }, options: { duration: 1000 } },
    { elements: $element2, properties: { translateX: 200 }, options: { duration: 1000 } },
    { elements: $element3, properties: { translateX: 300 }, options: { duration: 1000 } }
];
$.Velocity.RunSequence(mySequence);
```

序列暴露一个特殊的`sequenceQueue`选项，当设为false时，强制调用与它签名的调用并发执行：

```js
var mySequence = [
    { elements: $element1, properties: { translateX: 100 }, options: { duration: 1000 } },
    /* The call below will run at the same time as the first call. */
    { elements: $element2, properties: { translateX: 200 }, options: { duration: 1000, sequenceQueue: false },
    /* As normal, the call below will run once the second call is complete. */
    { elements: $element3, properties: { translateX: 300 }, options: { duration: 1000 }
];
$.Velocity.RunSequence(mySequence);
```

#### Effects: Pre-Registered

The UI pack includes a couple dozen pre-registered effects for you to use out of the box. Use cases for these effects include drawing the user's attention to an element, dynamically loading content, and displaying modals. Refer to the [tutorial](http://www.smashingmagazine.com/2014/06/18/faster-ui-animations-with-velocity-js/) for a full overview.

To trigger an effect, simply pass its name as Velocity's first argument (instead of a properties map), e.g. `$elements.velocity("callout.bounce");` UI pack effects do not accept the loop, easing, or progress options. Further, they cannot be used with parallel queueing (ie. queue: false).

Note that `display: inline` elements cannot take the CSS transform property (which most of the UI pack effects use). Accordingly, the UI pack automatically switches any display: inline elements that it animates to `display: inline-block`.

Below is a listing of all pre-registered effects:
（见http://julian.com/research/velocity/#uiPack）

#### Effects: Behavior

UI pack effects behave like normal Velocity calls; they can be chained and can take options.

Elements automatically switch to display: block/inline when transitioning in, and back to display: none after transitioning out. (To prevent this behavior, pass `display: null` as an option into the UI Pack call.)

Support the special **stagger**, **drag**, and **backwards** options. (Refer to the next section.)

Browser support: Below IE10 and Android 3.0, the flip and perspective transitions gracefully fall back to simply fading in and out. In IE8, all transitions gracefully fall back to just fading in and out, and callouts (except for callout.flash) have no effect.

There are three options that work only with UI pack effects — but not with traditional Velocity calls. They are passed into a UI pack call as standard Velocity call options:

- **Stagger**: Specify the stagger option in ms to successively delay the animation of each element in a set by the targeted amount. You can also pass in a value function to define your own stagger falloffs. Demo: http://codepen.io/julianshapiro/pen/mqsnk
- **Drag**: Set the drag option to true to successively increase the animation duration of each element in a set. The last element will animate with a duration equal to the sequence's original value, whereas the elements before the last will have their duration values gradually approach the original value. The end result is a cross-element easing effect. Demo: http://codepen.io/julianshapiro/pen/lxfie
- **Backwards**: Set the backwards option to true to animate starting with the last element in a set. This option is ideal for use with an effect that transitions elements out of view since the backwards option mirrors the behavior of elements transitioning into view (which, by default, animate in the forwards direction — from the first element to the last). Demo: http://codepen.io/julianshapiro/pen/fEKsw

Refer to the [tutorial](http://www.smashingmagazine.com/2014/06/18/faster-ui-animations-with-velocity-js/) for a step-by-step overview of using these options.

#### Effects: Registration

Demo: http://codepen.io/julianshapiro/pen/jaBgm

The UI pack allows you to register custom effects, which also accept the special stagger, drag, and backwards options. Once registered, an effect is called by passing its name as Velocity's first parameter: 

    $element.velocity("name").

Benefits of custom effects include separating UI animation design from UI interaction logic, naming animations for better code organization, and packaging animations for re-use across projects and for sharing with others.

A custom UI pack effect is registered with the following syntax:

    $.Velocity.RegisterEffect(name, {
        defaultDuration: duration,
        calls: [
            [ { property: value }, durationPercentage, { options } ],
            [ { property: value }, durationPercentage, { options } ]
        ],
        reset: { property: value, property: value }
    });

In the above template, we pass an optional `defaultDuration` property, which specifies the duration to use for the full effect if one is not passed into the triggering Velocity call, e.g. `$element.velocity("name")`. Like a value function, `defaultDuration` also accepts a function to be run at an animation's start. This function is called once per UI pack call (regardless of how many elements are passed into the call), and is passed the raw DOM element set as both its context and its first argument.

Next is the array of Velocity calls to be triggered (in order). Each call takes a standard properties map, followed by the percentage (as a decimal) of the effect's total animation duration that the call should consume (defaults to 1 if unspecified), followed by an optional animation options object. This options object only accepts Velocity's easing and delay options.

Lastly, you may optionally pass in a reset property map (using standard Velocity properties and values), which immediately applies the specified properties upon completion of the effect. This is useful for when you're, say, scaling an element down to 0 (out of view) and want to return the element to scale:1 once the element is hidden so that it’s normally scaled when it’s made visible again sometime in the future.

Sample effect registrations:

Callout:

    $.Velocity.RegisterEffect("callout.pulse", {
        defaultDuration: 900,
        calls: [
            [ { scaleX: 1.1 }, 0.50 ],
            [ { scaleX: 1 }, 0.50 ]
        ]
    });
    $element.velocity("callout.pulse");

Transition:

    /* Registration */
    $.Velocity
        .RegisterEffect("transition.flipXIn", {
            defaultDuration: 700,
            calls: [
                [ { opacity: 1, rotateY: [ 0, -55 ] } ]
            ]
        });
        .RegisterEffect("transition.flipXOut", {
            defaultDuration: 700,
            calls: [
                [ { opacity: 0, rotateY: 55 } ]
            ],
            reset: { rotateY: 0 }
        });
    /* Usage */
    $element
        .velocity("transition.flipXIn")
        .velocity("transition.flipXOut", { delay: 1000 });

Note that, if your effects' names end with In or Out, Velocity will automatically set the display option to "none" or the element’s default type for you. In other words, elements are set to display: block Before beginning an “In” transition or display: none after completing an Out transition.

    /* Bypass the UI pack's automatic display setting. */
    $element.velocity("transition.flipXIn", { display: null });

