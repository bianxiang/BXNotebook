[toc]

### 2.4 const限定符

{{const 属于变量声明的“基础类型”部分。}}

想令变量不可改变，可以限定 `const`：

```cpp
const int bufSize = 512;  // input buffer size
```

因为const对象创建后不能改变，因此必须初始化。初始化器可以使用任意复杂的表达式：

```cpp
const int i = get_size(); // 运行时初始化是可以的
const int j = 42; // 编译时初始化
const int k; // 错误
```

---

**const对象局限于单个文件**

如果常量是被一个编译时常量初始化的，如：

```cpp
const int bufSize = 512;
```

编译器一般会在编译时将变量的使用替换为变量的值。

要替换变量的值，编译器必须能看到变量的初始化器。当把程序分成多个文件时，每个使用此常量的文件都必须能访问它的初始化器。为了看到初始化器，变量必须在每个文件中的定义。为支持这种用法，同时避免多次的定义同一个变量，**const变量局限于文件**。在多个文件中使用使用相同的名字定义一个const，就好像定义的是不同的变量。

有时期望多个文件共享的const变量的初始化器不是一个常量表达式。此时，我们不想让编译器在每个文件中产生一个变量。我们想在一个文件中定义const，在其他文件中声明。要定义一个const变量的单个实例，我们需要在定义和声明时都加 `extern` 关键字。

```cpp
// file_1.cc定义并初始化const，并令其可被其他文件访问
extern const int bufSize = fcn();
// file_1.h
extern const int bufSize; // same bufSize as defined in file_1.cc
```

---

#### 2.4.1 到const的引用

可以绑定到const类型的引用。到常量的引用与常量本身一样，不能用于改变绑定的对象：

```cpp
const int ci = 1024;
const int &r1 = ci; // ok: ci和r1都是常量
r1 = 42; // 错误：r1指向的是常量
int &r2 = ci; // 错误：r2不是一个常量引用，因此不能引用一个常量
```

> 术语：常量引用（const Reference）是一个到常量的引用。C++程序员一般称“reference to const”为“const reference”。这是个缩略语。技术上讲，并没有const references。引用不是对象，因此不能令引用本身是常量。但另一方面，因为不能改变让引用指向的对象，引用就像一个常量。“引用指向的是常量”决定我们能对被引用做什么。而不是是否能改变引用的绑定。

##### 初始化常量引用

之前我们提到，引用的类型必须匹配被应用对象的类型。但有两个例外。第一是，常量引用，可以使用任意可以转换为匹配类型的表达式初始化。特别的，可以将到常量的引用绑定到**非常量**对象：

```cpp
int i = 42;
const int &r1 = i; // 可以将 const int & 绑定到非常量的int
const int &r2 = 42; // ok
const int &r3 = r1 * 2; // ok: r3 is a reference to const
int &r4 = r * 2; // 错误，r4不是一个常量引用
```

例子：绑定不同类型：

```cpp
double dval= 3.14;
const int &ri = dval;
```

以上代码，编译器实际会出下面的变换：

```cpp
const int temp = dval; // 从double创建一个临时 const int
const int &ri = temp; // 将ri绑定到一个临时变量
```

即 `ri` 绑定到了一个临时对象。

为什么不同类型的绑定，引用必须是常量的？如下面是错误的：

```cpp
double dval= 3.14;
int &ri = dval; // 不允许
```

因为如果不是常量，程序员可能向通过为引用赋值改变引用绑定的原对象。但引用此时绑定的实际是一个临时对象，不会影响原对象。于是在C++中，不同类型的绑定，引用必须是const的。

##### 常量引用可以引用一个不是常量的对象

常量引用只会限制我们可以对引用做什么，但限制不了对底层变量的直接操作。因为底层的对象不是常量，它可能通过其他方式改变：

```cpp
int i = 42;
int &r1 = i; // r1 bound to i
const int &r2 = i; // r2也绑定到i；但是不能用于改变i
r1 = 0; // r1 is not const; i is now 0
r2 = 0; // error: r2 is a reference to const
```

#### 2.4.2 指针与const

指针可以指向常量或非常量类型。*到常量的指针* 不能用于修改指向的对象。常量对象的地址只能放在 *到常量的指针* 里。

```cpp
const double pi = 3.14;
double *ptr = &pi; // 错误，ptr只是普通指针
const double *cptr = &pi; // 正确
*cptr = 42; // 错误，不能赋值给*cptr
```

指针与指向的对象类型必须匹配。但有两个例外。第一个是，到常量的指针可以指向非常量对象。

```cpp
double dval = 3.14;
cptr = &dval; // 可以。但不同通过cptr改变dval
```

注意，如果底层对象不是常量，它可能被其他方式改变。

*指向常量的指针*，指针本身是变量，指针指向的是对象是常量。但下面的 *常量指针*，指针本身是常量，指针指向的对象可能是常量或变量。

##### const指针

指针本身是对象，于是指针本身可以是常量。与其他常量对象一样，常量指针必须被初始化；一旦初始化，不能改变。要表示指针是常量，需要在 `*` 后加 `const`。

