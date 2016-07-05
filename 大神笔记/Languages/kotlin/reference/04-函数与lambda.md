[toc]

# 函数与lambda

## 函数

### 函数声明

函数声明要加 `fun` 关键字：

```kotlinfun double(x: Int): Int {}
```

### 函数的使用

调用方式与 Java 一样，

```kotlinval result = double(2)

Sample().foo() // create instance of class Sample and calls foo
```

#### 中缀（Infix）

函数还可以使用中缀语法调用，只要- 它们是成员函数或扩展函数
- 它们只有一个参数
- 它们带 `infix` 关键字

```kotlin
// Define extension to Intinfix fun Int.shl(x: Int): Int {	...}// call extension function using infix notation1 shl 2// is the same as1.shl(2)
```

#### 参数

函数参数的定义采用 Pascal 的语法。如 `name: type`。参数类型必须显式声明。

```kotlinfun powerOf(number: Int, exponent: Int) {	...}
```

#### 默认实参

函数参数可以有默认值。默认值减少了重载的需要。

```kotlinfun read(b: Array<Byte>, off: Int = 0, len: Int = b.size()) {	...}
```

#### 命名实参

定义有具名的参数，

```kotlinfun reformat(str: String,	normalizeCase: Boolean = true,	upperCaseFirstLetter: Boolean = true,	divideByCamelHumps: Boolean = false,	wordSeparator: Char = ' ') {	...}
```
可以使用默认值调用：

```kotlinreformat(str)
```
或，

```kotlinreformat(str, true, true, false, '_')
```
具名调用：

```kotlinreformat(str, normalizeCase = true, uppercaseFirstLetter = true,
	divideByCamelHumps = false, wordSeparator = '_')
```
and if we do not need all arguments

```kotlin
reformat(str, wordSeparator = '_')
```
注意，命名实参不能用于调用 Java 函数，因为 Java 的字节码不总是保留函数参数的名字。

#### 返回 Unit 的函数

若函数不返回任何有用的值，它的返回类型是 `Unit`。`Unit` 是一种类型，且该类型只有一个值：`Unit`。This value does not have to be returned explicitly

```kotlinfun printHello(name: String?): Unit {	if (name != null)		println("Hello ${name}")	else		println("Hi there!")	// `return Unit` or `return` is optional}
```
The `Unit` return type declaration is also optional. The above code is equivalent to

```kotlinfun printHello(name: String?) {...}
```

#### 单个语句的函数

若函数只是返回一个表达式的值，括号可以省略，在表达式前加 `=`，

```kotlinfun double(x: Int): Int = x * 2
```
此时，函数的返回值类型声明也可以省略，
```kotlinfun double(x: Int) = x * 2
```

#### 显式声明返回类型

若函数正文是一个块，必须显式声明返回类型。Kotlin 不会推断函数块的返回类型。
例外：返回 `Unit` 时声明可以省略。

#### 可变数量参数（Varargs）

A parameter of a function (normally the last one) may be marked with `vararg` modifier:

```kotlinfun asList<T>(vararg ts: T): List<T> {	val result = ArrayList<T>()	for (t in ts) // ts is an Array		result.add(t)	return result}
```

允许可变数量的实参传入函数：

```kotlinval list = asList(1, 2, 3)
```
在函数内 `vararg` 参数是一个数组，即 `Array<out T>`。
最多只有一个参数可以被标记为 `vararg`。若 `vararg` 参数不是最后一个参数，后面的参数可以通过命名实参的语法传入；或者如果最后又一个函数类型的参数，在调用的括号后面放一个 lambda。
`vararg` 参数的实参，，除了可以一个个传递，也可以传入一个数组，并使用 **spread** 运算符展开：

```kotlinval a = array(1, 2, 3)val list = asList(-1, 0, *a, 4)
```

### 函数作用域

在 Kotlin 中函数可以在文件顶级声明，不用放在类中。In addition to top level functions, Kotlin functions can also be declared local, as member functions and extension functions.

