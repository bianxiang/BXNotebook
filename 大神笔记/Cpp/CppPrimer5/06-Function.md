[toc]

## 6. 函数

### 6.1 函数基础

为了与C兼容，当函数形参列表为空时，可以用 `void`：

```cpp
void f1(){/* ... */ } // implicit void parameter list
void f2(void){ /* ... */ } // explicit void parameter list
```

返回类型可以是 `void`。返回类型不可以是**数组类型或函数类型**。但可以返回到数组或函数的指针。

#### 6.1.1 Local Objects

**名字有作用域，对象有生命周期**。

- 名字的作用域指，在哪块程序中名字是可见的
- 对象的生命周期指运行中对象存在的时间

定义在函数体内的参数和变量称为局部（local）变量。局部变量隐藏外部作用域的同名变量。

##### 自动对象

与普通局部变量相对应的对象。

参数是自动对象。参数在函数体的作用域内定义。

对应局部变量的自动对象，如果定义中不含初始化器，则它们将被**默认初始化**(§2.2.1)。即未初始化的内建类型的局部变量的值是未定的。

##### Local static Objects

Each local static object is initialized before the first time execution passes through the object’s definition. Local statics are not destroyed when a function ends; 它们在程序终止时被销毁。

例如，下面的函数能记录被调用的次数：

```cpp
size_t count_calls()
{
    static size_t ctr = 0;
    return ++ctr;
}
int main()
{
    for(size_t i = 0; i != 10; ++i)
        cout << count_calls() << endl;
    return 0;
}
```

未显式初始化局部静态变量，它们将被值初始化(§3.3.1)。内建类型的局部静态变量会被初始化为0。

#### 6.1.2 函数声明

与其他名字一样，函数名要先声明再使用。与变量名一样，函数名只能被定义一次，却能被声明多次。只要我们不使用，函数可以只声明不定义（一个例外见§15.3）。

函数声明类似于函数定义，只是没有Body。函数声明也称为函数原型。

函数声明的参数可以没有参数名。但列出参数名有助于帮助理解语义：

```cpp
    void print(vector<int>::const_iterator beg,
    	vector<int>::const_iterator end);
```

之前提出，变量应该在头文件中声明，在源文件中定义。出于相同目的，函数应该在头文件中声明，在源文件中定义。源文件应该包含函数声明的头，这样编译器能检查声明与定义是否一致。

#### 6.1.3 独立编译（Separate Compilation）

独立（Separate）编译允许我们将程序分成多个文件。每个文件可以被独立编译。例子，假设函数 `fact` 定义在文件 `fact.cc`，它的声明在头文件 `Chapter6.h`。`main` 函数在第二个文件中 `factMain.cc`。要产生可执行文件，需要所有文件：

```
$ CC factMain.cc fact.cc # generates factMain.exe or a.out
$ CC factMain.cc fact.cc -o main # generates main or main.exe
```

其中CC是编译器。

改变某个文件后，期望只重新编译改变的文件。

编译器可以将多个对象文件链接成一个可执行文件。On the system we use, we would separately compile our program as follows:

```
$ CC -c factMain.cc # generates factMain.o
$ CC -c fact.cc  # generates fact.o
$ CC factMain.o fact.o  # generates factMain.exe or a.out
$ CC factMain.o fact.o -o main # generates main or main.exe
```

### 6.2 参数传递

> 参数初始化与变量初始化的方式相同。

如果形参是引用(§2.3.1)则形参绑定到实参（按引用传递）。否则实参拷贝到形参，此时形参与实参是独立的对象（按值传递）。

#### 6.2.1 按值传递

> 熟悉C的程序员常通过指针访问函数外的对象，**但C++程序员一般使用引用**。

如果参数是指针，也是按值传递 —— 指针值被拷贝。于是形参和实现指向同一个对象。

#### 6.2.2 按引用传递

最佳实践：若函数内不会改变引用参数，引用应该被置为`const`（指向常量的引用）。

```cpp
// compare the length of two strings
bool isShorter(const string &s1, const string &s2)
{
    return s1.size() < s2.size();
}
```

