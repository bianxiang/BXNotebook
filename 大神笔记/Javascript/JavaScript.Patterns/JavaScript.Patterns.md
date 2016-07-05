Copyright © 2010 Yahoo!, Inc.. All rights reserved.

[toc]

## 1 介绍

### 1.2 Javascript：概念

定义一个变量，变量自动称为一个对比对象（Activation对象）的属性（如果是全局变量，则是全局变量的属性）。Second, this variable is actually also object-like because it has its own properties (called *attributes*), which determine whether the variable can be changed, deleted, or enumerated in a for-inloop. These attributes are not directly exposed in ECMAScript 3, but edition 5 offers special descriptor methods for manipulating them.

### 1.3 ECMAScript 5

ECMAScript Version 5 添加了一些新特性。最重要的是引入了*strict mode*。它实际是为了移除某些语言特性。例如`with`语句在 ES5 严格模式下会报错，在非严格模式下不会。开启严格模式只需要一个字符串。老的浏览器会忽略这个字符串，因此这个特性是向后兼容的。在每个作用域（函数作用域、全局作用域，传入`eval()`的字符串的开头）可以添加：
```javascript
function my() {
	"use strict";
	// rest of the function...
}
```

本书不涉及 ES5 新特性，因为目前还有浏览器支持它。

## 2 基础

### 2.2 最小化全局

每个Javascript环境都有一个全局对象。在函数外可以通过`this`访问。全局变量都是这个全局对象的属性。在浏览器中，有个全局变量叫`window`，指向全局对象自己。
```javascript
myglobal = "hello"; // antipattern
console.log(myglobal); // "hello"
console.log(window.myglobal); // "hello"
console.log(window["myglobal"]); // "hello"
console.log(this.myglobal); // "hello"
```

JavaScript允许不声明变量就使用变量，此时变量会变为全局对象的属性（隐式全局）。例如下面的代码，`result`会称为全局变量：
```javascript
function sum(x, y) {
	// antipattern: implied global
	result = x + y;
	return result;
}
```

小心另一种形式。如下面代码，`b`会称为全局变量：
```javascript
// antipattern, do not use
function foo() {
	var a = b = 0;
	// ...
}
```

通过`var`显示定义的全局变量不能被`delete`删除。但隐式全局变量可以。

#### 单`var`模式

在函数开头通过单个`var`声明全部变量。

```javascript
function func() {
	var a = 1,
		b = 2,
		sum = a + b,
		myobject = {},
		i,
		j;
	// function body...
}
```

#### Hoisting

你在函数任何地方使用`var`，与在函数开头使用相同。这种特性称为 hoisting。这种特性可能导致问题。如：
```javascript
// antipattern
myname = "global"; // global variable
function func() {
	alert(myname); // "undefined"
	var myname = "local";
	alert(myname); // "local"
}
func();
```

上面的代码相当于：
```javascript
myname = "global"; // global variable
function func() {
	var myname; // same as -> var myname = undefined;
	alert(myname); // "undefined"
	myname = "local";
	alert(myname); // "local"
}
func();
```

> For completeness, let’s mention that actually at the implementation level things are a little more complex. There are two stages of the code handling, where variables, function declarations, and formal parameters are created at the first stage, which is the stage of parsing and entering the context. In the second stage, the stage of runtime code execution, function expressions and unqualified identifiers (undeclared variables) are created. But for practical purposes, we can adopt the concept of hoisting, which is actually not defined by ECMAScript standard but is commonly used to describe the behavior.

### 2.3 for 循环

下面循环的问题是，每次循环都要访问`length`属性。这种方法效率较低，特别是当`myarray`不是数组时。

```javascript
// sub-optimal loop
for (var i = 0; i < myarray.length; i++) {
// do something with myarray[i]
}
```

解决方法是缓存长度变量：
```javascript
for (var i = 0, max = myarray.length; i < max; i++) {
// do something with myarray[i]
}
```

这种方法的问题是，违反了单`var`原则。`i`和`max`需要在开头声明。但这种方式不利于剪切粘贴for循环。

```javascript
function looper() {
var i = 0,
	max,
	myarray = [];
// ...
for (i = 0, max = myarray.length; i < max; i++) {
	// do something with myarray[i]
}
}
```

另一种方法：
```javascript
var i, myarray = [];
for (i = myarray.length; i--;) {
	// do something with myarray[i]
}
```

这种方法的好处是，少一个变量（`max`），且比0比较比与数组长度比较效率更高。

最后一种方式：
```javascript
var myarray = [],
	i = myarray.length;
while (i--) {
	// do something with myarray[i]
}
```

### （未）2.4 for-in 循环

### 2.7 避免隐式类型转换

比较变量时Javascript会做隐式类型转换。于是`false == 0`或`"" == 0`返回`true`。

避免使用隐式转换，总是使用`===`和`!==`。

### 2.8 `parseInt()`

parseInt()的第二个参数可选，表示radix。
在ES3中，如果字符串以0开头，会被当成八进制。但在ES5中已经改变。为了安全，总是指定第二个参数。
```javascript
var month = "06", year = "09";
month = parseInt(month, 10);
year = parseInt(year, 10);
```

另一种更快的方式是：
```javascript
+"08" // result is 8
Number("08") // 8
```

二者有其他区别。例如对于“08  hello”，`parseInt()`返回数字，但其他方式返回`NaN`。

### 2.12 API 文档

And for JavaScript there are two excellent tools, both free and open source: the JSDoc Toolkit (http://code.google.com/p/jsdoc-toolkit/) and YUIDoc (http://yuilibrary.com/projects/yuidoc)

## 3. 字面量与构造器

### 3.3 `new`

调用构造器时，若忘记`new`，可能造成隐式创建全局变量。因为不加`new`时，函数中`this`指向全局对象。

```javascript
// constructor
function Waffle() {
	this.tastes = "yummy";
}

// antipattern:
// forgotten `new`
var good_morning = Waffle();
console.log(typeof good_morning); // "undefined"
console.log(window.tastes); // "yummy"
```

This undesired behavior is addressed in ECMAScript 5, and in strict mode this  will no longer point to the global object.

### （未）3.4 数组字面量

### （未）3.5 正则表达式字面量

### （未）3.6 Primitive Wrappers

### 3.7 错误对象

JavaScript有一些Error构造器，如`Error()`, `SyntaxError()`, `TypeError()`等。这些对象有两个属性：
- `name`：The name property of the constructor function that created the object; it could be the general “Error” or a more specialized constructor such as “RangeError”
- `message`： The string passed to the constructor when creating the object

The error objects have other properties, such as the line number and filename where the error happened, but these extra properties are browser extensions inconsistently implemented across browsers and are therefore unreliable.




