[toc]

# Javascript: The Good Part

## 1. 精华

### 1.1 为什么要使用JavaScript

浏览器的API和文档对象模型（DOM）相当糟糕，导致Javascript遭到不公平指责。在任何语言中处理DOM都是一件痛苦的事情，它的规范制定的拙劣且实现不一致。本书很少涉及DOM，我认为写一本关于DOM的精华的书是不可能完成的任务。

### 1.2 分析JavaScript

Javascript中好的想法包括函数、弱类型、动态对象、对象字面量表示法。

JavaScript's functions are first class objects with (mostly) lexical scoping.
译注3：Javascript中的函数根据词法来划分作用域，而不是动态地划分作用域。参见《Javascript权威指南》第5版，8.8.1，词法作用域。

Javascript是第一个主流的lambda语言。

Javascript的原型继承（prototypal inheritance）是一项有争议的特性。Javascript的对象系统没有类的概念，对象直接从其他对象继承属性。

Javascript有一项特性是非常糟糕的：JavaScript depends on global variables for linkage. All of the top-level variables of all compilation units are tossed together in a common namespace called the global object. This is a bad thing because global variables are evil, and in JavaScript they are fundamental. Fortunately, as we will see, JavaScript also gives us the tools to mitigate this problem.

### 1.3 一个简单的试验场

本书自始至终将使用下面的method方法定义新方法：

```js
Function.prototype.method = function (name, func) {
    this.prototype[name] = func;
    return this;
};
```

第4章将详细解释。

## 2 语法

### 2.1 空白

Javascript提供两种注释：`/* */`和`//`。

### 2.2 标识符

保留字：
```
abstract
boolean break byte
case catch char class const continue
debugger default delete do double
else enum export extends
false final finally float for function
goto
if implements import in instanceof int interface
long
native new null
package private protected public
return
short static super switch synchronized
this throw throws transient true try typeof
var volatile void
while with
```

### 2.3 数字

Javascript只有一种数字类型。内部表示为64位的浮点数。没有单独的整数，因此1和1.0是相同值。

> Javascript没有整数类型

指数形式，如`1e2`，值是100。

值`NaN`是一个数值。它表示一个不能产生正常结果的运算结果。`NaN`不等于任何值，包括它自己。可以使用函数`isNaN(number)`检测`NaN`。

> 判断一个数字`number`是不是`NaN`，须用`isNaN(number)`。不能用`number === NaN`

值`Infinity`表示所有大于`1.79769313486231570e+308`的值。

数字拥有方法（参见第8章）。Javascript有一个对象Math，包含一套用于数字的方法。例如`Math.floor(number)`方法将一个数字转换成一个整数。

### 2.4 字符串

字符串字面量可以被包围在单引号或双引号中。\是转义字符。

由于创建Javascript的时候Unicode是一个16位的字符集，所有Javascript中所有字符都是16位的。

Javascript没有字符类型（至哟字符串）。

字符串有一个`length`属性。如`”seven”.length`。

字符串是不可变的。

可以使用`+`连接字符串。下面的表达式值为true。
```js
'c' + 'a' + 't' === 'cat'
```

字符串的方法见第8章。
```js
'cat'.toUpperCase() === 'CAT'
```

### 2.5 语句

定义变量：

```js
var a;
var a, b;
var a = 1, b;
```

在web浏览器中，每个`<script>`标签都是一个编译单元，被编译并立即执行。由于缺少连接器，Javascript把它们一起抛入一个公共的全局名字空间中。附录A有更多关于全局变量的内容。

switch, while, for, do语句可以带一个前置标签，供break语句使用。

控制语句：if, switch, while, for, do, break, return, throw。

下列值被当作false：

- false
- null
- undefined
- 空字符串
- 数字0
- 数字NaN

其他值都被当作true，包括true、字符串”false”、空数组`[]`，空对象`{}`，以及所有对象。

switch语句：表达式可以产生数字或字符串。
如果没有找到匹配执行default。
一个case子句可以包含一个或多个case表达式。case表达式不一定是常量。为防止继续执行下一个case，case语句之后应跟随一个强制跳转语句，如break。

for循环有两个形式。一种是

```js
for(initializtion;condition;increment){}
```

另一种是for-in。枚举对象的所有属性名（或键名）。

通常需要检查`object.hasOwnProperty(varibale)`来确定这个属性名就是该对象的成员，还是从其原型链中得到的。

```js
for (myvar in obj) {
    if (obj.hasownProperty(myvar)) {
        ...
    }
}
```

try–catch

throw语句中的表达式通常为一个对象字面量，它包含一个name属性和message属性。异常捕获后可以获取这些属性。

```js
try {
	throw {name: "xx", message: "xxx"};
} catch(e) {
	console.log(e.name);
}
```