#### 6.2.3 const形参与实参

回忆关于顶级常量的讨论（§2.4.3），顶级常量指对象本身是常量：

```cpp
const int ci = 42; // 常量是顶级的
int i = ci; // 可以：拷贝ci时，顶级常量会被忽略
int * const p = &i; // p是常量
*p = 0; // 但p指向的不是常量，因此可以赋值
```

当我们拷贝一个实参去初始化一个形参时，顶层常量会被忽略。可以将一个常量的实参或非常量的实参传给一个常量的形参：

```cpp
void fcn(const int i) { /* fcn不能修改i*/ }
```

可以向 `fcn` 传 `const int` 或普通 `int`。

因为顶级常量可以忽略，因此下面两个声明的差别不够大，因此不允许这种重载：

```cpp
void fcn(const int i) {}
void fcn(int i) {} // error: redefines fcn(int)
```

##### 指针、引用与const

我们可以用一个非常量的对象初始化一个底层常量，但反之不行。

```cpp
int i = 42;
const int *cp = &i; // 可以：cp不能修改i (§2.4.2)
const int &r = i; // 可以：r不能修改i (§2.4.1)
const int &r2 = 42; // ok: (§2.4.1)
int *p = cp; // 错误：cp指向常量，但p指向变量(§2.4.2)
int &r3 = r; // 错误：r指向常量，但r3指向变量(§2.4.1)
int &r4 = 42; // 错误：不能用字面量初始化一个非常量的引用(§ 2.3.1)
```

同样的规则适用于参数传递：

```cpp
void reset(int *ip);
void reset(int &i);
string::size_type find_char(const string &s, char c, string::size_type &occurs)
int i = 0;
const int ci = i;
string::size_type ctr = 0;
reset(&i); // calls the version of reset that has an int* parameter
reset(&ci); // 错误，不能用指向常量int的指针初始化指向非常量int的指针
reset(i); // 调用(int&)
reset(ci); // 错误：不能把常量引用赋给非常量引用
reset(42); // 错误：不能绑定普通引用到一个字面量
reset(ctr); // 错误：类型不匹配，ctr是无符号类型
// ok: find_char's first parameter is a reference to const
find_char("Hello World!", 'o', ctr);
```

##### 尽量使用到常量的引用

一个常见的错误是，将函数定义成非常量引用，但其实函数不会修改此参数。而且限制了实参可以使用的类型。

例如，把 `find_char` 改成错误的定义：

```cpp
string::size_type find_char(string &s, char c, string::size_type&occurs);
```

则 `find_char("Hello World", 'o', ctr);` 将不再能通过编译。

#### 6.2.4 数组参数

数组两个特性：不能拷贝数组(§3.5.1)，使用数组时，常被转换为指针(§3.5.3)。因为不能拷贝，因此不能按值传递。因为数组常被转换为指针，将数组传给函数时，实际传递的是数组第一个元素的指针。

```cpp
void print(const int*);
void print(const int[]);
void print(const int[10]); // 10只起到文档的作用
```

这三个声明是等价的。每个函数实际接受的参数都是 `const int*`。因此实参只要是 `const int*`：

```cpp
int i = 0, j[2] = {0, 1};
print(&i); // ok: &i is int*
print(j); // ok
```

> 当函数不需要修改数组元素时，数组参数应该是指向常量的指针(§2.4.2)。

如果实参是数组，实参会被自动转换为指向第一个元素的指针，数组的大小不被关心。由于函数不知道数组大小，因此一般需要附加参数。有三种策略：

1、数组结尾放一个标记元素。例如，C风格的字符串，最后一个是null字符。

```cpp
void print(const char *cp)
{
    if(cp)  // 如果cp不是空指针
        while(*cp)  // 不是空字符
            cout << *cp++;
}
```

2、使用标准库的约定。传递数组首指针和数组后指针。

```cpp
void print(const int *beg, const int *end)
{
    while(beg != end)
        cout << *beg++ << endl;
}

int j[2]= {0, 1};
print(begin(j), end(j)); // begin and end functions, see § 3.5.3
```

