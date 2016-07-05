[toc]

本文档针对：1.9.3
最新更新：2015年08月15日

## 基础

CoffeeScript 使用空白符区分代码块。不需要使用分号，行尾就是分隔。（分号用于分隔一行上的多个表达式）。使用缩进（而不是大括号）围绕 function、if、switch 和 try/catch 的代码块。

### 声明变量

不需要、不要使用 `var` 关键字。

CoffeeScript的输出会被包裹进一个匿名函数：`(function(){ ... })()`。包裹与自动产生`var`关键字，避免了不小心污染全局命名空间的情况。

```
outer = 1
changeNumbers = ->
  inner = -1
  outer = 10
inner = changeNumbers()
```
所有的变量声明都会都放在最近作用域（变量首次出现的）的顶端。

无法直接使用`var`，就无法掩盖外部变量。

如果想创建全局变量，可以将它们作为`window`对象的属性，或`exports`对象的属性（CommonJS环境）。

### 对象和数组字面量

```
bitlist = [
  1, 0, 1
  0, 0, 1
  1, 1, 0
]

kids =
  brother:
    name: "Max"
    age:  11
  sister:
    name: "Ida"
    age:  9
```

JavaScript中，保留字，如`class`，不能做对象属性。CoffeeScript允许。

```
$('.account').attr class: 'active'

log object.class
```

## 运算符与表达式

Because variable declarations occur at the top of scope, assignment can be used within expressions, even for variables that haven't been seen before:

```
six = (one = 1) + (two = 2) + (three = 3)
```

### 一切都是表达式

CoffeeScript编译器尝试将一些语句都看做表达式。

```
grade = (student) ->
  if student.excellentWork
    "A+"
  else if student.okayStuff
    if student.triedHard then "B" else "B-"
  else
    "C"

eldest = if 24 > 21 then "Liz" else "Ike"

alert(
  try
    nonexistent / undefined
  catch error
    "And the error is ... #{error}"
)
```

有一些Javascript语句不能转换成表达式，如`break`, `continue`, `return`。{{但并不表示它们不能在Coffee中。可以正常使用。只是它们本身不会被转换为表达式。它们不影响所在的for循环等，转换成表达式。}}

### 后缀表达式

一个使用后缀表达式改写的例子。改写前：

```
odd = (num) ->
    if typeof num is 'number'
        if num is Math.round num
            if num > 0
                num % 2 is 1
            else
                throw "#{num} is not positive"
        else
            throw "#{num} is not an integer"
    else
        throw "#{num} is not a number"
```

改写后：

```
odd = (num) ->
    unless typeof num is 'number'
        throw "#{num} is not a number"
    unless num is Math.round num
        throw "#{num} is not an integer"
    unless num >0
        throw "#{num} is not positive"
    num % 2 is 1
```

### 三元运算符用if/then/else表示

CoffeeScript can compile if statements into JavaScript expressions, using the ternary operator when possible, and closure wrapping otherwise. There is no explicit ternary statement in CoffeeScript — you simply use a regular if statement on a single line.

```
date = if friday then sue else jill   <=>   date = friday ? sue : jill;
```

### 运算符与别名

CoffeeScript 将 `==` 编译成 `===`，将 `!=` 编译成 `!==`。将 `is` 编译成 `===`，将 `isnt` 编译成`!==`。

`not` 是 `!` 的别名。`and` 编译成 `&&`， `or` 编译成 `||`。

在while、if/else、switch/when语句中，`then`可以代替换行或分号，分隔条件和表达式。

像YAML一样，`on`和`yes`与`true`等价，`off`和`no`与`false`等价。

`unless`是`if`的反义语句。

`in`可以测试数组元素的存在性。`of`测试Javascript对象键的存在性。

