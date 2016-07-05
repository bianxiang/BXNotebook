[toc]

## 5. 函数

如果函数只有一个参数。且实参是 **字符串字面量** 或 **表构造器**，则括号是可选的：

```lua
	print "Hello World" <--> print("Hello World")
	dofile 'a.lua' <--> dofile ('a.lua')
	print [[a multi-line <--> print([[a multi-line
    message]] message]])
	f{x=10, y=20} <--> f({x=10, y=20})
	type{} <--> type({})
```

Lua提供一种面向对象的调用方式：用冒号运算符。`o:foo(x)`只是`o.foo(o,x)`的另一种写法。面向对象详见第16章。

Lua程序可以使用C编写的函数（或宿主程序使用的其他语言）。例如，标准库中的函数都是C写的。调用函数时，C编写的函数与Lua编写的函数没有区别。

函数定义：

```lua
    -- add the elements of sequence 'a'
    function add (a)
    	local sum = 0
    	for i = 1, #a do
    		sum = sum + a[i]
    	end
    	return sum
    end
```

实参和形参数量可以不同。规则与多赋值一样。

```lua
    function f (a, b) print(a, b) end

	f(3) --> 3 nil
    f(3, 4) --> 3 4
    f(3, 4, 5) --> 3 4 (5 is discarded)
```

### 5.1 多个结果

Lua函数可以返回多个结果。

```lua
	s, e = string.find("hello Lua users", "Lua")
	print(s, e) --> 7 9
```

（注意字符串的第一个字符的下标是1。）

定义返回多值的函数：

```lua
    function maximum (a)
        local mi = 1 -- index of the maximum value
        local m = a[mi] -- maximum value
        for i = 1, #a do
        	if a[i] > m then
                mi = i; m = a[i]
            end
        end
        return m, mi
    end
    print(maximum({8, 10, 23, 12, 5})) --> 23 3
```

假如有下面几个函数定义：

```lua
    function foo0 () end -- returns no results
    function foo1 () return "a" end -- returns 1 result
    function foo2 () return "a", "b" end -- returns 2 results
```

返回多值与多赋值：

```lua
	x, y, z = 10, foo2() -- x=10, y="a", z="b"
```

**值列表中，函数调用后面的值无效，将被丢弃**！因此**函数调用只能作为列表中最后一项**：

```lua
    x, y = foo2(), 20 -- x="a", y=20
    x, y = foo0(), 20, 30 -- x = nil, y=20, 30 is discarded
```

如果函数调用作为另一个函数调用的最后一个实参，则内部函数调用的结果都将作为实参。例如`print`，可以接受任意数量的实参。

```lua
    print(foo0()) -->
    print(foo1()) --> a
    print(foo2()) --> a b
    print(foo2(), 1) --> a 1
    print(foo2() .. "x") --> ax (see next)
```

如果`foo2`调用在一个表达式内，Lua将返回值调整成一个，因此上面例子的最后一个调用，只有第一个返回值“a”被使用。

若`f(g())`，`f`有固定数量的参数，则Lua会调整g的结果数量以匹配f的参数数量。

表构造器也会收集函数调用的所有结果：

```lua
    t = {foo0()} -- t = {} (an empty table)
    t = {foo1()} -- t = {"a"}
    t = {foo2()} -- t = {"a", "b"}
```

但前提时函数调用在最后。否则每个函数调用只会留下一个结果：

```lua
	t = {foo0(), foo2(), 4} -- t[1] = nil, t[2] = "a", t[3] = 4
```

`return f()`返回`f`返回的所有值：

```lua
    function foo (i)
        if i == 0 then return foo0()
        elseif i == 1 then return foo1()
        elseif i == 2 then return foo2()
        end
    end
    print(foo(1)) --> a
    print(foo(2)) --> a b
    print(foo(0)) -- (no results)
    print(foo(3)) -- (no results)
```

若想函数调用只返回一个结果，可以在外面包一个括号：

```lua
    print((foo0())) --> nil
    print((foo1())) --> a
    print((foo2())) --> a
```

`table.unpack`接收一个数组，将所有元素返回。要求数下标从1开始：

```lua
	print(table.unpack{10,20,30}) --> 10 20 30
	a,b = table.unpack{10,20,30} -- a=10, b=20, 30 is discarded
```

