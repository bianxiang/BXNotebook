[toc]

## 7. 数组

JavaScript arrays are zero-based and use 32-bit indexes: the index of the first element is 0, and the highest possible index is 4294967294 (2^32−2). JavaScript arrays are dynamic: they grow or shrink as needed and there is no need to declare a fixed size for the array when you create it or to reallocate it when the size changes. JavaScript arrays may be sparse: the elements need not have contiguous indexes and there may be gaps. Every JavaScript array has a length property. For nonsparse arrays, this property specifies the number of elements in the array. For sparse arrays, length is larger than the index of all elements.

Arrays inherit properties from `Array.prototype`, which defines a rich set of array manipulation methods, covered in §7.8 and §7.9. Most of these methods are generic, which means that they work correctly not only for true arrays, but for any “array-like object.” We’ll discuss array-like objects in §7.11. In ECMAScript 5, strings behave like arrays of characters, and we’ll discuss this in §7.12.

### 7.1 创建数组

The values in an array literal need not be constants; they may be arbitrary expressions:

    var base = 1024;
    var table = [base, base+1, base+2, base+3];

If you omit a value from an array literal, the omitted element is given the value `undefined`:

    var count = [1,,3]; // An array with 3 elements, the middle one undefined.
    var undefs = [,,]; // An array with 2 elements, both undefined.

Array literal syntax allows an optional trailing comma, so `[,,]` has only two elements, not three.

创建数组的另一种方式是通过`Array()`构造器。有三种调用方式：

1、没有参数，创建空数组：

	var a = new Array();

2、指定长度。Note that no values are stored in the array, and the array index properties “0”, “1”, and so on are not even defined for the array.{{稀疏数组，见下一节}}

	var a = new Array(10);

3、显式指定数组元素，两个或多个元素，或至少一个非数字元素：

	var a = new Array(5, 4, 3, 2, 1, "testing, testing");

### 7.2 读写数组元素

数组只是对象的一个特例。JavaScript将数组索引转换为字符串，如索引`1`变成`"1"`，然后用这个字符串作为属性名。一个普通对象也可以这样索引：

	o = {}; // Create a plain object
    o[1] = "one"; // Index it with an integer

数组的特殊之处是，如果属性名是一个小于2^32的非负整数，会自动维护`length`属性。

数组索引与对象属性名的区别。所有的索引都是属性名，但只有从0到2^32–1的整数属性才是索引。

如果索引位置放置小树或负数，它们会变成属性，不会成为索引。

    a[-1.23] = true; // This creates a property named "-1.23"
    a["1000"] = 0; // This the 1001st element of the array
	a[1.000] // Array index 1. Same as a[1]

Javascript数组没有越界。访问不存在的元素只是得到`undefined`。

### 7.3 稀疏数组

If the array is sparse, the value of the `length` property is greater than the number of elements. Sparse arrays can be created with the `Array()` constructor or simply by assigning to an array index larger than the current array length.

    a = new Array(5); // No elements, but a.length is 5.
    a = [];
    a[1000] = 0; // Assignment adds one element but sets length to 1001.

We’ll see later that you can also make an array sparse with the `delete` operator.

非常稀疏的数组一般实现为速度慢但内存使用更高效的方式，此时在数组中寻找元素花费的时间与在普通对象中查找属性一样。

在数组字面量中省略值并不会创建稀疏数组。被省略的元素在数组中，只是值为`undefined`。这与数组元素根本不存在不完全相同。差别可以通过`in`运算符或**for/in**体现：

    var a1 = [,,,]; // This array is [undefined, undefined, undefined]
    var a2 = new Array(3); // This array has no values at all
    0 in a1 // => true: a1 has an element with index 0
    0 in a2 // => false: a2 has no element with index 0

### 7.4 数组长度

如果给`length`赋值`n`，其小于当前长度值，则多出的数组元素会被删除。

	a = [1,2,3,4,5];
    a.length = 3;

如果设置的值大于当前值，会产生稀疏数组。

In ECMAScript 5, you can make the `length` property of an array readonly with `Object.defineProperty()`:

	a = [1,2,3];
    Object.defineProperty(a, "length", {writable: false});
    a.length = 0; // a is unchanged.

Similarly, if you make an array element **nonconfigurable**, it cannot be deleted. If it cannot be deleted, then the `length` property cannot be set to less than the index of the nonconfigurable element. (See §6.7 and the Object.seal() and Object.freeze() methods in §6.8.3.)