return语句若不指定返回值，返回`undefined`。


### 2.6 表达式

> 不要死记运算符优先级。所有有歧义的地方直接加括号！

`typeof`运算符的值有'number', 'string', 'boolean', 'undefined', 'function', 'object'。第6章和附录A有更多关于`typeof`的内容。

> 数组或null的`typeof`结果是’object’！

`/`可能产生非整数结果（即使两个运算符都是整数）。

如果第一个运算数的值为假，那么运算符`&&`的结果为第一个运算符的值，否则取第二个运算符的值。

### 2.7 字面量

字面量包括数字字面量、字符串字面量、对象字面量、数组字面量、函数字面量、正则表达式字面量。

对象字面量，如：`{a: 1, b: true}`。数组字面量，如：`[1, 2, 'a', 4]`。

正则表达式字面量被两个斜杠包围。如：`/(.*)[a-z]/`。

## 3 对象

Javascript的简单类型包括数字、字符串、布尔值（true和false）、`null`值和`undefined`值。其他所有值都是对象。

数字、字符串和布尔值“貌似”是对象，因为它们拥有方法，但它们是不可变的。Javascript中的对象是可变的键值集合。在Javascript中，数组是对象、函数是对象、正则表达式是对象。

对象是属性的容器，其中每个属性都有名字和值。属性的名字可以是包含空字符串在内的任意字符串。属性值可以是除`undefined`值之外的任何值。

Javascript中的对象没有类。

JavaScript includes a prototype linkage feature that allows one object to inherit the properties of another. 如果使用正确，它可以减少初始化时间和内存消耗。

### 3.1 对象字面量

对象字面量是一对花括号内的零个或多个“名值对”。

```js
var empty_object = {};

var stooge = {
    "first-name": "Jerome",
    "last-name": "Howard"
};
```

在对象字面量中，若属性名是一个合法的Javascript标识符且不是保留字，可以不使用引号括住属性名 。例如，`"first–name"`的引号是必需的，`first_name`可以不用引号。

对象可以嵌套：

```js
var flight = {
    airline: "Oceanic",
    number: 815,
    departure: {
        IATA: "SYD",
        time: "2004-09-22 14:55",
        city: "Sydney"
    },
    arrival: {
        IATA: "LAX",
        time: "2004-09-23 10:42",
        city: "Los Angeles"
    }
};
```

### 3.2 检索

利用`.`或中括号。如果字符串是一个合法的Javascript标识符且非保留字，可以使用`.`否则应使用中括号。

```js
stooge["first-name"] // "Joe"
flight.departure.IATA // "SYD"
```

如果检索的属性不存在，返回`undefined`。注意不是`null`。

```js
stooge["middle-name"]    // undefined
flight.status            // undefined
stooge["FIRST-NAME"]     // undefined
```

`||`可用于帮助填充值：

```js
var middle = stooge["middle-name"] || "(none)";
var status = flight.status || "unknown";
```

尝试从`undefined`取值将抛出TypeError异常。可以使用`&&`预作判断。

```js
flight.equipment                              // undefined
flight.equipment.model                        // throw "TypeError"
flight.equipment && flight.equipment.model    // undefined
```

### 3.3 更新

直接赋值。

```js
stooge['first-name'] = 'Jerome';
```

如果对象之前没有某个属性值，赋值将添加该属性。

### 3.4 引用

对象通过引用来传递。它们永远不会被拷贝。

### 3.5 原型

每个对象都链接（linked）到一个原型对象，并且它可以从中继承属性。所有通过对象**字面量**创建的对象都链接到`Object.prototype`——JavaScript标准中的对象。

创建对象时可以选择某个对象做它的原型。Javascript本身提供的机制杂乱又复杂，可以被显著简化。我们将给Object对象增加一个`beget`方法。使用该对象可以将一个对象作为原型创建新对象。

```js
if (typeof Object.beget !== 'function') {
     Object.beget = function (o) {
         var F = function () {};
         F.prototype = o;
         return new F();
     };
}
var another_stooge = Object.beget(stooge);
```

原型链接不影响更新。当改变对象时，原型对象不会受影响。

```js
another_stooge['first-name'] = 'Harry';
another_stooge['middle-name'] = 'Moses';
another_stooge.nickname = 'Moe';
```

试验：

```js
var stooge={}
stooge['first-name'] = 'A';
var another_stooge = Object.beget(stooge);
another_stooge['first-name'] = 'B';

another_stooge['first-name']; //’B’
stooge['first-name']; //’A’
```

原型链接只对检索值有效：尝试获得某个对象的属性值，若该对象本身没有该属性，从它的原型中找属性，如果没有再从原型的原型中找。如果`Obejct.prototype`也没找到，返回`undefined`。该过程称为委托（delegation）。

