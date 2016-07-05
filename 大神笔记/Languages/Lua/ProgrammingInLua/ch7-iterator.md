[toc]

## 7. 迭代器与通用for

Lua中，迭代器一般表示为函数：每次调用函数，返回集合中下一个元素。

迭代器需要维护状态，如当前迭代到哪个元素。闭包适于完成此项任务。闭包构造器一般包含两个函数，闭包自己和工厂，工厂负责创建闭包和闭包所需的**非局部变量**。

例子，编写一个列表的迭代器。这个迭代器不像`ipairs`，只返回值，不返回索引：

```lua
    function values (t)
    	local i = 0
    	return function () i = i + 1; return t[i] end
    end
```

可以在while循环中使用该迭代器：

```lua
    t = {10, 20, 30}
    iter = values(t) -- creates the iterator
    while true do
    	local element = iter() -- calls the iterator
    	if element == nil then break end
    		print(element)
    end
```

但使用通用for更容易：

```lua
    t = {10, 20, 30}
    for element in values(t) do
   		print(element)
    end
```

通用for调用的是迭代器工厂，它自己负责保存工厂产生迭代器（闭包）。遇到`nil`时停止迭代。

### 7.2 通用for的语义

之前迭代器的缺点是，每次新循环都要创建一个新闭包。多数情况下，这个开销不是问题。如上面的例子中，`allwords`迭代器相对于读取文件的开销不大。但有时不能忽略此开销。在这种情况下，我们可以利用**通用for**自己维护遍历的状态。本节介绍通用for提供的维护状态的功能。

通用for实际会维护三个值：迭代器函数，一个不可变状态和一个控制变量。

通用for的语法是：

```lua
    for <var-list> in <exp-list> do
    	<body>
    end
```

`var-list`是一个或多个变量，逗号分隔。`exp-list`是一个或多个表达式，也是逗号分隔。多数情况下，表达式列表中只有一个元素，一般是迭代器工厂的调用。

```lua
	for k, v in pairs(t) do print(k, v) end
```

变量列表中第一个变量称为控制变量。循环过程中，它的值不能为`nil`。因为`nil`表示循环结束。

for循环做的第一件事是对表达式求值。这些表达式应该产生for循环感兴趣的三个值：迭代器函数、不可变状态和控制变量的初始值。与多赋值一样，只有列表中最后一个元素可以产生多于一个值；且最终所有结果只取前三个。（当我们使用的是简单的迭代器时，工程返回的是迭代器函数，因此不可变状态和控制变量都是`nil`。）

在初始化完成后，for循环调用迭代器函数时会传入两个实参：不可变状态和控制变量。然后迭代器函数返回的值会被赋给变量列表中的变量。如果第一个值（控制变量）返回nil，循环结束。否则，for执行代码并继续。

通用for的代码：

```lua
    for var_1, ..., var_n in <explist> do <block> end
```

等价于：

```lua
    do
        local _f, _s, _var = <explist>
        while true do
            local var_1, ... , var_n = _f(_s, _var)
            _var = var_1
            if _var == nil then break end
            	<block>
        end
    end
```

So, if our iterator function is f, the invariant state is s, and the initial value for the control variable is a0, the control variable will loop over the values a1 = f(s; a0), a2 = f(s; a1), and so on, until ai is nil. If the for has other variables, they simply get the extra values returned by each call to f.

### 7.3 无状态的迭代器

无状态的迭代器自身不维护任何状态。因此它可以被多个循环使用。避免了创建新闭包的开销。

之前讲到，for循环调用迭代器函数时会传递两个参数（不可变状态和控制变量）。无状态迭代器只依赖这两个值产生下一个元素。这种迭代器的典型例子是`ipairs`，用于迭代数组中的所有元素：

```lua
    a = {"one", "two", "three"}
    for i, v in ipairs(a) do
    	print(i, v)
    end
```

迭代的状态包括：被遍历的表（即不可变状态），当前下标（控制变量）。`ipairs`（工厂）和迭代器都相当简单：

```lua
    local function iter (a, i)
        i = i + 1
        local v = a[i]
        if v then
        	return i, v
        end
    end
    function ipairs (a)
        return iter, a, 0
    end
```

The pairs function, which iterates over all elements of a table, is similar, except that the iterator function is the `next` function, which is a primitive function in Lua:

```lua
    function pairs (t)
    	return next, t, nil
    end
```

