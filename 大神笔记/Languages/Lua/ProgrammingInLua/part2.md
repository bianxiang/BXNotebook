[toc]

## Part II Tables and Objects

## 11. 数据结构

本章的内容是，如何用table实现各种常见的数据结构，如数组、记录、列表、队列、集合等。More to the point, Lua tables implement all these structures efficiently.

For instance, we seldom write a search in Lua, because tables offer direct access to any type.

### 11.1 数组

在Lua中实现数组，只要用整数作为Table的索引。数组没有固定大小，按需增长。

```lua
    a = {} -- new array
    for i = 1, 1000 do
    	a[i] = 0
    end
```

数组的下标可以从任何值开始，不仅限于0或1：

```lua
    -- creates an array with indices from -5 to 5
    a = {}
    for i = -5, 5 do
    	a[i] = 0
    end
```

但在Lua中，习惯以1开始。Lua库依赖此约定。长度运算符也是。If your arrays do not start with 1, you will not be able to use these facilities.

利用构造器可以创建并同时初始化数组：

```lua
	squares = {1, 4, 9, 16, 25, 36, 49, 64, 81}
```

### 11.2 矩阵和多维数组

使用数组的数组：即Table的元素是另一个Table。例如：

```lua
    mt = {} -- create the matrix
    for i = 1, N do
        mt[i] = {} -- create a new row
        for j = 1, M do
        	mt[i][j] = 0
        end
    end
```

对于稀疏数组（内部有`nil`值），不能用长度运算符。不过，这没什么，因为从头到尾遍历稀疏数组是无效的。应该使用`pairs`直接遍历有值的元素。

```lua
    function mult (a, rowindex, k)
        local row = a[rowindex]
        for i, v in pairs(row) do
        	row[i] = v * k
        end
    end
```

注意Table没有本质顺序（intrinsic order），因此`pairs`不保证按顺序。若要顺序，可能需要链表等机制。

### 11.3 链表

Because tables are dynamic entities, it is easy to implement linked lists in Lua. Each node is represented by a table and links are simply table fields that contain references to other tables. For instance, let us implement a basic list, where each node has two fields, `next` and `value`. A simple variable is the list root:

```lua
	list = nil
```

To insert an element at the beginning of the list, with a value v, we do:

```lua
	list = {next = list, value = v}
```
To traverse the list, we write:

```lua
    local l = list
    while l do
    	<visit l.value>
    	l = l.next
    end
```

Other kinds of lists, such as double-linked lists or circular lists, are also implemented easily. However, you seldom need those structures in Lua, because usually there is a simpler way to represent your data without using linked lists. For instance, we can represent a stack with an (unbounded) array.

### 11.4 队列和双向队列

A simple way to implement queues in Lua is with functions `insert` and `remove` from the table library. These functions insert and remove elements in any position of an array, moving other elements to accommodate the operation.

However, these moves can be **expensive** for large structures. A more efficient implementation uses two indices, one for the first element and another for the last:

```lua
    function ListNew ()
    	return {first = 0, last = -1}
    end
```

To avoid polluting the global space, we will define all list operations inside a table, properly called `List` (that is, we will create a module). Therefore, we rewrite our last example like this:

```lua
    List = {}
    function List.new ()
    	return {first = 0, last = -1}
    end
```

Now, we can insert or remove an element at both ends in constant time:

```lua
    function List.pushfirst (list, value)
        local first = list.first - 1
        list.first = first
        list[first] = value
    end
    function List.pushlast (list, value)
        local last = list.last + 1
        list.last = last
        list[last] = value
    end
    function List.popfirst (list)
        local first = list.first
        if first > list.last then error("list is empty") end
        local value = list[first]
        list[first] = nil -- to allow garbage collection
        list.first = first + 1
        return value
    end
    function List.poplast (list)
        local last = list.last
        if list.first > last then error("list is empty") end
        local value = list[last]
        list[last] = nil -- to allow garbage collection
        list.last = last - 1
        return value
    end
```

If you use this structure in a strict queue discipline, calling only pushlast and popfirst, both first and last will increase continually. However, because we represent arrays in Lua with tables, you can index them either from 1 to 20 or from 16777216 to 16777236. Because Lua uses double precision to represent numbers, your program can run for two hundred years, doing one million insertions per second, before it has problems with overflows.

### 11.5 Sets 和 Bags

Suppose you want to list all identifiers used in a program source; somehow you need to filter the reserved words out of your listing. Some C programmers could be tempted to represent the set of reserved words as an array of strings, and then to search this array to know whether a given word is in the set. To speed up the search, they could even use a binary tree to represent the set.

In Lua, an efficient and simple way to represent such sets is to put the set elements as indices in a table. Then, instead of searching the table for a given element, you just index the table and test whether the result is `nil` or not. In our example, we could write the next code:

```lua
    reserved = {
    	["while"] = true, ["end"] = true,
    	["function"] = true, ["local"] = true,
    }
    for w in allwords() do
    	if not reserved[w] then
    		<do something with ’w’> -- 'w' is not a reserved word
    	end
    end
```

(Because these words are reserved in Lua, we cannot use them as identifiers; for instance, we cannot write `while=true`. Instead, we use the `["while"]=true` notation.)

You can have a clearer initialization using an auxiliary function to build the set:

```lua
    function Set (list)
        local set = {}
        for _, l in ipairs(list) do set[l] = true end
        	return set
    end
    reserved = Set{"while", "end", "function", "local", }
```

Bags, also called multisets, differ from regular sets in that each element can appear multiple times. An easy representation for bags in Lua is similar to the previous representation for sets, but with a counter associated to each key. To insert an element we increment its counter:

```lua
    function insert (bag, element)
    	bag[element] = (bag[element] or 0) + 1
    end
```

To remove an element we decrement its counter:

```lua
    function remove (bag, element)
    	local count = bag[element]
    	bag[element] = (count and count > 1) and count - 1 or nil
    end
```

We only keep the counter if it already exists and it is still greater than zero.

