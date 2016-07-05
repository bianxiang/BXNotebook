## 3. 动态类型 ##

动态类型比静态类型更具扩展性：可以在运行时向类注入新行为。 With the  *multimethods* tool, we can provide alternate behaviors to operations based on the arguments’ runtime types.

### 3.1 Java中的类型

### 3.2 动态类型 ###

动态类型有两个主要的优点。且其优点大过了缺点。首先，实现程度更高的多态。其次，大量减少了强制类型转换。

### 3.3 动态类型不等于弱类型 ###

Java是静态类型、强类型的语言。

Ruby和Groovy是动态类型但是强类型。

C和C++是静态类型但是弱类型。

JavaScript是弱类型、动态类型。

### 3.4 Design by Capability ###

Java程序员非常依赖接口。他们强调“**design by contract**”。由接口定义contracts。

但不我们不想让协议太过限制。基于接口的编程太过限制。

下面看一些静态类型和动态类型的区别。

#### 静态类型 ####

假设我们想找人搬东西，开始我们找男人：

	public void takeHelp(Man man) {
		//...
		man.helpMoveThings();
		//...
	}

后来发现女人也可以，于是我们把`helpMoveThings`放入男人女人的基类中：

	// Java code
	public abstract class Human {
		public abstract void helpMoveThings();
		//...
	}

	public void takeHelp(Human human) {
		//...
		human.helpMoveThings();
		//...
	}

后来发现不是人也可以帮忙，于是我们把能帮忙的特性抽象为接口：

	// Java code
	public interface Helper {
		public void helpMoveThings();
	}

	public void takeHelp(Helper helper) {
		//...
		helper.helpMoveThings();
		//...
	}

#### 使用动态类型 ####


利用Groovy的动态类型，我们会定义：

	def takeHelp(helper){
		//...
		helper.helpMoveThings()
		//...
	}


没有指定`helper`的类型。它唯一的特征是有`helpMoveThings()`方法。This is design by capability. This is called duck typing：如果它走起路来像鸭子，叫声也像鸭子那么它就是鸭子。

要想具备帮助能力的类只需要实现`helpMoveThings()`方法，不需要实现什么接口。

	class Man {
		void helpMoveThings() {
			//...
		}
		//...
	}
	class Woman {
		void helpMoveThings() {
			//...
		}
		//...
	}
	class Elephant {
		void helpMoveThings() {
			//...
		}
		//...
	}

#### 动态类型：注意事项 ####

静态类型可以检查出来的拼写错误动态类型是检查不出来的。解决方法是单元测试。

因为没有类型，有时难以知道参数要求传什么类型的值。

有可能传入了一个不具备指定能力的对象（没有期望的方法）。 Groovy的`respondsTo()`方法可以用来检查对象是否具有某项能力：
	
	def takeHelpAndReward(helper) {
		//...
		helper.helpMoveThings()
		if(helper.metaClass.respondsTo(helper, 'eatSugarcane')) {
			helper.eatSugarcane()
		}
		//...
	}

### 3.5 Optional Typing ###

Groovy is dynamically typed andoptionally typed; 我们可以选择两个极端：不指定任何类型，或精确指定类型。

有时使用一些Java库和框架，但Groovy的动态类型映射有时不能与这些库或框架匹配。此时我们可以采用指定类型的方式。其他情况也可能要求明确指定类型，例如需要类型信息推断数据库类型，使用GORM/Grails。

比如，JUnit要求测试方法的返回值是void。于是我们不能用`def`定义方法。必须明确定义返回值为void。

### 3.6 Multimethods ###

Groovy支持多态。但在Groovy中，选择执行哪个方法不只取决于目标对象的类型。

先看一个Java中多态的例子：

	// Java code
	public class Employee {
		// #1
		public void raise(Number amount) {
			System.out.println("call Employee.raise(Number)");
		}
	}

	// Java code
	public class Executive extends Employee {
		// #2
		public void raise(Number amount) {
			System.out.println("Executive got raise");
		}
		// #3
		public void raise(java.math.BigDecimal amount) {
			System.out.println("Executive got outlandish raise");
		}
	}

注意Number是BigDecimal的父类！

下面是调用代码：

	// Java code
	public class GiveRaiseJava {
		public static void giveRaise(Employee employee) {
			employee.raise(new BigDecimal(10000.00));
		}
		public static void main(String[] args) {
			giveRaise(new Employee()); // 执行方法1
			giveRaise(new Executive()); // 执行方法2
		}
	}


`Employee`的`raise()`方法是多态的，在运行时执行哪个方法取决于目标引用的类型（target reference’s type），而不是指向的对象的类型（type of the referenced object）。

但在Groovy中，当你拿一个BigDecimal调用`raise()`时，它会问对象是否有一个`raise()`方法接受`java.math.BigDecimal()`。`Executive`却有这个方法，因此就执行了这个方法：

	void giveRaise(Employee employee) {
		employee.raise(new BigDecimal(10000.00))
		// same as
		// employee.raise(10000.00)
	}
	giveRaise new Employee()
	giveRaise new Executive() // 执行方法3

Groovy在选择方法时，不仅基于目标对象，而且考虑参数类型，这种行为称为 *multiple dispatch* 或 *multimethods*。

另一个例子，参看下面的代码。`col`变量的类型是`Collection<String>`，但它指向的对象是一个`ArrayList<String>`。当我们调用`col.remove(0)`时，由于`Collection`的`remove()`方法接受一个`Object`,于是Java会把`0`装箱成`Integer`。And since an instance of Integer is not part of the list, the method call did not remove anything.

	// Java code
	public class UsingCollection {
		public static void main(String[] args) {
			ArrayList<String> lst = new ArrayList<String>();
			Collection<String> col = lst;
			lst.add("one");
			lst.add("two");
			lst.add("three");
			lst.remove(0);
			col.remove(0);
			System.out.println("Added three items, removed two, so 1 item to remain.");
			System.out.println("Number of elements is: " + lst.size());
			System.out.println("Number of elements is: " + col.size());
		}
	}

在Groovy中就不会有上述问题。上述代码，原封不动作为Groovy文件执行，就能得到期望的结果：删除第一个元素。

### 3.7 动态：用不用

作者选择，除非不得已，否则尽量省略类型。

从使用者角度触发，社区倾向于在方法签名中总是给出类型。The benefit here is knowing the types for arguments during method calls and  avoiding unnecessary runtime typechecking within methods.

### （未）3.8 关闭动态类型

Groovy的动态类型也是有代价的。编译时能检查出来的错误被推迟到运行时。动态方法分发是有开销的。尽管Java 7引入了动态调用特性减少性能问题。但在之前的JVM中仍有性能问题。

We  can  ask  the  Groovy  compiler  to  tighten  its  type-checking  from  its dynamic relaxed mode to the levels we’d expect from a statically typed compiler  like javac.  We  can  also  trade  in  the  benefits  of  dynamic  typing  and
metaprogramming  capabilities,  and  ask  the  Groovy  compiler  to  statically compile code down to more-efficient bytecode.

In  this  section  we’ll  look  at  two  features,  one  to  ask  that  Groovy  perform more-rigorous checks at compile time and the other to ask that it create more efficient statically compiled bytecode.


























