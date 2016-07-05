[toc]

## 13 拷贝控制

类通过五个成员函数 —— 拷贝构造器、移动构造器、拷贝赋值运算符、移动赋值运算符与析构器 —— 控制其对象的拷贝、赋值、移动和销毁。这五个函数称为拷贝控制器。拷贝和移动构造器定义了对象从另一个同类型对象初始化时的行为。拷贝赋值和移动赋值运算符定义了把对象赋给另一个同类型对象的行为。

类没有定义的拷贝控制成员函数，编译器将自动定义。于是多数类可以忽略拷贝控制（§7.1.5）。但对于某些类，使用默认定义将导致灾难。

> 警告：拷贝控制是定义任何 C++ 类必备的部分。

### 13.1 拷贝、赋值、销毁

#### 13.1.1 拷贝构造函数

当一个构造器的第一个参数是类的引用类型，且后续参数都有默认值时，构造器是拷贝构造函数：

```cpp
class Foo{
public:
    Foo(); // 默认构造器
    Foo(const Foo&); // 拷贝构造器
    // ...
};
```

第一个参数是引用类型，且一般是到常量的引用，虽然非常量也是可以的。拷贝构造函数会在多种环境下被隐式使用。因此拷贝构造函数不能是 explicit(§ 7.5.4)。

##### 编译器合成的（Synthesized）拷贝构造器

若我们没有为类定义一个拷贝构造器，编译器会合成一个。与合成的默认构造器不同，即使我们自己定义了其他构造器，拷贝构造器仍会合成。合成的拷贝构造器可能会阻止我们拷贝其对象 （§13.1.6）；若不阻止，合成的拷贝构造器会智能的将所有成员（非静态）拷贝到正在被创建的对象中 （§7.1.5）。

成员的类型决定如何拷贝：类类型的成员通过拷贝构造器拷贝；内建类型的成员被直接拷贝。尽管无法直接拷贝数组（§3.5.1），合成的拷贝构造器拷贝数组的每一个元素。

`Sales_data` 的拷贝构造器等价于：

```cpp
class Sales_data{
public:
    // 其他成员与构造器略
    Sales_data(const Sales_data &);
private:
    std::string bookNo;
    int units_sold = 0;
    double revenue = 0.0;
};

// 等价的拷贝构造器
Sales_data::Sales_data(const Sales_data &orig):
    bookNo(orig.bookNo), // uses the string copy constructor
    units_sold(orig.units_sold),// copies orig.units_sold
    revenue(orig.revenue) // copies orig.revenue
{ } // empty body
```

##### 拷贝初始化

区别**直接初始化**和**拷贝初始化**（§3.2.1）：

```cpp
string dots(10, '.'); // 直接
string s(dots); // 直接
string s2 = dots; // 拷贝
string null_book = "9-999-99999-9"; // 拷贝
string nines = string(100, '9'); // 拷贝
```

当我们使用直接初始化时，我们是在让编译器使用普通的函数匹配选择匹配实参的构造器。而使用拷贝初始化，我们是让编译器将右边的值拷贝到正在创建的对象，并做需要的转换（§7.5.4）。

拷贝初始化一般使用拷贝构造器。但如果类有**移动构造器**，拷贝初始化有时会使用移动构造器替代拷贝构造器（§13.6.2）。

不仅使用 `=` 定义变量时会使用拷贝初始化：

- 将对象作为实参传递给方法参数（不是引用类型的形参）
- 从函数返回对象（返回值不是引用类型）
- 大括号初始化数组，或聚合类（§7.5.5）

Some class types also use copy initialization for the objects they allocate. 例如，初始化容器，或调用容器的 `insert` 或 `push`方法，容器会拷贝初始化它们的元素。但 `emplace` 方法使用直接初始化。

##### explicit 构造器只能直接使用，不能用作拷贝

如果我们的初始化需要一个 explicit 构造器（§7.5.4）做转换，则拷贝或直接初始化是不同的：