### （未）11.6 String Buffers

### （未）11.7 Graphs

## （未）12. 数据文件与持久化

In this chapter, we will see how we can use Lua to eliminate all code for reading data from our programs, simply by writing the data in an appropriate format.

## 13. 元表和元方法

Lua中每个值能执行什么操作是确定的。例如，两个数可以相加，两个表却不能相加。除非使用元表（metatables）。Metatables allow us to change the behavior of a value when confronted with an undefined operation. 例如，利用元表我们可以定义两个表相加：当Lua尝试两个表相加时，它会检查是否有一个表有元表，且原表是否有`__add`字段。如果有，则字段的值，被称为元方法（metamethod），是一个函数，用于计算和。

Lua中每个值都可以关联元表。Tables and userdata have individual metatables; values of other types share one single metatable for all values of that type. 新创建的Table没有元表：

```lua
    t = {}
    print(getmetatable(t)) --> nil
```

可以通过`setmetatable`设置或改变任何表的元表：

```lua
    t1 = {}
    setmetatable(t, t1)
    print(getmetatable(t) == t1) --> true
```

通过Lua语言只能操纵表的元表；操纵其他类型值的元表只能通过C代码。（这么做的目的仅是增加使用难度，不要轻易使用其他类型的元表。之前的经验表名，这些全局设置经常导致不可重用的代码。）字符串库为字符串设置了元表（第21章）。其他类型默认没有元表：

```lua
    print(getmetatable("hi")) --> table: 0x80772e0
    print(getmetatable("xuxu")) --> table: 0x80772e0
    print(getmetatable(10)) --> nil
    print(getmetatable(print)) --> nil
```

任何表都可以做元表，表中可以有任何值；多个表可以共享同一个元表。表也可以使用只属于自己的元表。

### 13.1 算术元方法

本节引入一个集合的例子解释如何使用元表。若用表表示集合，定义函数执行集合交并运算。为了命名空间整洁，将这些函数存放到表`Set`。

Listing 13.1. A simple implementation for sets:

```lua
    Set = {}
    -- create a new set with the values of a given list
    function Set.new (l)
        local set = {}
        for _, v in ipairs(l) do set[v] = true end
        return set
    end
    function Set.union (a, b)
        local res = Set.new{}
        for k in pairs(a) do res[k] = true end
        for k in pairs(b) do res[k] = true end
        return res
    end
    function Set.intersection (a, b)
        local res = Set.new{}
        for k in pairs(a) do
        	res[k] = b[k]
        end
        return res
    end
    -- presents a set as a string
    function Set.tostring (set)
        local l = {} -- list to put all elements from the set
        for e in pairs(set) do
        	l[#l + 1] = e
        end
        return "{" .. table.concat(l, ", ") .. "}"
    end
    -- print a set
    function Set.print (s)
    	print(Set.tostring(s))
    end
```

我们像用相加运算符`+`计算两个集合的并集。所有表示集合的Table将共享一个元表。该元表定义相加运算符。下面是元表：

```lua
	local mt = {} -- metatable for sets
```

接下来，修改`Set.new`函数，让新创建的集合都获得元表`mt`：

```lua
    function Set.new (l) -- 2nd version
        local set = {}
        setmetatable(set, mt)
        for _, v in ipairs(l) do set[v] = true end
        return set
    end
```

```lua
    s1 = Set.new{10, 20, 30, 50}
    s2 = Set.new{30, 1}
    print(getmetatable(s1)) --> table: 00672B60
    print(getmetatable(s2)) --> table: 00672B60
```

接下来，向元表添加元方法：

```lua
	mt.__add = Set.union
```

于是可以：

```lua
    s3 = s1 + s2
    Set.print(s3) --> {1, 10, 20, 30, 50}
```

类似的，可以利用乘法做集合相交：

```lua
    mt.__mul = Set.intersection
    Set.print((s1 + s2)*s1) --> {10, 20, 30, 50}
```

其他运算符在元表中的字段名：`__add`、`__mul`、`__sub` (for subtraction), `__div` (for division), `__unm` (for negation), `__mod` (for modulo), and `__pow` (for exponentiation). 还有连接运算符`__concat`。

有时表达式涉及的两个值拥有不同的元表，如：

```lua
	s = Set.new{1,2,3}
	s = s + 8
```

寻找元方法时，步骤如下：如果第一个值的元表有`__add`字段，则使用该字段的值作为元方法，不管第二个值是什么。否则如果第二个值的元表有`__add`字段，则使用第二个值得；否则报错。

上面的例子，运行`s=s+8`会报错：`Set.union`: `bad argument #1 to 'pairs' (table expected, got number)`。或者，通过检查给出更明确的错误提示：

```lua
    function Set.union (a, b)
    	if getmetatable(a) ~= mt or getmetatable(b) ~= mt then
    		error("attempt to 'add' a set with a non-set value", 2)
        end
    <as before>
```

Remember that the second argument to error (2, in this example) directs the error message to where the operation was called.

### 13.2 关系元方法

Metatables also allow us to give meaning to the relational operators, through the metamethods `__eq` (equal to), `__lt` (less than), and `__le` (less than or equal to). There are no separate metamethods for the other three relational operators:

Lua translates `a~=b` to `not(a==b)`, `a>b` to `b<a`, and `a>=b` to `b<=a`. Until version 4.0, Lua translated all order operators to a single one, by translating `a<=b` to `not(b<a)`. However, this translation is incorrect when we have a partial order, that is, when not all elements in our type are properly ordered. For instance, floating-point numbers are not totally ordered in most machines, because of the value Not a Number (**NaN**). According to the IEEE 754 standard, NaN represents undefined values, such as the result of `0/0`. The standard specifies that any comparison that involves NaN should result in false.

This means that `NaN<=x` is always false, but `x<NaN` is also false. It also implies that the translation from `a<=b` to `not(b<a)` is not valid in this case. In our example with sets, we have a similar problem. An obvious (and useful) meaning for `<=` in sets is set containment: `a<=b` means that a is a subset of b. With this meaning, again it is possible that both `a<=b` and `b<a` are false; therefore, we need separate implementations for `__le` (less or equal) and `__lt` (less than):