3、显式传递 `size` 参数

```cpp
void print(const int ia[], size_t size)
{
    for(size_t i = 0; i != size; ++i) {
        cout << ia[i] << endl;
    }
}

int j[] = { 0, 1 };  // int array of size 2
print(j, end(j) - begin(j));
```

##### 数组引用参数

形参可以是到数组的引用。注意括号。

```cpp
void print(int (&arr)[10])
{
    for(auto elem : arr)
        cout << elem << endl;
}
```

数组的大小是引用类型的一部分。这样限制了函数的用途。实参数组只能是10个元素。We’ll see in §16.1.1 how we might write this function in a way that would allow us to pass a reference parameter to an array of any size.

##### 多维数组做参数

与任何数组一样，多维数组传入的实际是指向第一个元素的指针(§3.6)。第二维的大小事元素类型的一部分，必须被指定：

```cpp
void print(int (*matrix)[10], int rowSize) { /* . . .*/ }
```

也可以使用数组形式的语法。编译器一般会忽略第一个纬度，因此我们最好也省略：

```cpp
    // equivalent definition
    void print(int matrix[][10], int rowSize) { /* . . .*/ }
```

#### （未）6.2.5 main: Handling Command-Line Options

#### 6.2.6 变长参数

新标准提供了两种解决办法：如果参数类型相同，可以传入一个库类型 `initializer_list`。如果参数类型不同，可以编写一种特殊的函数，称为 *variadic template*，参见 §16.4。

CPP还有一种特殊的参数类型 **ellipsis**，可以用于传递数量可变的实参。但这种功能一般只需要用在需要与 C 函数接口的程序。

##### `initializer_list`

如果参数类型相同，可以用 `initializer_list` 传递数量不定的参数。`initializer_list` 是库类型，表示一个数组。该类型定义在头文件 `initializer_list`。

`initializer_list` 支持的操作：

- `initializer_list<T> lst;`：默认初始化，列表为空
- `initializer_list<T> lst{a, b, c...};`：从初始值列表中拷贝。列表中的元素是常量。
- `lst2(lst)`，`lst2 = lst`：拷贝或赋值一个 `initializer_list` 不拷贝元素，两个列表共享相同的元素。
- `lst.size()`：列表中的元素数
- `lst.begin()`, `lst.end()`：

`initializer_list`是一个模板类型。

```cpp
	initializer_list<string> ls;
	initializer_list<int> li;
```

与 `vector` 不同，`initializer_list` 中的值总是常量；没有办法改变`initializer_list` 中的元素的值。`initializer_list` 对应的实参放在一个大括号中。

有 `initializer_list` 参数的函数可以同时有其他参数。

因为 `initializer_list` 有 `begin` 和 `end` 方法，因此可以使用range for(§5.4.3)。

```cpp
    void error_msg(ErrCodee, initializer_list<string> il)
    {
        cout << e.msg() << ": ";
        for(const auto &elem : il)
        	cout << elem << " " ;
        cout << endl;
    }

    if (expected!= actual)
    	error_msg(ErrCode(42), {"functionX", expected, actual});
    else
    	error_msg(ErrCode(0), {"functionX", "okay"});
```

##### Ellipsis参数

Ellipsis参数的存在，是为了与C代码接口，这些代码用到了C库的功能 —— `varargs`。In particular, objects of most class types are not copied properly when passed to an ellipsis parameter. 除此之外，ellipsis参数不应被用于其他目的。Your C compiler documentation will describe how to use varargs.

An ellipsis parameter may appear only as the **last** element in a parameter list and may take either of two forms:

```cpp
void foo(parm_list, ...);
void foo(...);
```

The first form specifies the type(s) for some of foo’s parameters. Arguments that correspond to the specified parameters are type checked as usual. No type checking is done for the arguments that correspond to the ellipsis parameter. In this first form, the comma following the parameter declarations is optional.

### 6.3 返回类型

#### 6.3.2 有返回值的函数

##### 值是如何被返回的

返回值与初始化变量和参数的方式相同。返回值被用于初始化一个临时变量，这个临时变量作为函数调用的结果。