#### 局部函数

Kotlin 支持局部函数，即函数中的函数，

```kotlinfun dfs(graph: Graph) {	fun dfs(current: Vertex, visited: Set<Vertex>) {		if (!visited.add(current)) return		for (v in current.neighbors)		dfs(v, visited)	}	dfs(graph.vertices[0], HashSet())}
```
局部函数可以访问外层函数的局部变量（即闭包），
局部函数可以令外部函数 return，使用加限定的 return 表达式，

```kotlin
fun reachable(from: Vertex, to: Vertex): Boolean {	val visited = HashSet<Vertex>()	fun dfs(current: Vertex) {		// here we return from the outer function:		if (current == to) return@reachable true		// And here -- from local function:		if (!visited.add(current)) return		for (v in current.neighbors)			dfs(v)		}	dfs(from)	return false // if dfs() did not return true already}
```

#### 成员函数

成员函数是定义在类或对象内的函数，

```kotlinclass Sample() {	fun foo() { print("Foo") }}
```
### 泛型函数

泛型函数，在函数名前指定类型：

```kotlinfun <T> singletonList(item: T): List<T> {	// ...}
```

### 内联函数

后面讨论。

### Tail recursive functions

Kotlin supports a style of functional programming known as **tail recursion**. This allows some algorithms that would normally be written using loops to instead be written using a recursive function, but without the risk of stack overflow. When a function is marked with the `tailrec` modifier and meets the required form the compiler optimises out the recursion, leaving behind a fast and efficient loop based version instead.

```kotlintailrec fun findFixPoint(x: Double = 1.0): Double	= if (x == Math.cos(x)) x else findFixPoint(Math.cos(x))
```
This code calculates the fixpoint of cosine, which is a mathematical constant. It simply calls `Math.cos` repeatedly starting at 1.0 until the result doesn’t change any more, yielding a result of 0.7390851332151607. The resulting code is equivalent to this more traditional style:

```kotlinprivate fun findFixPoint(): Double {	var x = 1.0	while (true) {		val y = Math.cos(x)		if (x == y) return y		x = y	}}
```
To be eligible for the `tailrec` modifier, a function must call itself as the last operation it performs. You cannot use tail recursion when there is more code after the recursive call, and you cannot use it within try/catch/finally blocks. Currently tail recursion is only supported in the JVM backend.

## 高阶函数与Lambda

### 高阶函数

高阶函数以另一个函数做参数，或返回一个函数。例如 `lock()` 函数，它接受一个锁对象和一个函数，它负责请求锁，运行函数，然后释放锁：

```kotlinfun <T> lock(lock: Lock, body: () -> T): T {	lock.lock()	try {		return body()	} finally {		lock.unlock()	}}

fun toBeSynchronized() = sharedResource.operation()val result = lock(lock, ::toBeSynchronized) // :: 的用法见下一章
```
或直接传入一个函数字面量（一般称为 lambda 表达式）：

```kotlinval result = lock(lock, { sharedResource.operation() })
```
函数字面量被大括号包围；它的参数放在 `->` 前面。正文放 `->` 后面。Kotlin 中，如果函数的最后一个参数是函数类型，调用时习惯省略括号：

```kotlin
lock (lock) {	sharedResource.operation()}
```
另一个高阶函数的例子：`map()`，

```kotlinfun <T, R> List<T>.map(transform: (T) -> R): List<R> {	val result = arrayListOf<R>()	for (item in this)		result.add(transform(item))	return result}

val doubled = ints.map { it -> it * 2 }
```若函数字面量只有一个参数，它的声明可以省略：`ints.map { it * 2 }`。利用这一点可以写出 LINQ 风格的代码：

```kotlinstrings.filter { it.length == 5 }.sortBy { it }.map { it.toUpperCase() }
```

### 内联函数

Sometimes it is beneficial to enhance performance of higher-order functions using inline functions.

### 函数字面量与函数表达式