```lua
    mt.__le = function (a, b) -- set containment
        for k in pairs(a) do
        	if not b[k] then return false end
        end
        return true
    end
    mt.__lt = function (a, b)
        return a <= b and not (b <= a)
    end
```

Finally, we can define set equality through set containment:

```lua
    mt.__eq = function (a, b)
    	return a <= b and b <= a
    end
```

After these definitions, we are ready to compare sets:

```lua
    s1 = Set.new{2, 4}
    s2 = Set.new{4, 10, 2}
    print(s1 <= s2) --> true
    print(s1 < s2) --> true
    print(s1 >= s1) --> true
    print(s1 > s1) --> false
    print(s1 == s2 * s1) --> true
```

For types that have a complete order, we do not need to define a `__leq` metamethod. In its absence, Lua will use the `__lt` entry. The equality comparison has some restrictions. If two objects have different basic types or different metamethods, the equality operation results in false, without even calling any metamethod. So, a set will always be different from a number, no matter what its metamethod says.

### 13.3 库定义的元方法

之前见到的元方法都是Lua核心定义的。It is the virtual machine that detects that the values involved in an operation have metatables with metamethods for that particular operation. 库也定义了（期待）一些元表中的特殊字段。

例如，`print`总是调用`tostring`格式化输出。`tostring`会检查值是否有元方法`__tostring`。若有，`tostring`会调用元方法，将其输出作为自己的输出。

例子：

```lua
	mt.__tostring = Set.tostring
```

```lua
	s1 = Set.new{10, 4, 5}
	print(s1) --> {4, 5, 10}
```

函数`setmetatable`和`getmetatable`也使用元表中一个字段`__metatable`，in this case to protect metatables. Suppose you want to protect your sets, so that users can neither see nor change their metatables. If you set a `__metatable` field in the metatable, `getmetatable` will return the value of this field, whereas `setmetatable` will raise an error:

```lua
    mt.__metatable = "not your business"
    s1 = Set.new{}
    print(getmetatable(s1)) --> not your business
    setmetatable(s1, {})
    stdin:1: cannot change protected metatable
```

In Lua 5.2, `pairs` and `ipairs` also got metatables, so that a table can modify the way it is traversed (and non-table objects can be traversed).

### 13.4 Table-Access Metamethods

The metamethods for arithmetic and relational operators all define behavior for otherwise erroneous situations. 它们不改变语言的正常行为。但Lua还提供一种方式改变表在两种情况下的行为：查询和修改表中缺失的字段。

#### `__index`元方法

> `__index`元方法只用于访问表的字段，即读取，修改表字段不涉及`__index`，而是涉及`__newindex`。见下节。

之前介绍说，若访问的字段表中不存在，返回`nil`。但其实不一定。这种访问会触发询问`__index`元方法：若没有此方法，返回`nil`；否则，将有元方法提供结果。

典型例子是实现继承。首先，定义原型和构造器函数：

```lua
    -- create the prototype with default values
    prototype = {x = 0, y = 0, width = 100, height = 100}
    mt = {} -- create a metatable
    -- declare the constructor function
    function new (o)
    	setmetatable(o, mt)
    	return o
    end
```

现在定义`__index`元方法：

```lua
	mt.__index = function (_, key)
    	return prototype[key]
    end
```

现在，我们创建一个window对象，试着访问缺失的字段：

```lua
	w = new{x=10, y=20}
	print(w.width) --> 100
```

将`__index`用作继承非常常见，于是Lua提供了一种快捷方式。`__index`的值可以不是函数，直接是一个表。此时，缺失字段将在该表中查找。因此上面的例子可以简化为：

```lua
	mt.__index = prototype
```

`__index`的值为表，只能实现单继承。若用函数，可以使用多继承、缓存等。We will discuss these forms of inheritance in Chapter 16.

若向访问表时，忽略`__index`元方法，可以使用`rawget`函数。`rawget(t, i)`访问表t。

#### `__newindex`元方法

`__newindex`元方法用于表更新。若你给一个不存在的键赋值，解析器会寻找`__newindex`元方法：解析器会调用该方法。与`__index`类似，如果`__newindex`的值是表，则赋值发生在这个表。

Moreover, there is a raw function that allows you to bypass the metamethod: the call `rawset(t, k, v)` sets the value v associated with key k in table t without invoking any metamethod.

`__index`和`__newindex`的组合可以实现一些强大的功能。如只读表，带默认值的表，面向对象的继承等。

#### 带默认值的表

表中字段的默认值是`nil`。利用元表可以改变此行为：

```lua
    function setDefault (t, d)
    	local mt = {__index = function () return d end}
    	setmetatable(t, mt)
    end
    tab = {x=10, y=20}
    print(tab.x, tab.z) --> 10 nil
    setDefault(tab, 0)
    print(tab.x, tab.z) --> 10 0
```

The setDefault function creates a new closure plus a new metatable for each table that needs a default value. This can be expensive if we have many tables that need default values. However, the metatable has the default value d wired into its metamethod, so the function cannot use a single metatable for all tables.

To allow the use of a single metatable for tables with different default values, we can store the default value of each table in the table itself, using an exclusive field. If we are not worried about name clashes, we can use a key like “`___`” for our exclusive field:

```lua
    local mt = {__index = function (t) return t.___ end}
    function setDefault (t, d)
    	t.___ = d
    	setmetatable(t, mt)
    end
```

Note that now we create the table mt only once, outside the SetDefault function. If we are worried about name clashes, it is easy to ensure the uniqueness of the special key. All we need is to create a new table and use it as the key:

```lua
    local key = {} -- unique key
    local mt = {__index = function (t) return t[key] end}
    function setDefault (t, d)
    	t[key] = d
    	setmetatable(t, mt)
    end
```

