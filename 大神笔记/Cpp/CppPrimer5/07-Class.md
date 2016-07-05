[toc]

## 7 类

### 7.1 定义抽象数据类型

#### 7.1.1 定义 `Sales_data` 类

在学习定义之前，先看看如何使用：

```cpp
Sales_data total;
if (read(cin, total)) {
    Sales_data trans;
    while(read(cin, trans)) {
        if(total.isbn() == trans.isbn())
            total.combine(trans);
        else {
            print(cout, total) << endl;
            total = trans;
        }
    }
    print(cout, total) << endl;
} else {  // there was no input
    cerr << "No data?!" << endl;
}
```

#### 7.1.2 定义 `Sales_data`

成员函数可以**定义**在类内或类外。非成员函数但属于接口的函数（如`add`, `read`）定义在类外。

```cpp
struct Sales_data {
    // 操作
    std::string isbn() const { return bookNo; }
    Sales_data& combine(const Sales_data&);
    double avg_price() const;
    // 数据成员
    std::string bookNo;
    unsigned units_sold = 0;
    double revenue = 0.0;
};

// 非成员但属于接口
Sales_data add(const Sales_data&, const Sales_data&);
std::ostream &print(std::ostream&, const Sales_data&);
std::istream &read(std::istream&, Sales_data&);
```

> 定义在类内的函数是隐式内联的(§6.5.2)。

##### `this`

使用`this`重新实现`isbn`：

```cpp
std::string isbn() const { return this->bookNo; }
```

##### const成员函数

注意到 `isbn` 函数后面的 `const` 关键字。`const` 用于限定 `this` 指针。

一个指向类的、非常量版本的指针的默认类型是常量指针（指针本身是常量，因为不能改变 `this` 的绑定）。例如 `Sales_data` 的成员函数内的 `this` 的类型是 `Sales_data * const`。**这样的指针不能绑定到常量对象**。于是，我们不能在一个常量对象上调用一个普通的成员函数。

因为 `isbn` 其实不会修改对象，我们期望函数内的 `this` 是个 `const Sales_data * const`，即，指向常量的指针。实现方法是，在函数参数列表后添加 `const` 关键字。这种函数称为**常量成员函数**。

> 常量对象，或指向常量对象的指针、引用，**只能调用常量成员函数**。

##### 类作用域与成员函数

类自己是一个作用域。成员函数的作用域嵌套在类的作用域内。因此在 `isbn` 中使用 `bookNo`，会被解析成 `Sales_data` 中的数据成员。

注意到 `isbn` 可以使用 `bookNo`，虽然它定义在 `isbn` 之后。§7.4.1 会谈到，编译器会分两步处理类。成员的声明先被编译，然后是成员函数的函数体。因此在成员函数内部，可以使用其他成员，不论定义的先后顺序。

##### 在类外*定义*一个成员函数

如果成员被声明为常量成员函数，则函数*定义*也必须带 `const` 关键字。同时前面必须加类名限定：

```cpp
double Sales_data::avg_price() const {
    if(units_sold)
        return revenue/units_sold;
    else
     return 0;
}
```

当编译器遇见函数名时，剩下的代码就相当于位于类的作用域内。因此 `revenue` 和 `units_sold` 都会被解析为 `Sales_data` 的数据成员。

##### 函数返回当前对象

`combine` 需要返回当前对象，且以左值的形式。于是返回类型必须是引用类型。通过 `*this` 获得当前对象。

```cpp
Sales_data& Sales_data::combine(const Sales_data &rhs)
{
    units_sold += rhs.units_sold;
    revenue += rhs.revenue;
    return *this; // 返回对象自身
}
```

#### 7.1.3 与类相关的非成员函数

类作者常要定义一些辅助函数，如 `add`、 `read`、 `print` 函数。尽快这些函数概念上是接口的一部分，但它们却不属于类。与普通函数类似，它们的声明和定义一般是分开的。这些函数的**声明**一般与**类声明**放在同一个头文件。

#### 7.1.4 构造器

构造器是个很复杂的主题，后续还会有多个章节涉及。
构造器没有返回值。类可以有多个构造器。
构造器不能声明为常量函数(§7.1.2)。对象在构造完后才能是常量的。

##### 合成的默认构造器

之前的 `Sales_data` 类没有定义任何构造器。却可以被正常使用。

```cpp
Sales_data total;
Sales_data trans;
```

上面两个变量，将被默认初始化。类通过一个特殊的构造器控制默认初始化：默认构造器。默认构造器是无参构造器。

若类不定义默认构造器，编译器将自动产生一个，称为合成（synthesized）默认构造器。对于多数类，该默认构造器初始化成员的方式是：

- 如果有**类内初始化**(§2.6.1)，用它先初始化成员
- 否则，默认初始化(§2.2.1)成员。

##### 一些类不能依赖合成的默认构造器

只有相对简单的类可以依赖合成的默认构造器。最常见的原因是，仅当我们不定义其他构造器时，编译器才会产生默认构造器。