```cpp
string make_plural(size_t ctr, const string &word, const string &ending)
{
    return(ctr > 1) ? word + ending : word;
}
```

返回的结果，或者是 `word` 的一个拷贝，或者是 `word` 和 `ending` 相加后的临时字符串。

如果函数返回引用，则此引用只是它引用的对象的另一个名字。

##### 永远不要返回到局部对象的引用或指针

返回值被函数返回后，其存储空间会被释放。此时，到局部对象的引用不再有效：

```cpp
const string &manip()
{
    string ret;
    // transform ret in some way
    if (!ret.empty())
        return ret; // 错误！
    else
        return "Empty"; // 错误！"Empty"的局部的临时字符串
}
```

两个 `return` 返回的值都是未定义的。

同样的，返回到局部对象的指针也是错误的。

##### 返回引用返回的是左值

函数调用是左值还是右值取决于函数返回值类型。返回**引用**的函数调用是左值，否则是右值。如果引用不是常量，则可以给函数调用赋值：

```cpp
char &get_val(string &str, string::size_type ix)
{
    return str[ix];
}
int main()
{
    strings("a value");
    cout << s << endl;  // prints a value
    get_val(s, 0) = 'A'; // changes s[0] to A
    cout << s << endl;  // prints A value
    return 0;
}
```

如果返回类型是到常量的引用，则我们不能给调用结果赋值。

##### 列表初始化返回值

新标准允许函数返回花括号包围的一组值。与其他返回值一样，列表用于初始化用作返回值的临时量。如果列表是空的，则临时量将被值初始化(§3.3.1)。其他情况下，返回的值取决于函数的返回值。

```cpp
    vector<string> process()
    {
        // . . .
        // expected and actual are strings
        if(expected.empty())
        	return {};  // 返回一个空的向量
        else if (expected == actual)
        	return {"functionX", "okay"}; // 返回列表初始化的向量
        else
            return {"functionX", expected, actual};
    }
```

如果函数返回一个内建类型，花括号中至多有一个值，and that value must not require a narrowing conversion (§2.2.1)。若函数返回一个类类型，则由类自己定义初始化器的使用（§3.3.1）。

##### Return from main

There is one exception to the rule that a function with a return type other than void must return a value: The mainfunction is allowed to terminate without a return. If control reaches the end of mainand there is no return, then the compiler implicitly inserts a return of 0.

As we saw in §1.1, the value returned from mainis treated as a status indicator. A zero return indicates success; most other values indicate failure. A nonzero value has a machine-dependent meaning. To make return values machine independent, the cstdlibheader defines two preprocessor variables (§2.3.2)
that we can use to indicate success or failure:

```cpp
    int main()
    {
        if(some_failure)
        	return EXIT_FAILURE;  // defined in cstdlib
        else
        	return EXIT_SUCCESS;  // defined in cstdlib
    }
```

Because these are preprocessor variables, we must not precede them with `std::`, nor may we mention them in `using` declarations.

#### 6.3.3 返回到数组的指针

因为无法拷贝数组，因此函数无法返回数组。但可以返回到数组的指针或引用(§3.5.1)。返回数组指针或引用的函数的定义很难看。需要一些方式简化。最直接的方式是使用类别别名：

```cpp
typedef int arrT[10]; // arrT是别名
using arrtT = int[10]; // 等价，定义arrT
arrT* func(int i); // func返回一个指针，指向一个5个元素的数组
```

##### 返回到数组的指针

不使用别名时，注意括号：

```cpp
int arr[10]; // arr is an array of ten ints
int *p1[10]; // p1是数组，10个指针
int (*p2)[10] = &arr; // p2是指针，指向具有10个元素的数组
```

返回到数组的指针，语法形如：
`Type(*function(parameter_list))[dimension]`

例如：`int (*func(int i))[10];`

##### 返回类型放后

新标准允许返回类型放后面。任何函数都可以这样做。特别适于返回类型复杂的函数。参数列表后加 `->`，接着是返回类型。同时原来返回类型的位置放 `auto`：

