http://www.typescriptlang.org/Handbook

[toc]

## 基本类型

布尔，类型 `boolean`。
与 JS 一样，TypeScript 中的数字都是浮点数，类型 `number`。
字符串类型是 `string`。字面量与 JS 一样，用单引号或双引号包围。

```
var isDone: boolean = false;
var height: number = 6;
var name: string = "bob";
```

### 数组

两种写法。第一，类型后加 `[]`：

```
var list:number[] = [1, 2, 3];
```

第二种使用泛型写法：`Array<elemType>`：

```
	var list:Array<number> = [1, 2, 3];
```

### 枚举

给数字值一个名字：

```
enum Color {Red, Green, Blue};
var c: Color = Color.Green;
```

名字对应值默认从0开始。可以改变起始值：

```
enum Color {Red = 1, Green, Blue};
```

或者，给所有名字赋值：

```
enum Color {Red = 1, Green = 2, Blue = 4};
```

可以通过数字反查名字：

```
enum Color {Red = 1, Green, Blue};
var colorName: string = Color[2];
```

### Any

不知道变量类型，用 `any` 类型。`any` 类型注意用于与已存在的 JS 代码交互。

```
var notSure: any = 4;
notSure = "maybe a string instead";
notSure = false; // okay, definitely a boolean
```

`any` 类型还用于混合的数组：

```
var list:any[] = [1, true, "free"];
```

### Void

You may commonly see this as the return type of functions that do not return a value:

```
function warnUser(): void {
    alert("This is my warning message");
}
```

## 接口

TypeScript 的类型检查关注的是值的“形状”。This is sometimes called "duck typing" or "structural subtyping". TypeScript 中接口负责命名这些类型。

例子：

```
function printLabel(labelledObj: {label: string}) {
  console.log(labelledObj.label);
}

var myObj = {size: 10, label: "Size 10 Object"};
printLabel(myObj);
```

`printLabel` 函数的 `labelledObj` 要求是一个对象，且有一个 `label` 字符串的属性。实参对象可以有更多属性，编译器只检查必需的属性。

重写上面的例子，显式定义接口：

```
interface LabelledValue {
  label: string;
}

function printLabel(labelledObj: LabelledValue) {
  console.log(labelledObj.label);
}

var myObj = {size: 10, label: "Size 10 Object"};
printLabel(myObj);
```

类型检查不要求属性以特定顺序出现。

### 可选的属性

声明属性是可选的：

```
interface SquareConfig {
  color?: string;
  width?: number;
}

function createSquare(config: SquareConfig): {color: string; area: number} {
  var newSquare = {color: "white", area: 100};
  if (config.color) {
    newSquare.color = config.color;
  }
  if (config.width) {
    newSquare.area = config.width * config.width;
  }
  return newSquare;
}

var mySquare = createSquare({color: "black"});
```

可选属性能让编译器检查出属性名的错拼：

```
interface SquareConfig {
  color?: string;
  width?: number;
}

function createSquare(config: SquareConfig): {color: string; area: number} {
  var newSquare = {color: "white", area: 100};
  if (config.color) {
    newSquare.color = config.collor; // Type-checker can catch the mistyped name here
  }
  if (config.width) {
    newSquare.area = config.width * config.width;
  }
  return newSquare;
}

var mySquare = createSquare({color: "black"});
```

### 函数类型

接口不仅能描述含有属性的对象，也能描述函数类型。

To describe a function type with an interface, we give the interface a call signature. This is like a function declaration with only the parameter list and return type given.

```
interface SearchFunc {
  (source: string, subString: string): boolean;
}
```

例子，创建该函数类型的变量：

```
var mySearch: SearchFunc;
mySearch = function(source: string, subString: string) {
  var result = source.search(subString);
  if (result == -1) {
    return false;
  }
  else {
    return true;
  }
}
```

函数类型的类型检查，不关心参数名。

### 数组类型

接口描述一个数组类型。描述一个数组类型，需要指出 `index` 的类型，和数组中元素的类型。

```
interface StringArray {
	[index: number]: string;
}

var myArray: StringArray;
myArray = ["Bob", "Fred"];
```

