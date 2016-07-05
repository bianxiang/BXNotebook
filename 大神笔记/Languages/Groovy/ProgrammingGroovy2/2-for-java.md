[toc]

## 2. Java视角

本章的例子先用传统Java编写，然后改成Groovy版本，以展示区别和意义。

### 2.1 从Java到Groovy

#### Hello, Groovy

下面的Java代码也是Groovy代码。保存文件Greetings.groovy：

    // Java code
    public class Greetings {
    	public static void main(String[] args) {
    		for(int i=0; i<3; i++){
			    System.out.print("ho");
    		}
    		System.out.println("MerryGroovy!");
    	}
    }

执行：

	groovy Greetings.groovy


> Groovy默认引入以下包：java.lang, java.util, java.io, java.net。此外还有两个类：java.math.BigDecimal 和 java.math.BigInteger。还有Groovy的包：groovy.lang 和 groovy.util。


第一次简化：去掉分号。去掉类和方法。
LightGreetings.groovy：

    for(int i=0; i<3; i++){
	    System.out.print("ho")
    }
    System.out.println("Merry Groovy!")

进一步简化：Groovy understands `println()` because it has been added on java.lang.Object. 理由`Range`对象简化for循环。省略调用函数时的括号。注意到`i`没有类型声明。

    for(i in 0..2) { print 'ho' }
    println 'Merry Groovy!'

#### 循环

除了for循环，Groovy还提供其他循环。

Groovy向java.lang.Integer添加了一个`upto()`方法：

	0.upto(2) { print "$it" }

将输出`012`。

`upto()`方法接受一个闭包。如果闭包只有一个形参，则其默认名是`it`。

执行指定次数的循环：

	3.times { print "$it" }

利用`step`指定循环步长：

	0.step(10, 2) { print "$it" }

#### GDK快速

Groovy向String类添加了一个`execute()`方法。用于执行系统命令。如：

	println "svn help".execute().text

等价的Java代码非常复杂：

    // Java code
    import java.io.*;
    public class ExecuteProcess {
        public static void main(String[] args) {
            try {
            	Process proc = Runtime.getRuntime().exec("svn help");
	            BufferedReader result = new BufferedReader(
  		          new InputStreamReader(proc.getInputStream()));
    	        Stringline;
        	    while((line = result.readLine()) != null) {
                	System.out.println(line);
    	        }
            } catch(IOException ex){
            	ex.printStackTrace();
            }
        }
    }

#### safe-navigation运算符

Groovy中

	str?.reverse()

等价于Java的：

	if(str != null){ str.reverse() }

#### 异常处理

Groovy中可以不捕获已检查异常。

若想捕获所有异常，可以省略异常类型：

    try {
    	openFile("non existent file")
    } catch(ex) {
    	println "Oops:" + ex
    }

> 注意其实只会捕获`Exception`，不会捕获其他的`Errors`或`Throwables`。若要真的捕获全部异常，使用`catch(Throwable throwable)`。

#### Groovy简化Java的其他地方

- `return`语句是可省略的。
- 分号几乎总是可以省略的。
- 方法和类默认都是public的。
- 可以使用具名参数初始化JavaBeans。
- 在静态类中，`this`指向`Class`对象。（进一步方便了静态方法的链式调用。）

### 2.2 JavaBeans

先看传统的Java代码：

	//Java code
	public class Car {
		private int miles;
		private final int year;
		public Car(int theYear) { year = theYear;}
		public int getMiles() { returnmiles;}
		public void setMiles(int theMiles) { miles=theMiles;}
		public int getYear() { return year;}

		public static void main(String[] args) {
			Car car = new Car(2008);
			System.out.println("Year:" + car.getYear());
			System.out.println("Miles:" + car.getMiles());
			System.out.println("Setting miles");
			car.setMiles(25);
			System.out.println("Miles:" + car.getMiles());
		}
	}

