[toc]
## Part III 标准库

## （未）18. 数学库

## （未）19. 位运算库

## 20. Table 库

Table库提供的函数把表当作数组看待。It provides functions to insert and remove elements from lists, to sort the elements of an array, and to concatenate all strings in an array.

### 20.1 插入和移除

`table.insert`用于向数组执行位置插入元素。如果如果`t`是数组`{10,20,30}`，在调用`table.insert(t,1,15)`后，`t`变成`{15,10,20,30}`。若调用时不指定位置，则在数组最后插入。As an example, the following code reads the program input line by line, storing all lines in an array:

```lua
    t = {}
    for line in io.lines() do
    	table.insert(t, line)
    end
    print(#t) --> (number of lines read)
````

但我更倾向于使用`t[#t+1]=line`。

`table.remove`移除数组特定位置的元素并返回。后续元素前提。若调用时不指定位置，则移除最后一个元素。

### 20.2 排序

`table.sort`用于排序。可以指定排序函数。排序函数返回true表示第一个参数应该在第二个之前。若不提供排序函数，默认使用`<`运算符。

不要对表的键排序。它们没有顺序。

Let us see an example. Suppose that you read a source file and build a table that gives, for each function name, the line where this function is defined; something like this:

    lines = {
        luaH_set = 10,
        luaH_get = 24,
        luaH_present = 48,
    }

Now you want to print these function names in alphabetical order. If you traverse this table with pairs, the names appear in an arbitrary order. You cannot sort them directly, because these names are keys of the table. However, when you put them into an array, then you can sort them. First, you must create an array with these names, then sort it, and finally print the result:

    a = {}
    for n in pairs(lines) do a[#a + 1] = n end
    table.sort(a)
    for _, n in ipairs(a) do print(n) end

Some people get confused here. After all, for Lua, arrays also have no order (they are tables, after all). But we know how to count, so we impose an order when we access the array with ordered indices. That is why you should always traverse arrays with `ipairs`, rather than `pairs`. The first function imposes the key order 1, 2, . . . , whereas the latter uses the natural arbitrary order of the table.

### 20.3 连接

We have already seen `table.concat` in Section 11.6. It takes a list of strings and returns the result of concatenating all these strings. An optional second argument specifies a string separator to be inserted between the strings of the list. The function also accepts two other optional arguments that specify the indices of the first and the last string to concatenate.

The next function is an interesting generalization of `table.concat`. It accepts nested lists of strings:

    function rconcat (l)
    	if type(l) ~= "table" then return l end
    	local res = {}
    	for i = 1, #l do
    		res[i] = rconcat(l[i])
    	end
    	return table.concat(res)
    end

For each list element, rconcat calls itself recursively to concatenate a possible nested list. Then it calls the original table.concat to concatenate all partial results.

    print(rconcat{{"a", {" nice"}}, " and", {{" long"}, {" list"}}})
    --> a nice and long list

## 21 字符串库

The string library exports its functions as a module called string. Since Lua 5.1, it also exports its functions as methods of the string type (using the metatable of that type). 例如，要转大写，可以使用`string.upper(s)`或`s:upper()`。

### 21.1 基本函数

`string.len(s)`返回字符串的长度，等价于`#s`。`string.rep(s,n)`（或`s:rep(n)`）重复字符串`s` n次。`string.lower(s)`返回全小写的字符串；`string.upper`相反。

`string.sub(s,i,j)`取字符串，从第i到第j（包括）。Lua中，第一个字符的下标是1。也可以使用负下标，从后面向前算。Therefore, the call `string.sub(s,1,j)` (or `s:sub(1,j)`) gets a prefix of the string s with length j; `string.sub(s,j,-1)` (or simply `s:sub(j)`, since the default for the last argument is 1) gets a suffix of the string, starting at the j-th character; and `string.sub(s,2,-2)` returns a copy of the string s with the first and last characters removed:

```lua
	s = "[in brackets]"
	print(s:sub(2, -2)) --> in brackets
```

记住Lua中字符串是不可变的，隐藏函数不会改变原字符串，而是返回新字符串。

The `string.char` and `string.byte` functions convert between characters and their internal numeric representations. Function `string.char` gets zero or more integers, converts each one to a character, and returns a string concatenating all these characters. Function `string.byte(s,i)` returns the internal numeric representation of the i-th character of the string s; the second argument is optional, so that the call `string.byte(s)` returns the internal numeric representation of the first (or single) character of s. In the following examples, we assume that characters are represented in ASCII:

```lua
    print(string.char(97)) --> a
    i = 99; print(string.char(i, i+1, i+2)) --> cde
    print(string.byte("abc")) --> 97
    print(string.byte("abc", 2)) --> 98
    print(string.byte("abc", -1)) --> 99
```

In the last line, we used a negative index to access the last character of the string. Since Lua 5.1, `string.byte` accepts an optional third argument. A call like `string.byte(s,i,j)` returns multiple values with the numeric representation  of all characters between indices i and j (inclusive):

```lua
	print(string.byte("abc", 1, 2)) --> 97 98
```

The default value for j is i, so a call without this argument returns only the i-th character. A nice idiom is {s:byte(1,-1)}, which creates a table with the codes of all characters in s. Given this table, we can recreate the original string by calling string.char(table.unpack(t)). This technique does not work for very long strings (say, longer than 1 MB), because Lua puts a limit on how many values a function can return.

Function string.format is a powerful tool for formatting strings, typically for output. It returns a formatted version of its variable number of arguments following the description given by its first argument, the so-called format string. The format string has rules similar to those of the printf function of standard C: it is composed of regular text and directives, which control where and how to place each argument in the formatted string. A directive is the character ‘%’ plus a letter that tells how to format the argument: ‘d’ for a decimal number, ‘x’ for hexadecimal, ‘o’ for octal, ‘f’ for a floating-point number, ‘s’ for strings, plus some other variants. Between the ‘%’ and the letter, a directive can include other options that control the details of the formatting, such as the number of decimal digits of a floating-point number:

```lua
    print(string.format("pi = %.4f", math.pi)) --> pi = 3.1416
    d = 5; m = 11; y = 1990
    print(string.format("%02d/%02d/%04d", d, m, y)) --> 05/11/1990
    tag, title = "h1", "a title"
    print(string.format("<%s>%s</%s>", tag, title, tag))
    --> <h1>a title</h1>
```

In the first example, the %.4f means a floating-point number with four digits after the decimal point. In the second example, the %02d means a decimal number with at least two digits and zero padding; the directive %2d, without the zero, would use blanks for padding. For a complete description of these directives, see the Lua reference manual. Or, better yet, see a C manual, as Lua calls the standard C library to do the hard work here.