`index` 支持两种类型：`string` 和 `number`。It is possible to support both types of index, with the restriction that the type returned from the numeric index must be a subtype of the type returned from the string index.

While index signatures are a powerful way to describe the array and 'dictionary' pattern, they also enforce that all properties match their return type. In this example, the property does not match the more general index, and the type-checker gives an error:

```
interface Dictionary {
    [index: string]: string;
    length: number; // error, the type of 'length' is not a subtype of the indexer
}
```

### 类类型

#### 实现一个接口

让类实现一个接口：

```
interface ClockInterface {
    currentTime: Date;
    setTime(d: Date);
}

class Clock implements ClockInterface  {
    currentTime: Date;
    setTime(d: Date) {
        this.currentTime = d;
    }
    constructor(h: number, m: number) { }
}
```

Interfaces describe the public side of the class, rather than both the public and private side. This prohibits you from using them to check that a class also has particular types for the private side of the class instance.

#### 类的静态/实例部分的区别

When working with classes and interfaces, it helps to keep in mind that a class has two types: the type of the static side and the type of the instance side. You may notice that if you create an interface with a construct signature and try to create a class that implements this interface you get an error:

```
interface ClockInterface {
    new (hour: number, minute: number);
}

class Clock implements ClockInterface  {
    currentTime: Date;
    constructor(h: number, m: number) { }
}
```

因为当类实现一个接口，只有类的实例的部分被检查。因为构造器属于类的静态的部分，因此不会被检查。

Instead, you would need to work with the 'static' side of the class directly. In this example, we work with the class directly:

```
interface ClockStatic {
    new (hour: number, minute: number);
}

class Clock {
    currentTime: Date;
    constructor(h: number, m: number) { }
}

var cs: ClockStatic = Clock;
var newClock = new cs(7, 30);
```

### 扩展接口

接口可以扩展一个或多个接口。

```
    interface Shape {
        color: string;
    }

    interface PenStroke {
        penWidth: number;
    }

    interface Square extends Shape, PenStroke {
        sideLength: number;
    }

    var square = <Square>{};
    square.color = "blue";
    square.sideLength = 10;
    square.penWidth = 5.0;
```

### 混合（Hybrid）类型

一个对象可能是上述几种类型的组合。例如一个函数可以待属性：

```
interface Counter {
    (start: number): string;
    interval: number;
    reset(): void;
}

var c: Counter;
c(10);
c.reset();
c.interval = 5.0;
```

When interacting with 3rd-party JavaScript, you may need to use patterns like the above to fully-describe the shape of the type.

## 类

### Classes

例子：

```
class Greeter {
    greeting: string;
    constructor(message: string) {
        this.greeting = message;
    }
    greet() {
        return "Hello, " + this.greeting;
    }
}
var greeter = new Greeter("world");
```

上面的类有三个成员，一个属性，一个构造器和一个方法。注意在类内，通过 `this.` 引用类的成员。最后我们用 `new` 实例化一个对象。

### 继承

```
class Animal {
    name:string;
    constructor(theName: string) { this.name = theName; }
    move(meters: number = 0) {
        alert(this.name + " moved " + meters + "m.");
    }
}

class Snake extends Animal {
    constructor(name: string) { super(name); }
    move(meters = 5) {
        alert("Slithering...");
        super.move(meters);
    }
}

class Horse extends Animal {
    constructor(name: string) { super(name); }
    move(meters = 45) {
        alert("Galloping...");
        super.move(meters);
    }
}

var sam = new Snake("Sammy the Python");
var tom: Animal = new Horse("Tommy the Palomino");

sam.move();
tom.move(34);
```

利用 `extends` 创建子类。可以覆盖父类方法。

### 公有/私有修饰符

成员默认是 `public`。You may still mark members a `private`.

```
class Animal {
    private name:string;
    constructor(theName: string) { this.name = theName; }
    move(meters: number) {
        alert(this.name + " moved " + meters + "m.");
    }
}
```

#### 私有与类型兼容

当我们判断两个不同类型是否兼容时，不论它们来自哪里，如果每个成员的类型是兼容的，则两个类型自己是兼容的。

对于含有私有成员的类型，有一些不同：For two types to be considered compatible, if one of them has a private member, then the other must have a private member that originated in the same declaration.