原型关系是一种动态的关系。如果我们添加一个新的属性到原型中。该属性会立即对所有基于该原型创建的对象可见。

```js
stooge.profession = 'actor';
another_stooge.profession    // 'actor'
```

### 3.6 反射

检查对象有什么属性，可以通过检索该属性并验证取得的值。

`typeof`操作符可以用于确定属性的类型：

```js
typeof flight.number     // 'number'
typeof flight.status      // 'string'
typeof flight.arrival     // 'object'
typeof flight.manifest   // 'undefined'
```

注意原型链中的有些属性也会产生一个值，但它们是函数（函数产生值）：

````js
typeof flight.toString    // 'function'
typeof flight.constructor // 'function'
```

有些值是函数（产生的），可以通过`typeof`去除。

`hasOwnProperty()`可以确认对象自身（注意Own这个词）是否有某个属性（返回true）。`hasOwnProperty()`不检查原型链。

```js
flight.hasOwnProperty('number')       // true
flight.hasOwnProperty('constructor')    // false
```

注意如果flight对象定义有方法fun1()，则flight.hasOwnProperty(‘func1’)返回true！

### 3.7 枚举

**for in** 语句用来遍历一个对象的所有属性名。该枚举过程会列出所有属性——包括**函数**和原型中的属性。可以使用`hasOwnProperty()`方法过滤掉原型中的属性，在此基础上使用`typeof`排除函数：

```js
var name;
for (name in another_stooge) {
    if (typeof another_stooge[name] !== 'function') {
        document.writeln(name + ': ' + another_stooge[name]);
    }
}
```

> 属性名的出现顺序是不确定的。


### 3.8 删除

`delete`运算符用于删除对象的属性。它不会触及原型链中的任何对象。

删除属性可能让原型链中的属性浮现出来。

```js
another_stooge.nickname    // 'Moe'
// Remove nickname from another_stooge, revealing the nickname of the prototype.
delete another_stooge.nickname;
another_stooge.nickname    // 'Curly'
```

### 3.9 减少全局变量污染

最小化全局变量的方法是在程序中只创建一个全局变量，让此变量成为应用的容器：

```js
var MYAPP = {};

MYAPP.stooge = {
    "first-name": "Joe",
    "last-name": "Howard"
};

MYAPP.flight = {
    airline: "Oceanic",
    number: 815,
    departure: {
        IATA: "SYD",
        time: "2004-09-22 14:55",
        city: "Sydney"
    },
    arrival: {
        IATA: "LAX",
        time: "2004-09-23 10:42",
        city: "Los Angeles"
    }
};
```

闭包也可以有效的减少全局污染，见下章。

## 4 函数

### 4.1 函数对象

对象字面量产生的对象连接到`Object.prototype`。而函数对象连接到`Function.prototype`（该原型对象本身连接到`Object.prototype`）。每个函数在创建时附有两个附加的隐藏属性：函数的上下文和实现函数行为的代码（the function's context and the code that implements the function's behavior）。

译注：Javascript创建一个函数对象时，会给该对象设置一个“调用”属性。当Javascript调用一个函数时，可以理解为调用此函数的“调用”属性。

Every function object is also created with a `prototype` property. Its value is an object with a constructor property whose value is the function（就是这个函数本身）. This is distinct from the hidden link to `Function.prototype`. 这个令人费解的构造过程将在下一节展示。

因为函数是对象，**函数可以存放在变量中，作为参数传递**。函数可以返回函数。因为函数也是对象，因此**函数可以拥有方法**。

### 4.2 函数字面量

通过函数字面量创建函数：

```js
var add = function (a, b) {
    return a + b;
};
```

函数字面量包含四个部分。

- 第一部分是保留字function。
- 第二部分是函数名，可以被省略。如果没有名字，函数是匿名函数，如上面的例子。
- 第三部分是参数列表。
- 第四部分是包含为大括号内的语句。

函数定义可以嵌套。内部函数可以访问到外围函数的**参数和变量**。通过函数字面量创建的函数对象包含到外部上下文的连接，这被称为 **闭包** 。这是Javascript强大表现力的根基。


### 4.3 调用

除了声明时定义的形式参数，每个函数还接收两个附加参数：`this`和`arguments`。`this`的值取决于调用函数的方式。Javascript中一共有四种调用模式：**方法调用模式**、**函数调用模式**、**构造器调用模式**和**apply调用模式**。这些模式影响参数`this`的指向。

当实参与形参个数不匹配时不会导致运行时错误。如果实参过多，多出的参数被忽略。如果参数值过少，缺失的值将被替换为`undefined`。对参数值不会进行类型检查。

#### 方法调用模式

当函数被存储为对象的一个属性时，称其为**方法**。当方法被调用时，`this`绑定到那个对象。

```js
var myObject = {
    value: 0;
    increment: function (inc) {
        this.value += typeof inc === 'number' ? inc : 1;
    }
};

