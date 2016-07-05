## 5. 字符串 ##

### 5.1 字面量与表达式 ###

Groory中单引号创建的也是`java.lang.String`，不是字符。如果想使用字符串字面量，必须使用`'a'  as char`。Of course, Groovy may implicitly create `Character` objects if any method calls demand it.

Groovy将单引号包围的字符串看做纯字面量。即里面的表达式不会被展开。

Groovy中字符串也是不可变对象。可以是用`[]`访问单个字符。

We can create an expression with either double quotes ("") or slashes (//). However, double quotes are often used to define string expressions, 斜杠常用于定义正则表达式。Here’s an example for creating an expression:

	value = 12
	println "He paid \$${value} for that."

若表达式是单个变量名，则`{}`可以被省略。因此我们可以写`println "He paid \$${value} for that."`，或`println "He paid \$$value for that."`，或`println(/He paid $$value for that/).`。

使用单引号创建的字符串一定是String类型。但使用双引号或斜杠创建的，但不能表示成String类型的字符串被称为GString，它们一般含有表达式。

	def printClassInfo(obj) {
	  println "class: ${obj.getClass().name}"
	  println "superclass: ${obj.getClass().superclass.name}"
	}
	
	val = 125
	printClassInfo ("The Stock closed at ${val}")
	printClassInfo (/The Stock closed at ${val}/)
	printClassInfo ("This is a simple String")

输出：

	class: org.codehaus.groovy.runtime.GStringImpl
	superclass: groovy.lang.GString
	class: org.codehaus.groovy.runtime.GStringImpl
	superclass: groovy.lang.GString
	class: java.lang.String
	superclass: java.lang.Object

注意最后一个是java.lang.String，虽然字符串由双引号包围。

### 5.2 GString Lazy求值问题 ###

GString中表达式的求值，发生在每次GString的`toString()`方法被调用时。于是下面的代码，两次调用产生不同的结果：

	what = new StringBuilder('fence')
	text = "The cow jumped over the $what"
	println text
	
	what.replace(0, 5, "moon")
	println text

输出：

	The cow jumped over the fence
	The cow jumped over the moon

但要注意，GString表达式若有变量，表达式绑定的是变量（引用）指向的对象，不是变量本身。因此下面的代码，三次输出都是：`Today Google stock closed at 684.71`。因为修改的“变量指向”，而不是指向对象的内容。
	
	price = 684.71
	company = 'Google'
	quote = "Today $company stock closed at $price"
	println quote
	
	stocks = [Apple : 663.01, Microsoft : 30.95]
	
	stocks.each { key, value ->
	  company = key
	  price = value
	  println quote
	}

解决办法是利用闭包。若GString中有一个闭包：

* 若闭包没有参数，则直接输出闭包的返回值
* 若闭包有一个参数，则这个参数将被传值`Writer`对象，
* 若闭包有超过一个参数，报错。

利用`Writer`对象解决：

	companyClosure = { it.write(company) }
	priceClosure = { it.write("$price") }
	quote = "Today ${companyClosure} stock closed at ${priceClosure}"
	stocks.each { key, value ->
	  company = key
	  price = value
	  println quote
	}

或者利用一种情况，省略参数，直接取闭包的返回值：

	quote = "Today ${-> company } stock closed at ${-> price }"
	
	stocks.each { key, value ->
	  company = key
	  price = value
	  println quote
	}

### 5.3 多行字符串 ###

多行字符串可以用三个单引号包围。

	memo = '''Several of you raised concerns about long meetings.
	To discuss this, we will be holding a 3 hour meeting starting
	at 9AM tomorrow. All getting this memo are required to attend.
	If you can't make it, please have a meeting with your manager to explain.
	'''

三个双信号包围的字符串中可以有表达式：

	message = """We're very pleased to announce
	that our stock price hit a high of \$${price} per share
	on December 24th. Great news in time for...
	"""
	println message

### 5.4 字符串方法 ###

`-=`用于移除部分字符。如：
	
	str = "It's a rainy day in Seattle"
	println str
	
	str -= "rainy "
	println str

除此之外，Groovy还向字符串添加了以下方法：`plus()`(+), `multiply()`(*), `next()`(++), `replaceAll()`, `tokenize()`等方法。


### 5.5 正则表达式 ###

Groovy中，`~`运算符可以创建一个正在表达式模式。这个运算符映射到String的`negate()`方法。

	obj = ~"hello"
	println obj.getClass().name

输出：`java.util.regex.Pattern`。

`~`后的字符串可以是单引号、双引号或斜线包围的。斜线有一个好处，不需要转义。即`/\d*\w*/`可以等价于`"\\d*\\w*"`。

有两个运算符可以用于正则表达式匹配：`=~`和`==~`，分用于部分匹配和精确匹配。例子：

	pattern = ~"(G|g)roovy"
	text = 'Groovy is Hip'
	if (text =~ pattern)
	  println "match"
	else
	  println "no match"
	
	if (text ==~ pattern)
	  println "match"
	else
	  println "no match"

`=~`运算符返回一个`java.util.regex.Matcher`对象。用在布尔表达式中，Groovy对这个对象特殊处理：若有匹配返回true否则false。If the match results in multiple matches, then the matcher contains an array of the matches. This helps quickly get access to parts of the text that match the given RegEx.

	matcher = 'Groovy is groovy' =~ /(G|g)roovy/
	print "Size of matcher is ${matcher.size()} "
	println "with elements ${matcher[0]} and ${matcher[1]}."

接下来可以替换掉匹配到的部分，使用`replaceFirst()`或`replaceAll()`：

	str = 'Groovy is groovy, really groovy'
	println str
	result = (str =~ /groovy/).replaceAll('hip')
	println result