利用`unpack`把数组元素作为函数调用的实参：

```lua
	f = string.find
	a = {"hello", "ll"}
	print(f(table.unpack(a)))
```

`unpack`利用长度运算符确定元素数量，因此默认它只能处理**序列**。否则需要显式限定范围：

```lua
	print(table.unpack({"Sun", "Mon", "Tue", "Wed"}, 2, 3))
	--> Mon Tue
```

预定义的`unpack`函数是C写的。下面用Lua模拟一个：

```lua
    function unpack (t, i, n)
    	i = i or 1
    	n = n or #t
    	if i <= n then
    		return t[i], unpack(t, i + 1, n)
    	end
    end
```

### 5.2 可变参数函数

可变参数的函数，值函数可以接收多个参数。

例如，返回所有参数的和：

```lua
    function add (...)
        local s = 0
        for i, v in ipairs{...} do
        	s = s + v
        end
        return s
    end
    print(add(3, 4, 10, 25, 12)) --> 54
```

参数列表中的三个点`...`表示函数是可变参数的。我们称这些参数为数组的额外参数。函数内仍然使用三个点访问这些参数。例如`{...}`产生一个数组，收集所有参数。我们称表达式`...`为变参数表达式。For instance, the command `print(...)` prints all extra arguments of the function. 类似的，下面的命令取前两个参数（若实参不足，给`nil`）：

```lua
	local a, b = ...
```

实际上，可以把传统的`foo (a, b, c)`函数改写成：

```lua
	function foo (...)
		local a, b, c = ...
```

下面的函数只是原样返回所有实参：

```lua
	function id (...) return ... end
```

这个特性可以用于包装、拦截函数：

```lua
	function foo1 (...)
		print("calling foo:", ...)
		return foo(...)
	end
```

另一个有用的例子，将格式化函数`string.format`和输出函数`io.write`合并为一个可变参数的函数：

```lua
    function fwrite (fmt, ...)
        return io.write(string.format(fmt, ...))
    end
```

在可变参数前，函数可以有任意数量的固定参数。


要遍历额外参数，可以用`{...}`将它们转换成一个表。

若额外参数中允许有nil，此时`{...}`产生的表不是序列（有洞）。为解决该问题，Lua 5.2引入了`table.pack`函数。该函数与`{...}`类似，只是产生的表有一个额外的字段`n`，给出参数数量。The following function uses `table.pack` to test whether none of its arguments is `nil`:

```lua
    function nonils (...)
        local arg = table.pack(...)
        for i = 1, arg.n do
        	if arg[i] == nil then return false end
        end
        return true
    end
    print(nonils(2,3,nil)) --> false
    print(nonils(2,3)) --> true
    print(nonils()) --> true
    print(nonils(nil)) --> false
```

Remember, however, that `{...}` is cleaner and faster than `table.pack(...)` when the extra arguments **cannot** be `nil`.

### 5.3 具名实参

Lua中参数传递是基于位置的。Lua本身不支持命名实参。但可以通过表实现类似效果：所有的实参打包进一个表，函数只取一个参数，即这个表。函数调用时，若参数是表，可以省略括号：

```lua
	rename{old="temp.lua", new="temp1.lua"}
```

`rename`的定义：

```lua
	function rename (arg)
		return os.rename(arg.old, arg.new)
	end
```

## 6. 函数深入

函数是一等公民。可以存入变量和表。

函数与其他值一样，是匿名的。感受一下：

```lua
    a = {p = print}
    a.p("Hello World") --> Hello World
    print = math.sin -- 'print' now refers to the sine function
    a.p(print(1)) --> 0.841470
    sin = a.p -- 'sin' now refers to the print function
    sin(10, 20) --> 10 20
```

Lua中`function foo (x) return 2*x end`只是下面写法的糖衣：`foo = function (x) return 2*x end`。

函数可以一直是匿名的。例如，库函数`table.sort`用于排序。它接收一个函数指定元素的顺序关系。

```lua
    network = {
        {name = "grauna", IP = "210.26.30.34"},
        {name = "arraial", IP = "210.26.30.23"},
        {name = "lua", IP = "210.26.23.12"},
        {name = "derain", IP = "210.26.23.20"},
    }

	table.sort(network,
    	function (a, b) return (a.name > b.name) end)
```



### 6.1 闭包

{{读的比较粗}}

