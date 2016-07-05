## 7. Nashorn Javascript 引擎 ##

Java已经捆绑了一个Javascript引擎很多年，名叫Rhino。但它不够快。使用新JVM为动态语言设计的指令，现在开发出了非常快的Nashorn引擎。

本章关键点：

- Nashorn is a pleasant environment for experimenting with the Java API.
- You can run JavaScript through the jjs interpreter, or from Java via the scripting API.
- Use the predefined JavaScript objects for the most common packages, or the `Java.type` function to access any package.
- Beware of intricacies in the conversion of strings and numbers between JavaScript and Java.
- JavaScript offers a convenient syntax for working with Java lists and maps, as well as JavaBeans properties.
- You can convert JavaScript functions to Java interfaces in a way that is very similar to using lambda expressions.
- You can extend Java classes and implement Java interfaces in JavaScript, but there are limitations.
- Nashorn has good support for writing shell scripts in JavaScript.
- You can write JavaFX programs in JavaScript, but the integration is not as good as it might be.

### 7.1. 从命令行运行 Nashorn

Java 8 自带一个命令行工具称为 **jjs**。启动它，随便写一些Javascript。

	$ jjs jjs> 'Hello, World'
	Hello, World


	jjs> 'Hello, World!'.length
	13


定义并调用函数：

	jjs> function factorial(n) { return n <= 1 ? 1 : n * factorial(n - 1) }
	function factorial(n) { return n <= 1 ? 1 : n * factorial(n - 1) }
	jjs> factorial(10)
	3628800

可以调用Java方法：

	var input = new java.util.Scanner(
		new java.net.URL('http://horstmann.com').openStream())
	input.useDelimiter('$')
	var contents = input.next()

Now, when you type `contents`, you see the contents of the web page.

利用上面的特性，可以快速实验Java代码。不需要写冗长的main函数。

### 7.2. 在Java中运行Nashorn ###

要运行脚本需要一个`ScriptEngine`对象。若引擎已注册，可以用名字获取它。Java 8 包含一个名叫"nashorn"的引擎。

	ScriptEngineManager manager = new ScriptEngineManager();
	ScriptEngine engine = manager.getEngineByName("nashorn");
	Object result = engine.eval("'Hello, World!'.length");
	System.out.println(result);

或者从一个`Reader`读取脚本：

	Object result = engine.eval(Files.newBufferedReader(path));

若想在脚本中访问一个Java对象，需要调用`ScriptEngine`接口的`put`方法。对象将被放入全局作用域。

	public void start(Stage stage) {
		engine.put("stage", stage);
		engine.eval(script);
		// Script code can access the object as stage
	}

除了放入全局作用域，还可以将值放入一个`Bindings`对象并传给`eval`方法：

	Bindings scope = engine.createBindings();
	scope.put("stage", stage);
	engine.eval(script, scope);

### 7.3. 调用方法

把Java对象暴露给Javascript脚本后，可以调用这些对象的方法。例如，如果先

	engine.put("stage", stage);

则JavaScript代码中可以调用

	stage.setTitle('Hello')

实际上可以直接

	stage.title = 'Hello'

Nashorn支持用属性语法替换getters和setters。当`stage.title`出现在赋值左边时，相当于调用`setTitle`。甚至可以用Javascript语法访问属性：

	stage['title'] = 'Hello'

> Javascript中分号是可选的。

JavaScript中没有方法重载的概念。Nashorn尝试根据类型和参数数量匹配正确的方法。一般是能匹配到的。若不能，可以显式指定调用哪个方法，语法：

	list['remove(Object)'](1)

Here, we specify the `remove(Object)` method that removes the Integer object 1 from the list. (There is also a `remove(int)` method that removes the object at position 1.)

### 7.4. 构造对象

若想在Javascript中构造对象，需要知道如何访问Java包。有两种机制。There are global objects java, javax, javafx, com, org, and edu that yield package and class objects via the dot notation. For example,

	var javaNetPackage = java.net // A JavaPackage object
	var URL = java.net.URL // A JavaClass object

If you need to access a package that does not start with one of the above identifiers, you can find it in the `Package` object, such as `Package.ch.cern`.

Alternatively, call the `Java.type` function:

	var URL = Java.type('java.net.URL')

This is a bit faster than `java.net.URL`, and you get better error checking. (If you make a spelling error such as `java.net.Url`, Nashorn will think it is a package.) But if you want speed and good error handling, you probably shouldn’t be using a scripting language in the first place, so I will stick with the shorter form.

The Nashorn documentation suggests that class objects should be defined at the top of a script file, just like you place imports at the top of a Java file:

	var URL = Java.type('java.net.URL')
	var JMath = Java.type('java.lang.Math')

一旦获取到类对象，就可以调用静态方法：

	JMath.floorMod(-3, 10)

要构造对象，仍使用new运算符：

	var URL = java.net.URL
	var url = new URL('http://horstmann.com')

或者若你不在乎效率（efficiency），可以：

	var url = new java.net.URL('http://horstmann.com')

If you use `Java.type` with new, you need an extra set of parentheses

	var url = new (Java.type('java.net.URL'))('http://horstmann.com')

If you need to specify an inner class, you can do so with the dot notation:

	var entry = new java.util.AbstractMap.SimpleEntry('hello', 42)

Alternatively, if you use `Java.type`, use a `$`, like the JVM does:

	var Entry = Java.type('java.util.AbstractMap$SimpleEntry')