Let's look at an example to better see how this plays out in practice:

```
class Animal {
    private name:string;
    constructor(theName: string) { this.name = theName; }
}

class Rhino extends Animal {
	constructor() { super("Rhino"); }
}

class Employee {
    private name:string;
    constructor(theName: string) { this.name = theName; }
}

var animal = new Animal("Goat");
var rhino = new Rhino();
var employee = new Employee("Bob");

animal = rhino;
animal = employee; //error: Animal and Employee are not compatible
```

因为 `Animal` 和 `Rhino` 的私有部分来自相同声明：`Animal` 的 `private name: string`，因此它们是兼容的。
`Employee` 看起来跟 `Animal` 一样。但把一个 `Employee` 对象赋给一个 `Animal` 类型时会报错：二者不兼容。尽管 `Employee` 也有一个私有成员 `name`，它与 `Animal` 的并不相同。

#### 参数属性

利用 `public` 和 `private` 可以直接创建并初始化一个成员，称为 parameter properties。

```
class Animal {
    constructor(private name: string) { }
    move(meters: number) {
        alert(this.name + " moved " + meters + "m.");
    }
}
```

### Accessors

TypeScript 支持 getters/setters，作为拦截访问的方式。例子：

```
var passcode = "secret passcode";

class Employee {
    private _fullName: string;

    get fullName(): string {
        return this._fullName;
    }
    set fullName(newName: string) {
        if (passcode && passcode == "secret passcode") {
            this._fullName = newName;
        }
        else {
            alert("Error: Unauthorized update of employee!");
        }
    }
}

var employee = new Employee();
employee.fullName = "Bob Smith";
if (employee.fullName) {
    alert(employee.fullName);
}
```

> Note: Accessors require you to set the compiler to output ECMAScript 5.

### 静态属性

静态成员标准 `static`。通过类名访问：

```
class Grid {
    static origin = {x: 0, y: 0};
    calculateDistanceFromOrigin(point: {x: number; y: number;}) {
        var xDist = (point.x - Grid.origin.x);
        var yDist = (point.y - Grid.origin.y);
        return Math.sqrt(xDist * xDist + yDist * yDist) / this.scale;
    }
    constructor (public scale: number) { }
}

var grid1 = new Grid(1.0);  // 1x scale
var grid2 = new Grid(5.0);  // 5x scale

alert(grid1.calculateDistanceFromOrigin({x: 10, y: 10}));
alert(grid2.calculateDistanceFromOrigin({x: 10, y: 10}));
```

### 高级技术

#### 构造器函数

类的声明创造了两项内容：表示类的实例的类型，和一个构造函数。

```
class Greeter {
    static standardGreeting = "Hello, there";
    greeting: string;
    greet() {
    }
}

var greeterMaker: typeof Greeter = Greeter;
greeterMaker.standardGreeting = "Hey there!";
var greeter2:Greeter = new greeterMaker();
alert(greeter2.greet());
```

`greeterMaker` 持有类自身。`typeof Greeter` 给出的是 `Greeter` 类自身的类型（一个类，或者说一个构造器函数）。This type will contain all of the static members of Greeter along with the constructor that creates instances of the Greeter class.

#### 类用作接口

因为类创造了一种“类型”，因此使用接口的地方也可以使用类。

```
class Point {
    x: number;
    y: number;
}

interface Point3d extends Point {
    z: number;
}

var point3d: Point3d = {x: 1, y: 2, z: 3};
```

## 模块

This post outlines the various ways to organize your code using modules in TypeScript. We'll be covering internal and external modules and we'll discuss when each is appropriate and how to use them. We'll also go over some advanced topics of how to use external modules, and address some common pitfalls when using modules in TypeScript.

示例程序，用于后面的讨论：