内存函数可以访问外层函数的局部变量，此特性称为词法作用域（lexical scoping）。

```lua
    function sortbygrade (names, grades)
    	table.sort(names, function (n1, n2)
    		return grades[n1] > grades[n2] -- compare the grades
    	end)
    end
```

在匿名函数中，`grades`既不是全局变量也不是局部变量，我们称其为**非局部变量**。(For historical reasons, non-local variables are also called **upvalues** in Lua.)

### 6.2 非全局函数

库函数（如`io.read`）就是将函数存储为表字段的例子。

```lua
    Lib = {}
    Lib.foo = function (x,y) return x + y end
    Lib.goo = function (x,y) return x - y end
    print(Lib.foo(2, 3), Lib.goo(2, 3)) --> 5 -1
```

也可以写成：

```lua
    Lib = {
    	foo = function (x,y) return x + y end,
    	goo = function (x,y) return x - y end
    }
```

或者写成：

```lua
    Lib = {}
    function Lib.foo (x,y) return x + y end
    function Lib.goo (x,y) return x - y end
```

代码块中可以声明局部函数，只在块内可见。

```lua
    local f = function (<params>)
    	<body>
    end
    local g = function (<params>)
    	<some code>
    	f() -- 'f' is visible here
    	<some code>
    end
```

局部函数的语法糖衣：

```lua
    local function f (<params>)
    	<body>
    end
```


定义局部的递归函数时要额外注意。正常的写法是不行的：

```lua
    local fact = function (n)
    	if n == 0 then return 1
    	else return n*fact(n-1) -- buggy
    	end
    end
```

Lua在编译`fact(n-1)`时，局部函数`fact`尚未定义，它会找一个全局函数。解决办法是，先定义局部变量，再定义函数：

```lua
    local fact
    fact = function (n)
        if n == 0 then return 1
        else return n*fact(n-1)
        end
    end
```

当Lua展开局部函数的糖衣形式时，如`local function foo (<params>) <body> end`，会被展开成：`local foo; foo = function (<params>) <body> end`。因此这个语法形式可以用于递归函数，没有任何问题。

Of course, this trick does not work if you have indirect recursive functions. In such cases, you must use the equivalent of an explicit forward declaration:

```lua
    local f, g -- 'forward' declarations
    function g ()
    	<some code> f() <some code>
    end
    function f ()
    	<some code> g() <some code>
    end
```

注意最后一个定义不要写成`local function f`！！Otherwise, Lua would create a fresh local variable f, leaving the original f (the one that g is bound to) undefined.

### 6.3 Proper Tail Calls

Lua does tail-call elimination. (This means that Lua is properly tail recursive, although the concept does not involve recursion directly; see Exercise 6.3.)

A tail call is a goto dressed as a call. 尾调用指函数在其最后一步调用另外一个函数。例如，下面的代码，调用`g`是尾调用：

	function f (x) return g(x) end

在`f`调用完`g`，后`f`没有其他事情做了。程序在完成嵌套的函数调用后不必再回到上一层函数。因此在尾调用之后，程序不需要在栈中保存函数`f`的信息。当`g`返回后，控制可以直接返回到调用`f`的地方。Some language implementations, such as the Lua
interpreter, take advantage of this fact and actually do not use any extra stack space when doing a tail call. We say that these implementations do tail-call elimination.

由于尾调用不需要栈控件，嵌套的尾调用的数量可以是无限的。例如，下面的函数，实参可以是任何大的值：

    function foo (n)
    	if n > 0 then return foo(n - 1) end
    end

永远不会栈溢出。

A subtle point when we assume tail-call elimination is what a tail call is. Some apparently obvious candidates fail the criterion that the calling function has nothing else to do after the call. For instance, in the following code, the call to `g` is not a tail call:

	function f(x) g(x) end

The problem in this example is that, after calling `g`, `f` still has to discard occasional results from g before returning. Similarly, all the following calls fail the criterion:

    return g(x) + 1 -- must do the addition
    return x or g(x) -- 必须把结果调整为1个
    return (g(x)) -- must adjust to 1 result

In Lua, only a call with the form `return func(args)` is a tail call. However, both `func` and its arguments can be complex expressions, because Lua evaluates them before the call. For instance, the next call is a tail call:

	return x[i].foo(x[j] + a*b, i + j)