函数字面量或函数表达式是一个匿名函数，即，函数没有声明，但直接当做表达式传递。

```kotlinmax(strings, { a, b -> a.length() < b.length() })
```
`max` 是一个高阶函数，第二个参数是一个函数字面量。

#### 函数类型

上面的 `max` 函数的定义应是：

```kotlinfun <T> max(collection: Collection<T>, less: (T, T) -> Boolean): T? {	var max: T? = null	for (it in collection)		if (max == null || less(max, it))			max = it	return max}
```
一个函数类型，可以带具名的参数，方便具名实参的使用。如，

```kotlin
val compare: (x: T, y: T) -> Int = ...
```

#### 函数字面量语法

函数字面量的完整语法，如

```kotlinval sum = { x: Int, y: Int -> x + y }
```
函数字面量最外层是大括号，完整形式的写法，参数声明在括号内，参数类型可选；函数正文在 `->` 后面。

完整的写法省掉所有可选的部分，结果如：

```kotlinval sum: (Int, Int) -> Int = { x, y -> x + y }
```
若函数字面量只有一个参数，可以省略该参数的声明，用 `it` 表示它：

```kotlinints.filter { it > 0 } // this literal is of type '(it: Int) -> Boolean'
```
若函数的最后一个参数是另一个函数，函数字面量作为实参可以放在实参列表圆括号外面。

#### 函数表达式

**函数字面量的问题是没法指定返回类型**。虽然多数情况返回类型可以推断。但如果你就想显式指定，你可以使用函数表达式。

```kotlinfun(x: Int, y: Int): Int = x + y
```
函数表达式与普通函数声明很像，只是没有名字。它的正文可以是一个表达式或块：

```kotlinfun(x: Int, y: Int): Int {	return x + y}
```
同样，若可以从上下文中推测，参数类型可以省略：

```kotlinints.filter(fun(item) = item > 0)
```
与普通函数一样，对于函数表达式，若正文是一个语句，返回类型可以自动推测；若正文是块，则必须指定返回类型（否则认为是 `Unit`）。
注意，函数表达式作为参数，调用时必须放在括号内。省略括号只对函数字面量有效。

函数表达式与字面量的区别还在于 `return` 语句。不带标签的 `return` 语句总是从通过 `fun` 关键字声明的函数中返回。从一个函数字面量中返回，将从字面量的外层函数中返回。但从一个函数表达式中返回，仅从函数表达式自身返回。

#### 闭包

函数字面量与表达式（包括局部函数和对象表达式）可以访问它的闭包。与 Java 不同的是，被闭包捕获的变量可以被修改，

```kotlinvar sum = 0ints.filter { it > 0 }.forEach { sum += it }print(sum)
```

#### 扩展函数表达式

Kotlin 还支持扩展函数字面量与表达式。One of the most important examples of their usage is Type-safe Groovy-style builders.
An extension function expression differs from an ordinary one in that it has a receiver type specification.

```kotlinval sum = fun Int.(other: Int): Int = this + other
```
Receiver type may be specified explicitly only in function expressions, not in function literals. Function literals can be used as extension function expressions, but only when the receiver type can be inferred from the context.
The type of an extension function expression is a function type with receiver:

```kotlinsum : Int.(other: Int) -> Int
```
The function can be called as if it were a method on the receiver object:

```kotlin1.sum(2)
```

## 内联函数

使用高阶函数有一定的性能损失：每个函数都是一个对象，它要捕获一个闭包。Memory allocations (both for function objects and classes) and virtual calls introduce runtime overhead.
多数情况下通过内联函数字面量可以消除这些开销。The functions shown above are good examples of this situation. I.e., the `lock()` function could be easily inlined at call-sites. Consider the following case:

```kotlinlock(l) { foo() }
```
Instead of creating a function object for the parameter and generating a call, the compiler could emit the following code