```
interface StringValidator {
    isAcceptable(s: string): boolean;
}

var lettersRegexp = /^[A-Za-z]+$/;
var numberRegexp = /^[0-9]+$/;

class LettersOnlyValidator implements StringValidator {
    isAcceptable(s: string) {
        return lettersRegexp.test(s);
    }
}

class ZipCodeValidator implements StringValidator {
    isAcceptable(s: string) {
        return s.length === 5 && numberRegexp.test(s);
    }
}

// Some samples to try
var strings = ['Hello', '98052', '101'];
// Validators to use
var validators: { [s: string]: StringValidator; } = {};
validators['ZIP code'] = new ZipCodeValidator();
validators['Letters only'] = new LettersOnlyValidator();
// Show whether each string passed each validator
strings.forEach(s => {
    for (var name in validators) {
        console.log('"' + s + '" ' + (validators[name].isAcceptable(s) ? ' matches ' : ' does not match ') + name);
    }
});
```

As we add more validators, we're going to want to have some kind of organization scheme so that we can keep track of our types and not worry about name collisions with other objects. Instead of putting lots of different names into the global namespace, let's wrap up our objects into a module.

In this example, we've moved all the Validator-related types into a module called `Validation`. Because we want the interfaces and classes here to be visible outside the module, we preface them with `export`. Conversely, the variables lettersRegexp and numberRegexp are implementation details, so they are left unexported and will not be visible to code outside the module. In the test code at the bottom of the file, we now need to qualify the names of the types when used outside the module, e.g. `Validation.LettersOnlyValidator`.

```
module Validation {
    export interface StringValidator {
        isAcceptable(s: string): boolean;
    }

    var lettersRegexp = /^[A-Za-z]+$/;
    var numberRegexp = /^[0-9]+$/;

    export class LettersOnlyValidator implements StringValidator {
        isAcceptable(s: string) {
            return lettersRegexp.test(s);
        }
    }

    export class ZipCodeValidator implements StringValidator {
        isAcceptable(s: string) {
            return s.length === 5 && numberRegexp.test(s);
        }
    }
}

// Some samples to try
var strings = ['Hello', '98052', '101'];
// Validators to use
var validators: { [s: string]: Validation.StringValidator; } = {};
validators['ZIP code'] = new Validation.ZipCodeValidator();
validators['Letters only'] = new Validation.LettersOnlyValidator();
// Show whether each string passed each validator
strings.forEach(s => {
    for (var name in validators) {
        console.log('"' + s + '" ' + (validators[name].isAcceptable(s) ? ' matches ' : ' does not match ') + name);
    }
});
```

### 多文件

Here, we've split our Validation module across many files. Even though the files are separate, they can each contribute to the same module and can be consumed as if they were all defined in one place. Because there are dependencies between files, we've added reference tags to tell the compiler about the relationships between the files. Our test code is otherwise unchanged.

Validation.ts

```
module Validation {
    export interface StringValidator {
        isAcceptable(s: string): boolean;
    }
}
```

LettersOnlyValidator.ts

```
    /// <reference path="Validation.ts" />
    module Validation {
        var lettersRegexp = /^[A-Za-z]+$/;
        export class LettersOnlyValidator implements StringValidator {
            isAcceptable(s: string) {
                return lettersRegexp.test(s);
            }
        }
    }
```

ZipCodeValidator.ts

```
    /// <reference path="Validation.ts" />
    module Validation {
        var numberRegexp = /^[0-9]+$/;
        export class ZipCodeValidator implements StringValidator {
            isAcceptable(s: string) {
                return s.length === 5 && numberRegexp.test(s);
            }
        }
    }
```

Test.ts

```
    /// <reference path="Validation.ts" />
    /// <reference path="LettersOnlyValidator.ts" />
    /// <reference path="ZipCodeValidator.ts" />

    // Some samples to try
    var strings = ['Hello', '98052', '101'];
    // Validators to use
    var validators: { [s: string]: Validation.StringValidator; } = {};
    validators['ZIP code'] = new Validation.ZipCodeValidator();
    validators['Letters only'] = new Validation.LettersOnlyValidator();
    // Show whether each string passed each validator
    strings.forEach(s => {
        for (var name in validators) {
            console.log('"' + s + '" ' + (validators[name].isAcceptable(s) ? ' matches ' : ' does not match ') + name);
        }
    });
```

Once there are multiple files involved, we'll need to make sure all of the compiled code gets loaded. There are two ways of doing this.

First, we can use concatenated output using the `--out` flag to compile all of the input files into a single JavaScript output file:

	tsc --out sample.js Test.ts