```cpp
// fcn返回一个指针，指向10个元素的数组
auto func(int i) -> int(*)[10];
```

##### 使用decltype

```cpp
int odd[] = {1,3,5,7,9};
int even[] = {0,2,4,6,8};
// returns a pointer to an array of five int elements
decltype(odd) *arrPtr(int i)
{
    return (i % 2) ? &odd : &even; // 返回指针
}
```

记住decltype不会自动将数组转换为相应的指针类型。这里decltype返回一个数组类型，因此必须在 `arrPtr` 前加 `*`。

### 6.4 重载函数

同一作用域、同名但参数列表不同的函数重载。例子：

```cpp
void print(const char *cp);
void print(const int *beg, const int *end);
void print(const int ia[], size_t size);
```

> main函数不能被重载

**定义重载函数**

We can call `lookup` passing a value of any of several types:

```cpp
Record lookup(const Account&);  // find by Account
Record lookup(const Phone&);  // find by Phone
Record lookup(const Name&);  // find by Name
```

参数列表相同，只是返回值类型不同的两个函数是不被允许的。

```cpp
Record lookup(const Account&);
bool lookup(const Account&);  // 错误
```

**两个参数类型是否不同**

看起来不同的两个参数列表其实是相同的：

```cpp
// each pair declares the same function
Record lookup(const Account &acct);
Record lookup(const Account&); // 参数名不重要
typedef Phone Telno;
Record lookup(const Phone&);
Record lookup(const Telno&); // Telno和Phone是同一类型
```

**重载与常量参数**

顶级常量不影响可以向方法传递的对象（§6.2.3）。带顶级常量的参数与不带的参数等价：

```cpp
Record lookup(Phone);
Record lookup(const Phone); // 重复声明
Record lookup(Phone*);
Record lookup(Phone* const); // 重复声明
```

但使用低级常量的两个参数不等价：

```cpp
Record lookup(Account &); // 引用指向的是非常量
Record lookup(const Account&); // 引用指向的是常量
Record lookup(Account *);
Record lookup(const Account *); // 新函数，指向常量的指针
```

若实参为常量对象（或指向常量的指针），则它只能传给一个常量参数。
若实参不是常量对象，则既可以传给常量参数，也可以传给非常量参数，编译器趋向于后者。

**`const_cast`与重载**

§4.11.3 中我们提到 `const_cast` 在重载函数中最有用。例子：

```cpp
const string &shorterString(const string &s1, const string &s2)
{
    return s1.size() <= s2.size() ? s1 : s2;
}
```

函数返回到常量的引用。调用这个函数时，可以传入两个非常量的字符串，但返回的却是一个常量的字符串。我们期望有一个 `shorterString` 版本，传入的字符串不是常量时，返回的也不是常量。可以通过 `const_cast` 实现：

```cpp
string &shorterString(string &s1, string &s2)
{
    auto &r = shorterString(const_cast<const string&>(s1),
        const_cast<const string&>(s2));
    return const_cast<string&>(r);
}
```

#### 6.4.1 重载与作用域

> Ordinarily, it is a bad idea to declare a function locally. However, to explain how scope interacts with overloading, we will violate this practice and use local function declarations.

重载与作用域没有特殊关系：与平时一样，如果我们在内层作用域声明一个名字，它将隐藏外层作用域的名字。**名字不会跨作用域重载**：

```cpp
string read();
void print(const string &);
void print(double); // 重载
void fooBar(int ival)
{
    bool read = false; // 新作用域：隐藏外部的read
    strings = read(); // 错误：read不再是函数
    // 不好的做法：不要在内层作用域声明函数
    void print(int); // 将隐藏外面的 所有 重载版本的print
    print("Value:"); // 错误：隐藏了print(const string &)
    print(3.14); // 正确：调用的是print(int)；但print(double)被隐藏了
}
```

调用 `print` 时，当编译器在内层作用域找到声明时，便会忽略外层的**所有**名字。

> CPP中名字查找在类型检查之前。

### 6.5 其他特性