```cpp
    vector<int> v1(10); // 可以，这里是直接初始化
    vector<int> v2 = 10; // 错误，取size的构造器是 explicit
    void f(vector<int>); // f的参数被拷贝初始化
    f(10); // 错误！不能使用 explicit 构造器
    f(vector<int>(10)); // 可以：这里使用了直接初始化初始化临时向量
```

##### 编译器可以绕过拷贝构造器

在拷贝初始化过程中，允许编译器跳过拷贝/移动构造器，直接创建对象。例如，允许编译器将

```cpp
string null_book= "9-999-99999-9"; // 拷贝初始化
```

改写成：
```cpp
string null_book("9-999-99999-9"); // 省略拷贝构造器
```

#### 13.1.2 拷贝赋值运算符

类也可以控制类的对象如何被赋值：

```cpp
Sales_data trans, accum;
trans = accum; // 将用到 Sales_data 的拷贝赋值运算符
```

若我们没有自己定义，编译器将合成一个**拷贝赋值运算符**。

重载运算符是函数。名称是 `operator` 加运算符，如 `operator=`。若运算符是成员函数，则运算符的左操作数是 `this`，右操作数作为函数参数传入。

为了与内建类型的赋值一致，赋值运算符一般返回到左操作数的引用。

赋值运算符例子：

```cpp
class Foo {
public:
	Foo& operator=(const Foo&); // assignment operator
// ...
};
```

合成的**拷贝赋值运算符**可能会禁止赋值。否则它将右边的所有非静态成员拷贝到左边，利用成员的拷贝赋值运算符。数组成员也会被逐元素赋值。

下面是 `Sale_data` 合成的拷贝赋值运算符的例子：

```cpp
Sales_data& Sales_data::operator=(const Sales_data &rhs)
{
    bookNo = rhs.bookNo;
    units_sold = rhs.units_sold;
    revenue = rhs.revenue;
    return *this; // return a reference to this object
}
```

#### 13.1.3 析构器

析构器也是成员函数。名取类名前缀 `~`。无返回值无参数。

```cpp
class Foo {
public:
	~Foo(); // destructor
// ...
};
```

构造器有初始化部分和函数体两部分。析构器也有函数体和析构部分。在构造器中，成员初始化在函数体执行前，成员初始化顺序与它们在类中出现的顺序相同。在析构器中，函数体先执行，然后成员被销毁。成员销毁的顺序与初始化的顺序相反。

析构器的析构不会不像构造器的初始化列表，它是隐式的。成员如何销毁取决于其类型。类类型的成员被其析构器销毁。内建类型的成员没有析构器，什么也不做。

若成员是内建指针类型，隐式的析构器不会自动释放它。与普通指针不同，智能指针是类，有析构器，会在析构阶段自动调用。

**析构器何时被调用**

对象被销毁时，包括：

- 变量离开作用域后被销毁
- 成员随着对象的销毁而销毁
- 容器中的元素（包括库容器和数组），随容器销毁
- 动态分配的对象，被调用 delete 时销毁。
- 临时对象在表达式结束后被销毁

到对象的引用或指针离开作用域后，所指对象不会被销毁。

**合成的析构器**

要是我们不自己定义，编译器会合成一个。合成曾的析构器可能会禁止对象被销毁（§13.1.6），否则，合成的析构器有一个空的函数体。

在（空的）析构器函数体运行后，成员会被自动销毁。特别的，`string` 的析构器会运行，释放内存。

注意析构器函数体不会直接销毁成员。成员随着隐式的析构阶段（在函数体执行后）销毁。A destructor body executes in addition to the memberwise destruction that takes place as part of destroying an object.

#### 13.1.4 The Rule of Three/Five

目前为止，我们看到三个基本操作控制着类对象的拷贝：拷贝构造器、拷贝赋值构造器和析构器。但在新标准下，类还可以有移动构造器和移动赋值运算符，见 §13.6。

**需要析构器的类需要拷贝和赋值**