等价的Groovy代码：

	class Car {
		def miles = 0
		final year
		Car(theYear) {year = theYear}
	}
	
	Car car = new Car(2008)
	println "Year: $car.year"
	println "Miles: $car.miles"
	println 'Setting miles'
	car.miles = 25
	println "Miles: $car.miles"


在这里，`def`定义一个属性。声明属性的另一种方式是给它一个类型，如`int miles`或`int miles = 0`。Groovy会为我们创建getter和setter方法。当我们在代码里使用`miles`时，引用的不是字段，而是调用getter方法。若属性只读，声明为`final`，类型信息可选。Groovy会提供getter方法但不会提供setter方法。尝试改变final字段值会导致异常。将字段标记为private是没用的，Groovy不会照办。因此若想实现私有字段，只能定义一个setter，拒绝改变：

	class Car {
		final year
		privat emiles = 0
		Car(theYear) { year = theYear}
		def getMiles() {
			println "getMiles called"
			miles
		}
		private void setMiles(miles) {
			throw new IllegalAccessException("you're not allowed to chang emiles")
		}
		def drive(dist) {if(dist > 0) miles += dist}
	}

访问属性，不需要通过getter和setter方法：

	Calendar.instance
	// instead of Calendar.getInstance()
	str = 'hello'
	str.class.name
	// instead of str.getClass().getName()
	// Caution: Won't work for Maps, Builders, ...
	// use str.getClass().name to be safe

### 2.3 灵活初始化与具名实参


创建对象时，可以利用具名实参初始化对象。初始化发生在构造后（利用无参构造器）。方法也可以接受命名实参。此时方法的第一个参数必须是一个`Map`。例子：

	class Robot{
		def type, height, width
		def access(location, weight, fragile) {
			println "Received fragile? $fragile, weight: $weight, loc: $location"
		}
	}
	robot = new Robot(type: 'arm', width: 10, height: 40)
	println "$robot.type, $robot.height, $robot.width"
	robot.access(x: 30, y: 20, z: 10, 50, true)
	// You can change the order
	robot.access(50, true, x: 30, y: 20, z: 10)

输出：

	arm,40,10
	Received fragile? true, weight: 50, loc: [x:30, y:20, z:10]
	Received fragile? true, weight: 50, loc: [x:30, y:20, z:10]