myObject.increment();
document.writeln(myObject.value);    // 1

myObject.increment(2);
document.writeln(myObject.value);    // 3
```

将`this`绑定到对象发生在调用时。这种极其延迟的绑定提高了使用`this`的函数的可重用性。Methods that get their object context from this are called public methods.

#### 函数调用模式

当函数不是对象的一个属性时，它被当作函数来调用：

```js
var sum = add(3, 4); // sum is 7
```

当函数以此模式调用时，`this`被绑定到全局对象，**这是一个设计错误**。正确的设计应该是当内部函数被调用时，`this`应绑定到外部函数的`this`变量。

这个设计错误导致的问题是，若函数A内部定义了函数B，在函数A找那个调用B，B的`this`指向全局对象，而不是函数A的`this`。一个约定俗成的解决方法是：让方法定义一个变量并将`this`赋给它，那么内部函数就可以通过那个变量访问`this`。按照约定，我给那个变量命令为`that`：

```js
myObject.double = function () {
    var that = this; // Workaround.
    var helper = function () {
        that.value = add(that.value, that.value)
    };
    helper(); // Invoke helper as a function.
};

// Invoke double as a method.
myObject.double();
document.writeln(myObject.getValue()); // 6
```

#### 构造器调用模式

Javascript是一门基于原型的语言。但Javascript本身对其原型的本质也缺乏自信，于是提供了一套和基于类的语言类似的对象构造语法。

如果调用一个函数时加`new`前缀，则将创建一个新对象。该对象隐式链接到函数的`prototype`的值上，`this`将被绑定到这个新对象。

`new`也会改变`return`语句的行为。我们将会在后面看到更多内容。

```js
// 创建一个构造器函数Que。它创建一个对象，带有status属性。
var Quo = function (string) {
    this.status = string;
};
// 给Quo的所有实例一个公有方法，名为get_status。
Quo.prototype.get_status = function () {
    return this.status;
};
// Make an instance of Quo.
var myQuo = new Quo("confused");
document.writeln(myQuo.get_status());  // confused
```

我不推荐构造器函数，下一章将介绍更好的方法。

#### apply调用模式

因为Javascript是一门函数式的面向对象编程语言，所以函数可以拥有方法。`apply`是函数的一个方法，用于调用此函数。`apply`方法接受两个参数。第一个是将被绑定给this的值。第二个是一个参数数组。

例如，正常的调用`sum`的方式：
```js
var sum = add(3, 4);
```
等价的调用方式：
```js
var array = [3, 4];
var sum = add.apply(null, array); // sum is 7
```

```js
// statusObject未继承Quo.prototype，但我们可以对statusObject对象调用get_status方法，即便statusObject对象没有get_status方法。

var statusObject = {
    status: 'A-OK'
};

var status = Quo.prototype.get_status.apply(statusObject);
// status is 'A-OK'
```

### 4.4 参数

通过`arguments`数组可以访问所有实参，包括那些多余参数。这使得编写一个无需指定参数个数的函数成为可能：

```js
var sum = function ( ) {
    var i, sum = 0;
    for (i = 0; i < arguments.length; i += 1) {
        sum += arguments[i];
    }
    return sum;
};

document.writeln(sum(4, 8, 15, 16, 23, 42)); // 108
```

这不是一个特别有用的模式。在第6章中，我们将介绍如何给数组添加一个相似的方法来达到同样的效果。

因为语言的设计错误，`arguments`并不是一个真正的数组。它只是一个“类似数组”的对象。`arguments`有`length`属性，但缺少所有的数组方法。我们将在本章结尾看到这个设计错误的后果。

### 4.5 返回

**函数总是会返回一个值**。如果没有指定返回值，返回`undefined`。

如果调用函数时有加`new`关键字，若函数内return没有返回值，将返回`this`（即新创建的对象）。

### 4.6 异常

抛出：

```js
var add = function (a, b) {
    if (typeof a !== 'number' || typeof b !== 'number') {
        throw {
            name: 'TypeError',
            message: 'add needs numbers'
        }
    }
    return a + b;
}
```

捕获：

```js
// Make a try_it function that calls the new add
// function incorrectly.
var try_it = function () {
    try {
        add("seven");
    } catch (e) {
        document.writeln(e.name + ': ' + e.message);
    }
}

