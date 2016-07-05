[toc]

## 6. 对象

Any value in JavaScript that is not a string, a number, true, false, null, or undefined is an object. And even though strings, numbers, and booleans are not objects, they behave like immutable objects (see §3.6).

属性包括一个名和一个值。属性名是字符串。值可以是任意Javascript值，或（ES5）可以是一个getter或setter函数（或同时是）。除了名和值，每个树形还有一些关联的值，我们称之为属性的特性（property attributes）：

- `writable`特性规定属性值是否可以被设置
- `enumerable`特性规定for/in循环是否包含该属性
- `configurable`特性规定属性是否可以被删除，以及它的特性是否可以被修改。

ES5之前，你代码创建的所有的属性都是可写、可枚举，可配置的。ES5允许你配置属性的特性。

每个对象都有三个关联的对象特性（object attributes）：

- An object’s *prototype* is a reference to another object from which properties are
inherited.
- 对象的*class*是一个字符串，分类对象的类型。
- （ES5）对象的*extensible*标志控制新属性是否可以加到对象。

### 6.1 创建对象

创建对象可以通过对象字面量、new关键字，`Object.create()`函数（ES5）。

#### 6.1.1 对象字面量

ES3一般要求属性名为关键字时加引号。ES5和部分ES3实现可以允许不加引号。ES5和部分ES3实现忽略最后的逗号，但IE会报错。

#### 6.1.2 使用new创建对象

Core JavaScript includes built-in constructors for native types. For example:

```js
var o = new Object();
var a = new Array();
var d = new Date();
var r = new RegExp("js");
```

#### 6.1.3 Prototypes

通过对象字面量创建的所有对象的原型对象都是`Object.prototype`。通过new创建的对象的原型是构造器的`prototype`树形。如`new Object()`创建的对象的原型是`Object.prototype`；`new Array()`的原型是`Array.prototype`。

`Object.prototype`是少有的没有原型的对象。所有内建的构造器的原型继承自`Object.prototype`。如`Date.prototype`继承自`Object.prototype`。因此`new Date()`从`Date.prototype`和`Object.prototype`两处继承属性。该特性称为原型链。

#### 6.1.4 Object.create()

ES5定义了一个方法`Object.create()`，创建一个新对象，使用第一个参数作为对象原型。第二个参数可选，描述新对象的属性。`Object.create()`是一个静态方法。

```js
var o1 = Object.create({x:1, y:2}); // o1 inherits properties x and y.
```

传入`null`创建的对象没有原型，此时对象什么也不继承，包括`toString()`等方法（因此不能使用`+`运算符）：

```js
var o2 = Object.create(null); // o2 inherits no props or methods.
```

创建空对象，即等价于`{}`或`new Object()`的写法是：

```js
var o3 = Object.create(Object.prototype); // o3 is like {} or new Object().
```

在ES3中模拟类似的行为：

```js
function inherit(p) {
	if (p == null) throw TypeError();
    if (Object.create)
		return Object.create(p);
	var t = typeof p;
	if (t !== "object" && t !== "function") throw TypeError();
    function f() {};
    f.prototype = p;
    return new f();
}
```

`inherit`函数的一个用法是，创建对象的一个替身。把这个替身传给不受信任的函数，函数若修改对象值，修改的是替身，不是原对象：

```js
var o = { x: "don't change this value" };
library_function(inherit(o)); // Guard against accidental modifications of o
```

该方法成立的理由，见下一节，如何读取和设置对象属性。

### 6.2 查询和设置属性

ES3中，标识符是保留字的，不能使用点运算符。例如不能写`o.for`或`o.class`。必须使用括号语法，如`o["for"]`和`o["class"]`。ES5（和部分ES3实现）放松了该限制。

使用中括号语法时，括号内的表达式必须求值为字符串，或能转换为字符串。例如数组，大量出现中括号内是数字。

#### 6.2.2 继承

假设要查询对象`o`的`x`属性。若对象`o`自己没有该属性。则从对象的原型中找。若原型也没有，则从原型的原型中找。直到找到`x`，或所有原型都被检查。

若给对象`o`的属性`x`赋值，若对象`o`有属性`x`（不是继承的），则直接设置该属性。否则，赋值在对象`o`上创建一个新属性`x`。如果对象`o`继承了过`x`属性，继承的属性被新创建的自有属性隐藏。