An alternative approach for associating each table with its default value is to use a separate table, where the indices are the tables and the values are their default values. However, for the correct implementation of this approach, we need a special breed of table called weak tables, and so we will not use it here; we will return to the subject in Chapter 17.

Another alternative is to memorize metatables in order to reuse the same metatable for tables with the same default. However, that needs weak tables too, so that again we will have to wait until Chapter 17.

#### （未）Tracking table accesses

#### （未）Read-only tables

## 14. （未）环境

Lua将所有的全局变量放在一个表中，称为全局环境。（更准确的锁，Lua将环境变量放在多个环境中，但我们先暂时简化成一个）。我们可以像操纵其他表一样操纵这个表。Lua将环境自己存放在全局变量`_G`中。于是`_G._G`等价于`_G`。

下面的代码邪乎所有全局变量的名字：

```lua
	for n in pairs(_G) do print(n) end
```

### 14.1 Global Variables with Dynamic Names

没什么。

### 14.2 全局变量声明

Lua中的全局变量不需要声明。于是用户有拼错的风险且很难觉察。为防止这种问题，有多种方法。我们可以使用metatable改变访问全局变量时的行为。

第一步是侦测到全局变量并不存在：

```lua
    setmetatable(_G, {
    	__newindex = function (_, n)
    		error("attempt to write to undeclared variable " .. n, 2)
        end,
    	__index = function (_, n)
    		error("attempt to read undeclared variable " .. n, 2)
    	end,
    })
```

此后，尝试访问一个不存在的全局变量将报错：

```lua
    > print(a)
    stdin:1: attempt to read undeclared variable a
```

但如何设置新的变量？一个办法是使用`rawset`，绕过元方法：

```lua
    function declare (name, initval)
    	rawset(_G, name, initval or false)
    end
```

`or`是为了全局总是获取到一个非`nil`的值。

更简单的方法是限制，给新全局变量赋值不能发生在函数内部，allowing free assignments in the outer level of a chunk。要检查赋值发生在哪，必须使用debug库。The call `debug.getinfo(2, "S")` returns a table whose field what tells whether the function that called the metamethod is a main chunk, a regular Lua function, or a C function. (We will see `debug.getinfo` in more detail in Chapter 24.) Using this function, we can rewrite the `__newindex` metamethod like this:

```lua
    __newindex = function (t, n, v)
        local w = debug.getinfo(2, "S").what
        if w ~= "main" and w ~= "C" then
        	error("attempt to write to undeclared variable " .. n, 2)
        end
        rawset(t, n, v)
    end
```

This new version also accepts assignments from C code, as this kind of code usually knows what it is doing.

To test whether a variable exists, we cannot simply compare it to nil because, if it is nil, the access will throw an error. Instead, we use rawget, which avoids the metamethod:

```lua
    if rawget(_G, var) == nil then
        -- 'var' is undeclared
        ...
    ends
```

As it is, our scheme does not allow global variables with nil values, as they would be automatically considered undeclared. But it is not difficult to correct this problem. All we need is an auxiliary table that keeps the names of declared variables. Whenever a metamethod is called, it checks in this table whether the variable is undeclared or not. The code can be like in Listing 14.1. Now even an assignment like x=nil is enough to declare a global variable.

The overhead for both solutions is negligible. With the first solution, the metamethods are never called during normal operation. In the second, they can be called, but only when the program accesses a variable holding a nil. The Lua distribution comes with a module `strict.lua` that implements a global-variable check that uses essentially the code we just reviewed. It is a good habit to use it when developing Lua code.

### 14.3 非全局环境

One of the problems with the environment is that it is global. Any modification you do on it affects all parts of your program. For instance, when you install a metatable to control global access, your whole program must follow the guidelines.

If you want to use a library that uses global variables without declaring them, you are in bad luck. In Lua, global variables do not need to be truly global. We can even say that Lua does not have global variables. That may sound strange at first, as we have been using global variables all along this text. Clearly, Lua goes to great lengths to give the programmer an illusion of global variables. Let us see how Lua builds this illusion.(Note that this mechanism was one of the parts of Lua that changed most from version 5.1 to 5.2. The following discussion refers to Lua 5.2 and very little of it applies to previous versions.)

Listing 14.1. Checking global-variable declaration:

```lua
    local declaredNames = {}
    setmetatable(_G, {
        __newindex = function (t, n, v)
            if not declaredNames[n] then
                local w = debug.getinfo(2, "S").what
                if w ~= "main" and w ~= "C" then
                    error("attempt to write to undeclared variable "..n, 2)
                end
                declaredNames[n] = true
            end
            rawset(t, n, v) -- do the actual set
        end,
    	__index = function (_, n)
        	if not declaredNames[n] then
        		error("attempt to read undeclared variable "..n, 2)
        	else
        		return nil
        	end
        end,
    })
```

Let us start with the concept of free names. A free name is a name that is
not bound to an explicit declaration, that is, it does not occur inside the scope of
a local variable (or a for variable or a parameter) with that name. For instance,
both var1 and var2 are free names in the following chunk:
	var1 = var2 + 3
Unlike what we said earlier, a free name does not refer to a global variable (at
least not in a direct way). Instead, the Lua compiler translates any free name
var to _ENV.var. So, the previous chunk is equivalent to this one:
	_ENV.var1 = _ENV.var2 + 3
But what is this new _ENV variable? It cannot be a global variable; otherwise,
we would be back to the original problem. Again, the compiler does the trick.
I already mentioned that Lua treats any chunk as an anonymous function.
Actually, Lua compiles our original chunk as the following code:
    local _ENV = <some value>
    return function (...)
    _ENV.var1 = _ENV.var2 + 3
    end
That is, Lua compiles any chunk in the presence of a predefined upvalue called
_ENV.
Usually, when we load a chunk, the load function initializes this predefined
upvalue with the global environment. So, our original chunk becomes equivalent
to this one:
    local _ENV = <the global environment>
    return function (...)
    _ENV.var1 = _ENV.var2 + 3
    end