第二个原因是，合成的构造器有时会做错事。例如，内建或复合类型（例如数组和指针）的对象在一个块内定义时，若被默认初始化，则其值是未定的(§2.2.1)。**类的成员也适用此规则**。于是这类成员应在类中被初始化，或为其定义构造器。否则类的实例中此成员的值将是未定义的。

第三个原因是，编译器有时无法合成。例如，一个成员是**类**类型，且那个类没有默认构造器，此时编译器将无法初始化该成员。此时，需要我们自己定义默认构造器。否则类将没有可用的默认构造器。

We’ll see in § 13.1.6 additional circumstances that prevent the compiler from generating an appropriate default constructor.

##### 定义 `Sales_data` 的构造器

为`Sales_data`定义四个构造器：

```cpp
struct Sales_data {
    Sales_data() = default;
    Sales_data(const std::string &s): bookNo(s) { }
    Sales_data(const std::string &s, unsigned n, double p):
        bookNo(s), units_sold(n), revenue(p*n) { }
    Sales_data(std::istream&);

    // other members as before
    std::string isbn() const { return bookNo; }
    Sales_data& combine(const Sales_data&);
    double avg_price() const;
    std::string bookNo;
    unsigned units_sold = 0;
    double revenue = 0.0;
};
```

##### `= default`

先解释默认构造器：

```cpp
	Sales_data() = default;
```

在新标准下，可以用 `= default` 告诉编译器为我们产生默认构造器。`= default` 可以放在类内声明处**或**在类外定义处。与其他函数一样，如果 `= default` 出现在类内，则默认构造器将被内联；如果在类外定义处，成员默认将不会内联。

##### 构造器初始化列表

```cpp
Sales_data(const std::string &s): bookNo(s) { }
Sales_data(const std::string &s, unsigned n, double p):
    bookNo(s), units_sold(n), revenue(p*n) { }
```

冒号之后的是构造器的**初始化列表**：定义一个或多个数据成员的初始值。列表中多个成员逗号分隔，成员的初始值在括号（**或花括号**）内。未在初始化列表中出现的成员的初始化与合成构造器的初始化方式一致。

构造器最好使用类内初始化器。但如果编译器还不支持类内初始化器，则构造器要现实的初始化每个内建类型的成员。

> 最佳实践：构建器不应覆盖类内初始化，除非想用另一个值。

##### 在类外定义构造器

```cpp
Sales_data::Sales_data(std::istream &is)
{
    read(is, *this);
}
```

注意，即使上面的构造器的初始化列表为空，**成员仍会先被初始化，再执行构造器内的代码**。未出现在初始化列表中的成员被**类内初始化**或**默认初始化**。对于 `Sales_data` 意味着，当构造器执行时，`bookNo` 将是空字符串，`units_sold` 和 `revenue` 是0。

#### 7.1.5 拷贝、赋值和销毁

对象在多种情况下可能被拷贝：如初始化一个变量时，向函数传值参时，或函数返回对象时(§6.2.1, §6.3.2)。使用赋值运算符时对象被赋值(§4.4)。如果我们不定义这些函数，编译器将为我们合成。

**一些类不能依赖合成的版本**

尽管编译器会合成拷贝、赋值、析构操作，但有时默认的版本的行为并不正确。特别是当类会在类对象外部分配资源时（第12章）。§13.1.4 会讲到，涉及管理动态内存的类，一般不能依赖合成的默认版本。

值得注意的是，多数需要动态内存的类，可以（并且应该）使用 `vector` 或 `string` 来管理所需内存。

合成拷贝、赋值、析构操作，能够正确处理含有 `vector` 或 `string` 成员的类。当我们拷贝或赋值的一个 `vector` 成员，`vector `类会负责正确的拷贝或赋值。当对象被销毁时，会销毁 `vector` 成员，销毁 `vector` 内的所有成员。

### 7.2 访问控制与封装

```cpp
class Sales_data {
public: // 访问控制
    Sales_data() = default;
    Sales_data(conststd::string &s, unsigned n, double p):
        bookNo(s),units_sold(n), revenue(p*n) { }
    Sales_data(conststd::string &s): bookNo(s) { }
    Sales_data(std::istream&);
    std::string isbn() const { return bookNo; }
    Sales_data& combine(const Sales_data&);
private: // 访问控制
    double avg_price() const {
        return units_sold ? revenue/units_sold : 0;
    }
    std::string ookNo;
    unsigned units_sold = 0;
    double revenue = 0.0;
};
```

类可以包含零个或多个访问说明符。同一个说明符可以出现多次。说明符的影响直到下一个说明符出现为止。

** `class` 和 `struct` 关键字**

`class` 和 `struct` 都能定义类。二者**唯一的区别**是默认的访问级别不同。用 `struct` 则默认是 `public`，用 `class` 默认是 `private`。

#### 7.2.1 友元

现在 `Sales_data` 的数据成员是私有的，于是 `read`,  `print` 将不再能通过编译。

类可以令其他类或函数成为类的友元，获得对非公有成员的访问。要声明一个函数是友元，在类中列出函数的声明，并添加 `friend` 关键字：