tryIt();
```

一个`try`只能有一个`catch`，捕获所有（各种）异常。

### 4.7 给类型增加方法

Javascript允许给语言的基本类型添加方法。在第3章中介绍了通过给`Object.prototype`添加方法来使得该方法对所有对象可用。此方式也适用于函数、数组、字符串、数字、正则表达式和布尔值。

例如，通过加强`Function.prototype`方法，可以令所有函数获得`method`方法：

```js
Function.prototype.method = function (name, func) {
    this.prototype[name] = func;
    return this;
};
```

By augmenting `Function.prototype` with a method method, we no longer have to type the name of the `prototype` property. That bit of ugliness can now be hidden.


Javascript并没有单独的整数类型，因此有时只提取数字中的整数部分是必要的。Javascript本身提供的取整方法有些丑陋。我们可以通过给`Number.prototype`添加一个`integer`方法来改善它：根据数字的正负来判断使用`Math.ceiling`还是`Math.floor`。

```js
Number.method('integer', function () {
    return Math[this < 0 ? 'ceiling' : 'floor'](this);
});

document.writeln((-10 / 3).integer(  ));  // -3
```


Javascript缺少移除字符串末尾空白的方法，可以实现为：

```js
String.method('trim', function ( ) {
    return this.replace(/^\s+|\s+$/g, '');
});

document.writeln('"' + "   neat   ".trim( ) + '"');
```

通过给基本类型增加方法，我们可以大大提高语言的表现力。因为Javascript原型继承的动态本质，新的方法立刻被赋予到所有的值（对象实例）上，哪怕值（对象实例）是在方法被创建之前就创建好了。

基本类型的原型是公共的结构，所以在类库混用时务必小心。一个保险的做法是只在确定没有该方法时才添加它。

```js
// Add a method conditionally.
Function.prototype.method = function (name, func) {
    if (!this.prototype[name]) {
        this.prototype[name] = func;
    }
};
```

### 4.8 递归

Javascript并未优化**尾递归**。深度递归的函数可能因为返回栈溢出而运行失败。

### 4.9 作用域

Javascript不支持块级作用域。

Javascript有函数作用域。

很多现代语言都推荐尽可能迟地声明变量。而用在Javascript上却很糟糕，因为它缺少块级作用域。所以最好的做法是在函数体的**顶部**声明函数中可能用到的所有变量。

### 4.10 闭包

作用域的好处是内部函数可以访问定义它们的外部函数的参数和变量（`this`和`arguments`除外）。

一个更有趣的情形是内部函数拥有比它的外部函数更长的生命周期。

```js
var myObject = function () {
    var value = 0;

    return {
        increment: function (inc) {
            value += typeof inc === 'number' ? inc : 1;
        },
        getValue: function () {
            return value;
        }
    }
}();
```

我们并没有把函数赋给`myObject`，而是把该函数的**返回值**赋给了`myObject`。注意最后的`()`。函数返回一个包含两个方法的对象，并且这些方法继续享有访问`value`变量的特权。

本章之前定义的`Que`构造器，产生带有`status`属性和`get_status`方法的对象。注意那个`status`是公有的，不提供过`get_status`也能访问。下面我们将定义另一个形式的que函数，使得`status`只能通过getter函数方法：

```js
var quo = function (status) {
    return {
        get_status: function () {
            return status;
        }
    };
};
// Make an instance of quo.
var myQuo = quo("amazed");
document.writeln(myQuo.get_status());
```

这个`que`函数被设计成无须在前面加上new来使用，所以名字也没有首字母大写。当调用`que`时，返回一个新对象。该对象包含一个`get_status`方法。即使`que`已经返回了，但`get_status`方法仍能访问`que`对象的`status`属性（其实是函数参数）。`get_status`方法并不访问该参数的一个拷贝，它访问的是该参数本身。函数可以访问被创建时所处的上下文环境，这被称为**闭包**。


一个更有用的例子：
```js
// 定义一个函数，设置一个DOM节点为黄色，然后让它逐渐变白
var fade = function (node) {
    var level = 1;
    var step = function ( ) {
        var hex = level.toString(16);
        node.style.backgroundColor = '#FFFF' + hex + hex;
        if (level < 15) {
            level += 1;
            setTimeout(step, 100);
        }
    };
    setTimeout(step, 100); // #1
};
fade(document.body);
```

fade函数定义了函数step，然后调用`setTimeout(step, 100)`;，意思是等100毫秒后调用step函数。然后fade返回，fade函数结束。100毫秒后，step函数被调用，然后可能继续调用setTimeout()。fade函数返回后，因step函数仍要用到level，因此level继续存在。

理解内部函数访问外部函数的变量的实际值而非副本是很重要的，见下面这个错误：

```js
var add_the_handlers = function (nodes) {
    var i;
    for (i = 0; i < nodes.length; i += 1) {
        nodes[i].onclick = function (e) {
            alert(i);
        }
    }
};
```

原本想每次弹出时显示节点序号，但最后每个节点显示的都是数字都等于节点数组（nodes.length）。原因是事件处理函数绑定的是变量`i`，而非`i`当时的值。

正确的修改：

```js
var add_the_handlers = function (nodes) {
    var i;
    for (i = 0; i < nodes.length; i += 1) {
        nodes[i].onclick = function (i) {
            return function (e) {
                alert(i);
            };
        }(i);
    }
};
```

### 4.11 回调

```js
request = prepare_the_request();
send_request_asynchronously(request, function (response) {
        display(response);
    });