属性赋值检查原型链来决定赋值是否被允许。如果`o`继承了只读属性`x`，则赋值不被允许。若允许，赋值总是在对象`o`自身创建或修改属性，不会碰原型链。

一个例外情况。若`o`继承的属性`x`是一个访问器树形，定义了setter方法。则调用setter方法，而不是在o上创建新属性x。但setter方法是在对象o上调用的，不是在定义它的原型上调用的。因此如果setter方法定义了树形，它们定义在对象o上，原型链不会被修改。

#### 6.2.3 属性访问错误

Property access expressions do not always return or set a value. This section explains the things that can go wrong when you query or set a property.

查询不存在的属性不会报错，返回`undefined`。

`null`和`undefined`没有属性。因此对其查询属性会报错：

```js
// Raises a TypeError exception. undefined doesn't have a length property
var len = book.subtitle.length;
```

尝试给`null`或`undefined`设置属性抛TypeError。

尝试设置只读属性，或者向不允许添加新属性的对象添加属性，会失败。但这些失败尝试不会抛任何错误。

```js
// The prototype properties of built-in constructors are read-only.
Object.prototype = 0; // Assignment fails silently; Object.prototype unchanged
```

这个历史的怪异问题在ES5严格模式下被修正，会抛TypeError异常。

属性是否可以赋值规则比较复杂，以下情况会失败：

- p是o本身的属性，但是只读的：不能设置只读属性。(See the `defineProperty()` method, however, for an exception that allows configurable read-only properties to be set.)
- p是o继承来的属性，但是只读的：不能用自有属性因此一个继承来的只读属性。
- p不是o的属性，p也不是继承来的带有setter的属性，但o的extensible特性是false。If p does not already exist on o, and if there is no setter method to call, then p must be added to o. But if o is not extensible, then no new properties can be defined on it.

### 6.3 删除属性

`delete`运算符从对象中删除属性。它的运算数是一个属性访问表达式。

```js
delete book.author;
delete book["main title"];
```

`delete`只删除自己的属性，不删除继承的。(To delete an inherited property, you must delete it from the prototype object in which it is defined. Doing this affects every object that inherits from that prototype.)

删除成功，删除操作返回true。返回false表示删除无效，如不存在该属性。当表达式不是一个属性访问表达式时，`delete`也返回true。

`delete`不会删除不可配置的属性（configurable为false）。(Though it will remove configurable properties of nonextensible objects.) 内建对象的部分属性是不可配置的。通过变量声明和函数声明创建的全局对象的属性也是不可配置的。严格模式下，尝试删除一个不可配置的属性会导致TypeError。在非严格模式下（ECMAScript 3），`delete`操作只是返回false。

```js
delete Object.prototype; // Can't delete; property is non-configurable
var x = 1; // Declare a global variable
delete this.x; // Can't delete this property
function f() {} // Declare a global function
delete this.f; // Can't delete this property either
```

在非严格模式下删除可配置属性，你可以省略全局对象引用。`delete`后面直接使用属性名：

```js
this.x = 1; // Create a configurable global property (no var)
delete x; // And delete it
```

但在严格模式下，如果属性前不加限定，会抛出SyntaxError。

### 6.4 测试属性

用`in`运算符测试对象是否有某个属性（自有或继承都可以）。若有返回true：

```js
var o = { x: 1 }
"x" in o; // true: o has an own property "x"
"y" in o; // false: o doesn't have a property "y"
"toString" in o; // true: o inherits a toString property
```

`hasOwnProperty()`方法测试对象是否有自有属性。

```js
var o = { x: 1 }
o.hasOwnProperty("x"); // true: o has an own property x
o.hasOwnProperty("y"); // false: o doesn't have a property y
o.hasOwnProperty("toString"); // false: toString is an inherited property
```

`propertyIsEnumerable()`仅当属性是对象的自有属性且属性的`enumerable`特性为true时才返回true。部分内建的属性是不可枚举的。

```js
var o = inherit({ y: 2 });
o.x = 1;
o.propertyIsEnumerable("x"); // true: o has an own enumerable property x
o.propertyIsEnumerable("y"); // false: y is inherited, not own
Object.prototype.propertyIsEnumerable("toString"); // false: not enumerable
```

直接查询属性，用`!==`测试`undefined`为true，则属性可能不存在，或虽存在但尚未被赋值。