The result of all these arrangements is that the var1 field of the global environment
gets the value of the var2 field plus 3.
At first sight, this may seem a rather convoluted way to manipulate global
variables. I will not argue that it is the simplest way, but it offers a flexibility
that is difficult to achieve with a simpler implementation.
Before we go on, let us summarize the handling of global variables in Lua 5.2:
 Lua compiles any chunk in the scope of an upvalue called _ENV.
 The compiler translates any free name var to _ENV.var.
 The load (or loadfile) function initializes the first upvalue of a chunk with
the global environment.
After all, it is not that complicated.
Some people get confused because they try to infer extra magic from these
rules. There is no extra magic. In particular, the first two rules are done entirely
by the compiler. Except for being predefined by the compiler, _ENV is a plain
regular variable. Outside compilation, the name _ENV has no special meaning at
all to Lua.3 Similarly, the translation from var to _ENV.var is a plain syntactic
translation, with no hidden meanings. In particular, after the translation, _ENV
will refer to whatever _ENV variable is visible at that point in the code, following
the standard visibility rules.

## 15. 模块与包

从5.1开始，Lua为模块和包定义了一组策略（包是模块的集合）。这些策略并未要求语言增加新的特性；使用的都是已有的特性：tables, functions, metatables, and environments。

从用户的视觉看，模块是通过`require`加载的代码（Lua代码或C代码），它创建并返回一个表。模块导出的所有内容，包括函数和常量，都定义在表中。这个表作为一个命名空间。

所有的标准库都是模块。例如，可以像这样使用数学库：

```lua
    local m = require "math"
    print(m.sin(3.14))
```

However, the stand-alone interpreter preloads all standard libraries with code equivalent to this:

```lua
    math = require "math"
    string = require "string"
    ...
```

模块是表，于是具有表的所有功能。例如，用户调用模块中的函数，有多种方法：

```lua
    local mod = require "mod"
    mod.foo()
```

模块可以用任意的本地名：

```lua
    local m = require "mod"
    m.foo()
```

In any way, remember that the module itself is loaded only once; it is up to the module to handle conflicting initializations.

### 15.1 require函数

The require function tries to keep to a minimum its assumptions about what a module is. For require, a module is just any code that defines some values (such as functions or tables containing functions). Typically, that code returns a table comprising the module functions. However, because this action is done by the module code, not by require, some modules may choose to return other values or even to have side effects.

要加载模块，只要调用`require"modname"`。第一步是检查表`package.loaded`，看模块是否已加载。如果已加载，直接返回。

若模块尚未加载，`require`寻找一个以模块命名的Lua文件。如果找到了Lua文件，用`loadfile`加载。加载的结果是一个函数，我们称之为loader。（loader是一个函数，调用该函数加载模块）。

如果没有以模块命名的Lua文件，则查找以模块命名的C库。若找到了一个C库，则用`package.loadlib`加载它（参见8.3），寻找一个名叫`luaopen_modname`的函数。`loadlib`的结果作为加载器，that is, the function `luaopen_modname` represented as a Lua function.

不管模块是Lua文件还是C库，`require`现在有了一个模块的加载器。为完成加载，`require`调用加载器，传入两个参数：模块名和加载器所在的文件名。（多数模块会忽略这两个参数）加载器返回的值，即`require`返回的值；同时该值会被存储到`package.loaded`表以缓存。如加载器没有返回，`require` behaves as if the module returned true. Without this correction, a subsequent call to require would run the module again.

若想强制重新加载，我们只要删掉缓存：`package.loaded.<modname> = nil`。

#### 重命名模块

Usually, we use modules with their original names, 但有时必须重命名模块以防止名字冲突。例如，我们需要加载同一模块的不同版本。Lua modules do not have their names fixed internally, 因此只要重命名`.lua`文件就好了。但我们无法编译二进制的库，改变`luaopen_*`函数的名字。为了能在此情况下重命名，`require`使用了一个小伎俩：如果模块名包含中划线，中划线之前的部分被当作前缀，后面的部分才会参与`luaopen_*`名的构成。例如，如果模块名叫`a-b`，`require`期望open函数名叫`luaopen_b`，而不是`luaopen_a-b`（后者本身也不是有效的函数名）。因此如果我们想使用两个都叫`mod`的模块，我们可以将其中一个命名为`v1-mod`。然后调用`m1=require"v1-mod"`，就可以找到`v1-mod`模块。在这个文件中，函数的名字还是原来的：`luaopen_mod`。

#### 路径搜索

`require`使用的路径与典型的路径不同。典型的路径是一组目录。但ANSI C没有目录的概念。因此`require`的路径是一组模板，利用模板将模块名（`require`函数的参数）转换为文件名。模板中可能带问号，问号会被替换为模块名。路径中的模板分号分隔。例如：

	?;?.lua;c:\windows\?;/usr/local/lua/?/?.lua

当调用`require"sql"`时上述模板会被转换为：

    sql sql.lua
    c:\windows\sql
    /usr/local/lua/sql/sql.lua

The require function assumes only the semicolon (as the component separator) and the question mark; everything else, including directory separators and file extensions, is defined by the path itself.

`require`的使用的路径是`package.path`的值。当Lua启动时，这个变量被赋予`LUA_PATH_5_2`。若未定义此变量，Lua尝试另一个变量`LUA_PATH`。If both are not defined, Lua uses a compiled-defined default path.(In Lua 5.2, the command-line option -E prevents the use of those environment variables and forces the default.) 若环境变量的值中的`;;`会被替换为默认路径。例如，若把`LUA_PATH_5_2`设为`mydir/?.lua;;`，最终的模板是`mydir/?.lua`加上默认路径。

用于搜索C库的路径来自`package.cpath`，而不是`package.path`。该变量的初始值来自环境变量`LUA_CPATH_5_2`或`LUA_CPATH`。Unix下的典型值是：

	./?.so;/usr/local/lib/lua/5.2/?.so

注意到路径中包含扩展名。Windows下则一般是：

	.\?.dll;C:\Program Files\Lua502\dll\?.dll