The call `next(t,k)`, where k is a key of the table t, returns a next key in the table, in an arbitrary order, plus the value associated with this key as a second return value. `next(t, nil)`返回第一个项。如果没有别的项了，返回nil。

有些人更喜欢直接用`next`，而不用`pairs`：

```lua
    for k, v in next, t do
    	<loop body>
    end
```

An iterator to traverse a linked list is another interesting example of a stateless iterator. (As we already mentioned, linked lists are not frequent in Lua, but sometimes we need them.)

```lua
    local function getnext (list, node)
        if not node then
        	return list
        else
        	return node.next
        end
    end
    function traverse (list)
    	return getnext, list, nil
    end
```

The trick here is to use the list main node as the invariant state (the second value returned by traverse) and the current node as the control variable. The first time the iterator function getnext is called, node will be nil, and so the function will return list as the first node. In subsequent calls, node will not be nil, and so the iterator will return node.next, as expected. As usual, it is trivial to use the iterator:

```lua
    list = nil
    for line in io.lines() do
    	list = {val = line, next = list}
    end
    for node in traverse(list) do
    	print(node.val)
    end
```

### 7.4 复杂状态

常常，迭代器除了不可变状态和控制变量，还需要维护其他状态。最简单的解决方法是使用闭包。另一种方法是，将迭代器需要的参数打包仅表，用这个表作为迭代的不可变状态。有了这个表，甚至可以忽略控制变量。

As an example of this technique, we will rewrite the iterator `allwords`, which traverses all the words from the current input file. This time, we will keep its state using a table with two fields: `line` and `pos`. The function that starts the iteration is simple. It must return the iterator function and the initial state:

```lua
    local iterator -- to be defined later
    function allwords ()
    	local state = {line = io.read(), pos = 1}
    return iterator, state
    end
```

The iterator function does the real work:

```lua
    function iterator (state)
        while state.line do -- repeat while there are lines
            -- search for next word
            local s, e = string.find(state.line, "%w+", state.pos)
            if s then -- found a word?
            	-- update next position (after this word)
            	state.pos = e + 1
            	return string.sub(state.line, s, e)
            else -- word not found
            	state.line = io.read() -- try next line...
            	state.pos = 1 -- ... from first position
            end
        end
        return nil -- no more lines: end loop
    end
```

Whenever possible, you should try to write stateless iterators, those that keep all their state in the for variables. With them, you do not create new objects when you start a loop. If you cannot fit your iteration into this model, then you should try closures. Besides being more elegant, typically a closure is more efficient than an iterator using tables: first, it is cheaper to create a closure than a table; second, access to non-local variables is faster than access to table fields. Later we will see yet another way to write iterators, with coroutines. This is the most powerful solution, but a little more expensive.

### 7.5 真正的迭代器

The name “iterator” is a little misleading, because our iterators do not iterate: what iterates is the for loop. Iterators only provide the successive values for the iteration. Maybe a better name would be “**generator**”, but “iterator” is already well established in other languages, such as Java.

However, there is another way to build iterators wherein iterators actually do the iteration. When we use such iterators, we do not write a loop; instead, we simply call the iterator with an argument that describes what the iterator must do at each iteration. More specifically, the iterator receives as argument a function that it calls inside its loop.

As a concrete example, let us rewrite once more the `allwords` iterator using this style:

```lua
    function allwords (f)
        for line in io.lines() do
            for word in string.gmatch(line, "%w+") do
            	f(word) -- call the function
            end
        end
    end
```

To use this iterator, we must supply the loop body as a function. If we want only to print each word, we simply use print:

allwords(print)

Often, we use an anonymous function as the body. For instance, the next code fragment counts how many times the word “hello” appears in the input file:

local count = 0
allwords(function (w)
if w == "hello" then count = count + 1 end
end)
print(count)

The same task, written with the previous iterator style, is not very different:
local count = 0
for w in allwords() do
if w == "hello" then count = count + 1 end
end
print(count)

True iterators were popular in older versions of Lua, when the language did not have the for statement. How do they compare with generator-style iterators? Both styles have approximately the same overhead: one function call per iteration. On the one hand, it is easier to write the iterator with true iterators (although we can recover this easiness with coroutines). On the other hand, the generator style is more flexible. First, it allows two or more parallel iterations. (For instance, consider the problem of iterating over two files comparing them word by word.) Second, it allows the use of break and return inside the iterator body. With a true iterator, a return returns from the anonymous function, not from the function doing the iteration. Overall, I usually prefer generators.