本节讲默认实参，内联和 `constexpr` 函数。

#### 6.5.1 默认实参

```cpp
typedef string::size_type sz;
string screen(sz ht = 24, sz wid = 80, char backgrnd = ' ');
```

这里，为每个形参都提供给了一个默认值。若一个参数带默认值，则它后面的参数都要带。

**调用带默认实参的函数**

下面的调用都是有效的：

```cpp
string window;
window = screen(); // equivalent to screen(24,80,' ')
window = screen(66);// equivalent to screen(66,80,' ')
window = screen(66, 256); // screen(66,256,' ')
window = screen(66, 256, '#'); // screen(66,256,'#')
```

调用时，只能省略最右端的实参。

**默认实参声明**

尽管最常见的做法是在头文件中只声明函数一次。但声明多次也是合法的。但在一个作用域中，参数只能被指定一次默认值。后续的声明，可以向之前没有指定过默认值的参数指定默认值。

```cpp
// no default for the height or width parameters
string screen(sz, sz, char = ' ');
```

不能改变之前声明的默认值：
`string screen(sz, sz, char = '*'); // 错误`

但可以增加默认值：
`string screen(sz= 24, sz = 80, char);`

> 最佳实践：默认实参一般应该在头文件中的函数**声明**中指定

**默认实参初始化**

局部变量不能用于默认实参。除此之外，任何表达式都可以，只要类型能兼容：

```cpp
// wd, def, ht必须位于函数之外（不是局部变量）
sz wd = 80;
char def = ' ';
sz ht();
string screen(sz = ht(), sz = wd, char = def);
string window = screen(); // calls screen(ht(), 80, ' ')
```

Names used as default arguments are resolved in the scope of the function declaration. 这些名字表示的值在调用时求值：

```cpp
void f2()
{
    def = '*'; // 改变默认实参的值
    sz wd = 100; // 隐藏了外层的定义
    window = screen(); // calls screen(ht(), 80, '*')
}
```

调用 `screen` 时，`def` 参数将使用更新后的值。但注意，重定义的内层的局部变量 `wd` 与默认实参无关，因此不影响调用。

#### 6.5.2 内联与 `constexpr` 函数

**inline函数避免函数调用的开销**

定义内联函数，在函数返回类型前加`inline`关键字：

```cpp
inline const string & shorterString(const string &s1, const string &s2)
{
    return s1.size() <= s2.size() ? s1 : s2;
}
```

> 内联只是给编译器的请求，编译器可以选择忽略

**constexpr函数**

constexpr函数是可以被用于常量表达式(§2.4.4)的函数。constexpr函数的限制是：返回类型和参数类型必须是字面量类型(§2.4.4)，函数体必须包含且只一条返回语句：

```cpp
	constexpr int new_sz() { return 42; }
    constexpr int foo = new_sz(); // ok
```

如果可能，编译器会将函数调用替换为为其返回值。为此，constexpr 函数是隐式内联的。

A constexpr function body may contain other statements so long as those statements generate no actions at run time. 例如，一个 constexpr 函数可以包含 null 语句，类型别名，using 声明等。

A constexpr function is permitted to return a value that is not a constant:

```cpp
// 如果arg是常量表达式，则scale(arg)是常量表达式
constexpr size_t scale(size_t cnt) {
    return new_sz() * cnt;
}

int arr[scale(2)]; // ok: scale(2)是常量表达式
int i = 2; // i不是一个常量表达式
int a2[scale(i)]; // 错误：scale(i)不是一个常量表达式
```

**将内联与constexpr函数放入头文件**

与其他函数不同，内联和constexpr函数可以在程序中被定义多次。毕竟编译器需要的是定义，不只是声明，才能展开代码。但内联和constexpr函数的多次定义必须一致。因此内联和constexpr函数一般定义在头文件内。

#### （未）6.5.3. 帮助调试

### 6.6 函数匹配

若重载的函数参数数量相同，且其中的有些参数的类型可以转换，则匹配并不是那么容易。例如：

```cpp
void f();
void f(int);
void f(int, int);
void f(double, double = 3.14);
f(5.6); // calls void f(double, double)
```