```

### 4.12 模块

我们可以使用函数和闭包来构造模块。模块是一个提供接口却隐藏 **状态** 与 **实现** 的函数或对象。通过**使用函数**去产生模块，我们几乎可以完全摒弃全局变量的使用。

假定要给String增加一个`deentityify`方法。任务是寻找字符串中HTML实体并替换为对应的字符。那实体名字与对应字符的映射表保存在哪？不能保存到全局变量。但如果放在`deentityify`方法内，则每次运行时都要对映射表字面量求值。增加开销。理想方式是放入闭包：

```js
String.method('deentityify', function () {
    var entity = {
        quot: '"',
        lt:   '<',
        gt:   '>'
    };

    return function () {
        return this.replace(/&([^&;]+);/g, function (a, b) {
            var r = entity[b];
            return typeof r === 'string' ? r : a;
        });
    };
}());
```

赋给`String.deentityify`是函数产生的函数，注意最后的`()`。

```js
document.writeln('&lt;&quot;&gt;'.deentityify());  // <">
```

模块模式利用函数作用域和闭包来创建绑定对象与私有成员的关联，在这个例子中，至哟`deentityify`方法有权访问那个映射表。

模块模式的一般形式是：一个定义了私有变量和函数的函数；利用闭包创建可以访问私有变量和函数的特权函数；最后返回这个特权函数，或者把它们保存到一个可访问的地方。

模块模式的目的是促进信息封装，类似于面向对象（Java/C++）中封装的概念。

模块模式也可以用来产生安全的对象。假定我们想要构造一个用来产生序列号的对象：

```js
var serial_maker = function () {
// 产生一个对象，这个对象能够产生唯一的字符串。字符串包括前缀和顺序数两部分。
// 对象包括两个方法用来设置前缀和顺序数
// gensym方法产生唯一的字符串
    var prefix = '';
    var seq = 0;
    return {
        set_prefix: function (p) {
            prefix = String(p);
        },
        set_seq: function (s) {
            seq = s;
        },
        gensym: function () {
            var result = prefix + seq;
            seq += 1;
            return result;
        }
    };
}( );

var seqer = serial_maker();
seqer.set_prefix = 'Q';
seqer.set_seq = 1000;
var unique = seqer.gensym();    // unique is "Q1000"
```

如果我们把`seqer.gensym`作为一个值传递给第三方函数，这个第三方函数可以用它产生唯一字符串，却不能用来改变prefix和seq的值。

### 4.13 级联

让函数返回this。

### 4.14 套用（Curry）

套用允许我们将函数与传递给它的参数相结合，产生一个新函数。

```js
var add1 = add.curry(1);
document.writeln(add1(6));    // 7
```

Javascript本身并未提供`curry()`方法。我们可以通过`Function.prototype`添加一个：

```js
Function.method('curry', function () {
    var args = arguments, that = this;
    return function () {
        return that.apply(null, args.concat(arguments));
    };
}); // 这段代码有问题。
```

使用Array的concat方法去连接两个数组。糟糕的是，arguments并非真正的数组，所以它没有concat方法。要避开该问题，我们必须在两个arguments数组上都应用数组的slice()方法。这样产生出拥有concat方法的常规数组。

```js
Function.method('curry', function ( ) {
    var slice = Array.prototype.slice,
        args = slice.apply(arguments),
        that = this;
    return function ( ) {
        return that.apply(null, args.concat(slice.apply(arguments)));
    };
});
```

### 4.15 记忆

函数可以利用对象记住之前操作的结果，避免重复的运算。

```js
var fibonacci = function (n) {
    return n < 2 ? n : fibonacci(n - 1) + fibonacci(n - 2);
};