```cpp
class Sales_data {
// 友元声明
friend Sales_data add(const Sales_data&, const Sales_data&);
friend std::istream &read(std::istream&, Sales_data&);
friend std::ostream &print(std::ostream&, const Sales_data&);
// other members and access specifiers as before
public:
    Sales_data() = default;
    Sales_data(conststd::string &s, unsigned n, double p):
        bookNo(s),units_sold(n), revenue(p*n) { }
    Sales_data(const std::string &s): bookNo(s) { }
    Sales_data(std::istream&);
    std::string isbn() const { return bookNo; }
    Sales_data& combine(const Sales_data&);
private:
    std::string bookNo;
    unsigned units_sold = 0;
    double revenue = 0.0;
};

// 函数声明
Sales_data add(const Sales_data&, const Sales_data&);
std::istream &read(std::istream&, Sales_data&);
std::ostream &print(std::ostream&, const Sales_data&);
```

友元声明只能放在类定义内。可以出现在类的任意位置。友元不是类的成员，因此不受访问控制节的影响。

友元的声明只声明访问性。**并不是在声明函数**。因此，函数仍需要在某处声明。为了方便类使用者使用友元，一般将友元声明与类定义放在一起。

### 7.3 类的其他特性

#### 7.3.1 类成员再探

##### 类型成员

除了数据成员和函数成员，类的成员还有*类型成员*；即定义类型的别名。且此名字的可访问性遵从一般成员：公有或私有。

```cpp
class Screen {
public:
    typedef std::string::size_type pos;
private:
    pos cursor = 0;
    pos height = 0, width = 0;
    std::string contents;
};
```

除了使用 `typedef` 还可以使用 `using`：

```cpp
class Screen {
public:
    using pos = std::string::size_type;
    // other members as before
};
```

与其他成员不同，数据成员要在使用前定义。因此，数据成员一般在类的最前面。原因在 7.4.1 解释。

##### 令成员内联

定义在类内的成员函数自动是内联的。

可以在类内显式声明函数是内联的。还可以在类外函数定义处指定函数是内联的：

```cpp
inline Screen &Screen::move(pos r, pos c)
{
    pos row = r * width;
    cursor= row + c ;
    return *this;
}
```

可以在声明和定义处都指定函数是内联的。但根本不需要这么做。

> 内联成员函数与内联函数一样，应该定义在头文件，与类定义在一起。

##### 重载成员函数

成员函数一样可以被重载。

##### `mutable` 数据成员

极端情况下，我们想修改一个数据成员，即使在一个常量成员函数中。这种数据成员需要在声明时加 `mutable` 关键字。mutable 数据成员不能是 `const`，即使它是一个常量对象的成员。

例子，`access_ctr` 是 `Screen` 的一个 mutable 成员，用于追踪所有成员函数被调用次数：

```cpp
    class Screen {
    public:
    	void some_member() const;
    private:
    	mutable size_t access_ctr;
    // other members as before
    };
    void Screen::some_member() const
    {
    	++access_ctr;
    	// ...
    }
```

尽管 `some_member` 是一个常量成员函数，它可以改变 `access_ctr` 的值。

##### 类类型的成员的初始化

新标准中，提供默认值**最好的方式是使用类内初始化**(§2.6.1)：

```cpp
class Window_mgr {
    private:
        std::vector<Screen> screens {Screen(24, 80, ' ')};
};
```

类内初始化必须使用 `=` 形式的初始化，或**直接初始化**（使用花括号）。

#### 7.3.2 返回 `*this` 的函数

在常量方法内返回 `*this`，返回类型应是 `const Sales_data &`。

##### 根据 const 重载

我们可以基于成员函数是否常量成员函数重载函数，其原因，与可以根据指针形参指向的是否是常量重载函数的原因(§6.4)相同。非常量的版本不会成为常量对象的可行（viable）函数。对常量对象，只能调用常量成员函数。对于非常量对象，可以调用任意版本，但非常量版本将更好的匹配。

下面将定义一个私有成员函数 `do_display`， `display` 函数将调用该函数，最后返回调用对象本身：

```cpp
class Screen {
public:
    // 根据对象是否是常量重载
    Screen& display(std::ostream &os)
        {do_display(os); return *this; }
    const Screen &display(std::ostream &os) const
        {do_display(os); return *this; }
private:
    void do_display(std::ostream &os) const {
        os << contents;
    }
    // other members as before
};
```

与其他上下文一样，当成员函数调用另一个成员函数时，`this` 指针会被隐式的传递。因此当 `display` 调用 `do_display`，它自己的 `this` 指针会被隐式传递给 `do_display`。当非常量版本的 `display` 调用 `do_display`，它的 `this` 指针会被隐式的从指向非常量的指针转换为指向常量的指针(§4.11.2)。

最后 `display` 会返回对象自身。对于非常量函数，`this` 指向非常量对象，因此函数返回的引用指向非常量。常量成员函数返回的引用指向常量。

当我们调用 `display` 时，调用哪个函数取决于对象是否是常量：

```cpp
Screen myScreen(5,3);
const Screen blank(5, 3);
myScreen.set('#').display(cout); // 调用非常量版本
blank.display(cout); // 调用常量版本
```

#### 7.3.3 类类型