The compiler will automatically order the output file based on the reference tags present in the files. You can also specify each file individually:

	tsc --out sample.js Validation.ts LettersOnlyValidator.ts ZipCodeValidator.ts Test.ts

Alternatively, we can use per-file compilation (the default) to emit one JavaScript file for each input file. If multiple JS files get produced, we'll need to use `<script>` tags on our webpage to load each emitted file in the appropriate order, for example:

MyTestPage.html (excerpt)

```
	<script src="Validation.js" type="text/javascript" />
    <script src="LettersOnlyValidator.js" type="text/javascript" />
    <script src="ZipCodeValidator.js" type="text/javascript" />
    <script src="Test.js" type="text/javascript" />
```

### Going External

TypeScript also has the concept of an external module. External modules are used in two cases: **node.js** and **require.js**. Applications not using node.js or require.js do not need to use external modules and can best be organized using the internal module concept outlined above.

In external modules, relationships between files are specified in terms of imports and exports **at the file level**. In TypeScript, any file containing a top-level import or export is considered an external module.

Below, we have converted the previous example to use external modules. Notice that we no longer use the `module` keyword – the files themselves constitute a module and are identified by their filenames.

The reference tags have been replaced with `import` statements that specify the dependencies between modules. The `import` statement has two parts: the name that the module will be known by in this file, and the `require` keyword that specifies the path to the required module:

```
import someMod = require('someModule');
```

We specify which objects are visible outside the module by using the `export` keyword on a top-level declaration, similarly to how export defined the public surface area of an internal module.

To compile, we must specify a module target on the command line. For node.js, `use --module commonjs`; for require.js, `use --module amd`. For example:

	tsc --module commonjs Test.ts

When compiled, each external module will become a separate .js file. Similar to reference tags, the compiler will follow import statements to compile dependent files.

Validation.ts

```
export interface StringValidator {
    isAcceptable(s: string): boolean;
}
```

LettersOnlyValidator.ts

```
import validation = require('./Validation');
var lettersRegexp = /^[A-Za-z]+$/;
export class LettersOnlyValidator implements validation.StringValidator {
    isAcceptable(s: string) {
        return lettersRegexp.test(s);
    }
}
```

ZipCodeValidator.ts

```
import validation = require('./Validation');
var numberRegexp = /^[0-9]+$/;
export class ZipCodeValidator implements validation.StringValidator {
    isAcceptable(s: string) {
        return s.length === 5 && numberRegexp.test(s);
    }
}
```

Test.ts

```
import validation = require('./Validation');
import zip = require('./ZipCodeValidator');
import letters = require('./LettersOnlyValidator');

// Some samples to try
var strings = ['Hello', '98052', '101'];
// Validators to use
var validators: { [s: string]: validation.StringValidator; } = {};
validators['ZIP code'] = new zip.ZipCodeValidator();
validators['Letters only'] = new letters.LettersOnlyValidator();
// Show whether each string passed each validator
strings.forEach(s => {
    for (var name in validators) {
        console.log('"' + s + '" ' + (validators[name].isAcceptable(s) ? ' matches ' : ' does not match ') + name);
    }
});
```

#### Code Generation for External Modules

Depending on the module target specified during compilation, the compiler will generate appropriate code for either node.js (commonjs) or require.js (AMD) module-loading systems. For more information on what the `define` and `require` calls in the generated code do, consult the documentation for each module loader.

This simple example shows how the names used during importing and exporting get translated into the module loading code.

SimpleModule.ts

```
import m = require('mod');
export var t = m.something + 1;
```

AMD / RequireJS SimpleModule.js:

```
define(["require", "exports", 'mod'], function(require, exports, m) {
    exports.t = m.something + 1;
});
```

CommonJS / Node SimpleModule.js:

```
var m = require('mod');
exports.t = m.something + 1;
```

### Export =

In the previous example, when we consumed each validator, each module only exported one value. In cases like this, it's cumbersome to work with these symbols through their qualified name when a single identifier would do just as well.

The `export =` syntax specifies a single object that is exported from the module. This can be a class, interface, module, function, or enum. When imported, the exported symbol is consumed directly and is not qualified by any name.