函数`package.searchpath`封装了用于搜索库的所有规则。传入模块名和路径，它就会按上述规则搜索。它返回找到的第一个文件，或`nil`加一段错误消息：

    > path = ".\\?.dll;C:\\Program Files\\Lua502\\dll\\?.dll"
    > print(package.searchpath("X", path))
    nil
    no file '.\X.dll'
    no file 'C:\Program Files\Lua502\dll\X.dll'

#### Searchers

In reality, `require` is a little more complex than we have described.搜索Lua文件和搜索C库只是更一般的searchers概念的两个特列。一个搜索器（searcher）是一个函数，传入模块名，返回模块的加载器或`nil`（表示未找到）。

数组`package.searchers`列出了`require`使用的所有搜索器。When looking for a module, require calls each searcher in the list passing the module name, until one of them finds a loader for the module. If the list ends without a positive response, require raises an error.

The use of a list to drive the search for a module allows great flexibility to require. 例如，如果模块压缩在一个zip文件中，你只需要将响应的搜索器函数添加到列表。However, more often than not, programs do not change the default contents of `package.searchers`. 默认配置，Lua文件的搜索器和C库的搜索器分别是列表的第二和第三个参数。在她们之前，有一个预加载搜索器。

The preload searcher allows the definition of an arbitrary function to load a module. It uses a table, called `package.preload`, to map module names to loader functions. When searching for a module name, this searcher simply looks for the given name in the table. If it finds a function there, it returns this function as the module loader. Otherwise, it returns nil. This searcher provides a generic method to handle some non-conventional situations. For instance, a C library statically linked to Lua can register its `luaopen_` function into the preload table, so that it will be called only when (and if) the user requires that module. In this way, the program does not waste time opening the module if it is not used.

`package.searchers`的默认内容包含第四个函数，只与子模块有关，见15.4。

### 15.2 使用Lua编写模块的基本过程

最简单的方式是，创建一个表，将所有要导出的函数放入表，然后返回该表。Listing 15.1 illustrates this approach. Note how we define `inv` as a private function simply by declaring it local to the chunk.

Some people do not like the final return statement. One way of eliminating it is to assign the module table directly into `package.loaded`:

```lua
    local M = {}
    package.loaded[...] = M
    <as before>
```

Remember that require calls the loader passing the module name as the first argument. So, the vararg expression ... in the index results in that name. After this assignment, we do not need to return M at the end of the module: if a module does not return a value, require will return the current value of `package.loaded[modname]` (if it is not nil). Anyway, I still prefer to write the final return, because it looks clearer.

Another variant for writing a module is to define all functions as locals and build the returning table at the end, as in Listing 15.2. What are the advantages of this approach? You do not need to prefix each name with `M.` or something similar; there is an explicit export list; and you define and use exported and internal functions in the same way inside the module. What are the disadvantages? The export list is at the end of the module instead of at the beginning, where it would be more useful as a quick documentation; and the export list is somewhat redundant, as you must write each name twice. (This last disadvantage may become an advantage, as it allows functions to have different names inside and outside the module, but I think programmers seldom do this.) I particularly like this style, but tastes may differ.

Anyway, remember that no matter how a module is defined, users should be able to use it in a standard way:

```lua
    local cpx = require "complex"
    print(cpx.tostring(cpx.add(cpx.new(3,4), cpx.i)))
    --> (3,5)
```

Listing 15.1. A simple module for complex numbers:

```lua
    local M = {}
    function M.new (r, i) return {r=r, i=i} end
    -- defines constant 'i'
    M.i = M.new(0, 1)
    function M.add (c1, c2)
    	return M.new(c1.r + c2.r, c1.i + c2.i)
    end
    function M.sub (c1, c2)
    	return M.new(c1.r - c2.r, c1.i - c2.i)
    end
    function M.mul (c1, c2)
    	return M.new(c1.r*c2.r - c1.i*c2.i, c1.r*c2.i + c1.i*c2.r)
    end
    local function inv (c)
    	local n = c.r^2 + c.i^2
    	return M.new(c.r/n, -c.i/n)
    end
    function M.div (c1, c2)
    	return M.mul(c1, inv(c2))
    end
    function M.tostring (c)
    	return "(" .. c.r .. "," .. c.i .. ")"
    end
    return M
```

Listing 15.2. Module with export list:

```lua
    local function new (r, i) return {r=r, i=i} end
    -- defines constant 'i'
    local i = complex.new(0, 1)
    <other functions follow the same pattern>
    return {
        new = new,
        i = i,
        add = add,
        sub = sub,
        mul = mul,
        div = div,
        tostring = tostring,
    }
```

### （未）15.3 Using Environments

### 15.4 子模块和包

Lua allows module names to be hierarchical, using a dot to separate name levels. 例如`mod.sub`是`mod`的子模块。A package is a complete tree of modules; it is the unit of distribution in Lua.

When you require a module called `mod.sub`, require queries first the table `package.loaded` and then the table `package.preload` using the original module name “`mod.sub`” as the key; here, the dot is just a character like any other in the module name.

However, when searching for a file that defines that submodule, `require` translates the dot into another character, usually the system’s directory separator (e.g., ‘/’ for UNIX or ‘\’ for Windows). After the translation, require searches for the resulting name like any other name. For instance, assume ‘/’ as the directory separator and the following path:

	./?.lua;/usr/local/lua/?.lua;/usr/local/lua/?/init.lua

The call `require"a.b"` will try to open the following files:

    ./a/b.lua
    /usr/local/lua/a/b.lua
    /usr/local/lua/a/b/init.lua

This behavior allows all modules of a package to live in a single directory. For instance, if a package has modules p, p.a, and p.b, their respective files can be p/init.lua, p/a.lua, and p/b.lua, with the directory p within some appropriate directory.

The directory separator used by Lua is configured at compile time and can be any string (remember, Lua knows nothing about directories). For instance, systems without hierarchical directories can use a ‘_’ as the “directory” separator, so that require"a.b" will search for a file `a_b.lua`.