当类型是类时，可以直接使用类名。或者，可以在类名前加 `class` 或 `struct`：

```cpp
Sales_data item1;
class Sales_data item1; // 等价
```

第二种形式是从 C 继承来的。

##### 类声明

就像可以分离函数的声明与定义，可以**声明**一个类，而不是**定义**它：

```cpp
class Screen; // 声明Screen类
```

这种声明，有时被称为前向声明。

声明之后，定义之前，类 `Screen` 是不完全的 —— 其内部成员是未知的。不完整类型的使用是受限的：可以定义此类型的指针或引用类型，可以声明（不是定义）使用此类型做参数或返回值的函数。

创建类的对象前，类必须被定义。否则编译器不知道对象占用多少空间。

若数据成员的类型是类A，则需要类A需要是已定义的（一个例外，见§7.6）。类型必须完整定义的原因是，编译器需要知道此成员占据多少空间。因为类在类正文结束后才算定义完成，因此**类不能定义自己类类型的成员**。但可以定义指向自己类类型的成员的指针或引用：

```cpp
class Link_screen {
    Screen window;
    Link_screen *next;
    Link_screen *prev;
};
```

#### 7.3.4 再探友元

类的友元可以是另一个类。
友元函数可以定义在类正文之内，这样的函数是隐式内联的。

##### 类之间的友元

让`Window_mgr`类成为`Screen`的友元类：