for (var i = 0; i <= 10; i += 1) {
    document.writeln('// ' + i + ': ' + fibonacci(i));
}
```

fibonacci函数将被调用453次。
使用一个名为memo的数组保存结果，结果隐藏在闭包中。

```js
var fibonacci = function ( ) {
    var memo = [0, 1];
    var fib = function (n) {
        var result = memo[n];
        if (typeof result !== 'number') {
            result = fib(n - 1) + fib(n - 2);
            memo[n] = result;
        }
        return result;
    };
    return fib;
}( );
```

一个通用的记忆函数：

```js
var memoizer = function (memo, fundamental) {
    var shell = function (n) {
        var result = memo[n];
        if (typeof result !== 'number') {
            result = fundamental(shell, n);
            memo[n] = result;
        }
        return result;
    };
    return shell;
};
```

使用memoizer调用fibonacci。

```js
var fibonacci = memoizer([0, 1], function (shell, n) {
    return shell(n - 1) + shell(n - 2);
});
```

计算阶乘：

```js
var factorial = memoizer([1, 1], function (shell, n) {
    return n * shell(n - 1);
});
```

## （未）5 继承

（略）

## 6 数组

Javascript中没有真正的数组，而是提供一些类数组的对象。它把数组下表转换为字符串，用其作为属性。它明显比真正的数组慢。数组有它们自己的字面量格式。数组有一套有用的内置方法，见第8章。

### 6.1 数组字面量

方括号内，逗号分隔。

```js
var empty = [];
var numbers = [
    'zero', 'one', 'two', 'three', 'four',
    'five', 'six', 'seven', 'eight', 'nine'
];

empty[1]          // undefined
numbers[1]        // 'one'

empty.length      // 0
numbers.length    // 10
```

数组下标以0开始。

`numbers`继承自（原型是）`Array.prototype`。

Javascript允许数组元素为混合类型：

```js
var misc = [
    'string', 98.6, true, false, null, undefined,
    ['nested', 'array'], {object: true}, NaN,
    Infinity
];
misc.length    // 10
```

### 6.2 长度

每个数组都有一个`length`属性。若使用下标时超过`length`，数组将扩大以容纳新元素。

下标运算符`[]`将其中的表达式转换为一个字符串，转换使用表达式的`toString`方法。产生的字符串将作为属性名。如果这个字符串看起来像一个正整数，大于等于数组当前长度且小于4,294,967,295，则数组长度将增加到容纳这个下标。

可以直接设置`length`值。Making the length larger does not allocate more space for the array. 令`length`减小相当于裁剪数组。

```js
numbers.length = 3;
// numbers is ['zero', 'one', 'two']
```

通过下面的方式可以向数组添加元素：

```js
numbers[numbers.length] = 'shi';
// numbers is ['zero', 'one', 'two', 'shi']
```

更好的方式是使用`push()`方法：

```js
numbers.push('go');
// numbers is ['zero', 'one', 'two', 'shi', 'go']
```

### 6.3 删除

使用`delete`删除数组元素，但会留下“空洞”。`delete`不是从数组中移除一个元素（后续元素上提），而是将指定位置上的值置为undefined。

```js
delete numbers[2];
// numbers is ['zero', 'one', undefined, 'shi', 'go']
```

`splice()`用于去除数组中一部分。第一个参数指定开始删除的位置，第二个参数指定删除个数。

```js
numbers.splice(2, 1);
// numbers is ['zero', 'one', 'shi', 'go']
```

要对被删除的元素之后的每个元素调整下标值，对大数组来说效率很低。

### 6.4 枚举

因为Javascript数组是对象，因此若使用for–in遍历数组，不仅无法保证按顺序遍历，而且从原型链中属性干扰的问题也存在。

显式使用下标遍历不会有问题：

```js
var i;
for (i = 0; i < myArray.length; i += 1) {
    document.writeln(myArray[i]);
}
```

### 6.5 混淆的地方

Javascript本身对于数组和对象的区别是混乱的。`typeof`运算符报告数组的类型是’object’，这没有什么意义。

下面是一个可以判定值是否是数组的方法：

```js
var is_array = function (value) {
    return value &&
        typeof value === 'object' &&
        value.constructor === Array;
};
```

Unfortunately, it fails to identify arrays that were constructed in a different window or frame. If we want to accurately detect those foreign arrays, we have to work a little harder:

```js
var is_array = function (value) {
    return value &&
        typeof value === 'object' &&
        typeof value.length === 'number' &&
        typeof value.splice === 'function' &&
        !(value.propertyIsEnumerable('length'));
};
```

### 6.7 维度

Javascript没有多维数组，但支持元素为数组的数组：

```js
    var matrix = [
        [0, 1, 2],
        [3, 4, 5],
        [6, 7, 8]
    ];
    matrix[2][1] // 7
```

## 7 正则表达式

（略）

## 8 方法

（略，需要时百度即可）

## 9 代码风格

Javascript没有类似C语言的块级作用域，只有函数级作用域。因此我在每个函数的开始声明我的所有变量。

## A 糟粕

### A.3 自动插入分号

有时自动插入分号会导致错误，例如：

```lua
    return
    {
        status: true
    };
```

实际返回`undefined`。

### A.6 typeof

对`null`做`typeof`结果为’object’而非’null’。检查null的方式：

```js
	my_value===null