判断类是否需要拷贝控制成员，可以先判断类是否需要析构器。是否需要析构器一般更容易确定。且**一个类如果需要析构器，一般也需要拷贝构造器和拷贝赋值运算符**。

例如，若类 `HasPtr` 有一个动态分配的内建指针成员，这时我们肯定会想到需要析构器是否动态指针：

```cpp
class HasPtr {
public:
    HasPtr(const std::string &s = std::string()):
    	ps(new std::string(s)), i(0) { }
    ~HasPtr() { delete ps; }
};
```

此时，若我们不重新定义拷贝构造器和拷贝赋值运算符，使用自动合成的。在拷贝时，指针拷贝，导致两个指针指向同一块内存区域。当一个对象被销毁时，动态指针释放，另一个对象的指针成员悬空！

**需要拷贝就需要赋值，反之亦然**

有些类需要拷贝和赋值，但不需要析构。例如，一个类的成员，每个需要一个唯一编号。拷贝或赋值时，需要产生新编号。

#### 13.1.5 使用 `= default`

利用 `= default` 我们可以显式请求编译器合成拷贝控制成员。

```cpp
class Sales_data {
public:
    // copy control; use defaults
    Sales_data() = default;
    Sales_data(const Sales_data&) = default;
    Sales_data& operator=(const Sales_data &); // 声明
    ~Sales_data() = default;
    // other members as before
};
Sales_data& Sales_data::operator=(const Sales_data&) = default; // 定义
```

若再类正文内使用 `= default`，合成的函数将是隐式内联的（与其他成员函数一样）。如果不想内联的，可以将 `= default` 放在成员定义处，如上面的拷贝赋值运算符。

#### 13.1.6 阻止拷贝

> 多数类应该定义（隐式或显式的定义）默认构造器、拷贝构造器和拷贝赋值运算符。

有些类不应支持拷贝，如 `iostream`；阻止多个对象读写同一个IO缓冲区。

不定义拷贝控制成员函数并不能阻止拷贝。如果我们不定义，系统会自动合成。

新标准，允许我们将拷贝构造器和拷贝赋值运算符声明为 deleted 函数。deleted 函数声明了，但不能使用。

```cpp
struct NoCopy {
	NoCopy() = default; // use the synthesized default constructor
	NoCopy(const NoCopy&) = delete; // no copy
	NoCopy &operator=(const NoCopy&) = delete; // no assignment
	~NoCopy() = default; // use the synthesized destructor
	// other members
};
```

与 `= default` 不同的是，`= delete` 必须放在函数首次声明时。

与 `= default` 不同的是，`= delete` 不仅可以用于编译器合成的默认构造器和拷贝控制成员函数，deleted functions are sometimes also useful when we want to guide the function-matching process。

析构器不能是 deleted 成员。The compiler will not let us define variables or create temporaries of a type that has a deleted destructor. Moreover, we cannot define variables or temporaries of a class that has a member whose type has a deleted destructor.

**合成的拷贝控制成员可能是 Deleted**

对于某些类，编译器会把合成的成员（包括构造器、析构器、拷贝控制器）定义成 deleted：

- 如果类有一个成员，其析构器是 deleted 或不可访问（如是私有的），则类的析构器是 deleted。
- 若某成员的拷贝构造器是 deleted 或不可访问的；或某成员的析构器是 deleted 或不可访问的，则类的拷贝构造器是 deleted。
- 合成的拷贝赋值运算符定义为 deleted，如果，
  1. 若类某成员的拷贝赋值运算符是 deleted 或不可访问的。
  2. 某成员是常量。常量意味着不能赋值，就没赋值运算符什么事了。
  3. 某成员是引用。赋值并不难改变引用的指向（引用的指向是不能改变的），引用不会指向右运算符指向的对象。这一般不是你期望的。
- 合成的默认构造器被定义为 deleted，如果：
  1. 某成员的析构器是 deleted 或不可访问的；
  2. 有引用成员但没有类内初始化（§2.6.1）；
  3. 有一个常量成员，其类型未显式定义默认构造器，且没有类内初始化。

