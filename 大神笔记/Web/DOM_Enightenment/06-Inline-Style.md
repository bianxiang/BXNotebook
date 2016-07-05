[toc]

## 6 元素的内联样式

### 6.1 内联CSS属性：`style`

每个HTML元素豆油一个 `style` 特性，用于插入内联的CSS属性。

```js
var divStyle = document.querySelector('div').style;
// logs CSSStyleDeclaration {0="background-color", ...}
console.log(divStyle);
```

`style` 属性返回一个 `CSSStyleDeclaration` 对象。

### 6.2 读写、移除单个CSS属性

内联CSS样式是 `style` 对象的属性。

```js
var divStyle = document.querySelector('div').style;
// set
divStyle.backgroundColor = 'red';
divStyle.border = '1px solid black';
divStyle.width = '100px';
divStyle.height = '100px';
// get
console.log(divStyle.backgroundColor);
console.log(divStyle.border);
console.log(divStyle.width);
console.log(divStyle.height);
/* remove
divStyle.backgroundColor = '';
divStyle.border = '';
divStyle.width = '';
divStyle.height = '';
*/
```

注意到 `style` 的属性名是对应CSS属性名驼峰化后的。

若CSS属性名是JavaScript关键字，则 `style` 的属性名需要加 `css` 前缀，如 `float = cssFloat`。{{貌似只有这一个属性需要这样。}}

Shorthand properties are available as properties as well. So you can set `margin` as well as `marginTop`.

对于需要单位的CSS属性，记得加单位，如 `style.width = '300px'`。

`style` 对象还提供方法 `setPropertyValue(propertyName, value)`、 `getPropertyValue(propertyName)` 和 `removeProperty()`。这里的属性名使用CSS标准的中划线分隔的，如 `background-color`。

```js
var divStyle = document.querySelector('div').style;
//set
divStyle.setProperty('background-color', 'red');
divStyle.setProperty('border', '1px solid black');
divStyle.setProperty('width', '100px');
divStyle.setProperty('height', '100px');
//get
console.log(divStyle.getPropertyValue('background-color'));
console.log(divStyle.getPropertyValue('border'));
console.log(divStyle.getPropertyValue('width'));
console.log(divStyle.getPropertyValue('height'));
/* remove
divStyle.removeProperty('background-color');
divStyle.removeProperty('border');
divStyle.removeProperty('width');
divStyle.removeProperty('height');
*/
```

### 6.3 读写、删除所有内联CSS属性

Using the `cssText` property of the `CSSStyleDeclaration` object, as well as the `getAttribute()` and `setAttribute()` methods, it’s possible to get, set, and remove the entire value (i.e., all inline CSS properties) of the style attribute using a JavaScript string.

```js
var div = document.querySelector('div');
var divStyle = div.style;
// set using cssText
divStyle.cssText = 'background-color:red;border:1px solid black;height:100px;width:100px;';
// get using cssText
console.log(divStyle.cssText);
// remove
divStyle.cssText = '';

// exactly that same outcome using setAttribute() and getAttribute()
// set using setAttribute
div.setAttribute('style','background-color:red;border:1px solid black;
height:100px;width:100px;');
// get using getAttribute
console.log(div.getAttribute('style'));
// remove
div.removeAttribute('style');
```

Replacing the `style` attribute value with a new string is the fastest way to make multiple changes to an element’s style.

### 6.4 利用 `getComputedStyle()` 获取元素的 Computed Styles（实际样式，包括级联来的）

The `style` property only contains the CSS that is defined via the `style` attribute. To get an element’s CSS from the cascade (i.e., cascading from inline stylesheets, external stylesheets, and browser stylesheets) as well as its inline styles, you can use `getComputedStyle()`. This method provides a read-only `CSSStyleDeclaration` object similar to style.

```js
var div = document.querySelector('div');
//logs rgb(0, 128, 0) or green, this is an inline element style
console.log(window.getComputedStyle(div).backgroundColor);
/* logs 1px solid rgb(128, 0, 128) or 1px solid purple, this is an inline
element style */
console.log(window.getComputedStyle(div).border);
//logs 100px, note this is not an inline element style
console.log(window.getComputedStyle(div).height);
//logs 100px, note this is not an inline element style
console.log(window.getComputedStyle(div).width);
```

The `getComputedStyles()` method returns color values in the `rgb(#,#,#)` format, regardless of how they were originally authored.

Shorthand properties are not computed for the `CSSStyleDeclaration` object; you will have to use nonshorthand property names for property access (e.g., `marginTop`, not `margin`).

### 6.5 添加、删除ID和类

```js
//set
div.setAttribute('id','bar');
div.classList.add('foo');
/* remove
div.removeAttribute('id');
div.classList.remove('foo');
*/
```



