```cpp
class Screen {
    // Window_mgr 的成员可以访问 Screen 的私有成员
    friend class Window_mgr;
    // ... rest of the Screen class
};
````

注意友元不是传递的。例如，`Window_mgr` 的友元对 `Screen` 没有特殊的访问权限。

##### 让成员函数做友元

如果不想让整个 `Window_mgr` 类做友元，只想其 `clear` 函数做友元。可以

```cpp
class Screen {
    // Window_mgr::clear 必须在类 Screen 之前声明
    friend void Window_mgr::clear(ScreenIndex);
    // ... rest of the Screen class
};
```

让成员函数做友元，需要仔细安排程序结构，安排声明和定义的顺序。这里顺序必须是：

- 先定义 `Window_mgr`，声明、但不能定义 `clear`。`Screen` 必须在 `clear` 使用其成员前声明。
- 然后，定义 `Screen`，包含对友元函数 `clear` 的声明。
- 最后，定义 `clear`。

##### 重载函数与友元

一次只能令一个版本成为友元。

```cpp
extern std::ostream& storeOn(std::ostream &, Screen &);
extern BitMap& storeOn(BitMap &, Screen &);
class Screen {
    friend std::ostream& storeOn(std::ostream &, Screen &);
    // . . .
};
```

##### 友元声明与作用域

Classes and nonmember functions need not have been declared before they are used in a friend declaration. When a name first appears in a friend declaration, that name is implicitly assumed to be part of the surrounding scope. 但友元本身并不在这个作用域中(§7.2.1)。即使函数定义在类内部，也必须在类外提供一个函数声明。即使这个友元函数只会被类的成员函数调用，也必须有此声明：

```cpp
struct X {
    friend void f() { /* 友元函数定义在类内部 */}
    X(){ f(); } // 错误：f未声明
    void g();
    void h();
};
void X::g() { return f(); } // 错误：f未被声明
void f(); // 声明在这
void X::h() { return f(); } // 现在可以了！
```

记住友元声明只影响访问能力，本身并不是声明。

> Remember, some compilers do not enforce the lookup rules for friends (§7.2.1).

### 7.4 类作用域

类定义了自己的新的作用域。在类作用域之外，普通数据成员和函数成员需要通过成员访问运算符(§4.6)才能访问。在类外通过作用域运算符访问类型成员。

```cpp
Screen::pos ht = 24, wd = 80; // pos是Screen中定义的类型
Screen scr(ht, wd, ' ');
Screen *p = &scr;
char c = scr.get();
c = p->get();
```

**作用域与在类外定义的成员**

类是一个作用域解释了为什么在类外定义“成员函数”时需要提供类名(§7.1.2)。从类名出现处往后，包括参数列表和函数体，都在类的作用域内。于是可以不加限定符访问其他成员。

例如
```cpp
void Window_mgr::clear(ScreenIndex i)
{
    Screen &s = screens[i];
    s.contents = string(s.height * s.width, ' ');
}
```

其中 `ScreenIndex` 是定义在类 `WindowMgr` 内的一个类型。

但函数返回类型一般在函数名前。于是此类型在类作用域之外。于是返回类型必须指定所属类。

```cpp
class Window_mgr {
public:
    // add a Screen to the window and returns its index
    ScreenIndex addScreen(const Screen&);
    // other members as before
};
// return type is seen before we're in the scope of Window_mgr
Window_mgr::ScreenIndex Window_mgr::addScreen(const Screen &s)
{
    screens.push_back(s);
    returnscreens.size() - 1;
}
```

#### 7.4.1 名字查找与类作用域

目前所写的代码，名字查找（搜索哪个声明匹配对此名字的使用）都比较直接：首先，在使用名字的代码块内查找名字的声明。只考虑使用处之前的声明。如果名字没有找到，搜索嵌套作用域。如果没有声明，程序报错

定义在类内的成员函数中的名字的解析，看似与上述规则不同。However, in this case, appearances are deceiving. Class definitions are processed in two phases:

- 首先编译成员声明
- 整个类可见后才会编译函数正文

> 成员函数定义的处理，发生在编译器处理了类中所有的声明之后。

两阶段处理使得类代码的组织变得简单。由于整个类可见后才会编译函数正文，于是可以使用定义在类中的任意名字。{{比如可以出现成员函数在前，成员变量在后}}

##### 类成员声明的名字查找

两阶段的处理只适用于在成员函数正文内使用的名字。而声明中使用的名字，包括用作返回类型的名字，参数列表中的名字，必须在使用前可见。若某个成员声明用到了尚未在类中见到过的名字，编译器将在类定义所在的作用于寻找该名字。例子：

```cpp
typedef double Money;
string bal;
class Account {
public:
    Money balance() { return bal; }
private:
    Money bal;
    //...
};
```

当编译器遇到 `balance` 函数声明时，它将在 `Account` 中寻找 `Money` 的声明。但编译器只会考虑 `Account` 中出现在使用 `Money` 之前的声明。因为找不到，于是寻找外层作用域。与此不同的是，`balance` 的函数体在整个类可见后才会被处理。此时虽然 `bal` 声明在后面，但仍是可见的。

##### 类型名有特殊性

一般来说，内层作用域可以重新定义外层作用域的名字，即使在内层作用域内，该名字已被用过。但在类中，如果成员使用了外层作用域中的一个名字，且这个名字是一个类型名，则类不能在后面重新定义这个名字：

```cpp
typedef double Money;
class Account {
public:
    Money balance() { return bal; } // 使用外层作用域的Money
private:
    typedef double Money; // 错误：不能重新定义Money
    Money bal;
//...
};
```

Although it is an error to redefine a type name, compilers are not required to diagnose this error. Some compilers will quietly accept such code, even though the program is in error.

> 类型名定义最好在类开始。这样使用这个类型的任何成员将能看到这个类型名。

##### 在成员定义中的普通块级名字查找

在成员函数体内使用的名字的解析方式如下：

- 首先在成员函数内寻找名字的声明。与一般情况一样，只有在使用前声明的名字可用。
- 如果未找到，则在类内找声明。类的所有成员都会被考虑。
- 若未周到，在成员函数定义之前的作用域找。

一般情况下，不要使用成员的名字做“成员函数参数”的名字。但下面为了展示名字的解析过程，我们破坏这条规则：

```cpp
int height; // defines a name subsequently used inside Screen
class Screen {
public:
    typedef std::string::size_type pos;
    void dummy_fcn(pos height) {
        cursor = width * height; // which height? the parameter
    }
private:
    pos cursor = 0;
    pos height = 0, width = 0;
};
```

`dummy_fcn` 内的 `height` 是参数列表中的 `height`。若想访问数据成员 `height`，可以：

```cpp
void Screen::dummy_fcn(pos height) {
    cursor= width * this->height; // member height
    // 等价方式
    cursor= width * Screen::height; // member height
}
```

##### 在类作用域后，查找外围作用域

如果在函数内和类内都找不到名字，则在外层作用域中寻找。

若想使用外部作用域的名字，可以显式使用作用域运算符：

```cpp
void Screen::dummy_fcn(pos height) {
    cursor = width * ::height; // which height? the global one
}
```

##### Names Are Resolved Where They Appear within a File

当成员定义在类外，名字查找的第三步包括 成员定义的作用域中的名字，及类定义的作用域的名字。例如：

```cpp
int height; // defines a name subsequently used inside Screen
class Screen {
public:
    typedef std::string::size_type pos;
    void setHeight(pos);
    pos height = 0; // hides the declaration of height in the outer scope
};

Screen::pos verify(Screen::pos);