概括讲，这些规则意味着，如果类有某个数据成员，不能被默认构造、拷贝、赋值或销毁，则相应的成员函数会变成 deleted 函数。

为什么若类有一个成员的析构器是 deleted 或不可访问的，则合成的默认构造器和拷贝构造器会变成 deleted。原因是，若不，创建好的对象将无法销毁。

We’ll see in §13.6.2, §15.7.2, and §19.6 that there are other aspects of a class that can cause its copy members to be defined as deleted.

**私有的拷贝控制**

新标准之前，阻止拷贝的方法是将拷贝构造器和拷贝赋值运算符声明为私有。

```cpp
class PrivateCopy {
    PrivateCopy(const PrivateCopy&);
    PrivateCopy &operator=(const PrivateCopy&);
    // other members
public:
    PrivateCopy() = default; // 合成的默认构造器
    ~PrivateCopy(); // users can define objects of this type but not copy them
};
```

定义成私有后，成员和友元仍可以使用它们。要进一步阻止使用，需要只声明不定义。With one exception, which we’ll cover in §15.2.1, it is legal to declare, but not define, a member function (§6.1.2). An attempt to use an undefined member results in a link-time failure.

#### 13.2 拷贝控制和资源管理

什么是对象的拷贝？两个选择：让类表现的像一个值还是一个指针。若对象是值，则拷贝后二者独立。若对象是指针，则拷贝后共享状态，底层数据共享，修改一个影响另一个。`string` 像一个值，而 `shared_ptr` 像一个指针。

对于内建类型的成员（指针除外），拷贝都是直接的值拷贝。但指针的拷贝方式决定对象像值还是像指针。

#### 13.2.1 像值一样的类

类 `HasPtr` 有两个成员，一个 `int` 和一个到 `string` 的指针。要让 `HasPtr` 像值，我们需要：

- 一个拷贝构造器，拷贝字符串，而不是指针
- 一个析构器是否字符串
- 一个拷贝赋值运算符，释放对象已有的字符串，拷贝来自右操作数的字符串

```cpp
class HasPtr {
public:
	// 动态分配一个string
    HasPtr(const std::string &s = std::string()):
    	ps(new std::string(s)), i(0) { }
    // 拷贝构造器，拷贝字符串
    HasPtr(const HasPtr &p):
    	ps(new std::string(*p.ps)), i(p.i) { }
    HasPtr& operator=(const HasPtr &);
    // 析构器，释放内存
    ~HasPtr() { delete ps; }
private:
    std::string *ps;
    int i;
};

HasPtr& HasPtr::operator=(const HasPtr &rhs)
{
    auto newp = new string(*rhs.ps); // 拷贝底层的字符串
    delete ps; // free the old memory
    ps = newp; // copy data from rhs into this object
    i = rhs.i;
    return *this; // return this object
}
```

赋值要考虑自己赋给自己的情况，不能出错。一个号的模式是，先把右边操作数拷贝到临时变量。然后销毁左边操作数。然后把临时变量赋给左操作数。

#### 13.2.2 像指针一样的类

若想让 HasPtr 像一个指针，则拷贝构造器和拷贝赋值运算符要拷贝指针成员，注意不是拷贝指针指向的对象。析构器要负责释放内存，但仅当不再有 HasPtr 指向一个字符串时。最简单的解决办法是用 `shared_ptr` 管理类的资源。拷贝（或赋值）一个 `shared_ptr` 拷贝指针，由 `shared_ptr` 负责追踪引用计数。

如果想直接管理资源，我们就得自己实现类似于引用计数的机制。

引用计数工作方式如下：

- 每个构造器（拷贝构造器除外）要创建一个计数器。创建对象时，计数器初始化为1。
- 拷贝构造器不创建**新**计数器。拷贝数据成员，**包括计数器**！且增加该计数器。
- 析构器减少计数。如果计数归零，删除状态。
- 拷贝赋值运算符增加右操作数的计数器，减少左操作数的计数器。当左边的计数归零时，表示没有用户了。拷贝赋值运算符要负责销毁左操作数的状态。