### 7.5. 字符串

Nashorn中的字符串，当然，是Javascript对象。例如：

	'Hello'.slice(-2)

调用的是JavaScript的方法`slice`。Java中没有此方法。但下面的方法也是可行的：

	'Hello'.compareTo('World')

尽管JavaScript没有`compareTo`方法（使用`<`运算符）。在这种情况下，Javascript字符串转换为Java字符串。

当把一个Javascript字符串传给Java方法时，总会被转换为Java字符串。若把一个Javascript对象传给Java方法且方法的参数类型是字符串，则Javascript对象会被转换为字符串。

	var path = java.nio.file.Paths.get(/home/)
	// A JavaScript RegExp is converted to a Java String!

这里`/home/`是一个正则表达式。

类似的转换也发生在数字和布尔。例如`'Hello'.slice('-2')`是有效的，`'-2'`会被转换为数字`–2`。

### 7.6. 数字

JavaScript不支持整数。它的`Number`类型等价于Java的`double`类型。若一个数字被传给一个Java函数，函数期望接收int或long，分数部分会被悄悄移除。例如`'Hello'.slice(-2.99)`等价于`'Hello'.slice(-2)`。For efficiency, Nashorn keeps computations as integers when possible, but that difference is generally transparent. Here is one situation when it is not:

	var price = 10
	java.lang.String.format('Price: %.2f', price)
	// Error: f format not valid for java.lang.Integer

The value of `price` happens to be an integer, and it is assigned to an Object since the format method has an `Object...` varargs parameter. Therefore, Nashorn produces a `java.lang.Integer`. That causes the format method to fail, since the f format is intended for floating-point numbers. In this case, you can force conversion to `java.lang.Double` by calling the `Number` function:

	java.lang.String.format('Unit price: %.2f', Number(price))

### 7.7. 数组

To construct a Java array, first make a class object:

	var intArray = Java.type('int[]')
	var StringArray = Java.type('java.lang.String[]')

Then call the **new** operator and supply the length of the array:

	var numbers = new intArray(10)
	var names = new StringArray(10)

Then use the bracket notation in the usual way:

	numbers[0] = 42
	print(numbers[0])

You get the length of the array as `numbers.length`. To iterate through all values of the names array, use

	for each (var elem in names)
		Do something with elem

This is the equivalent of the enhanced for loop in Java. If you need the index values, use the following loop instead:

	for (var i in names)
		Do something with i and names[i]

Java和JavaScript数组非常不同。当你向期望Java数组的地方传一个Javascript数组时，Nashorn会负责转换。But sometimes, you need to help it along. Given a JavaScript array, use the `Java.to` method to obtain the equivalent Java array:

	var javaNames = Java.to(names, StringArray)
	
Conversely, use `Java.from` to turn a Java array into a JavaScript array:

	var jsNumbers = Java.from(numbers)
	jsNumbers[-1] = 42

You need to use `Java.to` to resolve overload ambiguities. For example,

	java.util.Arrays.toString([1, 2, 3])

is ambiguous since Nashorn can’t decide whether to convert to an `int[]` or `Object[]` array. In that situation, call

	java.util.Arrays.toString(Java.to([1, 2, 3], Java.type('int[]')))

or simply

	java.util.Arrays.toString(Java.to([1, 2, 3], 'int[]'))

### 7.8. Lists 和 Maps

Nashorn provides “syntactic sugar” for Java lists and maps. You can use the bracket operator with any Java List to invoke the get and set methods:

	var names = java.util.Arrays.asList('Fred', 'Wilma', 'Barney')
	var first = names[0]
	names[0] = 'Duke'

The bracket operator also works for Java maps:

	var scores = new java.util.HashMap
	scores['Fred'] = 10 // Calls scores.put('Fred', 10)

To visit all elements in the map, you can use the JavaScript for each loops:

	for (var key in scores) ...
	for each (var value in scores) ...

If you want to process keys and values together, simply iterate over the entry set:

	for each (var e in scores.entrySet())
		Process e.key and e.value

> The **for each** loop works for any Java class that implements the `Iterable` interface.

### 7.9. Lambdas

JavaScript有匿名函数，如：

	var square = function(x) { return x * x }
	var result = square(2)

这种匿名函数跟Java的Lambda表达式很像。可以将匿名函数作为函数接口的实参，就像在Java中使用Lambda表达式做实参一样。例如

	java.util.Arrays.sort(words,
		function(a, b) { return java.lang.Integer.compare(a.length, b.length) })

Nashorn supports shorthand for functions whose body is a single expression. For such functions, you can omit the braces and the return keyword:

	java.util.Arrays.sort(words,
		function(a, b) java.lang.Integer.compare(a.length, b.length))

Again, note the similarity with a Java lambda expression `(a, b) -> Integer.compare(a.length, b.length)`.

> That shorthand notation (called an “expression closure”) is not part of the official JavaScript language standard (ECMAScript 5.1), but it is also supported by the Mozilla JavaScript implementation.

### （未）7.10. 继承Java类，实现Java接口

### 7.11. 异常

Java抛出的异常，可以在Javascript中正常捕获：

	try {
		var first = list.get(0)
		...
	} catch (e) {
		if (e instanceof java.lang.IndexOutOfBoundsException)
			print('list is empty')
	}

注意到只有一个`catch`子句。

### （未）7.12. Shell Scripting ###

### （未）7.13. Nashorn and JavaFX ###