Note that the code above uses the `!==` operator instead of `!=`. `!==` and `===` distinguish between `undefined` and `null`.

```js
// If o has a property x whose value is not null or undefined, double it.
if (o.x != null) o.x *= 2;
// If o has a property x whose value does not convert to false, double it.
// If x is undefined, null, false, "", 0, or NaN, leave it alone.
if (o.x) o.x *= 2;
```

### 6.5 枚举属性

遍历属性，一般通过**for/in**循环。ECMAScript 5提供两种额外的方式。

**for/in**循环遍历自有和继承的属性。循环变量是属性名。对象继承的内建方法是不可枚举的。

```js
var o = {x:1, y:2, z:3};
o.propertyIsEnumerable("toString") // => false: not enumerable
for(p in o)
	console.log(p);
```

一些工具库向`Object.prototype`添加了新方法或属性。在ES5前，没有办法令这些属性不可枚举，因此它们会出现在**for/in**中。一些常见的过滤：

```js
for(p in o) {
	if (!o.hasOwnProperty(p)) continue; // Skip inherited properties
}
for(p in o) {
	if (typeof o[p] === "function") continue; // Skip methods
}
```

ECMAScript 5定义了两个函数，用于枚举属性名。第一个是`Object.keys()`，枚举对象的可枚举的自有属性。`Object.getOwnPropertyNames()`返回自有属性，包括不可枚举的属性。There is no way to write this function in ECMAScript 3, because ECMAScript 3 does not provide a way to obtain the nonenumerable properties of an object.

### 6.6 Property Getters and Setters

ES5开始，通过getters和setters定义的属性有时被称作访问器属性。

访问器属性不像数据属性有writable特性。若属性有getter和setter两个方法，则它是一个读写属性。如果属性只有getter方法，它是一个只读属性。若只有setter方法，它是一个只可写的属性；此时读取它直接返回`undefined`。

The easiest way to define accessor properties is with an extension to the object literal syntax:

```js
var o = {
    // An ordinary data property
    data_prop: value,
	// An accessor property defined as a pair of functions
    get accessor_prop() { /* function body here */ },
	set accessor_prop(value) { /* function body here */ }
};
```

访问器属性的函数名就是属性名。`function`关键字分别由get/set替代。注意数据属性和访问函数之间，访问函数相互之间是逗号而不是分号分隔的。

例子：

```js
var p = {
	x: 1.0,
	y: 1.0,
	get r() { return Math.sqrt(this.x*this.x + this.y*this.y); },
    set r(newvalue) {
		var oldvalue = Math.sqrt(this.x*this.x + this.y*this.y);
        var ratio = newvalue/oldvalue;
		this.x *= ratio;
		this.y *= ratio;
	},
	// theta is a read-only accessor property with getter only.
	get theta() { return Math.atan2(this.y, this.x); }
};
```

注意属性直接逗号分隔。注意getter和setter方法里的`this`。getter和setter函数被Javascript作为方法调用。

访问器属性与数据属性一样是可以被继承的。

本节只介绍了通过字面量如何创建访问器属性，下一节介绍如何向已存在的对象添加访问器属性。

### 6.7 属性特性

特性决定属性是否可写、可枚举、可配置。ES3无法设置这些特性：所有属性都是可写、可枚举、可配置的。ES5提供API查询和设置属性的特性。

本节我们将会把getter和setter方法看做属性的特性。并且数据属性的值也可以看做它的特性。于是，我们可以说一个属性包含一个名称和四个特性。对于数据属性，四个特性是`value`、`writable`、`enumerable`和`configurable`。对于访问器属性，四个特性`get`、`set`、`enumerable`和`configurable`。

ES5中，属性特性通过属性描述符对象查询或设置。The ECMAScript 5 methods for querying and setting the attributes of a property use an object called a property descriptor to represent the set of four attributes. 属性描述符对象包含几个属性，与特性名相同：对于数据属性，描述符对象的属性有`value`、`writable`、`enumerable`和`configurable`。访问器属性的描述符包含`get`、`set`、`enumerable`和`configurable`。`writable`、`enumerable`和`configurable`属性都是布尔值，`get`和`set`属性是函数。

To obtain the property descriptor for a named property of a specified object, call `Object.getOwnPropertyDescriptor()`:

```js
Object.getOwnPropertyDescriptor({x:1}, "x"); // Returns {value: 1, writable:true, enumerable:true, configurable:true}
// Now query the octet property of the random object defined above.
Object.getOwnPropertyDescriptor(random, "octet"); // Returns { get: /*func*/, set:undefined, enumerable:true, configurable:true}
Object.getOwnPropertyDescriptor({}, "x"); // undefined, no such prop
Object.getOwnPropertyDescriptor({}, "toString"); // undefined, inherited
```

`Object.getOwnPropertyDescriptor()`只能用于自有属性。要查询继承来的属性，必须显式的走继承链（见`Object.getPrototypeOf()`，§6.8.1）。

要设置属性的特性，会创建新属性，配以指定的特性，可以调用`Object.defineProperty()`：

```js
var o = {}; // Start with no properties at all
// 添加属性x
Object.defineProperty(o, "x", { value : 1, writable: true, enumerable: false, configurable: true});
// Check that the property is there but is nonenumerable
o.x; // => 1
Object.keys(o) // => []
// Now modify the property x so that it is read-only
Object.defineProperty(o, "x", { writable: false });
// Try to change the value of the property
o.x = 2; // Fails silently or throws TypeError in strict mode
o.x // => 1
// The property is still configurable, so we can change its value like this:
Object.defineProperty(o, "x", { value: 2 });
o.x // => 2
// Now change x from a data property to an accessor property
Object.defineProperty(o, "x", { get: function() { return 0; } });
o.x // => 0
```

若传给`Object.defineProperty()`的属性描述符没有带全四个特性，则省略的特性取值`false`或`undefined`。若是修改存在的属性，则省略的特性将保留原值。该方法可以创建新属性，或修改已有的属性，但不会修改继承来的属性。

若想一次修改或创建多个属性，利用`Object.defineProperties()`方法。例如：

```js
var p = Object.defineProperties({}, {
	x: { value: 1, writable: true, enumerable:true, configurable:true },
    y: { value: 1, writable: true, enumerable:true, configurable:true },
    r: {
        get: function() { return Math.sqrt(this.x*this.x + this.y*this.y) },
        enumerable:true,
		configurable:true
	}
});
```

`Object.defineProperties()`和`Object.defineProperty()`都返回修改后的对象。

We saw the ECMAScript 5 method Object.create() in §6.1. We learned there that the first argument to that method is the prototype object for the newly created object. This method also accepts a second optional argument, which is the same as the second argument to Object.defineProperties(). If you pass a set of property descriptors to Object.create(), then they are used to add properties to the newly created object.
`Object.defineProperty()` and `Object.defineProperties()` throw `TypeError` if the attempt to create or modify a property is not allowed. 例如尝试向不可扩展的对象添加新属性（§6.8.3）。The other reasons that these methods might throw TypeError have to do with the attributes themselves. The `writable` attribute governs attempts to change the value attribute. And the `configurable` attribute governs attempts to change the other attributes (and also specifies whether a property can be deleted). The rules are not completely straightforward, however. It is possible to change the value of a nonwritable property if that property is configurable, for example. Also, it is possible to change a property from writable to nonwritable even if that property is nonconfigurable. 下面是完整的规则。Calls to Object.defineProperty() or Object.defineProperties() that attempt to violate them throw TypeError:

- 若对象是不可扩展的，则你可以编辑已存在的属性，但不能添加新属性。
- 若属性不可配置，则不能修改`configurable`和`enumerable`特性。
- If an accessor property is not configurable, you cannot change its getter or setter method, and you cannot change it to a data property.
- If a data property is not configurable, you cannot change it to an accessor property.
- If a data property is not configurable, you cannot change its writable attribute from false to true, but you can change it from true to false.
- If a data property is not configurable and not writable, you cannot change its value. You can change the value of a property that is configurable but nonwritable, however (because that would be the same as making it writable, then changing the value, then converting it back to nonwritable).

Example 6-2 included an extend() function that copied properties from one object to another. That function simply copied the name and value of the properties and ignored their attributes. Furthermore, it did not copy the getter and setter methods of accessor properties, but simply converted them into static data properties. Example 6-3 shows a new version of extend() that uses Object.getOwnPropertyDescriptor() and Object.defineProperty() to copy all property attributes. Rather than being written as a function, this version is defined as a new Object method and is added as a nonenumerable property to Object.prototype.

#### 6.7.1 Legacy API for Getters and Setters














