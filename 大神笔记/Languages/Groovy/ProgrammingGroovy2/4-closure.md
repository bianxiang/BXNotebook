## 4. 使用闭包

### 4.1 闭包有多方便 ###

定义一个对偶数操作的函数：
	
	def pickEven(n, block) {
		for(int i=2; i<=n; i+=2) {
			block(i)
		}
	}
	pickEven(10, { println it} )

`pickEven()`是一个高阶函数。

如果闭包是函数的最后一个参数，可以放到括号外：

	pickEven(10) { println it}

如果闭包只有一个参数，`it`可以指代这个参数。也可以显式指定参数名：

	pickEven(10) { evenNumber -> println evenNumber}

### 4.2 使用闭包 ###

### 4.3 使用闭包的方法 ###

`return` is optional even in closures; the value of the last expression (possibly null) is automatically returned to the caller if we don’t have an explicit return.

### 4.4 向闭包传参数 ###

支持多个参数：

	def tellFortune(closure) {
		closure new Date("09/20/2012"), "Your day is filled with ceremony"
	}
	tellFortune() { date, fortune ->
		println "Fortune for ${date} is' ${fortune}'"
	}

支持可选类型：

	tellFortune() { Date date, fortune->
		println "Fortune for ${date} is' ${fortune}'"
	}

### 4.5 闭包做资源清理 ###

Groovy向`FileWriter`添加了一个`withWriter()`方法。闭包返回后流会被自动关闭。

	new FileWriter('output.txt').withWriter{ writer ->
		writer.write('a')
	} // no need to close()

自动关闭功能的实现思路：

	def static use(closure) {
		def r= new Resource()
		try {
			r.open()
			closure(r)
		} finally {
			r.close()
		}
	}

### 4.6 闭包与Coroutines ###

### （未）4.7 Curried Closure ###

### 4.8 动态闭包 ###

本节讲闭包的两个属性`maximumNumberOfParameters`和`parameterTypes`。

判断闭包是否存在：

	def doSomeThing(closure) {
		if(closure) {
			closure()
		} else {
			println "Using default implementation"
		}
	}

闭包的`maximumNumberOfParameters`属性给出闭包接受多少个参数。利用它判断传入个闭包是哪种闭包：
	
	def completeOrder(amount, taxComputer) {
		tax = 0
		if(taxComputer.maximumNumberOfParameters == 2) {
			tax = taxComputer(amount, 6.05)
		} else { // uses a default rate
			tax = taxComputer(amount)
		}
		println "Sales tax is ${tax}"
	}
	completeOrder(100) {it * 0.0825}
	completeOrder(100) {amount, rate -> amount * (rate  /100)}

In  addition  to  the  number  of  parameters,  we  can  find  the  types  of  these parameters using the `parameterTypes` property. Here is an example examining the parameters of the closures provided:

	def examine(closure) {
		println "$closure.maximumNumberOfParameters parameter(s) given:"
		for(aParameter in closure.parameterTypes) 
			{ println aParameter.name }
	
		println"--"
	}
	
	examine(){}
	examine(){it}
	examine(){->}
	examine(){val1->}
	examine(){Dateval1->}
	examine(){Dateval1,val2->}
	examine(){Dateval1,Stringval2->}

### 4.9 Closure Delegation ###

Groovy支持方法代理。

闭包与三个属性：`this`, `owner`, `delegate`。这三个属性决定了在闭包内，方法调用的目标对象是谁。`delegate`一般等于`owner`。通过修改`delegate`的值指向其他对象（代理对象）可以实现元数据编程。例如，一些Groovy函数，如`with()`，会修改`delegate`实现动态路由。

先来看这三个属性的内容：

	def examiningClosure(closure) {
	  closure()
	}
	
	examiningClosure() { 
	  println "In First Closure:"
	  println "class is " + getClass().name
	  println "this is " + this + ", super:" + this.getClass().superclass.name
	  println "owner is " + owner + ", super:" + owner.getClass().superclass.name
	  println "delegate is " + delegate +
	              ", super:" + delegate.getClass().superclass.name
	  
	  examiningClosure() { 
	    println "In Closure within the First Closure:"
	    println "class is " + getClass().name
	    println "this is " + this + ", super:" + this.getClass().superclass.name
	    println "owner is " + owner + ", super:" + owner.getClass().superclass.name
	    println "delegate is " + delegate +
	                ", super:" + delegate.getClass().superclass.name
	  }  
	}

输出：

	In First Closure:
	class is ThisOwnerDelegate$_run_closure1
	this is ThisOwnerDelegate@55e6cb2a, super:groovy.lang.Script
	owner is ThisOwnerDelegate@55e6cb2a, super:groovy.lang.Script
	delegate is ThisOwnerDelegate@55e6cb2a, super:groovy.lang.Script

	In Closure within the First Closure:
	class is ThisOwnerDelegate$_run_closure1_closure2
	this is ThisOwnerDelegate@55e6cb2a, super:groovy.lang.Script
	owner is ThisOwnerDelegate$_run_closure1@15c330aa, super:groovy.lang.Closure
	delegate is ThisOwnerDelegate$_run_closure1@15c330aa, super:groovy.lang.Closure

从上面的输出可以看出，闭包作为内部类被创建。

在闭包中调用一个方法，这个方法的目标对象是谁？会先在`this`指向的对象中找，找不到再从`owner`指向的对象找，最后从`delegate`的指向对象找。例如：

	class Handler {
	  def f1() { println "f1 of Handler called ..."}
	  def f2() { println "f2 of Handler called ..."}  
	}
	
	class Example {
	  def f1() { println "f1 of Example called ..."}
	  def f2() { println "f2 of Example called ..."}
	
	  def foo(closure) {
	    closure.delegate = new Handler()
	    closure()
	  }
	}
	
	def f1() { println "f1 of Script called..." }
	
	new Example().foo {
      // 这个闭包直接定义在脚本中（最外层），其上下文是脚本
	  f1()
	  f2()
	}

输出：

	f1 of Script called...
	f2 of Handler called...

In the previous example we set the delegate property on a closure. This has side effects, especially if the closure can be used in other functions or in other threads. 如果你确信闭包不会在其他地方使用，你可以设置`delegate`。If it is used elsewhere, avoid the side effect—clone the closure, set the `delegate` on the clone, and use the clone. Groovy提供一个遍历方法实现此功能。你不再需要：

	def clone = closure.clone()
	clone.delegate = handler
	clone()

上面三步代码简化为`with()`方法：

	handler.with closure

Section 19.7, Closures and DSLs, on page 301, covers how the concepts from this section are used to build DSLs. Also refer to Section 7.1, Using Object Extensions,  on  page  128,  and  Section  13.2, Injecting  Methods  Using ExpandoMetaClass, on page 198. `ExpandoMeta` Classuses delegateto proxy methods
of classes.

### （未）4.10 尾递归 ###

### （未）4.11 Improving Performance Using Memoization ###