Names in C cannot contain dots, so a C library for submodule a.b cannot export a function luaopen_a.b. Here require translates the dot into another character, an underscore. So, a C library named a.b should name its initialization function `luaopen_a_b`. We can use the hyphen trick here too, with some subtle results. For instance, if we have a C library a and we want to make it a submodule of mod, we can rename the file to mod/v-a. When we write require"mod.v-a", require correctly finds the new file mod/v-a as well as the function luaopen_a inside it.

As an extra facility, require has one more searcher for loading C submodules. When it cannot find either a Lua file or a C file for a submodule, this last searcher searches again the C path, but this time looking for the package name. For example, if the program requires a submodule a.b.c this searcher will look for a. If it finds a C library for this name, then require looks into this library for an appropriate open function, `luaopen_a_b_c` in this example. This facility allows a distribution to put several submodules together into a single C library, each with its own open function.

From the point of view of Lua, submodules in the same package have no explicit relationship. Requiring a module a does not automatically load any of its submodules; similarly, requiring a.b does not automatically load a. Of course, the package implementer is free to create these links if she wants. For instance, a particular module a may start by explicitly requiring one or all of its submodules.

## 16. 面向对象编程

Like objects, tables have an identity (a self ) that is independent of their values. Like objects, tables have a life cycle that is independent of who created them or where they were created.

使用冒号运算符和`self`定义对象方法：

```lua
	Account = {balance = 0}
    function Account:withdraw (v)
    	self.balance = self.balance - v
    end
```

函数调用：

```lua
	a:withdraw(100.00)
```

冒号的作用是，向方法形参添加一个隐藏参数，向调用添加一个实参。但冒号其实只是语法糖衣。我们可以显式写出隐藏的内容。我们可以用点定义函数，使用冒号调用；反之也可以；只要正确处理额外的参数：

```lua
    Account = { balance=0,
    	withdraw = function (self, v)
    		self.balance = self.balance - v
    	end
    }

    function Account:deposit (v)
	    self.balance = self.balance + v
    end
    Account.deposit(Account, 200.00)
    Account:withdraw(100.00)
```

### 16.1 类

Lua没有类的概念。Nevertheless, it is not difficult to emulate classes in Lua, following the lead from prototype-based languages like Self and NewtonScript. 在这些语言中，对象没有类。但可能有一个原型。类和原型的作用让多个对象都是共享行为。

关于原型和继承参见Section 13.4。如果有两个对象 a 和 b，若想让 b 做 a 的原型：

```lua
	setmetatable(a, {__index = b})
```

回到银行账户的例子。为了让其他账户拥有类似Account的行为，我们让这些新对象以Account做原型。一个小优化是，不用创建额外的元表，让Account自己做元表：

```lua
    function Account:new (o)
        o = o or {} -- create table if user does not provide one
        setmetatable(o, self)
        self.__index = self
        return o
    end
```

（当调用`Account:new`时，`self`等于`Account`；so, we could have used Account directly, instead of self. However, the use of self will fit nicely when we introduce class inheritance, in the next section.）

After this code, what happens when we create a new account and call a method on it, like this?

```lua
	a = Account:new{balance = 0}
	a:deposit(100.00)
```

`a:deposit(100.00)`就像是在写：

```lua
	getmetatable(a).__index.deposit(a, 100.00)
```

a的元表示Account，`Account.__index`的元表也是Account（由于`self.__index=self`）。因此之前的表达式简化成`Account.deposit(a, 100.00)`。即，Lua调用之前的函数，但传入`a`作为self参数。

若对b对象调用`deposit`方法，等价调用：

```lua
	b.balance = b.balance + v
```

上面的方法代码执行后。后续访问`b.balance`将不再需要`__index`元方法，因为`balance`是`b`自己的字段。

### 16.2 继承

Lua中实现继承非常简单。

假设我们有基类`Account`：

```lua
    Account = {balance = 0}
    function Account:new (o)
    	o = o or {}
    	setmetatable(o, self)
    	self.__index = self
    	return o
    end
    function Account:deposit (v)
    	self.balance = self.balance + v
    end
    function Account:withdraw (v)
    	if v > self.balance then error"insufficient funds" end
    	self.balance = self.balance - v
    end
```

我们想要一个子类`SpecialAccount`，允许用户透支。

```lua
	SpecialAccount = Account:new()
```

此时`SpecialAccount`只是`Account`的一个实例。

我们可以重定义继承来的方法。

```lua
    function SpecialAccount:withdraw (v)
    	if v - self.balance >= self:getLimit() then
    		error"insufficient funds"
    	end
    	self.balance = self.balance - v
    end
    function SpecialAccount:getLimit ()
    	return self.limit or 0
    end
```

如果仅是单个对象需要新行为，不必再定义新类。执行让这个对象重新定义相关方法即可。如

```lua
    function s:getLimit ()
    	return self.balance * 0.10
    end
```

### 16.3 多继承

Lua中实现面向对象编程的方法有几种。The approach that we have seen, using the index metamethod, is probably the best combination of simplicity, performance, and flexibility. Nevertheless, there are other implementations, which may be more appropriate for some particular cases. Here we will see an alternative implementation that allows multiple inheritance in Lua.

The key to this implementation is the use of a function for the metafield `__index`. Remember that, when a table’s metatable has a function in the `__index` field, Lua will call this function whenever it cannot find a key in the original table. Then, `__index` can look up for the missing key in how many parents it wants.

Multiple inheritance means that a class can have more than one superclass. Therefore, we cannot use a class method to create subclasses. Instead, we will define a specific function for this purpose, createClass, which has as arguments the superclasses of the new class; see Listing 16.1. This function creates a table to represent the new class, and sets its metatable with an `__index` metamethod that does the multiple inheritance. Despite the multiple inheritance, each object instance still belongs to one single class, where it looks for all its methods.

Therefore, the relationship between classes and superclasses is different from the relationship between classes and instances. Particularly, a class cannot be the metatable for its instances and for its subclasses at the same time. In Listing 16.1, we keep the class as the metatable for its instances, and create another table to be the metatable of the class.