```cpp
int errNumb = 0;
int *const curErr = &errNumb; // curErr总是指向errNumb
const double pi = 3.14159;
const double *const pip = &pi;
```

为方便理解，从右向左读。`curErr` 标识符最近的是const，表示**curErr自己是常量**。

指针本身是常量，不影响是否可以利用指针改变底层对象。是否允许只取决于指针指向什么。

#### 2.4.3. 顶级（Top-Level）常量

指针本身是常量称为顶级（top-level）常量。如果指针指向常量对象，这种常量称为底层（low-level）常量。更一般的说，**顶级常量表示对象自己是常量**。任何类型都可以是顶级常量。底层常量表示复合类型的**基础类型**是常量。而指针，可以同时为顶级或底层常量：

```cpp
int i= 0;
int *const p1 = &i; // 顶级常量。不能改变p1的值
const int ci = 42; // 顶级常量。不能改变ci
const int *p2 = &ci; // 常量是底层常量。可以改变p2
const int *const p3 = p2;
const int &r = ci; // 常量引用总是底层的
```

顶级和底层常量的区别在**拷贝**对象时很重要。拷贝一个对象时，顶级常量会被忽略：

```cpp
i = ci; // 可以：拷贝ci的值；ci中的顶级常量被忽略
```

拷贝对象｛｛注意，不是赋值！！！｝｝不改变被拷贝对象。因此，拷贝或被拷贝对象是否是常量都不重要。

与此相反，底层常量永不会被忽略。拷贝一个对象时，**两个对象必须具有相同的底层常量的限定**，或者在两个对象类型之间存在一种转换。一般来说，可以将非常量转换为常量，但反之不行：

```cpp
int *p = p3; // 错误：p3有低级常量但p没有
p2 = p3; // 可以
p2 = &i; // ok：int*可以转换成const int*
int &r = ci; // 错误
const int &r2 = i; // 可以
```

#### 2.4.4 `constexpr` 和常量表达式

常量表达式是值不会改变的表达式。于是可以在编译期求值。字面量是常量表达式。一个常量对象，且被一个常量表达式初始化的对象，也是一个常量表达式。

一个对象（或表达式）是否为常量表达式取决于类型和**初始化器**。例子：

```cpp
const int max_files = 20; // max_files是常量表达式
const int limit = max_files + 1; // 是
int staff_size = 27; // 不是
const int sz = get_size(); // sz不是一个常量表达式，因为get_size()在运行时才知道
```

##### `constexpr`变量

在大型系统中，有时很难知道一个初始化器是否是常量表达式。可能仅当将常量放在一个需要常量表达式的地方时才发现其实它不是。经常发现，对象的定义和使用它的上下文不在一起。

新标准下，我们可以令编译器验证一个变量是否是常量表达式：将变量声明为 `constexpr`。 `constexpr` 变量是隐式的常量，因此必须初始化：

```cpp
constexpr int mf = 20; // 20是常量表达式
constexpr int limit = mf + 1; // mf + 1是常量表达式
constexpr int sz = size(); // 仅当size是constexpr函数时
```

对于constexpr变量，不能用普通函数初始化。（§6.5.2）新标准引入了**constexpr函数**。这类函数必须非常简单，令编译器可以在编译时求值。constexpr函数可以作为constexpr变量的初始化器。

一般来说，对准备在常量表达式中使用的变量，应该声明为 `constexpr`。

##### 字面量类型

因为常量表达式在编译期求值，于是可以在 `constexpr` 声明中使用的类型受限。可以在constexpr 中使用的类型称为**字面量类型**（literal types）。

之前用的所有类型中，算术、引用和指针是字面类型。`Sales_item`、IO库、**字符串**等不是。因此不能定义这些类型的变量是constexprs。We’ll see other kinds of literal types in §7.5.6 and §19.3.

尽管可以将指针和引用定义为constexpr，但用于初始化它们的对象是严格受限的。初始化constexpr指针可以用 `nullptr` 字面量或字面量 `0`。We can also point to (or bind to) an object that remains at a fixed address.

定义在函数内的变量一般没有固定地址（§6.1.1）。因此，不能令一个constexpr指针指向这种变量。定义在函数外的对象*的地址*是一个常量表达式，可以用于初始化一个constexpr指针。

##### 指针和 constexpr

在一个 constexpr 声明中定义的指针，constexpr 限定符应用于指针而不是指针指向的类型：

```cpp
const int *p = nullptr;  // p指向一个常量int
constexpr int *q = nullptr; // q是一个常量指针，指向普通int
```

`constexpr` 产生的是一个顶级常量。与其他常量指针一样，指针可以指向常量也可以指向非常量。

```cpp
constexpr int *np = nullptr;
int j = 0;
constexpr int i = 42;  // type of i is const int
// i and j must be defined outside any function
constexpr const int *p = &i; // p是常量指针，指向const int i
constexpr int *p1 = &j;  // p1是常量指针，指向int j
```