**找出候选和可行（Viable）函数**

第一步找出候选函数。候选函数要求名字匹配，且其声明在调用时可见。在上面的例子中，有四个候选函数 `f`。

第二步，匹配参数。参数匹配的函数是可行（viable）函数。要可行，函数的参数数量首先要匹配。其次，实参形参类型能够转换。根据参数数量可以去掉两个候选函数。

在这个例子中，剩下的两个函数都是可行的：

- `f(int)` 可行，因为可以将 `double` 转换为 `int`
- `f(double, double)` 可行，因为第二个参数有默认值，第一个参数精确匹配。

**寻找最佳匹配**

第三步，从可行（viable）函数中找最佳匹配。寻找形参与实参的最佳匹配。下一节将解释什么是“最佳”，简单来说就是，形参与实参类型越接近，匹配越好。

例如上面的例子，形参 `double` 与实参 `double` 是最近的匹配。

**多个参数的函数匹配**

若实参有多个，情况将更加复杂。例如：

```cpp
f(42, 2.56);
```

按照参数类型先筛选，可行函数式 `f(int, int)` 和 `f(double, double)`。然后逐个参数检查哪个函数是最佳的匹配。There is an overall best match if there is one and only one function for which

- The match for each argument is no worse than the match required by any other viable function
- There is at least one argument for which the match is better than the match provided by any other viable function
- If after looking at each argument there is no single function that is preferable, then the call is in error. The compiler will complain that the call is ambiguous.

上面的例子中，对于第一个实参，`f(int, int)` 相比 `f(double, double)` 是更好的匹配。对于第二个实参，`f(double, double)` 是更好地匹配。

于是编译器将拒绝这个调用，因为有歧义。可以通过显式的强制类型转换消除歧义。However, in well-designed systems, argument casts should not be necessary.

> **最佳实践**：Casts should not be needed to call an overloaded function. The need for a cast suggests that the parameter sets are designed poorly.

#### 6.6.1 实参类型转换

为了得出最佳的匹配，编译器对实参到形参的转换进行了排序。Conversions are ranked as follows:

1. 精确匹配。An exact match happens when: 实参形参类型完全一致；实参从数字或函数类型转换为响应的指针类型；顶级常量不重要。
1. 需要常量转换(§4.11.2)。
1. 需要提升(§4.11.1)。
1. 需要算术(§4.11.1)或指针转换(§4.11.2)
1. Match through a class-type conversion. (§14.9)

**需要提升或算术转换**

> Warning
Promotions and conversions among the built-in types can yield surprising results in the context of function matching. Fortunately, well-designed systems rarely include functions with parameters as closely related as those in the following examples.

记住，小的整数类型总是会被提升为int或更大的整数类型。Given two functions, one of which takes an int and the other a short, the short version will be called only on values of type short. Even though the smaller integral values might appear to be a closer match, those values are promoted to int, whereas calling the short version would require a conversion:

```
void ff(int);
void ff(short);
ff('a'); // char提升为int，因此调用f(int)
```

各种算术转换被认为差别。例如，int到unsigned int的转换，并不比int到double的优先级更高。例如：

```
void manip(long);
void manip(float);
manip(3.14); // 错误：有歧义
```

**常量实参**

若重载的函数区别在于引用或指针指向的，是不是常量。则调用时，编译器根据实参是不是常量决定调用哪个函数：

```
Record lookup(Account&);
Record lookup(const Account&); // 新函数
const Account a;
Account b;
lookup(a); // calls lookup(const Account&)
lookup(b); // calls lookup(Account&)
```

In the second call, we pass the nonconst object b. For this call, both functions are viable. We can use b to initialize a reference to either const or nonconst type. However, initializing a reference to const from a nonconst object requires a conversion. The version that takes a nonconst parameter is an exact match for b. Hence, the nonconst version is preferred.

### 6.7 指向函数的指针

与其他指针一样，函数指针指向一个特定的类型。函数的类型由返回值类型和参数类型决定。函数名不是函数类型的一部分。声明函数指针时，指针名放在原来函数名的地方：