### 7.5 添加删除数组元素

利用`push()`追加元素：

    a = [];
    a.push("zero")
    a.push("one", "two") // Add two more values. a = ["zero", "one", "two"]

利用`unshift()`在数组开头添加元素，已存在的元素后移。

You can delete array elements with the `delete` operator, just as you can delete object properties:

    a = [1,2,3];
    delete a[1]; // a now has no element at index 1
    1 in a // => false: no array index 1 is defined
    a.length // => 3: delete不影响数组长度

`delete`操作会产生稀疏数组。不改变长度值。类似于给指定元素赋值`undefined`。

删除数组尾部的元素可以通过设置`length`属性实现。`pop()`方法移除最后一个元素并返回。`shift()`移除开头元素，将后续元素前移。这两个方法参见7.8节。

Finally, `splice()` is the general-purpose method for inserting, deleting, or replacing array elements. It alters the `length` property and shifts array elements to higher or lower indexes as needed. See §7.8 for details.

### 7.6 遍历数组

不要每次循环都访问数组`length`属性，可以：

    for(var i = 0, len = keys.length; i < len; i++) {
    	// loop body remains the same
    }

排除`null`、`undefined`和不存在的元素：

	for(var i = 0; i < a.length; i++) {
		if (!a[i]) continue;
        // loop body here
	}

若只想排除`undefined`和不存在的元素：

    for(var i = 0; i < a.length; i++) {
    	if (a[i] === undefined) continue;
        // loop body here
    }

只想排除不存在的元素：

    for(var i = 0; i < a.length; i++) {
    	if (!(i in a)) continue;
        // loop body here
    }

ECMAScript 5定义了一个新方法，按索引顺序依次遍历、传入每个元素：

	var data = [1,2,3,4,5];
    var sumOfSquares = 0;
    data.forEach(function(x) {sumOfSquares += x*x;});

`forEach()`详细讨论见 §7.9。

### 7.7 多维数组

### 7.8 数组方法

ECMAScript 3在`Array.prototype`上定义了多个数组操纵方法。

#### 7.8.1 join()

`Array.join()`将所有数组元素连接成一个字符串并返回。可以传入一个分隔符，默认为逗号。

    var a = [1, 2, 3];
    a.join();
    a.join(" ");
    a.join("");

与`Array.join()`相对照的方法是`String.split()`。

#### 7.8.2 reverse()

`Array.reverse()`翻转所有元素顺序，并返回翻转后的数组。注意此方法改变原数组{{返回的数组就是原数组}}

    var a = [1,2,3];
    a.reverse().join() // => "3,2,1" and a is now [3,2,1]

#### 7.8.3 sort()

`Array.sort()`对数组进行原地排序，并返回排序好的数组。若不带参数调用`sort()`，按字母顺序排序(temporarily converting them to strings to perform the comparison, if necessary)：

	var a = new Array("banana", "cherry", "apple");
    a.sort();
	var s = a.join(", "); // s == "apple, banana, cherry"

未定义元素排在数组最后。

传入一个函数，指定排序规则。如果传入的第一个参数应该排在第二个前面，返回负值；相等返回0；第二个排前面返回正值。

	var a = [33, 4, 1111, 222];
	a.sort(function(a,b) {
    	return a-b;
    });

#### 7.8.4 concat()

`Array.concat()`创建并返回一个新的数组。开始先是原来数组的所有元素，然后是所有参数。如果参数是数组，则将数组中的元素取出追加到新数组。但不会地柜的展平数组的数组。原数组不会被修改。例子：

    var a = [1,2,3];
    a.concat(4, 5) // Returns [1,2,3,4,5]
    a.concat([4,5]); // Returns [1,2,3,4,5]
    a.concat([4,5],[6,7]) // Returns [1,2,3,4,5,6,7]
    a.concat(4, [5,[6,7]]) // Returns [1,2,3,4,5,[6,7]]

#### 7.8.5 slice()

`Array.slice()`返回一个子数组。两个参数分别指定开始（包括）和结束位置（不包括） 。若省略第二个参数，则子数组一直到数组最后一个元素。两个参数为负值时表示相对于最后一个元素。例如`-1`指数组最后一个元素。`slice()`不修改原数组。

    var a = [1,2,3,4,5];
    a.slice(0,3); // Returns [1,2,3]
    a.slice(3); // Returns [4,5]
    a.slice(1,-1); // Returns [2,3,4]
    a.slice(-3,-2); // Returns [3]