void Screen::setHeight(pos var) {
    // height: refers to the class member
    // verify: refers to the global function
    height = verify(var);
}
```

注意到全局函数 `verify` 位于类 `Screen` 定义之后。但 `verify` 位于 `setHeight` 定义之前，因此可用。

### 7.5 构造器再探

#### 7.5.1 构造器的初始化列表

定义变量时，我们一般会直接初始化它们；而不是先定义再初始化。

初始化跟赋值的区别，在对象数据成员上也一样。如果我们不在初始化列表中显式初始化成员，则在构造器体执行前，数据成员会被默认初始化。例如：

```cpp
// 不好的写法
Sales_data::Sales_data(const string &s, unsignedcnt, double price)
{
    bookNo = s;
    units_sold = cnt;
    revenue = cnt * price;
}
```

这里，是通过赋值给成员值，而不是初始化。

##### 构造器初始化{{列表}}有时是需要的

常常，但不总是可以忽略初始化成员和赋值的区别。常量成员和引用成员必须被初始化。类类型的成员，且类没有默认构造器的，也必须被初始化。例如：

```cpp
class ConstRef{
public:
    ConstRef(int ii);
private:
    int i;
    const int ci;
    int& ri;
};
```

`ci` 和 `ri` 必须被初始化。因此省略构造器的初始化列表是错误的：

```cpp
// 错误！
ConstRef::ConstRef(int ii)
{  // 下面是赋值，不是初始化！
    i = ii; // ok
    ci = ii; // 错误：不能向常量赋值
    ri = i; // 错误：ri还未曾被初始化
}
```

当构造器体开始执行后，初始化已经完成。初始化常量和引用唯一可能的地方是构造器**初始化器**。正确的写法是：

```cpp
ConstRef::ConstRef(int ii): i(ii), ci(ii), ri(i) {  }
```

除了对错问题，还有效率问题。赋值多了一步。

##### 成员初始化顺序

每个成员只能在构造器初始化器中出现一次。
初始化列表只是给定初始化值，但未规定初始化执行顺序。成员初始化的顺序由它们在类内定义的顺序决定。初始化列表顺序无关紧要。

少数情况下，初始化顺序是重要，例如：

```cpp
class X{
    int i;
    int j;
public:
    // undefined: i is initialized before j
    X(int val): j(val), i(j) { }
};
```

一些编译器，当发现构造器初始化列表中数据成员的顺序与成员声明时的顺序不一致时，会提示一个警告。

> 最佳实践：构造器初始化列表的顺序最好与成员定义的顺序一致。最好不要用一个成员初始化另一个。

##### 默认实参与构造器

`Sales_data` 的默认构造器，与只取一个 `string` 参数的构造器可以合并成一个 —— 利用默认实参：

```cpp
class Sales_data {
public:
    Sales_data(std::strings = ""): bookNo(s) { }
    // 剩下的构造器如前
    Sales_data(std::strings, unsigned cnt, double rev):
        bookNo(s),units_sold(cnt), revenue(rev*cnt) { }
    Sales_data(std::istream&is) { read(is, *this); }
    // remaining members as before
};
```

这样的类**仍算有默认构造器**。若一个构造器每个参数都有默认实参，则该构造器也算默认构造器。

#### 7.5.2 代理构造器

新标准扩展了构造器初始化，允许使用代理构造器（delegating constructors）。代理构造器利用同一个类内的其他构造器完成初始化。

与其他构造器一样，代理构造器有一个成员初始化列表和一个函数体。它的成员初始化列表只有一项：类名，接着是括号内的一组参数。实参列表必须匹配类内的另一个构造器。例子：

```cpp
class Sales_data{
public:
    // 非代理构造器
    Sales_data(std::strings, unsigned cnt, double price):
        bookNo(s), units_sold(cnt), revenue(cnt * price) {
    }
    // 下面都是代理构造器
    Sales_data(): Sales_data("", 0, 0) {}
    Sales_data(std::string s): Sales_data(s, 0, 0) {}
    Sales_data(std::istream& is): Sales_data() {read(is, *this); }
    // other members as before
};
```

取 `istream&` 的构造器，委托给默认构造器，默认构造器再委托到三个参数的构造器。

若构造器A代理到构造器B，则A的**函数体**要执行完，才能轮到B的函数体执行。

#### 7.5.3 默认构造器的角色

当对象需要被**默认初始化**或**值初始化**时，默认构造器会被自动使用。默认初始化发生在：

- 在块级作用域内定义非静态变量或数组，但没有初始化器
- 数据成员是类类型，且使用合成的默认构造器
- 数据成员是类类型，但未在构造器初始化列表中显式初始化

值初始化发生在：

- 数组初始化时，我们提供的初始化器数量少于数组大小(§ 3.5.1)
- 定义局部静态对象，但未提供初始化器(§ 6.1.1)
- 当我们**显式请求值初始化**时，即使用 `T()` 的形式。这里 `T` 是一个类型（`vector` 的构造器需要一个参数，指定 `vector` 大小）。

若数据成员的类，没有默认构造器，可能导致一些错误，如：

```cpp
class NoDefault {
public:
    NoDefault(const std::string&);
    // additional members follow, but no other constructors
};
struct A {
    NoDefault my_mem;
};
A a; // 错误：无法合成A的默认构造器
struct B {
    B(){} // 错误：未定义b_member的初始化
    NoDefault b_member;
};
```

##### 使用默认构造器

下面这段代码会报错，编译器认为 `obj` 是一个函数。而我们实际想做的是实例化一个对象。

Sales_data obj();  // ok: but defines a function, not an object
if (obj.isbn() == Primer_5th_ed.isbn())  // error: obj is a function

因为编译器会把 `Sales_data obj();` 当成一个函数声明：没有参数，返回一个 `Sales_data`， 名叫 `obj` 的函数。

若想使用默认初始器，正确的写法是去掉括号：`Sales_data obj;`。

#### 7.5.4 隐式类类型的转换

§4.11 讲过，语言定义了几种内建类型的自动转换。类也可以定义隐式转换。若构造器只有一个实参，则这个构造器定义了一种从实参类型到类的隐式转换。这类构造器有时被称为**转换构造器**。We’ll see in §14.9 how to define conversions from a class type to another type.

`Sales_data` 有两个这样的构造器。分别取 string 和 istream 做参数。即，在期待`Sales_data` 的地方，我们可以提供一个 string 或 istream：

```cpp
string null_book = "9-999-99999-9";
// constructs a temporary Sales_data object
item.combine(null_book);
```

`combine` 方法本来接收 `Sales_data` 参数。编译器自动从 string 创建一个 `Sales_data`。Because combine’s parameter is a reference to const, we can pass a temporary to that parameter.

**类类型的转换只允许做一次**

In §4.11.2 we noted that the compiler will automatically apply only one class-type conversion. 例如下面代码错误的原因是需要两次转换：


```cpp
// (1) convert "9-999-99999-9" to string
// (2) convert that (temporary) string to Sales_data
item.combine("9-999-99999-9");
```

我们可以显式的将字符字面量转换为 `string` 或 `Sales_data`：

```cpp
item.combine(string("9-999-99999-9"));
item.combine(Sales_data("9-999-99999-9"));
```

**阻止构造器产生隐式转换**

对构造器施加 `explicit`，则构造器将不能再用于隐式转换。

```cpp
class Sales_data{
public:
    Sales_data() = default;
    Sales_data(const std::string &s, unsigned n, double p):
        bookNo(s), units_sold(n), revenue(p*n) { }
    explicit Sales_data(const std::string &s): bookNo(s) { }
    explicit Sales_data(std::istream&);
    // remaining members as before
};
```

```cpp
item.combine(null_book); // error: string constructor is explicit
item.combine(cin); // error: istream constructor is explicit
```

`explicit`关键字用在只有一个参数的构造器才有意义。
`explicit`关键字只出现在类中的构造器声明处。不能在类外的定义处：

```cpp
// error: explicit allowed only on a constructor declaration
explicit Sales_data::Sales_data(istream& is)
{
    read(is,*this);
}
```

隐式转换的一种形式是“拷贝形式的初始化”（通过 `=`）(§ 3.2.1)。 explicit 构造器不能用于这种形式的初始化；只能使用**直接初始化**：

```cpp
Sales_data item1(null_book); // ok: 直接初始化
// error
Sales_data item2 = null_book;
```

**显式使用构造器做转换**

```cpp
    // ok: the argument is an explicitly constructed Sales_data object
    item.combine(Sales_data(null_book));
    // ok: static_cast 可以使用 explicit 构造器！
    item.combine(static_cast<Sales_data>(cin));