Below, we've simplified the **Validator** implementations to only export a single object from each module using the `export =` syntax. This simplifies the consumption code – instead of referring to 'zip.ZipCodeValidator', we can simply refer to 'zipValidator'.

Validation.ts

```
export interface StringValidator {
    isAcceptable(s: string): boolean;
}
```

LettersOnlyValidator.ts

```
import validation = require('./Validation');
var lettersRegexp = /^[A-Za-z]+$/;
class LettersOnlyValidator implements validation.StringValidator {
    isAcceptable(s: string) {
        return lettersRegexp.test(s);
    }
}
export = LettersOnlyValidator;
```

ZipCodeValidator.ts

```
import validation = require('./Validation');
var numberRegexp = /^[0-9]+$/;
class ZipCodeValidator implements validation.StringValidator {
    isAcceptable(s: string) {
        return s.length === 5 && numberRegexp.test(s);
    }
}
export = ZipCodeValidator;
```

Test.ts

```
import validation = require('./Validation');
import zipValidator = require('./ZipCodeValidator');
import lettersValidator = require('./LettersOnlyValidator');

// Some samples to try
var strings = ['Hello', '98052', '101'];
// Validators to use
var validators: { [s: string]: validation.StringValidator; } = {};
validators['ZIP code'] = new zipValidator();
validators['Letters only'] = new lettersValidator();
// Show whether each string passed each validator
strings.forEach(s => {
    for (var name in validators) {
        console.log('"' + s + '" ' + (validators[name].isAcceptable(s) ? ' matches ' : ' does not match ') + name);
    }
});
```

### Alias

Another way that you can simplify working with either kind of module is to use `import q = x.y.z` to create shorter names for commonly-used objects. Not to be confused with the `import x = require('name')` syntax used to load external modules, this syntax simply creates an alias for the specified symbol. You can use these sorts of imports (commonly referred to as aliases) for any kind of identifier, including objects created from external module imports.

```
module Shapes {
    export module Polygons {
        export class Triangle { }
        export class Square { }
    }
}

import polygons = Shapes.Polygons;
var sq = new polygons.Square(); // Same as 'new Shapes.Polygons.Square()'
```

Notice that we don't use the `require` keyword; instead we assign directly from the qualified name of the symbol we're importing. This is similar to using var, but also works on the type and namespace meanings of the imported symbol. Importantly, for values, import is a distinct reference from the original symbol, so changes to an aliased var will not be reflected in the original variable.

### Optional Module Loading and Other Advanced Loading Scenarios

In some cases, you may want to only load a module under some conditions. In TypeScript, we can use the pattern shown below to implement this and other advanced loading scenarios to directly invoke the module loaders without losing type safety.

The compiler detects whether each module is used in the emitted JavaScript. For modules that are only used as part of the type system, no require calls are emitted. This culling of unused references is a good performance optimization, and also allows for optional loading of those modules.

The core idea of the pattern is that the `import id = require('...')` statement gives us access to the types exposed by the external module. The module loader is invoked (through `require`) dynamically, as shown in the if blocks below. This leverages the reference-culling optimization so that the module is only loaded when needed. For this pattern to work, it's important that the symbol defined via import is only used in type positions (i.e. never in a position that would be emitted into the JavaScript).

To maintain type safety, we can use the `typeof` keyword. The `typeof` keyword, when used in a type position, produces the type of a value, in this case the type of the external module.

Dynamic Module Loading in node.js

```
declare var require;
import Zip = require('./ZipCodeValidator');
if (needZipValidation) {
    var x: typeof Zip = require('./ZipCodeValidator');
    if (x.isAcceptable('.....')) { /* ... */ }
}
```

Sample: Dynamic Module Loading in require.js

```
declare var require;
import Zip = require('./ZipCodeValidator');
if (needZipValidation) {
    require(['./ZipCodeValidator'], (x: typeof Zip) => {
        if (x.isAcceptable('...')) { /* ... */ }
    });
}
```

### Working with Other JavaScript Libraries

To describe the shape of libraries not written in TypeScript, we need to declare the API that the library exposes. Because most JavaScript libraries expose only a few top-level objects, modules are a good way to represent them. We call declarations that don't define an implementation "ambient". Typically these are defined in `.d.ts` files. If you're familiar with C/C++, you can think of these as .h files or 'extern'. Let's look at a few examples with both internal and external examples.