```kotlinl.lock()try {	foo()} finally {	l.unlock()}
```
Isn’t it what we wanted from the very beginning?
To make the compiler do this, we need to mark the `lock()` function with the `inline` modifier:

```kotlininline fun lock<T>(lock: Lock, body: () -> T): T {// ...}
```
`inline` 关键字影响函数自身和传入它的 lambda：它们二者都会被内联到调用它的地方。
内联将导致产生的代码数量变多。

### noinline

In case you want only some of the lambdas passed to an inline function to be inlined, you can mark some of your function parameters with the `noinline` modifier:

```kotlininline fun foo(inlined: () -> Unit, noinline notInlined: () -> Unit) {	// ...}
```
Inlinable lambdas can only be called inside the inline functions or passed as inlinable arguments, but noinline ones can be manipulated in any way we like: stored in fields, passed around etc.
Note that if an inline function has no inlinable function parameters and no reified type parameters, the compiler will issue a warning, since inlining such functions is very unlikely to be beneficial (you can suppress the warning if you are sure the inlining is needed).

### Non-local returns

In Kotlin, we can only use a normal, unqualified return to exit a named function or a function expression. This means that to exit a lambda, we have to use a label, and a bare return is forbidden inside a lambda, because a lambda can not make the enclosing function return:

```kotlinfun foo() {	ordinaryFunction {		return // ERROR: can not make `foo` return here	}}
```
But if the function the lambda is passed to is inlined, the return can be inlined as well, so it is allowed:

```kotlinfun foo() {	inlineFunction {		return // OK: the lambda is inlined	}}
```
Such returns (located in a lambda, but exiting the enclosing function) are called non-local returns. We are used to this sort of constructs in loops, which inline functions often enclose:

```kotlinfun hasZeros(ints: List<Int>): Boolean {	ints.forEach {		if (it == 0) return true // returns from hasZeros	}	return false}
```
Note that some inline functions may call the lambdas passed to them as parameters not directly from the function body, but from another execution context, such as a local object or a nested function. In such cases, non-local control flow is also not allowed in the lambdas. To indicate that, the lambda parameter needs to be marked with the crossinline modifier:```kotlininline fun f(crossinline body: () -> Unit) {	val f = object: Runnable {		override fun run() = body()	}	// ...}
```
break and continue are not yet available in inlined lambdas, but we are planning to support them too

### Reified type parameters

Sometimes we need to access a type passed to us as a parameter:```kotlinfun <T> TreeNode.findParentOfType(clazz: Class<T>): T? {	var p = parent	while (p != null && !clazz.isInstance(p)) {		p = p?.parent	}	@Suppress("UNCHECKED_CAST")	return p as T}
```
Here, we walk up a tree and use reflection to check if a node has a certain type. It’s all fine, but the call site is not very pretty:

```kotlinmyTree.findParentOfType(MyTreeNodeType::class.java)
```
What we actually want is simply pass a type to this function, i.e. call it like this:

```kotlinmyTree.findParentOfType<MyTreeNodeType>()
```
To enable this, inline functions support reified type parameters, so we can write something like this:

```kotlininline fun <reified T> TreeNode.findParentOfType(): T? {	var p = parent	while (p != null && p !is T) {		p = p?.parent	}	return p as T}
```
We qualified the type parameter with the reified modifier, now it’s accessible inside the function, almost as if it were a normal class. Since the function is inlined, no reflection is needed, normal operators like !is and as are working now. Also, we can call it as mentioned above: `myTree.findParentOfType<MyTreeNodeType>()`. Though reflection may not be needed in many cases, we can still use it with a reified type parameter:

```kotlininline fun membersOf<reified T>() = T::class.membersfun main(s: Array<String>) {	println(membersOf<StringBuilder>().joinToString("\n"))}
```
Normal functions (not marked as inline) can not have reified parameters. A type that does not have a run-time representation (e.g. a non-reified type parameter or a fictitious type like Nothing ) can not be used as an argument for a reified type parameter.
For a low-level description, see the spec document.