为避免混淆，最好显式指明第一个参数的类型：

	def access(Map location, weight, fragile) { ...

### 2.4 可选参数

在方法或构造器的形参列表中，若给某个参数一个值，则此参数可选。可选参数的数量是任意的，但必须放在列表的最后。

	def log(x, base = 10) {
		Math.log(x) / Math.log(base)
	}

	println log(1024)
	println log(1024, 10)
	println log(1024, 2)

最后一个数组参数也是可选的：

	def task(name, String[] details) {
		println "$name - $details"
	}

	task 'Call', '123-456-7890'
	task 'Call', '123-456-7890', '231-546-0987'
	task 'CheckMail'

注意到最后几个实参会被收集到数组中。

### 2.5 多赋值

例子。分割全名。

	def splitName(fullName) {fullName.split(' ')}
	def (firstName, lastName) = splitName('JamesBond')
	println "$lastName, $firstName $lastName"

例子。交换两个值。左面的两个值要放入圆括号。右边两个值要放入方括号。

	def name1 = "Thomson"
	def name2 = "Thompson"
	println "$name1 and $name2"
	(name1, name2) = [name2, name1]
	println "$name1 and $name2"

如果左右变量和值的数量不等。多余的变量被赋null，多余的值被丢弃。但如果多余的变量是基本类型，它们不能被赋值null，Groovy会抛出异常——这是Groovy 2的新特性——尽量使用基本类型。

可以指定变量类型：

	def (String cat, String mouse) = ['Tom', 'Jerry', 'Spike', 'Tyke']
	println "$cat and $mouse"

### 2.6 实现接口

Java中典型的事件回调代码：

	// Java code
	button.addActionListener(new ActionListener() {
		public void actionPerformed(ActionEvent ae) {
			JOptionPane.showMessageDialog(frame, "Youclicked!");
		}
	});

Groovy的写法：

	button.addActionListener(
		{ JOptionPane.showMessageDialog(frame,"Youclicked!") } as ActionListener
	)

另一例子，重用代码：

	displayMouseLocation = { positionLabel.setText("$it.x,$it.y")}
	frame.addMouseListener(displayMouseLocation as MouseListener)
	frame.addMouseMotionListener(displayMouseLocation as MouseMotionListener)

Groovy不要求我们必须实现接口中所有方法；我们只需要定义我们感兴趣的方法。未实现的方法若不被调用，则一切安全。但如果被调用了，将产生`NullPointerException`。

若需要实现接口中的多个方法，可以提供一个Map。键是方法名，值是方法。

	handleFocus=[
		focusGained: {msgLabel.setText("Goodtoseeyou!")},
		focusLost: {msgLabel.setText("Comebacksoon!")}
	]
	button.addFocusListener(handleFocus as FocusListener)

使用`as`运算符要求编译时指定接口名。但如果接口名在运行时才能获知，需要用`asType()`方法。

	events = ['WindowListener', 'ComponentListener']handler = { msgLabel.setText("$it")}
	for(event in events) {
		handlerImpl = handler.asType(Class.forName("java.awt.event.${event}"))
		frame."add${event}"(handlerImpl)
	}

一个完整的例子：

	import javax.swing.*
	import java.awt.*
	import java.awt.event.*
	
	frame = new JFrame(size: [300, 300],
	  layout: new FlowLayout(),
	  defaultCloseOperation: javax.swing.WindowConstants.EXIT_ON_CLOSE)
	button = new JButton("click")
	positionLabel = new JLabel("")
	msgLabel = new JLabel("")
	frame.contentPane.add button
	frame.contentPane.add positionLabel
	frame.contentPane.add msgLabel
	
	button.addActionListener(
	  { JOptionPane.showMessageDialog(frame, "You clicked!") } as ActionListener
	)
	
	displayMouseLocation = { positionLabel.setText("$it.x, $it.y") }
	frame.addMouseListener(displayMouseLocation as MouseListener)
	frame.addMouseMotionListener(displayMouseLocation as MouseMotionListener)
	
	handleFocus = [
	  focusGained : { msgLabel.setText("Good to see you!") },
	  focusLost : { msgLabel.setText("Come back soon!") }
	]
	button.addFocusListener(handleFocus as FocusListener)
	
	events = ['WindowListener', 'ComponentListener'] 
	// Above list may be dynamic and may come from some input
	handler = { msgLabel.setText("$it") }
	for (event in events) {
	  handlerImpl = handler.asType(Class.forName("java.awt.event.${event}"))
	  frame."add${event}"(handlerImpl)
	}
	
	frame.show()
	frame

### 2.7 Groovy 布尔求值


与Java不同，Groovy将根据上下文将一个表达式作为布尔值求值。

例如，下面的代码在Java里是错误的
	
	//Java code
	String obj = "hello";
	int val = 4;
	if(obj) {} //ERROR
	if(val) {} //ERROR


在一个期望布尔值的地方放一个对象引用，引用空相当于false。

	str = 'hello'
	if(str) {println 'hello'}

若引用非空，真假取决于对象类型。例如若是List，则空集导致false，非空导致true。

	lst0 = null
	println lst0 ? 'lst0 true' : 'lst0 false'
	lst1 = [1, 2, 3]
	println lst1 ? 'lst1 true' : 'lst1 false'
	lst2 = []
	println lst2 ? 'lst2 true' : 'lst2 false'


下面是各种类型为真的条件：

- Boolean：true
- Collection：非空
- Character：值不是0
- CharSequence：不是空串
- Enumeration：有更多的元素
- Iterator：有下一个元素
- Number：不是0
- Map：非空
- Matcher：有匹配
- Object[]：长度大于0
- 其他类型：变量不是null

除了上述内建的真值转换，我们自己的类通过实现`asBoolean()`方法实现我们自己的转换。


###  2.8 运算符重载


每个运算符映射到一个方法。在Java里我们可以使用这些方法，在Groovy里我们使用运算符，也可以使用方法。

例子：

	for(ch = 'a'; ch < 'd'; ch++){
		println ch
	}

这里的`++`运算符映射到String类的`next()`方法。


下面的写法也用到了`next()`方法：

	for(ch in 'a' .. 'c') {
		println ch
	}

向集合添加元素使用`<<`运算符，映射到`Collection`的`leftShift()`方法：

	lst = ['hello']
	lst << 'there'
	println lst

通过重载映射方法实现我们自己的重载，如：

	class ComplexNumber {
		def real, imaginary
		def plus(other) {
			new ComplexNumber(real: real + other.real,
				imaginary: imaginary + other.imaginary)
		}
		String toString() { "$real ${imaginary > 0 ? '+' : ''} ${imaginary}i"}
	}
	c1 = new ComplexNumber(real: 1, imaginary: 2)
	c2 = new ComplexNumber(real:4, imaginary:1)
	println c1 + c2

### 2.9 Support of Java 5 Language Features

这里指的Java 5特性包括：Autoboxing、for-each、enum、Varargs、Annotation、Static import、Generics。

#### Autoboxing ####

In fact, Groovy automatically treats primitives as objects where necessary. Groovy decides to store the instance as an int or
Integer based on how we use it. 

例如：

	int val = 5
	println val.getClass().name

结果是`java.lang.Integer`。

Groovy’s handling of autoboxing is a notch better than Java’s. In Java, autoboxing and unboxing involve constant casting. Groovy, on the other  hand, simply treats them as objects—so there’s no **repeated** casting involved.

2.0之前所有的基本类型都被当作对象。为了提高性能，且使用更多直接的字节码，Groovy 2.0 做了一些优化。尽在必要的时候才将基本类型当作对象，例如，例如，在它们身上调用方法时或将它们传给对象引用时。否则Groovy在字节码层面将它们作为基本类型。

#### for-each

Java中实现Iterable接口的对象可以使用for-each。例如：

	//Java code
	String[] greetings = {"Hello", "Hi", "Howdy"};
	for(String greet : greetings) {
		System.out.println(greet);
	}

换成Groovy代码：

	String[] greetings = ["Hello", "Hi", "Howdy"]
	for(String greet : greetings) {
		println greet
	}

若不想指定类型或def，可以使用`in`替代`:`

	for(greet in greetings){
		println greet
	}

我们更喜欢Groovy的for-in，而不是Java风格的写法。

#### 枚举 ####

Groovy支持枚举。

	enum CoffeeSize { SHORT, SMALL, MEDIUM, LARGE, MUG }
	def orderCoffee(size) {
	  print "Coffee order received for size $size: "
	  switch(size) {
	    case [CoffeeSize.SHORT, CoffeeSize.SMALL]:
	      println "you're health conscious"
	      break
	    case CoffeeSize.MEDIUM..CoffeeSize.LARGE: 
	      println "you gotta be a programmer"
	      break
	    case CoffeeSize.MUG:
	      println "you should try Caffeine IV"
	      break
	  }
	}
	orderCoffee(CoffeeSize.SMALL)
	orderCoffee(CoffeeSize.LARGE) 
	orderCoffee(CoffeeSize.MUG) 
	print 'Available sizes are: '
	for(size in CoffeeSize.values()) {
	    print "$size "
	}

注意到在`case`语句中，可以使用单个值、一组值或范围值。


#### varargs ####

Groovy有两种实现可变参数的方式。除了Java的`…`标记，最后一个参数是数组，也可以接受数量可变的实参。

	def receiveVarArgs(int a, int... b) {
		println "You passed $a and $b"
	}
	def receiveArray(int a, int[] b) {
		println "You passed $a and $b"
	}

	receiveVarArgs(1, 2, 3, 4, 5)
	receiveArray(1, 2, 3, 4, 5)

两种方式的可变参数，都接受离散的实参或一个数组。

注意Groovy中，`[]`是一个`ArrayList`对象，不是数组。因此不能直接让诸如`[2, 3, 4, 5]`做实参，否则会抛出`MethodMissingException`。要发送一个数组，需要使用`as`运算符：

	int[] values = [2, 3, 4, 5]
	receiveVarArgs(1, values)
	receiveVarArgs(1, [2, 3, 4, 5] as int[])


#### 注解 ####

在Groovy中可以定义和使用注解。The syntax for defining in Groovy is the same as in Java.

The Groovy compiler does not treat the Java compilation-related annotations the same way the Java compiler does. For example, **groovyc** ignores `@Override`.

### 静态import ###

Java中可以引入静态方法，如：

	import static Math.random;

除了支持Java的语法，Groovy还支持指定别名：

	import static Math.random as rand
	import groovy.lang.ExpandoMetaClass as EMC
	double value = rand()
	def metaClass = new EMC(Integer)
	assert metaClass.getClass().name == 'groovy.lang.ExpandoMetaClass'

#### 泛型

However, the Groovy compiler does not perform type-checking the same way the Java compiler does (see CompileTime Type-Checking Is Off by Default, on page 47); don’t expect the Groovy compiler to reject at the outset code with type violations, like the Java compiler does. Groovy’s dynamic typing will interplay here with generic types to get our code running, if possible.

Groovy supports Generics while favoring dynamic behavior. The   previous code example shows quite an interesting interplay of the two concepts. This dual nature of Groovy may be a surprise at first, but it will make sense when you learn the benefits of  Groovy metaprogramming (see Part III, MOPping Groovy, on page 173).

The usefulness of Generics is not totally lost in Groovy. Groovy 2.x provides rigorous type-checking on parts of the code if  we’re willing to compromise metaprogramming capabilities—see Section 3.8, Switching Off Dynamic Typing, on page 65.

### 2.10 Groovy代码产生注解

Groovy tactfully eases the tension that language designers often face between
a desire to evolve the language and a reluctance to modify the grammar due
to its impact on performance, complexity, and semantic correctness. Rather
than modifying the core syntax of the language, the Groovy compiler recognizes
select annotations and generates appropriate code. In this section you’ll learn
about a few such annotations. Chapter 16, Applying Compile-Time Metaprogramming, on page 235, covers how to create our own annotations for custom
transformations.

Groovy provides a number of code-generation annotations in the groovy.transform
package and a few other packages. We’ll talk about a few of these annotations
in this section.

#### @Canonical

如果我们需要一个`toString()`方法，它只是显示一些字段，逗号分隔，我们可以让Grooovy编译器产生，利用`@Canonical` transformation。默认包含所有字段；可以要求包含或排除字段。

	import groovy.transform.*

	@Canonical(excludes = "lastName, age")
	class Person {
		String firstName
		String lastName
		int age
	}
	def sara = new Person (firstName: "Sara", lastName: "Walker", age:49)
	println sara


#### @Delegate

Inheritance must be savored only where the derived class is truly substitutable and used in place of the base class. For most other purposes, delegation is better than inheritance from pure code-reuse point of view. Yet in Java we’re reluctant to use delegation, as that leads to code duplication and more effort.

Groovy makes delegation quite easy, so we can make the proper design choice.

To better understand delegation, let’s start with a `Worker` class that has a few methods in it. `Expert` has one method with the same name and signature as the `Worker` class. `Manager`, as  we’d expect, does nothing. But this manager is smart at delegating work, so the two fields are marked with the `@Delegate` annotation.


	class Worker {
		def work() { println 'get work done' }
		def analyze() { println 'analyze...' }
		def writeReport() { println 'get report written'}
	}
	class Expert {
		def analyze() { println "expert analysis..." }
	}
	class Manager {
		@Delegate Expert expert = new Expert()
		@Delegate Worker worker = new Worker()
	}

	def bernie = new Manager()
	bernie.analyze()
	bernie.work()
	bernie.writeReport()

编译时，Groovy examines the `Manager` class and brings in methods
from the delegated classes only if those methods don’t already  exist. As a result, it brings in the `analyze()` method of the `Expert` first. From the `Worker` class it brings in only the `work()` and `writeReport()` methods. At this time, since the `analyze()` method is present in the `Manager`, brought in from `Expert`, the one in `Worker` is ignored.


For each of the methods that are brought in, Groovy simply routes a call to the method on the instance, like so: 
`public Object analyze() { expert.analyze() }`.


The `Manager` class is extensible thanks to the `@Delegate `annotation. If we add or remove methods to the `Worker` or the `Expert` class, we don’t have to make any changes to `Manager` for  the  corresponding  change  to  take  effect. Simply recompile the code, and Groovy takes care of the rest.

#### @Immutable

Groovy makes it easier to do the right thing by marking the fields as final and, as a bonus, creating some convenience methods for us if we mark a class with the `@Immutable` annotation.

Let’s use this annotation in a `CreditCard` class.

	@Immutable
	class CreditCard {
		String cardNumber
		int creditLimit
	}

	println new CreditCard("4000-1111-2222-3333", 1000)

Groovy为我们提供一个构造器，参数顺序取决于字段出现顺序。In addition, Groovy  adds the hashCode(), equals(), and toString() methods.

#### @Lazy

We want to defer the construction of time-consuming objects until we actually need them. We can be lazy and productive at the same time, write less code, and reap all the benefits of lazy initialization.

In the next example we want to postpone the creation of the instances of `Heavy` until they’re needed. We can directly initialize instances at the point of declaration or we can wrap the logic for creation within a closure.

	class Heavy {
		def size = 10
		Heavy() { println "Creating Heavy with $size" }
	}
	class AsNeeded {
		def value
		@Lazy Heavy heavy1 = new Heavy()
		@Lazy Heavy heavy2 = { new Heavy(size: value) }()
		AsNeeded() { println "Created AsNeeded"}
	}
	
	def asNeeded = new AsNeeded(value: 1000)
	println asNeeded.heavy1.size
	println asNeeded.heavy1.size
	println asNeeded.heavy2.size

Groovy not only defers the creation, but also marks the field as volatile and ensures thread safety during creation. The instances are created on the first access to the fields, as we can see in the output.

	Created AsNeeded
	Creating Heavy with 10
	10
	10
	Creating Heavy with 10
	1000


The `@Lazy` annotation provides a painless way to implement the virtual proxy pattern with thread safety as a bonus.

#### @Newify

In Groovy we often follow the traditional Java syntax of using `new` to create an instance. Losing this keyword will improve the fluency when creating DSLs, however. The `@Newify` annotation can help us create Ruby-like constructors where new is a method on the class. It can also help us create Python-like
constructors (and Scala-like applicators) where we can do away  with `new` entirely. To create the Python-like constructor, we  must specify the list of types to the `@Newify` annotation. The Ruby-style constructor is created for us unless we set the value `auto=false` as a parameter to `@Newify`.

We  can  use  the `@Newify` annotation  in  various  scopes,  such  as  classes  or methods, as in the next example.

	@Newify([Person, CreditCard])
	def fluentCreate() {
		println Person.new (firstName: "John", lastName: "Doe", age: 20)
		println Person (firstName: "John", lastName:"Doe", age:20)
		println CreditCard("1234-5678-1234-5678", 2000)
	}
	fluentCreate()


#### @Singleton

To implement the singleton pattern, normally we’d create a  static field and a static method to initialize that field, then return that singleton instance. We have to ensure that this method is thread-safe and decide whether we want a lazy creation of the singleton. We can eliminate this effort entirely by using
the `@Singleton` transformation, as in the following example. 

	@Singleton(lazy = true)
	class TheUnique {
		private TheUnique() { println 'Instancecreated' }
		def hello() { println'hello' }
	}

	println "Accessing TheUnique"
	TheUnique.instance.hello()
	TheUnique.instance.hello()

When we run the code, the instance is created on the first call to the `instance` property, which maps to the `getInstance()`method.

We can examine the generated code by copying and pasting the previous code into **groovyConsole** and selecting the **script | Inspect AST** menu item.

There’s one caveat to using the @Singleton annotation. It makes the constructor of the target class private, as we’d expect, but since Groovy does not honor privacy,  we  can  still  create  instances  using  the new keyword  from  within Groovy. We must take care to use the class properly and heed the warnings from code-analysis tools and integrated development environments.

In  addition  to  what  we’ve  seen  so  far,  Groovy  provides  an  annotation  that
removes drudgery when extending classes with multiple constructors. Java
forces  us  to  mundanely  implement  the  multiple  constructors  even  if  we
merely want to route the calls back to the respective superconstructors. If we
annotate the class with `@InheritConstructors`, Groovy generates these constructors
for us.

### 2.11 陷阱

#### Groovy的`==`是Java的`equals` ####

Groovy将`==`运算符映射到Java的`equals()`方法。Groovy中要比较引用，需要使用`is()`。

	str1 = 'hello'
	str2 = str1
	str3 = new String('hello')
	str4 = 'Hello'
	println "str1==str2: ${str1==str2}"
	println "str1==str3: ${str1==str3}"
	println "str1==str4: ${str1==str4}"
	println "str1.is(str2): ${str1.is(str2)}"
	println "str1.is(str3): ${str1.is(str3)}"
	println "str1.is(str4): ${str1.is(str4)}"

如果类实现了`Comparable`接口，则`==`映射到`compareTo()`方法。


#### 编译时类型检查默认关闭

Groovy is optionally typed;however, for the most part the Groovy  compiler, **groovyc**, does not perform full type-checking. (Chapter 3, Dynamic Typing, on page 53, shows how Groovy allows selective type-checking.) Instead it performs casting when it encounters type definitions. Let’s assign a string to a variable
of the Integer type:

	Integer val = 4
	val = 'hello'

上述代码能通过编译。但在运行时会抛出GroovyCastException。

The Groovy compiler, instead of verifying the type, simply cast it and left it to the runtime to deal with. So, in Groovy, `x = y` is semantically equivalent to `x = (ClassOfX)(y)`.


如果调用一个不存在的方法，也不会编译报错。但运行时会报`MissingMethodException`。这其实是优点：我们可以在编译和运行之间，动态注入方法。见第13章。
The Groovy compiler may appear relaxed, but this behavior is necessary for the dynamic and metaprogramming strengths of Groovy. In version 2.x we can  turn  this  dynamic  typing  feature  off  and  enhance  compile-time  typechecking, as we’ll explore in Section 3.8, Switching Off Dynamic Typing, on
page 65, and Static Type-Checking, on page 66.


#### 注意新关键字

增加了new和in两个关键字。

最好不要让`it`做变量名。

#### 无代码块

下面的代码在Java中是有效的：

	// Java code
	public void method() {
		System.out.println("inmethod1");
		{
			System.out.println("inblock");
		}
	}

Groovy会以为我们在定义闭包。

#### （未）闭包：匿名内部类冲突

#### 创建基本类型数组的不同语法

Java的写法是不行的：

	int[] arr = new int[] { 1, 2, 3, 4, 5};

Groovy中上述写法无法通过编译，必须这样写：

	int[] arr = [1, 2, 3, 4, 5]

如果我们省略左侧的`int[]`，Groovy会当我们要创建一个ArrayList。

或者，利用`as`运算符创建数组：

	def arr = [1, 2, 3, 4, 5] as int[]

----

Visit http://groovy.codehaus.org/Differences+from+Java for a nice list of Groovy-Java differences.