```

判断一个对象是否为对象或数组：

```js
if (my_value && typeof my_value === 'object') {
    // my_value is an object or an array!
}
```

在对正则表达式的类型识别上，不太一致。有的实现会报’object’有的会报’function’。

```js
	typeof /a/
```

### A.7 parseInt

`parseInt`将一个字符串转换为整数。当遇到非数字时停止。因此`parseInt("16")`与`parseInt("16 tons")`返回值相同。

如果第一个字符是`0`，则字符串按八进制解析。因为8和9不是合法的八进制，因此`parseInt("08")`和`parseInt("09")`都返回0。这个问题会导致解析时间字符串时错误，可以强制指定基数`parseInt("08", 10)`。推荐总是指定基数。

### A.9 浮点数

二进制浮点数不能正确处理十进制小数。因此`0.1 + 0.2`不等于`0.3`。这是Javascript经常被报告的bug，并且它是遵循二进制浮点数算术标准（IEEE 754）而有意导致的结果。

**幸好浮点运算中的整数运算是精确的**，可以通过扩大精度避免小数。如将货币单位设为分而不是元。

### A.10 NaN

`NaN`是IEEE 754定义的一个值。它表示非数字，尽管：`typeof NaN === 'number'`是true。

`NaN`可能尝试将字符串转换为数字时产生：

```js
+ '0'       // 0
+ 'oops'    // NaN
```

如果运算中有一个运算数为`NaN`，结果为`NaN`。

只能使用`isNaN`函数判断一个值是否为数字。`NaN`不等于自己。

```js
NaN === NaN    // false
NaN !== NaN    // true

isNaN(NaN)       // true
isNaN(0)         // false
isNaN('oops')    // true
isNaN('0')       // false
```

判断一个值是否可用于数字运算的最佳方式是使用`isFinite()`函数，因为它会筛选掉`NaN`和`Infinity`。不幸的是，如果操作数不是数字，`isFinite()`会试图将操作数转换为数字。因此，最好定义函数：

```js
    function isNumber(value) {
    	return typeof value === 'number' && isFinite(value);
    }
```

### A.11 伪数组

`typeof`运算符无法区分数组和对象，要判断是否是数组，需要检查`constructor`属性：

```js
    if (my_value && typeof my_value === 'object' &&
            my_value.constructor === Array) {
        // my_value is an array!
    }
```

上面的检测对于在不同Frame或窗口创建的数组将会给出false。当数组有可能在其他Frame中创建时，下面的检测更为可靠：

```js
    if (my_value && typeof my_value === 'object' &&
            typeof my_value.length === 'number' &&
            !(my_value.propertyIsEnumerable('length')) {
        // my_value is truly an array!
    }
```

`arguments`不是一个真数组，尽管它带有length成员。上面的检测会将arguments识别为数组。

### A.12 假值

Javascript中的假值：`0`、`NaN`、`''`（空字符串）、`false`、`null`、`undefined`。

`undefined`和`NaN`不是常量。它们是全局变量，而且你可以改变它们的值。

## 附录B 鸡肋

### B.1 `==`

两组运算符`===`、`!==`和`==`、`!=`。`===`只有在两个操作数类型相同且值相同时才会返回true。但`==`和`!=`在做比较时，如果操作数类型不同，会尝试做转换，转换规则非常奇怪。一些例子：

```js
'' == '0'          // false
0 == ''            // true
0 == '0'           // true

false == 'false'   // false
false == '0'       // true

false == undefined // false
false == null      // false
null == undefined  // true

' \t\r\n ' == 0    // true
```

建议，始终使用`===`、`!==`。永远不要使用`==`、`!=`。

### B.9 函数语句 vs 函数表达式

Javascript既有**函数语句**，又有**函数表达式**。二者看起来类似。

函数语句：`function foo( ) {}`。等价的函数表达式：`var foo = function foo( ) {};`。

函数语句在解析时会被提升：不管函数语句放置在哪里，它会被移动到所在作用域的顶部。可使得函数不必先声明后使用。而我认为这会造成混乱。

一个语句不能以一个函数表达式开头，因为官方规定以function开头的语句是一个function语句。解决方法是把函数放在一个圆括号中：

```js
(function () {
    var hidden_variable;

    // This function can have some impact on
    // the environment, but introduces no new
    // global variables.
})();
```

### B.10 类型的包装对象

Javascript有一套类型的包装读写，例如：`new Boolean(false)`，会返回一个对象。这其实完全没有必要！不要使用new Boolean，new Number和new String。此外也避免使用new Object和new Array。直接使用`{}`和`[]`多好。

### B.12 void

In JavaScript, void is an operator that takes an operand and returns undefined.