计数器也应该是动态分配的：新创建对象，分配动态内存给计数器。拷贝对象，拷贝计数器的指针：让共享资源的对象访问相同的计数器。

```cpp
class HasPtr {
public:
	// constructor allocates a new string and a new counter, which it sets to 1
	HasPtr(const std::string &s = std::string()):
		ps(new std::string(s)), i(0), use(new std::size_t(1))
    {}
	// 拷贝所有成员，增加计数器
	HasPtr(const HasPtr &p):
		ps(p.ps), i(p.i), use(p.use) { ++*use; }
	HasPtr& operator=(const HasPtr&);
	~HasPtr();
private:
	std::string *ps;
	int i;
	std::size_t *use; // 计数器
};

HasPtr::~HasPtr()
{
    if (--*use == 0) { // if the reference count goes to 0
        delete ps; // delete the string
        delete use; // and the counter
    }
}

// 拷贝赋值运算符既要完成拷贝有要做析构器的事情
// 还要处理自己赋给自己的情况
HasPtr& HasPtr::operator=(const HasPtr &rhs)
{
    ++*rhs.use; // increment the use count of the right-hand operand
    if (--*use == 0) { // then decrement this object's counter
        delete ps; // if no other users
        delete use; // free this object's allocated members
    }
    ps = rhs.ps; // copy data from rhs into this object
    i = rhs.i;
    use = rhs.use;
    return *this; // return this object
}
```

### 13.3 Swap

管理资源类一般还要定义 `swap` 函数（§9.2.5）。重排元素的算法会调用 `swap` (§10.2.3)。如果类定义了自己的 `swap`，则算法会用类自己的。否则会用库的 `swap`。

{{库的}} swap 一般要涉及一次拷贝和两次赋值，如下。

```cpp
HasPtr temp = v1; // make a temporary copy of the value of v1
v1 = v2; // assign the value of v2 to v1
v2 = temp; // assign the saved value of v1 to v2
```

**编写我们自己的 swap 函数**

通过为我们自己的类定义 `swap`，我们可以覆盖 `swap` 的默认行为。典型实现是：

```cpp
class HasPtr {
	friend void swap(HasPtr&, HasPtr&);
	// other members as in §13.2.1
};

inline void swap(HasPtr &lhs, HasPtr &rhs)
{
    using std::swap;
    swap(lhs.ps, rhs.ps); // 交换指针，而不是字符串数据
    swap(lhs.i, rhs.i); // 交换整型成员
}
```

`swap` 声明为类的友元。因为 `swap` 是为了优化代码，因此我们将其定义为内联函数。

与拷贝控制成员不同，`swap` 不是必须的。但对于分配资源的类，它是一项重要的优化。

**swap 应该调用 swap，不要调用 std::swap**

例如，我们有一个类 `Foo`，有一个成员 `h`，是 `HasPtr` 类型的。现在我们尚未为 `Foo` 声明 `swap`，因此我们只能用库的。而库的 `swap` 会使用拷贝，造成 `HasPtr` 不必要的拷贝。为避免该问题，我们可以为 `Foo` 编写一个 `swap`。但下面的写法是错误的：

```cpp
void swap(Foo &lhs, Foo &rhs)
{
	// 错误：函数使用库版本的swap，而不是 HasPtr 的
	std::swap(lhs.h, rhs.h);
	// swap other members of type Foo
}
```

正确的写法是：

```cpp
void swap(Foo &lhs, Foo &rhs)
{
    using std::swap;
    swap(lhs.h, rhs.h); // uses the HasPtr version of swap
    // swap other members of type Foo
}
```

如果有类的 swap，该版本比库版本更优匹配，调用 `swap` 时会选择类的版本。若没有类的版本，且用了 `using` 引入作用域，则调用系统版本的。

`using` 声明引入的 `swap` 并不会覆盖 HasPtr 版本的 `swap`，原因见 §18.2.3。