````

**库类中使用 explicit 构造器的例子**

string 构造器中有一个取单个参数 `const char*`，且不是 explicit。

但 `vector` 构造器中，取一个大小值的的构造器是 explicit。

#### 7.5.5 聚合（Aggregate）类

聚合类可以直接访问其成员，且具有特殊的初始化语义。聚合需满足：

- 所有的数据成员是公有的
- 不定义任何构造器
- 无类内初始化(§2.6.1)
- 没有基类或虚函数

例如：

```cpp
struct Data{
    int ival;
    string s;
};
```

We can initialize the data members of an aggregate class by providing a braced list of member initializers:

```cpp
// val1.ival= 0; val1.s = string("Anna")
Data val1 = {0, "Anna" };
```

初始器的顺序与数据成员出现顺序相同。

与数组的初始器一样，如果初始器列表中元素个数少于类的成员个数，后面的成员将被值初始化（§3.5.1）。初始化列表中的元素个数不能多于类成员的个数。

It is worth noting that there are three significant drawbacks to explicitly initializing the members of an object of class type:

- 要求所有数据成员公开。
- 合理初始化类成员的负担交给了调用者。
- 若添加或删除了成员，所有的初始化都要更新。

#### 7.5.6 字面量类

§6.5.2 降到 `constexpr` 函数的参数类型和返回值类型都必须是字面量类型。除了数值类型、引用、指针，某些类也是字面量类型。与其他类不同，字面量类含有 `constexpr` 数据成员。这些成员必须满足 constexpr 函数的所有要求。These member functions are implicitly const(§7.1.2).

所有数据成员都是字面量类型的聚集类（§7.5.5）是一个字面量类。A nonaggregate class, that meets the following restrictions, is also a literal class:

- The data members all must have literal type.
- The class must have at least one constexpr constructor.
- If a data member has an in-class initializer, the initializer for a member of builtin type must be a constant expression (§ 2.4.4), or if the member has class type, the initializer must use the member’s own constexpr constructor.
- The class must use default definition for its destructor, which is the member that destroys objects of the class type (§ 7.1.5).

##### `constexpr` Constructors

Although constructors can’t be const(§ 7.1.4, p. 262), constructors in a literal class can be constexpr(§ 6.5.2, p. 239) functions. Indeed, a literal class must provide at least one constexpr constructor.

A constexpr constructor can be declared as = default(§ 7.1.4) (or as a deleted function, which we cover in § 13.1.6). Otherwise, a constexpr constructor must meet the requirements of a constructor—meaning it can have no return statement—and of a constexpr function—meaning the only executable statement it can have is a returnstatement (§ 6.5.2). As a result, the body of a constexpr constructor is typically empty. We define a constexpr constructor by preceding its declaration with the keyword constexpr:

```
class Debug{
public:
	constexpr Debug(bool b = true): hw(b), io(b), other(b) {
	}
	constexpr Debug(bool h, bool i, bool o): hw(h),io(i), other(o) {
	}
	constexpr bool any() { return hw || io || other; }
    voidset_io(bool b) { io = b; }
    voidset_hw(bool b) { hw = b; }
    voidset_other(bool b) { hw = b; }
private:
    boolhw; // hardware errors other than IO errors
    boolio; // IO errors
    boolother; // other errors
};
```

A constexpr constructor must initialize every data member. The initializers must either use a constexpr constructor or be a constant expression. A constexpr constructor is used to generate objects that are constexpr and for parameters or return types in constexpr functions:

```
constexpr Debug io_sub(false, true, false); // debugging IO
if (io_sub.any()) // equivalent to if(true)
	cerr << "print appropriate error messages" << endl;
constexpr Debug prod(false); // no debugging during production
if (prod.any()) // equivalent to if(false)
	cerr << "print an error message" << endl;
```

### 7.6 静态类成员

##### 声明静态成员

向声明添加 `static` 关键字，则成员附属于类。静态成员也可以公有、私有。静态数据成员可以是常量、引用、数组、类类型等。

```cpp
class Account {
public:
    void calculate() { amount += amount * interestRate; }
    static double rate() { return interestRate; }
    static void rate(double);
private:
    std::string owner;
    double amount;
    static double interestRate;
    static double initRate();
};
```

静态成员函数不与任何对象绑定，没有 `this` 指针。因此静态成员函数不能不能声明为 `const`。不能在其中使用 `this`，包括隐式的是使用 —— 访问非静态成员。

##### 使用类静态成员

可以通过作用域运算符访问静态成员：

```cpp
double r;
r = Account::rate();
```

尽管静态成员不属于类的任何特定对象，但可以用对象、应用、指针去访问它：

```cpp
Account ac1;
Account *ac2 = &ac1;
r = ac1.rate();
r = ac2->rate();
```

成员函数可以直接使用静态成员，不需要作用域运算符。

```cpp
class Account{
public:
    void calculate() { amount += amount * interestRate; }
private:
    static double interestRate;
    // remaining members as before
};
```

##### 定义静态成员

与其他成员函数一样，静态成员函数的定义可以在类内或类外。在类外定义时不要重复 `static` 关键字。`static` 只出现在声明处：

```cpp
void Account::rate(double newRate)
{
    interestRate = newRate;
}
```

静态成员不在构造器中初始化，也不在类内初始化，**定义和初始化要放在类外**！与其他对象一样，静态数据成员只能被定义一次。

与全局对象一样，静态数据成员定义在所有函数之外。因此一旦定义，它们存在直到程序结束。

```cpp
// define and initialize a static class member
double Account::interestRate = initRate();
```
看到类名后，后续代码就在类的作用域之中，因此可以调用类的静态函数 `initRate`。

> 最好将静态数据成员的定义，与类的非内联成员函数的定义放在同一个文件。

##### 静态数据成员的类内初始化

一般，类的静态成员不能在类正文内初始化。但可以为常量整型的静态成员提供类内初始化；以及必须为字面量类型(§7.5.6)的 `constexpr` 提供类内初始化。For example, we can use an initialized static data member to specify the dimension of an array member:

```cpp
class Account{
public:
    static double rate() { return interestRate; }
    static void rate(double);
private:
    static constexpr int period = 30;// period is a constant expression
    double daily_tbl[period];
};
````

If the member is used only in contexts where the compiler can substitute the member’s value, then an initialized const or constexpr static need not be separately defined. However, if we use the member in a context in which the value cannot be substituted, then there must be a definition for that member.

For example, if the only use we make of `period` is to define the dimension of `daily_tbl`, there is no need to define `period` outside of Account. However, if we omit the definition, it is possible that even seemingly trivial changes to the program might cause the program to fail to compile because of the missing definition. For example, if we pass `Account::period` to a function that takes a `const int&`, then `period` must be defined.

If an initializer is provided inside the class, the member’s definition must not specify an initial value:

```cpp
// definition of a static member with no initializer
constexpr int Account::period; // initializer provided in the class
definition
```

> 最佳实践：Even if a `const static` data member is initialized in the class body, that member ordinarily should be defined outside the class definition.

##### static成员的使用方式比普通成员

As one example, a static data member can have incomplete type (§7.3.3). 特别的，静态数据成员的类型可以是所属类。

```cpp
class Bar{
public:
//...
private:
    static Bar mem1; // ok: static member can have incomplete type
    Bar* mem2;  // ok: pointer member can have incomplete type
    Bar mem3;  // error: data members must have complete type
};
```

Another difference between static and ordinary members is that we can use a static member as a default argument (§ 6.5.1):

```cpp
class Screen{
public:
    // bkground refers to the static member
    // declared later in the class definition
    Screen&clear(char = bkground);
private:
    static const char bkground;
};
```

A non static data member may not be used as a default argument because its value is part of the object of which it is a member. Using a non static data member as a default argument provides no object from which to obtain the member’s value and so is an error.