**Ambient Internal Modules**

The popular library **D3** defines its functionality in a global object called 'D3'. Because this library is loaded through a script tag (instead of a module loader), its declaration uses internal modules to define its shape. For the TypeScript compiler to see this shape, we use an ambient internal module declaration. For example:

D3.d.ts (simplified excerpt)

```
declare module D3 {
    export interface Selectors {
        select: {
            (selector: string): Selection;
            (element: EventTarget): Selection;
        };
    }

    export interface Event {
        x: number;
        y: number;
    }

    export interface Base extends Selectors {
        event: Event;
    }
}

declare var d3: D3.Base;
```

**Ambient External Modules**

In node.js, most tasks are accomplished by loading one or more modules. We could define each module in its own `.d.ts` file with top-level export declarations, but it's more convenient to write them as one larger `.d.ts` file. To do so, we use the quoted name of the module, which will be available to a later import. For example:

node.d.ts (simplified excerpt)

```
declare module "url" {
    export interface Url {
        protocol?: string;
        hostname?: string;
        pathname?: string;
    }

    export function parse(urlStr: string, parseQueryString?, slashesDenoteHost?): Url;
}

declare module "path" {
    export function normalize(p: string): string;
    export function join(...paths: any[]): string;
    export var sep: string;
}
```

Now we can `/// <reference> node.d.ts` and then load the modules using e.g. `import url = require('url');`.

```
    ///<reference path="node.d.ts"/>
    import url = require("url");
    var myUrl = url.parse("http://www.typescriptlang.org");
```

### Pitfalls of Modules

In this section we'll describe various common pitfalls in using internal and external modules, and how to avoid them.

#### `/// <reference>` to an external module

A common mistake is to try to use the `/// <reference>` syntax to refer to an external module file, rather than using import. To understand the distinction, we first need to understand the three ways that the compiler can locate the type information for an external module.

The first is by finding a .ts file named by an import x = require(...); declaration. That file should be an implementation file with top-level import or export declarations.

The second is by finding a .d.ts file, similar to above, except that instead of being an implementation file, it's a declaration file (also with top-level import or export declarations).

The final way is by seeing an "ambient external module declaration", where we 'declare' a module with a matching quoted name.

myModules.d.ts

```
// In a .d.ts file or .ts file that is not an external module:
declare module "SomeModule" {
    export function fn(): string;
}
```

myOtherModule.ts

```
    /// <reference path="myModules.d.ts" />
    import m = require("SomeModule");
````

The reference tag here allows us to locate the declaration file that contains the declaration for the ambient external module. This is how the node.d.ts file that several of the TypeScript samples use is consumed, for example.

### Needless Namespacing

If you're converting a program from internal modules to external modules, it can be easy to end up with a file that looks like this:

shapes.ts

```
export module Shapes {
    export class Triangle { /* ... */ }
    export class Square { /* ... */ }
}
```

The top-level module here Shapes wraps up Triangle and Square for no reason. This is confusing and annoying for consumers of your module:

shapeConsumer.ts

```
import shapes = require('./shapes');
var t = new shapes.Shapes.Triangle(); // shapes.Shapes?
```

A key feature of external modules in TypeScript is that two different external modules will never contribute names to the same scope. Because the consumer of an external module decides what name to assign it, there's no need to proactively wrap up the exported symbols in a namespace.

To reiterate why you shouldn't try to namespace your external module contents, the general idea of namespacing is to provide logical grouping of constructs and to prevent name collisions. Because the external module file itself is already a logical grouping, and its top-level name is defined by the code that imports it, it's unnecessary to use an additional module layer for exported objects.

Revised Example:

shapes.ts

```
export class Triangle { /* ... */ }
export class Square { /* ... */ }
```
shapeConsumer.ts

```
import shapes = require('./shapes');
var t = new shapes.Triangle();
```

### Trade-offs for External Modules

Just as there is a one-to-one correspondence between JS files and modules, TypeScript has a one-to-one correspondence between external module source files and their emitted JS files. One effect of this is that it's not possible to use the --out compiler switch to concatenate multiple external module source files into a single JavaScript file.