**在赋值中使用 swap**

定义有 `swap` 的类一般会用 `swap` 定义它们的赋值运算符。These operators use a technique known as **copy and swap**.

```cpp
// 注意 rhs 是按值传递的，HasPtr copy constructor
// copies the string in the right-hand operand into rhs
HasPtr& HasPtr::operator=(HasPtr rhs)
{
    // 左操作数与局部变量 rhs 交换
    swap(*this, rhs); // rhs 现在时 this 之前的值
    return *this; // rhs 被销毁，删除 rhs 中的指针
}
```

这种形式的好处是能自动处理自己赋值给自己的情况。第二，对异常安全。The only code that might throw is the new expression inside the copy constructor. If an exception occurs, it will happen before we have changed the left-hand operand.

### （未）13.4 拷贝控制的例子

分配资源并不是需要拷贝控制的唯一原因。有时需要记账等其他操作。

例如我们有一个邮件处理应用。两个类 Message 和 Folder。分别表示邮件和目录。一个邮件可以同时属于多个目录，一个目录下也可以由多个邮件，即多对多关系。为此，Message 类有一个 set，持有 Folder 的指针；Folder 类也有一个 set，持有 Message 的指针。

Message 类提供 save 和 remove 操作，添加或删除 Folder 关系。
拷贝 Message，拷贝后的消息和原消息是独立的，但属于同一个 Folder。因此，拷贝 Message 不仅要拷贝内容和 set，也要将新增的 Message 放入所属 Folder。

而销毁一个 Message，也要在所属 Folder 中删除它。

一个 Message 赋值给另一个，左边的内容替换为右边的；也要更新相应 Folder。

The `Folder` class will need analogous copy control members to add or remove itself from the `Message`s it stores.

We’ll leave the design and implementation of the Folder class as an exercise. However, we’ll assume that the Folder class has members named addMsg and remMsg that do whatever work is need to add or remove this Message, respectively, from the set of messages in the given Folder.

**Message 类**

### （未）13.5 Classes That Manage Dynamic Memory

### （未）13.6 移动对象

新标准允许移动而非拷贝对象。如 §13.1.1 所示，很多情况下会发生拷贝。在某些条件下，对象被拷贝后会立即被销毁，这时如果是移动而非拷贝，性能会得到显著提升。

语言的之前版本不能直接移动对象。存储在容器中的类要求可能被拷贝。新标准则只要求能移动。

库容器、 `string`、 `shared_ptr` 支持移动和拷贝。IO 和 `unique_ptr` 只支持移动，不支持拷贝。

#### 13.6.1 右值引用

为支持移动，新标准引入了一个新的引用：右值引用。右值引用只能绑定到一个右值。右值引用通过 `&&` 获得（不是 `&`）。右值引用只能绑定到一个将要销毁的对象。于是我们可以将资源从右值移动到另一个对象。

左值和右值都是表达式的属性（§4.1.1）。一些表达式产生或需要左值，另一些产生或需要右值。一般来讲，左值表达式引用对象标示（identity）而右值表达式引用对象的值。

与引用一样，右值引用也只是一个对象的另一个名字。
我们不能将一个普通引用绑定到需要转换的表达式，或绑定到字面量，或返回右值的表达式（§2.3.1）。
右值引用与此相反，我们可以将右值引用绑定到这些表达式，但不能直接绑定到一个左值：

```cpp
int i = 42;
int &r = i;
int &&rr = i; // 错误：不能绑定到左值
int &r2 = i * 42; // 错误，i * 42 是右值
const int &r3 = i * 42; // 可以：普通引用可以绑定到一个**常量的**右值
int &&rr2 = i * 42; // ok: bind rr2 to the result of the multiplication
```

返回左值引用的函数，赋值，下标，解引用，前缀版的增减运算符，都是返回左值的表达式。

返回非引用类型的函数，算术、关系、二进制、后缀版的增减运算符，都产生右值。

