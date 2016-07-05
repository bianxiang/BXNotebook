## 7. 探索GDK ##

### 7.1 对 java.lang.Object 的扩展 ###

#### dump() 和 inspect()方法 ####

`dump()`显示对象内部信息，可以用于调试、日志、学习。
	
	str = 'hello'
	
	println str
	println str.dump()

输出

	hello
	<java.lang.String@5e918d2 value=hello offset=0 count=5 hash=99162322>

`inspect()`方法目的是告诉我们创建这样一个对象需要什么输入。若类未显式实现，默认返回`toString()`的结果。

#### with()与上下文 ####

`with()`接受一个闭包，在闭包内的所有方法调用的目标对象都是`with()`的目标对象。

例子，下面代码

	lst = [1, 2]
	lst.add(3)
	lst.add(4)
	println lst.size()
	println lst.contains(2)

可以被替换成：

	lst = [1, 2]
	lst.with {
	  add(3)
	  add(4)
	  println size()
	  println contains(2)
	}

`with()`的魔力来自于`delegate`属性。When we invoke the `with()` method, it sets the closure’s `delegate` property to the object on which `with()` is called.
	
	lst.with {
	  println "this is ${this},"
	  println "owner is ${owner},"
	  println "delegate is ${delegate}."
	}

得到：
	
	this is Identity@ce56f8,
	owner is Identity@ce56f8,
	delegate is [1, 2, 3, 4].


`with()`是`identity()`的别名，二者可以互换。

#### （未）sleep() ####

#### 间接访问属性 ####

利用`[]`运算符访问属性——映射到`getAt()`方法。若它出现在赋值的左边，则映射到`putAt()`方法。

	class Car {
	  int miles, fuelLevel
	}
	
	car = new Car(fuelLevel: 80, miles: 25)
	
	properties = ['miles', 'fuelLevel']
	// the above list may be populated from some input or
	// may come from a dynamic form in a web app
	
	properties.each { name ->
	  println "$name = ${car[name]}"
	}
	
	car[properties[1]] = 100
	
	println "fuelLevel now is ${car.fuelLevel}"

要获取对象的所有属性，可以通过对象的`properties`属性或`getProperties()`方法。

#### 间接调用方法 ####

若方法名放在一个字符串中，想通过这个字符串调用方法，Java中需要使用反射。拿到`Method`对象，调用`invoke()`方法。

Groovy中可以直接调用`invokeMethod()`方法。所有对象都支持该方法。

	class Person {
	  def walk() { println "Walking..." }
	  def walk(int miles) { println "Walking $miles miles..." }
	  def walk(int miles, String where) { println "Walking $miles miles $where..." }
	}
	
	peter = new Person()
	
	peter.invokeMethod("walk", null)
	peter.invokeMethod("walk", 10)
	peter.invokeMethod("walk", [2, 'uphill'] as Object[])

Groovy also provides `getMetaClass()` to get the metaclass object, which is a key object for taking advantage of dynamic capabilities in Groovy, as we’ll see in later chapters.

### 7.2 其他扩展 ###

#### 数组扩展 ####

通过`Range`对象获取数组片段：

	int[] arr = [1, 2, 3, 4, 5, 6]
	println arr[2..4]

得到：`[3, 4, 5]`。

#### （未）java.lang 扩展 ####

向基本类型的包装类型，如`Character`、`Integer`，添加了重载的运算符映射方法。如`plus()`（`+`）、`next()`（`++`）。

数字类型的基类`Number`，添加了一些迭代方法，如`upto()`、`downto()`和`step()`。


#### java.io 扩展 ####

向`File`类添加了大量方法。 如`eachFile()`和`eachDir()`。

Groovy向`BufferedReader`、`InputStream`和`File`添加了`text`属性，通过它可以将文件内容读到一个字符串。

	println new File('thoreau.txt').text

逐行读取：
	
	new File('thoreau.txt').eachLine { line ->
	  println line
	}

过滤行：

	println new File('thoreau.txt').filterLine { it =~ /life/ }

如果我们想在使用完后自动刷出和关闭一个输入流，可以使用`withStream()`方法。该方法接受一个闭包，传入一个`InputStream`对象。闭包返回后，流被自动刷出和关闭。对于`Writer`，有`withWriter()`方法。

`InputStream`的`withReader()`方法，创建一个`BufferedReader`并传入闭包。We can also obtain a new instance of `BufferedReader` by calling the `newReader()` method.

We can iterate over the stream of input in InputStream and DataInputStream using an `Iterator` we obtain by calling the `iterator()` method. Speaking of iterating, we can conveniently iterate over objects in an `ObjectInputStream`, as well.

If we want to use a `Reader` instead, we can. The convenience methods added to `InputStream` are still available on it.

向文件或流写内容也很简单。`OutputStream`、`ObjectOutputStream`和`Writer`类增加了一个`leftShift()`方法（`<<`运算符）。

	newFile("output.txt").withWriter{ file ->
		file << "somedata..."
	}

#### java.util 的扩展 ####

List, Set, SortedMap, and SortedSethave gained the method `asImmutable()` to obtain an immutable instance of their respective instances. They also have a method `asSynchronized()` to create an instance that is thread-safe.

The `Iterator` supports the inject() method.

### （未）7.3 Custom Methods Using the Extension Modules ###

Using  the  extension-modules  feature,  we  can  add  our  own
instance or static methods to existing classes at compile time and use them throughout the application at runtime. Let’s take a look at the simple steps we have to follow for this using an example.