#### 7.8.6 splice()

`Array.splice()`是一个插入、删除元素的通用方法。`splice()`直接修改原数组。

`splice()`的第一个参数指定插入/删除开始的位置。第二个参数指定要被移除的元素的数量。若省略第二个参数，则从开始到最后所有元素都被删除。`splice()`返回被删除的元素。

	var a = [1,2,3,4,5,6,7,8];
	a.splice(4); // Returns [5,6,7,8]; a is [1,2,3,4]
    a.splice(1,2); // Returns [2,3]; a is [1,4]
    a.splice(1,1); // Returns [4]; a is [1]

前两个参数后可以跟随任意数量的参数，这些参数会被插入到第一个参数指定的位置。

    var a = [1,2,3,4,5];
    a.splice(2,0,'a','b'); // Returns []; a is [1,2,'a','b',3,4,5]
    a.splice(2,2,[1,2],3); // Returns ['a','b']; a is [1,2,[1,2],3,3,4,5]

若参数是数组，`splice()`直接插入，不会展开。

#### 7.8.7 push() 和 pop()

`push()`想数组最后追加一个或多个元素，返回数组新长度。`pop()`移除最后一个元素，长度减一，返回被删除的元素。两个方法都直接修改数组。

若向`push()`传入一个数组，不会被展平，直接插入

    stack.push([4,5]); // stack: [1,[4,5]] Returns 2

#### 7.8.8 unshift() 和 shift()

`unshift()`和`shift()`方法与`push()`和`pop()`类似，只是插入删除发生在数组开头。`unshift()`在数组开头插入一个或多个元素，将存在的元素后移，返回新的数组长度。`shift()`删除数组第一个元素并返回，后面所有元素前移。

    a.unshift(3,[4,5]); // a:[3,[4,5],1] Returns: 3

#### 7.8.9 toString() 和 toLocaleString()

`toString()`方法将数组每个元素转换为一个字符串（可能调用元素的`toString()`方法），返回一个逗号分隔的列表。注意返回的字符串前后没有中括号之类的分界符。

	[1,2,3].toString() // Yields '1,2,3'
    ["a", "b", "c"].toString() // Yields 'a,b,c'
    [1, [2,'c']].toString() // Yields '1,2,c'

`toLocaleString()` is the localized version of `toString()`. It converts each array element to a string by calling the `toLocaleString()` method of the element, and then it concatenates the resulting strings using a locale-specific (and implementation-defined) separator string.

### 7.9 ECMAScript 5数组方法

ECMAScript 5 defines nine new array methods for iterating, mapping, filtering, testing, reducing, and searching arrays. The subsections below describe these methods.

先讨论一下这些方法的共性。首先，多数方法第一个参数接收一个数组，and invoke that function once for each element (or some elements) of the array。如果数组是稀疏的，对不存在的元素不会调用。多数情况，传入的函数可有三个参数，分别是元素的值，元素在数组中的索引，和数组自己。多数函数接受第二个可选的参数，函数作为这个参数的方法调用，即在函数内，第二个参数成为`this`。

#### 7.9.1 forEach()

`forEach()`方法遍历数组。

	var data = [1,2,3,4,5];
	var sum = 0;
	data.forEach(function(value) { sum += value; });

	// Now increment each array element
    data.forEach(function(v, i, a) { a[i] = v + 1; });

`forEach()`不支持中途结束遍历。If you need to terminate early, you must throw an exception, and place the call to `forEach()` within a try block.

#### 7.9.2 map()

The map() method passes each element of the array on which it is invoked to the function you specify, and returns an array containing the values returned by that function. For example:

    a = [1, 2, 3];
    b = a.map(function(x) { return x*x; }); // b is [1, 4, 9]

如果原函数是稀疏的，返回的函数也是同样稀疏的。

#### 7.9.3 filter()

`filter()`返回数组的子集。传入的函数是一个断言：需要返回true或false。对于一个元素，若断言返回true，则元素加入到最后的子集中。返回一个新数组，不修改原数组。

	a = [5, 4, 3, 2, 1];
    smallvalues = a.filter(function(x) { return x < 3 }); // [2, 1]
    everyother = a.filter(function(x, i) { return i%2 == 0 }); // [5, 3, 1]