**左值是持久的；右值是短暂的**

右值是字面量，或在求值过程中的临时对象。因为右值引用绑定到临时量，因此我们可以确定：

- 指向的对象快要被销毁
- 不可能有其他人使用该对象

于是我们可以接管右值引用指向的对象中的资源。

**变量是左值**

变量也是表达式：可以看成只有一个操作数，没有运算符。变量表达式是左值。于是我们绑定向其绑定一个右值引用：

```cpp
int &&rr1 = 42; // ok: literals are rvalues
int &&rr2 = rr1; // error: the expression rr1 is an lvalue!
```

**库函数 `move`**

定义在头 `utility`。

Although we cannot directly bind an rvalue reference to an lvalue, we can explicitly cast an lvalue to its corresponding rvalue reference type. 还可以利用库函数 `move`，获得一个左值的右值引用。The move function uses facilities that we’ll describe in §16.2.6 to return an rvalue reference to its given object.

```cpp
int &&rr3 = std::move(rr1); // ok
```

调用 `move` 告诉编译器我们打算将左值当做右值。调用 `move` 后我们不能再使用 `rr1`，除了对它赋值或销毁它。After a call to move, we cannot make any assumptions about the value of the moved-from object.

As we’ve seen, differently from how we use most names from the library, we do not provide a `using` declaration (§3.1) for move (§13.5). We call `std::move` not move. We’ll explain the reasons for this usage in §18.2.3.

#### 13.6.2 移动构造器与移动赋值

要使我们的类支持移动，需要定义移动构造器和移动赋值运算符。这些成员与拷贝策划那个呀类似，只是它们从对象偷资源，而非拷贝。

移动构造器与拷贝构造器类似，除第一个引用参数外，其他参数必须都有默认值。只是对于移动构造器，引用参数是右值引用。

移动要后确保原对象不再持有资源。切断关系，方式原对象销毁时引起资源销毁。

As an example, we’ll define the `StrVec` move constructor to move rather than copy the elements from one `StrVec` to another:

```cpp
StrVec::StrVec(StrVec &&s) noexcept // move won't throw any exceptions
	// member initializers take over the resources in s
	: elements(s.elements), first_free(s.first_free), cap(s.cap)
	{
		// leave s in a state in which it is safe to run the destructor
		s.elements = s.first_free = s.cap = nullptr;
	}
```

We’ll explain the use of `noexcept` (which signals that our constructor does not throw any exceptions) shortly, but let’s first look at what this constructor does.

**移动运算符，库容器和异常**

因为移动运算偷资源，而不分配资源，因此一般不会抛出异常。我们应告知编译器这个事实。否则库需要做一些额外工作。

One way inform the library is to specify `noexcept` on our constructor. We’ll cover `noexcept`, which was introduced by the new standard, in more detail in §18.1.4. For now what’s important to know is that `noexcept` is a way for us to promise that a function does not throw any exceptions. We specify noexcept on a function after its parameter list. In a constructor, `noexcept` appears between the parameter list and the : that begins the constructor initializer list.

We must specify `noexcept` on both the declaration in the class header and on the definition if that definition appears outside the class.

在移动操作中抛出异常，可能留下不一致的状态。例如类有三个成员，在前两个成员移动后出错，第三个成员未被移动。作为容器（如 vector）要保证，若操作（如 push_back）抛出异常，容器要恢复之前状态。拷贝不会出现不一致的问题。因此作为容器，除非它被告知移动操作不会抛出异常，否则它会使用拷贝。

**移动赋值运算符**

移动赋值运算符与**析构器**和**移动构造器**的工作相同。与移动构造器一样，如果移动赋值运算符不抛出异常，应标记 `noexcept`。移动赋值运算符要能处理自己赋值给自己。

```cpp
StrVec &StrVec::operator=(StrVec &&rhs) noexcept
{
    // direct test for self-assignment
    if (this != &rhs) {
        free(); // free existing elements
        elements = rhs.elements; // take over resources from rhs
        first_free = rhs.first_free;
        cap = rhs.cap;
        // leave rhs in a destructible state
        rhs.elements = rhs.first_free = rhs.cap = nullptr;
    }
    return *this;
}
```