Let us illustrate the use of createClass with a small example. Assume our previous class Account and another class, Named, with only two methods: setname and getname.

```lua
    Named = {}
    function Named:getname ()
    	return self.name
    end
    function Named:setname (n)
    	self.name = n
    end
```

Listing 16.1. An implementation of multiple inheritance:
    -- look up for 'k' in list of tables 'plist'
    local function search (k, plist)
    for i = 1, #plist do
    local v = plist[i][k] -- try 'i'-th superclass
    if v then return v end
    end
    end
    function createClass (...)
    local c = {} -- new class
    local parents = {...}
    -- class will search for each method in the list of its parents
    setmetatable(c, {__index = function (t, k)
    return search(k, parents)
    end})
    -- prepare 'c' to be the metatable of its instances
    c.__index = c
    -- define a new constructor for this new class
    function c:new (o)
    o = o or {}
    setmetatable(o, c)
    return o
    end
    return c -- return new class
    end

To create a new class NamedAccount that is a subclass of both Account and Named, we simply call createClass:

```lua
    NamedAccount = createClass(Account, Named)
```

To create and to use instances, we do as usual:
```lua
	account = NamedAccount:new{name = "Paul"}
    print(account:getname()) --> Paul
```

Now let us follow how this last statement works. Lua cannot find the field “getname” in account; so, it looks for the field `__index` of account’s metatable, which is NamedAccount. But NamedAccount also cannot provide a “getname” field, so Lua looks for the field `__index` of NamedAccount’s metatable. Because this field contains a function, Lua calls it. This function then looks for “getname” first in Account, without success, and then in Named, where it finds a non-nil value, which is the final result of the search.

Of course, due to the underlying complexity of this search, the performance of multiple inheritance is not the same as single inheritance. A simple way to improve this performance is to copy inherited methods into the subclasses. Using this technique, the index metamethod for classes would be like this:

    setmetatable(c, {__index = function (t, k)
    local v = search(k, parents)
    t[k] = v -- save for next access
    return v
    end})

With this trick, accesses to inherited methods are as fast as to local methods (except for the first access). The drawback is that it is difficult to change method definitions after the system is running, because these changes do not propagate down the hierarchy chain.

### （未）16.4 Privacy

### （未）16.5 The Single-Method Approach

## 17. Weak Tables and Finalizers

Lua does automatic memory management. Programs create objects (tables, threads, etc.), but there is no function to delete objects. Lua automatically deletes objects that become garbage, using garbage collection.

The use of a real garbage collector means that Lua has no problems with cycles. You do not need to take any special action when using cyclic data structures; they are collected like any other data. Nevertheless, sometimes even the smarter collector needs your help. No garbage collector allows you to forget all worries about resource management, such as hoarding memory and external resources.

Weak tables and finalizers are the mechanisms that you can use in Lua
to help the garbage collector. Weak tables allow the collection of Lua objects that are still accessible to the program, while finalizers allow the collection of external objects that are not directly under control of the garbage collector. In this chapter, we will discuss both mechanisms.

### （未）17.1 Weak Tables

存储在全局变量中的对象不会被GC。要通过给它们赋值`nil`让垃圾收集器回收它们。

However, simply cleaning your references is not always enough. Some constructions need extra collaboration between the program and the collector. A typical example happens when you want to keep a collection of all live objects of some kind (e.g., files) in your program. This task seems simple: all you have to do is to insert each new object into the collection. However, once the object is part of the collection, it will never be collected! Even if no one else points to it, the collection does. Lua cannot know that this reference should not prevent the reclamation of the object, unless you tell Lua about this fact.

弱表用于告诉Lua，此引用并不阻止回收。如果指向对象的所有引用都是弱引用，对象会被收集，弱引用会被删除。Lua implements weak references as weak tables: 弱表中的项都是弱的。即如果一个对象，只被弱表们引用，Lua最终将回调那个对象。

Tables have keys and values, and both can contain any kind of object. Under normal circumstances, the garbage collector does not collect objects that appear as keys or as values of an accessible table. 即键值都是强引用。但在若表中，键和值都可以是弱的。即有三种弱表：键是弱的，值是弱的，二者都弱。不管什么表，当键或值被收集后，整个项都会从表中消失。

表弱不弱区域元表的`__mode`字段。这个字段的值是字符串：if this string is “k”, the keys in the table are weak; if this string is “v”, the values in the table are weak; if this string is “kv”, both keys and values are weak. The following example, although artificial, illustrates the basic behavior of weak tables:

```lua
    a = {}
    b = {__mode = "k"}
    setmetatable(a, b) -- now 'a' has weak keys
    key = {} -- creates first key
    a[key] = 1
    key = {} -- creates second key
    a[key] = 2
    collectgarbage() -- forces a garbage collection cycle
    for k, v in pairs(a) do print(v) end
    --> 2
```

In this example, the second assignment key={} overwrites the reference to the first key. The call to collectgarbage forces the garbage collector to do a full collection. As there is no other reference to the first key, this key is collected and the corresponding entry in the table is removed. The second key, however, is still anchored in variable key, so it is not collected.

Notice that only objects can be collected from a weak table. Values, such as numbers and booleans, are not collectible. For instance, if we insert a numeric key in table a (from our previous example), the collector will never remove it. Of　course, if the value corresponding to a numeric key is collected in a table with　weak values, then the whole entry is removed from the table.

Strings present a subtlety here: although strings are collectible, from an implementation　point of view, they are not like other collectible objects. Other objects, such as tables and threads, are created explicitly. For instance, whenever Lua evaluates the expression {}, it creates a new table. However, does　Lua create a new string when it evaluates "a".."b"? What if there is already a string “ab” in the system? Does Lua create a new one? Can the compiler create this string before running the program? It does not matter: these are implementation　details. From the programmer’s point of view, strings are values, not objects. Therefore, like a number or a boolean, a string is not removed from　weak tables (unless its associated value is collected).