`filter()`会忽略稀疏数组中不存在的元素，因此返回的数组总是实的。To close the gaps in a sparse array, you can do this:

	var dense = sparse.filter(function() { return true; });

And to close gaps and remove `undefined` and `null` elements you can use filter like this:

	a = a.filter(function(x) { return x !== undefined && x != null; });

#### 7.9.4 every() 和 some()

`every()`和`some()`是数组断言。最终返回true或false。

    a = [1,2,3,4,5];
    a.every(function(x) { return x < 10; }) // => true: all values < 10.
    a.every(function(x) { return x % 2 === 0; }) // => false: not all values even.

	a.some(function(x) { return x%2===0; }) // => true a has some even numbers.
    a.some(isNaN) // => false: a has no non-numbers.

`every()`和`some()`在能确定值后会立即返回。

如果数组为空，按照数学惯例，`every()`返回true，`some()`返回false。

#### 7.9.5 reduce(), reduceRight()

`reduce()`和`reduceRight()`方法组合数组中得元素，产生一个值。This is a common operation in functional programming and also goes by the names “inject” and “fold.” Examples help illustrate how it works:

	var a = [1,2,3,4,5]
	var sum = a.reduce(function(x,y) { return x+y }, 0); // Sum of values
	var product = a.reduce(function(x,y) { return x*y }, 1); // Product of values
    var max = a.reduce(function(x,y) { return (x>y)?x:y; }); // Largest value

`reduce()`接受两个参数。第一个是reduction函数。这个函数的任务是讲两个值组合或减少为一个值。第二个参数（可选）是初始值。

传给`reduce()`的函数的参数与前面的函数不同。它接受四个值，第一个值是累计值，后面三个值是元素值、元素索引和数组。第一次函数调用是，第一个参数的值是`reduce()`的第二个参数。后续调用，第一个参数值是上一次调用返回的结果。

若调用`reduce()`时不指定初始值（第二个参数），则数组第一个元素会被用作初始值。因此上面求和和求乘积的例子，初始值也可以省略。

如果数组未空切未指定初始值，调用`reduce()`导致`TypeError`。如果只有一个只——either an array with one element and no initial value or an empty array and an initial value——it simply returns that one value without ever calling the reduction function.

`reduceRight()`与`reduce()`不同点在于，处于数组时从右向左。You might want to do this if the reduction operation has right-to-left precedence, for example:

	var a = [2, 3, 4]
    // Compute 2^(3^4). Exponentiation has right-to-left precedence
    var big = a.reduceRight(function(accumulator, value) {
    	return Math.pow(value, accumulator); });

Note that neither `reduce()` nor `reduceRight()` accepts an optional argument that specifies the `this` value on which the reduction function is to be invoked. The optional initial value argument takes its place. See the `Function.bind()` method if you need your reduction function invoked as a method of a particular object.

#### 7.9.6 indexOf() 和 lastIndexOf()

`indexOf()`和`lastIndexOf()`搜索数组中得指定值，返回索引，或-1表示每找到。

	a = [0,1,2,1,0];
    a.indexOf(1) // => 1: a[1] is 1
    a.lastIndexOf(1) // => 3: a[3] is 1

`indexOf()`和`lastIndexOf()`第一个参数是搜索目标。第二个可选的参数指定搜索开始的位置。允许负值，表示从最后一个元素开始向前。如`–1`表示最后一个元素。

The following function searches an array for a specified value and returns an array of all matching indexes. This demonstrates how the second argument to indexOf() can be used to find matches beyond the first.

### 7.10 数组类型

ECMAScript 5可以通过`Array.isArray()`判断对象是否为数组：

	Array.isArray([]) // => true
    Array.isArray({}) // => false

但在ECMAScript 5之前，区别数组和非数组对象很那。对于数组`typeof`返回`“object”`。The `instanceof` operator works in simple cases:

	[] instanceof Array // => true
    ({}) instanceof Array // => false

The problem with using `instanceof` is that in web browsers, there can be more than one window or frame open. Each has its own JavaScript environment, with its own global object. And each global object has its own set of constructor functions. Therefore an object from one frame will never be an instance of a constructor from another frame. 尽管跨帧问题不是很长久，但已经足够成为`instanceof`不可靠的理由。