**A Moved-from Object Must Be Destructible**

Moving from an object does not destroy that object: Sometime after the move operation completes, the moved-from object will be destroyed. Therefore, when we write a move operation, we must ensure that the moved-from object is in a state in which the destructor can be run. Our `StrVec` move operations meet this requirement by setting the pointer members of the moved-from object to `nullptr`.

**合成的移动操作**

若我们不定义，编译器总是会合成拷贝构造器或拷贝赋值运算符。合成的版本可能是 deleted。

但对于移动操作，若我们自定义了拷贝构造器、拷贝赋值运算符或析构器，则不会合成移动构造器和移动赋值运算符。于是，一些类最终没有移动构造器或移动赋值运算符。此时，相应的拷贝操作代替移动。

仅当我们没有定义拷贝控制成员，且每个非静态成员都可以被移动，编译器才会合成移动构造器或移动赋值运算符。编译器可以移动内建类型的成员，也可以移动拥有相应移动操作的成员。

```cpp
// the compiler will synthesize the move operations for X and hasX
struct X {
	int i; // 内建类型可以被移动
	std::string s; // string 定义了移动操作
};
struct hasX {
	X mem; // X has synthesized move operations
};
X x, x2 = std::move(x); // uses the synthesized move constructor
hasX hx, hx2 = std::move(hx); // uses the synthesized move constructor
```

与拷贝操作不同，移动操作从不会被隐式的定义诶 deleted 函数。但如果你通过 `= default` 显式要求编译器产生移动操作，有可能出现移动操作被定义成 deleted：

- 移动构造器/移动赋值被定义成 deleted，如果，
  1. 某成员定义了自己的拷贝构造器，但没有定义移动构造器
  2. 或，某成员没有定义自己的拷贝构造器，编译器不能合成移动构造器
- 移动构造器/移动赋值被定义成 deleted，如果某成员的移动构造器/移动赋值被定义成 deleted，或不可访问。
- 移动构造器被定义成 deleted，如果类的析构器被定义成 deleted 或不可访问的
- 移动赋值被定义成 deleted，如果某成员是常量或引用

反过来，如果类定义了移动构造器或移动赋值运算符，则合成的拷贝构造器和拷贝赋值运算符将被声明为 deleted。即，定义了移动操作就一定要定义拷贝操作。

**Rvalues Are Moved, Lvalues Are Copied ...**

When a class has both a move constructor and a copy constructor, the compiler uses ordinary function matching to determine which constructor to use (§ 6.4, p. 233). Similarly for assignment. For example, in our StrVec class the copy versions take a reference to const StrVec. As a result, they can be used on any type that can be converted to StrVec. The move versions take a StrVec&& and can be used only when the argument is a (nonconst) rvalue:

```cpp
StrVec v1, v2;
v1 = v2; // v2是左值，拷贝赋值；（因为右值引用不能绑定到左值）
StrVec getVec(istream &); // getVec 返回一个右值
v2 = getVec(cin); // getVec(cin) 是一个右值；移动赋值
```

**...But Rvalues Are Copied If There Is No Move Constructor**

若类只定义拷贝构造器，此时类是没有合成的移动构造器的。If a class has no move constructor, function matching ensures that objects of that type are copied, even if we attempt to move them by calling move:

```cpp
class Foo {
public:
	Foo() = default;
	Foo(const Foo&); // copy constructor
	// other members, but Foo does not define a move constructor
};
Foo x;
Foo y(x); // copy constructor; x is an lvalue
Foo z(std::move(x)); // 仍是拷贝构造器，因为没有移动版
```

The copy constructor for Foo is viable because we can convert a `Foo&&` to a `const Foo&`.

用拷贝构造器替代移动构造器是绝对安全的。对赋值也是。

**Copy-and-Swap Assignment Operators and Move**