```cpp
// pf是指针
bool (*pf)(const string &, const string &); // 为初始化指针
```

`*pf` 两端的括号是必要的。若省略就变成了一个返回指针的函数。

**使用函数指针**

将函数名用作值，函数会被自动转换为指针。例如把 `lengthCompare` 赋给 `pf`：

```cpp
pf = lengthCompare; // pf现在指向一个名为lengthCompare的函数
pf = &lengthCompare; // 等价；取地址是可选的
```

可以使用函数指针调用函数。不需要解引用指针：

```cpp
bool b1 = pf("hello", "goodbye");  // 调用lengthCompare
bool b2 = (*pf)("hello", "goodbye"); // 等价
```

指向不同函数类型的指针之间没有转换。但我们可以给一个函数指针赋值 `nullptr` 或字面量0，表示指针现在未指向任何函数。

**指向重载函数**

```cpp
void ff(int*);
void ff(unsigned int);
void (*pf1)(unsigned int) = ff; // pf1指向ff(unsigned)
```

编译器根据指针的类型那个决定使用哪个重载的函数。指针的类型必须与某个重载函数的类型精确匹配。

**函数指针参数**

与数组一样(§6.2.4)，函数参数不能是一个函数，但可以是函数指针。函数参数看起来是一个函数类型，但实际会被当成指针：

```cpp
// 第三个参数是一个函数类型，会被当作函数指针
void useBigger(const string &s1, const string &s2,
    bool pf(const string &, const string &));
// 等价声明，显式指明这是个指针
void useBigger(const string &s1, const string &s2,
    bool(*pf)(const string &, const string &));
```

将函数做实参，会被自动转换为指针：

```cpp
useBigger(s1, s2, lengthCompare);
```

利用类型别名(§2.5.1)、decltype可以简化代码 ：

```cpp
// Func and Func2 have function type
typedef bool Func(const string&, const string&);
typedef decltype(lengthCompare) Func2; // equivalent type
// FuncP and FuncP2 have pointer to function type
typedef bool(*FuncP)(const string&, const string&);
typedef decltype(lengthCompare) *FuncP2;  // equivalent type

// equivalent declarations of useBigger using type aliases
void useBigger(const string&, const string&, Func);
void useBigger(const string&, const string&, FuncP2);

```

`decltype`返回一个函数类型，若需要一个指针，需要显式添加 `*`。

**返回函数指针**

与数组一样(§6.3.3)，我们不能返回一个函数类型但可以返回函数指针。且必须显式声明为指针类型。使用时，最好结合别名以简化书写：

```cpp
using F = int(int*, int); // F是一个函数类型
using PF = int(*)(int*, int); // PF是指针类型
```

The thing to keep in mind is that, unlike what happens to parameters that have function type, the return type is not automatically converted to a pointer type. 必须显式指定返回值是一个指针类型：

```cpp
PF f1(int); // ok: PF是函数指针；f1返回这个指针
F f1(int); // 错误：F是函数类型；不能返回一个函数
F *f1(int); // ok: 显式指定返回类型是指向函数的指针
```

Of course, we can also declare `f1` directly, which we’d do as

```cpp
int (*f1(int))(int*,int);
```

Reading this declaration from the inside out, we see that `f1` has a parameter list, so `f1` is a function. `f1` is preceded by a `*` so f1 returns a pointer. The type of that pointer itself has a parameter list, so the pointer points to a function. That function returns an int.

For completeness, it’s worth noting that we can simplify declarations of functions that return pointers to function by using a trailing return (§6.3.3):

```cpp
auto f1(int) -> int (*)(int*, int);
```

**利用auto或decltype**

If we know which function(s) we want to return, we can use `decltype` to simplify writing a function pointer return type.

```cpp
string::size_type sumLength(conststring&, const string&);
string::size_type largerLength(const string&, const string&);
decltype(sumLength) *getFcn(const string &);
```

Remember that when we apply decltype to a function, it returns a function type, not a pointer to function type. We must add a `*` to indicate that we are returning a pointer, not a function.