The solution is to inspect the `class` attribute (see §6.8.2) of the object. For arrays, this attribute will always have the value “Array”, and we can therefore write an `isArray()` function in ECMAScript 3 like this:

    var isArray = Function.isArray || function(o) {
    	return typeof o === "object" && Object.prototype.toString.call(o) === "[object Array]";
    };

This test of the `class` attribute is, in fact, exactly what the ECMAScript 5 `Array.isArray()` function does. The technique for obtaining the class of an object using `Object.prototype.toString()` is explained in §6.8.2 and demonstrated in Example 6-4.

### 7.11 类数组对象

As we’ve seen, JavaScript arrays have some special features that other objects do not have:

- The length property is automatically updated as new elements are added to the list.
- Setting length to a smaller value truncates the array.
- Arrays inherit useful methods from Array.prototype.
- Arrays have a class attribute of “Array”.

很多数组算法对类数组对象都有效，特别是算法是只读的，不需要修改`length`属性。

The Arguments object that’s described in §8.3.2 is an array-like object. In client-side JavaScript, a number of DOM methods, such as `document.getElementsByTagName()`, return array-like objects. Here’s a function you might use to test for objects that work like arrays:

    // 判断o是否是类数组对象
    // Strings and functions have numeric length properties, but are
    // excluded by the typeof test. In client-side JavaScript, DOM text
    // nodes have a numeric length property, and may need to be excluded
    // with an additional o.nodeType != 3 test.
    function isArrayLike(o) {
        if (o && // o is not null, undefined, etc.
            typeof o === "object" // o is an object
            && isFinite(o.length) // o.length is a finite number
            && o.length >= 0 // o.length is non-negative
            && o.length===Math.floor(o.length) // o.length is an integer
            && o.length < 4294967296) // o.length < 2^32
            return true;
        else
            return false;
    }

JavaScript数组方法一般定义为通用的，能用于类数组对象。ECMAScript 5的所有数组方法都是通用的。ECMAScript 3中`toString()`和`toLocaleString()`之外的方法是通用的。(The concat() method is an exception: although it can be invoked on an array-like object, it does not property expand that object into the returned array.) 利用`Function.call`间接调用这些方法：

	var a = {"0":"a", "1":"b", "2":"c", length:3};
    Array.prototype.join.call(a, "+") // => "a+b+c"
    Array.prototype.slice.call(a, 0) // => ["a","b","c"]: true array copy
    Array.prototype.map.call(a, function(x) {
		return x.toUpperCase();
    });

The ECMAScript 5 array methods were introduced in Firefox 1.5. Because they were written generically, Firefox also introduced versions of these methods as functions de- fined directly on the Array constructor. With these versions of the methods defined, the examples above can be rewritten like this:

    var a = {"0":"a", "1":"b", "2":"c", length:3}; // An array-like object Array.join(a, "+")
    Array.slice(a, 0)
    Array.map(a, function(x) { return x.toUpperCase(); })

这些静态方法很好用，但毕竟是非标准的。可以通过下面的代码保证兼容性：

    Array.join = Array.join || function(a,sep) {
    	return Array.prototype.join.call(a,sep);
    };
    Array.slice = Array.slice || function(a,from,to) {
    	return Array.prototype.slice.call(a,from,to);
    };
    Array.map = Array.map || function(a, f, thisArg) {
    	return Array.prototype.map.call(a, f, thisArg);
    }

### 7.12 字符串作为数组

In ECMAScript 5 (and in many recent browser implementations—including IE8—prior to ECMAScript 5), strings behave like read-only arrays. Instead of accessing individual characters with the `charAt()` method, you can use square brackets:

	var s = test;
    s.charAt(0) // => "t"
    s[1] // => "e"

The `typeof` operator still returns “string” for strings, of course, and the `Array.isArray()` method returns false if you pass it a string.

The fact that strings behave like arrays also means, however, that we can apply generic array methods to them. For example:

	s = "JavaScript"
    Array.prototype.join.call(s, " ")
    Array.prototype.filter.call(s, function(x) {
		return x.match(/[^aeiou]/);
	}).join("")

字符串是不可变得，因此被当做数组时也是只读的数组。诸如`push()`、`sort()`、` reverse()`和`splice()`的方法对字符串无效。强制调用不会报错，只是忽略这些操作。