`**` 用于乘方（`Math.pow(a, b)`）。`//` 用于整除（`Math.floor(a / b)`）。`%%` provides “[dividend dependent modulo](http://en.wikipedia.org/wiki/Modulo_operation)”:

```
-7 % 5 == -2 # The remainder of 7 / 5
-7 %% 5 == 3 # n %% 5 is always between 0 and 4
```

注意，`+`用于连接字符串时需要在两端留空格。下面的写法会报错：

```
color = 'red'
x = color +5
```

因为`color +5`会被编译为`color(+5)`。因为前缀`+`是字符串转换为数字的捷径。

### 存在运算符与条件赋值

Javascript中没有判断变量存在的运算符。`if (variable) ...`在变量是0、空串、false的情况下也都不执行。CoffeeScript的存在运算符 `?` 仅当变量不是`null`或`undefined`时返回true。

```
solipsism = true if mind? and not world?
```

对于条件赋值，存在运算符比`||=`更安全：特别是处理数字或字符串时。

```
speed = 0
speed ?= 15

footprints = yeti ? "bear"
```

### 安全属性访问

`?.`代替`.`，可以用于消除属性链中的空指针。如果属性链中有`null`或`undefined`，将返回`undefined`。

```
zip = lottery.drawWinner?().address?.zipcode
```

{{用于数组：`a?[0]`}}

### Splats...

CoffeeScript provides splats `...`, both for function definition as well as invocation, making variable numbers of arguments a little bit more palatable.

```
gold = silver = rest = "unknown"

awardMedals = (first, second, others...) ->
  gold   = first
  silver = second
  rest   = others

contenders = [
  "Michael Phelps"
  "Liu Xiang"
  "Yao Ming"
  "Allyson Felix"
  "Shawn Johnson"
  "Roman Sebrle"
  "Guo Jingjing"
  "Tyson Gay"
  "Asafa Powell"
  "Usain Bolt"
]

awardMedals contenders...
```

在函数调用（不是声明！）中，Splats可以将一个数组**展开**为一个参数列表：

```
coffee> console.log 1, 2, 3, 4
1 2 3 4
coffee> arr = [1, 2, 3]
coffee> console.log arr, 4
[ 1, 2, 3] 4
coffee> console.log arr..., 4
1 2 3 4
```

Splats不一定非要呆在参数列表的最后：
`sandwich = (beginning, middle..., end)->`

非Splats参数具有更高优先权。因此，如果实参只有2个，则`beginning`和`end`获得。

一个函数只有一个Splats参数才有意义。

### 解构赋值

To make extracting values from complex arrays and objects more convenient, CoffeeScript implements ECMAScript Harmony's proposed destructuring assignment syntax.

```js
theBait   = 1000
theSwitch = 0

[theBait, theSwitch] = [theSwitch, theBait]
```

可以让函数返回多个值：

```
weatherReport = (location) ->
  [location, 72, "Mostly Sunny"]

[city, temp, forecast] = weatherReport "Berkeley, CA"
```

可以用于任意深度的对象和数组嵌套：

```
futurists =
  sculptor: "Umberto Boccioni"
  painter:  "Vladimir Burliuk"
  poet:
    name:   "F.T. Marinetti"
    address: [
      "Via Roma 42R"
      "Bellagio, Italy 22021"
    ]

{poet: {name, address: [street, city]}} = futurists
```

**与splats结合**：

```
    tag = "<impossible>"
    [open, contents..., close] = tag.split("")
```

Expansion can be used to retrieve elements from the end of an array without having to assign the rest of its values. **It works in function parameter lists as well.**

```
text = "Every literary critic believes he will
        outwit history and have the last word"

[first, ..., last] = text.split " "
```

解构赋值用在类的构造器，将选项赋给成员变量。

```
class Person
  constructor: (options) ->
    {@name, @age, @height} = options

tim = new Person age: 4
```

### Switch/When/Else

CoffeeScript会自动加break。switch也是一个表达式。每个when子句可以带多个值，之间是或关系。

```
switch day
  when "Mon" then go work
  when "Tue" then go relax
  when "Thu" then go iceFishing
  when "Fri", "Sat"
    if day is bingoDay
      go bingo
      go dancing
  when "Sun" then go church
  else go work
```

Switch子句可以不带控制表达式，这时它变成了类似 if/else 链的东西。

```
score = 76
grade = switch
  when score < 60 then 'F'
  when score < 70 then 'D'
  when score < 80 then 'C'
  when score < 90 then 'B'
  else 'A'
```

### Try/Catch/Finally

语义与Javascript类似。在CoffeeScript中，可以省略 catch 和 finally 两部分。catch 部分若不关心错误对象，可以省略。

```
try
  allHellBreaksLoose()
  catsAndDogsLivingTogether()
catch error
  print error
finally
  cleanUp()
```

### 链式比较

```
cholesterol = 127
healthy = 200 > cholesterol > 60
```

## 字符串插值、多行字符串、块字符串、块注释

Double-quoted strings allow for interpolated values, using `#{ ... }`, and single-quoted strings are literal. **You may even use interpolation in object keys**.

```
author = "Wittgenstein"
quote  = "A picture is a fact. -- #{ author }"

sentence = "#{ 22 / 7 } is a decent approximation of π"
```

---

CoffeeScript允许多行字符串。Lines are joined by a single space unless they end with a backslash. Indentation is ignored.

```
mobyDick = "Call me Ishmael. Some years ago --
  never mind how long precisely -- having little
  or no money in my purse, and nothing particular
  to interest me on shore, I thought I would sail
  about a little and see the watery part of the
  world..."
```

---

> 注意区别：多行字符串（不保留空白符、缩进）和块字符串（保留）

Block strings can be used to hold formatted or indentation-sensitive text (or, if you just don't feel like escaping quotes and apostrophes). The indentation level that begins the block is maintained throughout, so you can keep it all aligned with the body of your code.

```
html = """
       <strong>
         cup of coffeescript
       </strong>
       """
```

{{注意有回车}}

```
    var html;
    html = "<strong>\n  cup of coffeescript\n</strong>";
```

Double-quoted block strings, like other double-quoted strings, allow interpolation.

---

Sometimes you'd like to pass a block comment **through** to the generated JavaScript. For example, when you need to embed a licensing header at the top of a file. Block comments, which mirror the syntax for block strings, are **preserved** in the generated code.

```
    ###
    SkinnyMochaHalfCaffScript Compiler v1.0
    Released under the MIT License
    ###
```

### 块正则表达式

Similar to block strings and comments, CoffeeScript supports block regexes — extended regular expressions that ignore internal whitespace and can contain comments and interpolation. CoffeeScript's block regexes are delimited by `///` and go a long way towards making complex regular expressions readable. To quote from the CoffeeScript source:

```
OPERATOR = /// ^ (
  ?: [-=]>             # function
   | [-+*/%<>&|^!?=]=  # compound assign / compare
   | >>>=?             # zero-fill right shift
   | ([-+:])\1         # doubles
   | ([&|<>])\2=?      # logic / shift
   | \?\.              # soak access
   | \.{2,3}           # range or splat
) ///
```

## 范围、循环与Comprehensions

Most of the loops you'll write in CoffeeScript will be comprehensions over arrays, objects, and ranges. Comprehensions replace (and compile into) for loops, with optional guard clauses and the value of the current array index. Unlike for loops, array comprehensions are expressions, and can be returned and assigned.

```
eat food for food in ['toast', 'cheese', 'wine']

courses = ['greens', 'caviar', 'truffles', 'roast', 'cake']
menu i + 1, dish for dish, i in courses

foods = ['broccoli', 'spinach', 'chocolate']
eat food for food in foods when food isnt 'chocolate'
```

Comprehension不仅能替代循环，还可以替代 **each/forEach**、 **map**、 **select/filter**。例如：`shortNames = (name for name in list when name.length < 5)`。

### 对象循环

Comprehensions can also be used to iterate over the keys and values in an object. Use of to signal comprehension over the properties of an object instead of the values in an array.

```
yearsOld = max: 10, ida: 9, tim: 11

ages = for child, age of yearsOld
  "#{child} is #{age}"
```

If you would like to iterate over just the keys that are defined on the object itself, by adding a `hasOwnProperty` check to avoid properties that may be inherited from the prototype, use `for own key, value of object`

### 范围

两个点表示包含最后的值，如`3..6`为(3, 4, 5, 6)。
三个点表示不包含最后的值，如`3...6`为(3, 4, 5)。

### 范围与循环

```
countdown = (num for num in [10..1])
```

To step through a range comprehension in fixed-size chunks, use by, for example:

`evens = (x for x in [0..10] by 2)`

If you don't need the current iteration value you may omit it:

`browser.closeCurrentTab() for [0...count]`

### 利用范围切片数组

Slices indices have useful defaults. An omitted first index defaults to zero and an omitted second index defaults to the size of the array.

```
numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9]
start   = numbers[0..2]
middle  = numbers[3...-2]
end     = numbers[-2..]
copy    = numbers[..]
```

在赋值左侧，范围可以修改数组连续几个值：

```
numbers = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
numbers[3..6] = [-3, -4, -5, -6]
```

Note that JavaScript strings are immutable, and can't be spliced.

### while

The only low-level loop that CoffeeScript provides is the while loop. while循环也是一个表达式，返回一个数组，包含每次循环的结果。

```
if this.studyingEconomics
  buy()  while supply > demand
  sell() until supply > demand

num = 6
lyrics = while num -= 1
  "#{num} little monkeys, jumping on the bed.
    One fell out and bumped his head."
```

For readability, the `until` keyword is equivalent to `while not`, and the `loop` keyword is equivalent to `while true`.

## 函数

### 函数定义

```
square = (x) -> x * x
cube   = (x) -> square(x) * x
```
在Javascript中有两种函数声明方式。一种是：

`var cube1 = function(x) {...}`

另一种是：

`function cube2 = function(x) {...}`

两种方式的最大区别是，如果在cube1定义前使用它会报错。但在cube2作用域范围内可以在定义前调用它。
CoffeeScript只会生成cube1这样的风格（由于IE的问题）。**因此在使用前一定要先定义**。

### do

直接调用函数。替代`(-> name = 'Ren')()`的写法是：
`do -> name = 'Ren'`

语法`do -> name = 'Ren'`，编译成js是：

```js
(function() {
    // <your compiled program goes here>
}).call(this);
```

在循环中定义函数，很可能要用到`do`。若想让内部函数绑定循环变量的当前值，而不是循环变量本身或者说其最终值。一般做法是包围一个函数，并利用循环变量当前值立即调用这个函数。

```
for filename in list
  do (filename) ->
    fs.readFile filename, (err, contents) ->
      compile filename, contents.toString()
```


### 隐式调用

有实参时，调用函数不必使用括号。The implicit call wraps forward to the end of the line or block expression.

```
console.log sys.inspect object → console.log(sys.inspect(object));
```

函数调用时省略括号。到表达式结尾，隐式括号才会闭合。如，一个错误的写法：
`console.log(Math.round 3.1, Math.round 5.2)`

会被解释为：
`console.log(Math.round(3.1, Math.round(5.2)))`

为避免混乱，只在**最外层函数调用时**省略括号：
`console.log Math.round(3.1), Math.round(5.2)# 3,5`

### 参数默认值

which will be used if the incoming argument is missing (null or undefined).

```
fill = (container, liquid = "coffee") ->
  "Filling the #{container} with #{liquid}..."
```

Coffee在幕后使用存在运算符，于是**传入`null`或`undefined`等同于省略参数**。

### 函数作用域

考虑下面两段代码的区别：

```
age = 99
fun1 = -> age = 0
fun1()
console.log age
```
和

```
fun1 = -> age = 0
age = 99
fun1()
console.log age
```

第一个输出0，但第二个输出99。通过`coffee -p`看产生的js发现完全不同。第二种写法，在函数内，`age`被重新定义成局部变量。

```js
(function() {
  var age, fun1;
  age = 99;
  fun1 = function() {
    return age = 0;
  };
  fun1();
  console.log("age is " + age);
}).call(this);
```

和

```js
(function() {
  var age, fun1;
  fun1 = function() {
    var age;
    return age = 0;
  };
  age = 99;
  fun1();
  console.log("age is " + age);
}).call(this);
```

应该尽力避免**同名覆盖**。

## 对象

### `@`缩写

As a shortcut for `this.property`, you can use `@property`.`@`后的点是可有可无的。

### 属性参数（`@arg`）

下面两个写法是等价的：

```
setName = (name)-> @name = name
setName = (@name)-> # no code required!
```

### 访问原型：`::`

`::` gives you quick access to an object's prototype.

```
String::dasherize = ->
  this.replace /_/g, "-"

// 等价于
String.prototype.dasherize = function() {
  return this.replace(/_/g, "-");
};
```

例子：

```coffee
Boy = -> # by convention, constructor names are capitalized
Boy::sing = -> console.log "It ain't easy being a boy named Sue"
sue = new Boy()
sue.sing()
```

### 已绑定函数、胖箭头

解决`this`的动态性问题。胖箭头`=>`定义的函数，将绑定到当前的`this`值。

```
Account = (customer, cart) ->
  @customer = customer
  @cart = cart

  $('.shopping_cart').on 'click', (event) =>
    @customer.purchase @cart
```

在类定义中使用时，使用胖箭头声明的方法，将自动绑定到类的实例。

## 类与继承

new的意义是，创建对象，将其原型设为构造器，并执行构造器。

```coffee
Gift = (@name) ->
	Gift.count++
	@day = Gift.count
    @announce()

Gift.count = 0
Gift::announce = ->
	console.log "On day #{@day} of Christmas I received #{@name}"

gift1 = new Gift('a partridge in a pear tree')
gift2 = new Gift('two turtle doves')
```

JavaScript内建的继承机制有一些缺陷，如不能调用super，或很难正确设置原型链。

CoffeeScript提供一套基础的`class`结构，可以命名类，设置父类，为原型中的属性赋值，定义构造器。

构造器函数是具名的，以更好的支持追踪栈 。下面的第一个例子，`this.constructor.name`的值是"Animal"。

```
class Animal
  constructor: (@name) ->

  move: (meters) ->
    alert @name + " moved #{meters}m."

class Snake extends Animal
  move: ->
    alert "Slithering..."
    super 5

class Horse extends Animal
  move: ->
    alert "Galloping..."
    super 45

sam = new Snake "Sammy the Python"
tom = new Horse "Tommy the Palomino"

sam.move()
tom.move()
```

上面介绍的`extends`运算符能够建立正确的原型，在任意**构造函数**之间建立继承关系。`super()`转换为调用上**一**级祖先的同名方法。

### 静态属性与实例属性

定义类时，就是在书写一个对象定义。其中的属性唯一不会进入原型的是`constructor`函数。

```coffee
class Tribble
	constructor: ->
		@isAlive = true
        Tribble.count++
	# Prototype properties
	breed: -> new Tribble if @isAlive
    die: ->
    	Tribble.count-- if @isAlive
        @isAlive = false
	# Class-level properties
	@count: 0
	@makeTrouble: -> console.log ('Trouble!' for i in [1..@count]).join(' ')
```

`count`变量（出现在`Tribble.count++`和`@count:0`），它是类的**静态**属性。注意到在方法中使用的时`Tribble.count`，二在类的上下文中使用的时`@count`。记住这里有三个对象要区别开：`Tribble`对象自己（实际是构造函数）、`Tribble.prototype`和`Tribble`的实例。默认Tribble属性（构造器除外）添加到原型。但如果使用`@`前缀，则表示属性添加到**类对象**自己。

### `super`

`super`与`super()`不同。例如下面的代码是错误的：

```coffee
class Appliance
	constructor: (warranty) ->
		warrantyDb.save(this) if warranty

class Toaster extends Appliance
	constructor: (warranty) ->
		super()
```

在子类构造器中使用`super()`，没有传入参数，则父类的构造器的`warranty`参数就没有值。
应该调用`super(warranty)`。或者缩写`super`，注意没有括号！会自动把当前函数的参数都带过去。

## Generator函数

CoffeeScript functions also support ES6 generator functions through the yield keyword. There's no `function*(){}` nonsense — a generator in CoffeeScript is simply a function that yields.

```
perfectSquares = ->
  num = 0
  loop
    num += 1
    yield num * num
  return

window.ps or= perfectSquares()
```

`yield*` is called `yield from`, and `yield return` may be used if you need to force a generator that doesn't yield.

## 嵌入Javascript

Hopefully, you'll never need to use it, but if you ever need to intersperse snippets of JavaScript within your CoffeeScript, you can use backticks to pass it straight through.

```
hi = `function() {
  return [document.title, "Hello JavaScript"].join(": ");
}`
```







