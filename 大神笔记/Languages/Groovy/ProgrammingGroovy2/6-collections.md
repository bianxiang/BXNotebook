## 6. 集合 ##

### 6.1 使用List ###


可以使用字面量创建java.util.ArrayList。

	lst = [1, 3, 4, 1, 8, 9, 2, 6]
	println lst
	println lst.getClass().name

输出：

	[1, 3, 4, 1, 8, 9, 2, 6]
	java.util.ArrayList

可以直接使用`[]`访问数组元素：
	
	println lst[0]
	println lst[lst.size() - 1]

支持负索引：

	println lst[-1]
	println lst[-2]

利用Range对象获取片段：

	println lst[2..5]

得到：`[4, 1, 8, 9]`

范围也支持负索引：

	println lst[-6 .. -3]

注意获取的片段并不是拷贝，仍然指向原来的对象。修改会影响到原来的列表。

### 6.2 遍历ArrayList ###

#### 使用 each 方法 ####

This iterator, the method named `each()`, is also known as an internal iterator.


	lst = [1, 3, 4, 1, 8, 9, 2, 6]
	lst.each { println it }

> C++和Java常见的迭代器是外部迭代器，即有调用者控制迭代，判断结束，负责移动元素。支持闭包的语言更常见的是内部迭代器。调用者不能控制循环。

利用`reverseEach()`实现反向迭代。`eachWithIndex()`可以在迭代过程中取到索引。

#### 使用 collect 方法 ####

`collect()`方法对每一个元素调用，将产生的结果输出成另一个集合。

	println lst.collect { it * 2 }

输出是大小相同的集合，每个元素是原来的2倍：

	[2, 6, 8, 2, 16, 18, 4, 12]

### 6.3 查找方法 ###

`find()`找到第一个匹配元素后停止。

	lst = [4, 3, 1, 2, 4, 1, 8, 9, 2, 6]
	println lst.find { it == 2 }

若找不到返回null。

若要寻找所有复合条件的元素，使用`findAll()`。如果想获取的是第一个匹配的元素的位置，使用`findIndexOf()`。

### 6.4 List的其他方法 ###

例子，计算所有单词的总长度：

	lst = ['Programming', 'In', 'Groovy']
	println lst.collect { it.size() }.sum()

利用`inject()`可以实现相同功能：

	println lst.inject(0) { carryOver, element -> carryOver + element.size()}

可以直接对某个下标的元素赋值，改变那个值：

	lst[0] = ['Be', 'Productive']
	println lst

得到：`[[Be,Productive],In,Groovy]`。

展平：

	lst = lst.flatten()
	println lst

得到：`[Be, Productive, In, Groovy]`。


利用`-`运算符用于删除元素。右边可以是单个对象或一个列表。

不用显式迭代就能对每个元素操作：

	println lst*.size()

其相当于`lst.collect{ it.size() }`。输出得到`[2, 10, 2, 6]`，即每个元素（字符串）的长度。这里的`*`被称为spread运算符。

下面展示如何在方法调用时使用ArrayList。利用`*`运算符把列表展开。列表长度必须等于方法参数个数。

	def words(a, b, c, d) {
		println "$a $b $c $d"
	}
	words(*lst)

### 6.5 使用Map类 ###

	langs = ['C++' : 'Stroustrup', 'Java' : 'Gosling', 'Lisp' : 'McCarthy']
	println langs.getClass().name

得到`java.util.LinkedHashMap`。

利用`[]`访问键值：

	println langs['Java']
	println langs['C++']

直接把键当属性访问：

	println langs.Java

对`a`是Map，则不能通过`a.class`访问class属性。Groovy会把`class`当作键，若没有这个键会返回null。必须使用`a.getClass()`。

但像`C++`这种键，必须加引号访问：

	println langs.'C++'

Map字面量中的键名的两边的引号是可以省略的：

	langs = ['C++' : 'Stroustrup', Java : 'Gosling', Lisp : 'McCarthy']

### 6.6 遍历Map ###

#### each 方法 ####

若传给each的闭包只有一个参数，则闭包将收到一个`MapEntry`对象。若闭包有两个参数，则分别收到键值对。

	langs = ['C++' : 'Stroustrup', 'Java' : 'Gosling', 'Lisp' : 'McCarthy']
	langs.each { entry ->
	  println "Language $entry.key was authored by $entry.value"  
	}

	langs.each { language, author ->
	  println "Language $language was authored by $author"  
	}

Similarly, for other methods—such as `collect()`, `find()`, and so on—we use one parameter if we want only the `MapEntry` and two parameters if we want the key and the value separately.

#### collect方法 ####

`collect()`方法返回一个列表。

	println langs.collect { language, author ->
	  language.replaceAll("[+]", "P")
	}

得到：`[CPP, Java, Lisp]`

#### 搜索方法 ####

Map对象也支持`find()`和`findAll()`方法。

	entry = langs.find { language, author ->
	  language.size() > 3
	}
	println "Found $entry.key written by $entry.value"

找不到返回null。

### 6.7 Map的方法 ###

`groupBy()`方法。
