本书讲的非常细致，非常简单基础的内容也充分介绍了。于是有点冗长，不精练，不够归纳。有些内容讲的很分散。

[toc]

## 4. 字符串、格式化输入输出

The talkback.c Program

    // talkback.c -- nosy, informative program
    #include <stdio.h>
    #include <string.h> // for strlen() prototype 
    #define DENSITY 62.4 // human density in lbs per cu ft 
    int main()
    {
        float weight, volume;
        int size, letters;
        char name[40];
        // name is an array of 40 chars
        printf("Hi! What's your first name?\n");
        scanf("%s", name);
        printf("%s, what'syour weight in pounds?\n", name);
        scanf("%f", &weight);
        size = sizeof name;
        letters = strlen(name);
        volume = weight / DENSITY;
        printf("Well, %s, your volume is %2.2f cubic feet.\n", name, volume);
        printf("Also, your first name has %d letters,\n", letters);
        printf("and we have %d bytes to store it.\n", size);
        return 0;
    }

运行：

    Hi! What's your first name?
    Christine
    Christine, what's your weight in pounds?
    154
    Well, Christine, your volume is 2.47 cubic feet. Also, your first name has 9 letters,
    and we have 40 bytes to store it.

### 字符串：介绍

例子：

	"Zing went the strings of my heart!"

C没有特别的字符串类型。字符串存储在`char`数组中。最后有一个`\0`表示结束。

简单例子：

```c
    char name[40];
    printf("What's your name? ");
    scanf("%s", name);
    printf("Hello, %s. %s\n", name, PRAISE);
```

`scanf()`读取输入，遇到空白符（空格、换行等）停止。

`strlen()`返回字符串中有几个字符。 若使用ANSI C之前的编译器，需要移除下面行：

	#include <string.h>

**string.h**包含字符串相关函数的函数原型，包括`strlen()`。Chapter 11, “Character Strings and String Functions,” discusses this header file more fully. (By the way, some pre-ANSI Unix systems use **strings.h** instead of **string.h** to contain declarations for string functions.)

As mentioned in Chapter 3, “Data and C,” the C99 and C11 standards use a `%zd` specifier for the type used by the `sizeof` operator. This also applies for type returned by `strlen()`. For earlier versions of C you need to know the actual type returned by `sizeof` and `strlen()`; typically that would be `unsigned` or `unsigned long`.

### 常量与C预处理器

用C预处理器定义常量，例如：

	#define TAXRATE 0.015

程序编译时，所有出现TAXRATE的地方都会被0.015替换，这称为编译时替换。通过这种方式定义的常量常被称作**manifest constants**。

C90新增了一种创建符号常量的方式：利用`const`关键字：

	const int MONTHS = 12; // MONTHS a symbolic constant for 12

这种方式比`#define`更灵活；如可以指定常量类型，可以控制常量的作用域、可见性。

Actually, C has yet a third way to create symbolic constants, and that is the enum facility discussed in Chapter 14, “Structures and Other Data Forms.”

C的头文件**limits.h**和**float.h**列出了整数类和浮点类类型的大小范围。例如**limits.h**包含以下内容：

	#define INT_MAX +32767
    #define INT_MIN -32768

若你的系统使用32位整数，上述文件会不同。

	printf("Maximum int value on this system = %d\n", INT_MAX);

### （未）printf() 和 scanf()

## 5. 运算符、表达式和语句

### 基础运算符

C90添加了一元`+`运算符。

浮点数除法得到浮点数。整数除法产生整数——即会截断。

Until the C99 standard, C gave language implementers some leeway in deciding how integer division with negative numbers worked. One could take the view that the rounding procedure consists of finding the largest integer smaller than or equal to the floating-point number. Certainly, 3 fits that description when compared to 3.8. But what about −3.8? The largest integer method would suggest rounding to −4 because −4 is less than −3.8. But another way of looking at the rounding process is that it just dumps the fractional part; that interpretation, called truncating toward zero, suggests converting −3.8 to −3. Before C99, some implementations used one approach, some the other. C99规定向0截断，于是`−3.8`截断为`−3`。

### 其他运算符

#### `sizeof`运算符和`size_t`类型

如果操作数是一个类型，必须加括号，如`sizeof(char)`。其他情况括号是可选的。

    #include <stdio.h>
    int main(void)
    {
        int n = 0;
        size_t intsize;
        intsize = sizeof (int);
        printf("n = %d, n has %zd bytes; all ints have %zd bytes.\n",
	        n, sizeof n, intsize );
        return 0;
    }

`sizeof`的返回值是`size_t`。他不是全新的类型，只不过是某种无符号整数。

C99 goes a step further and supplies `%zd` as a `printf()` specifier for displaying a `size_t` value. If your system doesn’t implement `%zd`, you can try using `%u` or `%lu` instead.

#### 取模

What about negative numbers? Before C99 settled on the “truncate toward zero” rule for integer division, there were a couple of possibilities. But with the rule in place, you get a nega- tive modulus value if the first operand is negative, and you get a positive modulus otherwise.

### 类型转换

1. 当`char`和`short`出现的表达式中，不论`signed`还是`unsigned`都自动被转换为`int`或者`unsigned int`。（若`short`与`int`大小相同，`unsigned short`比`int`大，因此`unsigned short`需要转换为`unsigned int`）。Under K&R C, but not under current C, `float` is automatically converted to `double`. 这些操作称为提升（promotions）。
2. 若操作涉及两种不同的类型，级别低的值要被转换为级别高的值。
3. 类型的级别，从高到低是：`long double`, `double`, `float`, `unsigned long long`, `long long`, `unsigned long`, `long`, `unsigned int`, `int`。若`long`和`int`等大，则`unsigned int`比`long`级别高。The `short` and `char` types don’t appear in this list because they would have been already promoted to `int` or perhaps `unsigned int`.
4. 在赋值语句中，最终结果转换为变量的类型。可能发生提升，或demotion。
5. 作为函数参数传递时，`char`和`short`会被转换为`int`。`float`会被转换为`double`。This automatic promotion is overridden by function prototyping, as discussed in Chapter 9, “Functions.”

When floating types are demoted to integer types, they are truncated, or rounded toward zero.

强制类型转换。The parentheses and type name together constitute a cast operator.

	mice = (int) 1.6 + (int) 1.7;

### 函数参数

若有原型，原型指导可能的强制类型转换。

	void pound(int n)

ANSI C之前C使用函数声明而不是原型；声明只包含返回值、函数名，不包括参数类型。为向后兼容，C依然允许下面的形式：

	void pound(); /* pre-ANSI function declaration */

调用`pound(f)`时若`f`是`float`会有问题。因为若无原型，`float`自动提升为`double`。程序仍能运行，但可能有问题。You could fix it by using an explicit type cast in the function call:

	pound ((int) f); // force correct type

## 6. 循环

    while (expression)
    	statement

	for (initialize ; test ; update)
    	statement

	do
		statement
	while ( expression );

在C中，true表达式的值是1，false表达式的值是0。反过来，所有非0的值都被当做true。0被当做false。

	int true_val = (10 > 2);

其中`true_val`的值是1。

表示真假的变量传统上用`int`表示。C99添加了`_Bool`类型。`_Bool`变量只能有1和0两个值。若给`_Bool`赋一个非零值，变量将被设为1。

    _Bool input_is_good;
    while (input_is_good)
    {
        // ...
        input_is_good = (scanf("%ld", &num) == 1);
    }

C99还提供了**stdbool.h**头文件。该文件为`_Bool`提供了一个别名`bool`，定义了`true`和`false`作为1和0的符号常量。

逗号表达式将两个表达式链接为一个，并保证最左的表达式先被求值。它一般用于for循环的控制表达式。逗号表达式的值是最右表达式的值。

	for (step = 2, fargo = 0; fargo < 1000; step *= 2)
    	fargo += step;

## 7. 循环：分支与跳转

The `getchar()` function takes no arguments, and it returns the next character from input.

	ch = getchar();

This statement has the same effect as the following statement:

	scanf("%c", &ch);

The `putchar()` function prints its argument.

	putchar(ch);

This statement has the same effect as the following:

	printf("%c", ch);

Both functions are typically defined in the **stdio.h** file. (Also, typically, they are preprocessor macros rather than true functions)


Fortunately, C has a standard set of functions for analyzing characters; the **ctype.h** header file contains the prototypes. These functions take a character as an argument and return nonzero (true) if the character belongs to a particular category and zero (false) otherwise. For example, the `isalpha()` function returns a nonzero value if its argument is a letter.

    #include <ctype.h> // for isalpha()

	// ...
    while ((ch = getchar()) != '\n') {
        if (isalpha(ch))
        	putchar(ch + 1);
        else
        	putchar(ch);
		// ...


C was developed in the United States on systems using the standard U.S. keyboards. But in the wider world, not all keyboards have the same symbols as U.S. keyboards do. Therefore, the C99 standard added alternative spellings for the logical operators. They are defined in the **iso646.h** header file. If you use this header file, you can use `and` instead of `&&`, `or` instead of `||`, and `not` instead of `!`. For example, you can rewrite

	if (ch != '"' && ch != '\'') charcount++;

this way:

	if (ch != '"' and ch != '\'') charcount++;


三元运算符：

	expression1 ? expression2 : expression3

The `continue` and `break` statements enable you to skip part of a loop or even terminate it, depending on tests made in the body of the loop.


`switch`的测试表达式只能是一个整数值（或`char`类型）。The `case` labels must be integer-type (including `char`) constants or integer constant expressions (expressions containing only integer constants). You can’t use a variable for a case label. Here, then, is the structure of a switch:

    switch (integer expression) {
        case constant1:
        	statements
        case constant2:
        	statements
        default :
        	statements
    }

### （未）The goto Statement

## （未）8. 字符输入输出、输入校验


## （未）11. Character Strings and String Functions


## （未）13. 文件输入输出

## 14. 结构与其他数据格式

### 声明结构

结构声明形如：

    struct book {
    	char title[MAXTITL];
        char author[MAXAUTL];
        float value;
    };


It declares `library` to be a structure variable using the book structure design.

	struct book library;

最后的括号后面的分号结束结构的定义。结构声明可以放在函数外，也可以放在函数内。放在函数内，则只能在函数内使用。放在函数外，可以在文件后面所有函数内使用。

### 定义一个结构变量

In declaring a structure variable, `struct` book plays the same role that `int` or `float` does in simpler declarations. For example, you could declare two variables of the struct book type or even a pointer to that kind of structure:

	struct book doyle, panshin, * ptbook;

As far as the computer is concerned, the declaration

    struct book library;

is short for

	struct book {
    	char title[MAXTITL];
        char author[AXAUTL];
        float value;
    } library; /* follow declaration with variable name */

结构名可以省略：

    struct { /* no tag */
    	char title[MAXTITL];
        char author[MAXAUTL];
        float value;
    } library;

To initialize a structure (any storage class for ANSI C and later, but excluding automatic variables for pre-ANSI C), you use a syntax similar to that used for arrays:

    struct book library = {
        "The Pious Pirate and the Devious Damsel",
        "Renee Vivotte",
        1.95
    };

Chapter 12, “Storage Classes, Linkage, and Memory Management,” mentioned that if you initialize a variable with static storage duration (such as static external linkage, static internal linkage, or static with no linkage), you have to use constant values. This applies to structures, too.

使用`.`访问结构成员。For example, `library.value` is the `value` portion of library.

	scanf("%f", &library.value);

The dot has higher precedence than the `&` here, so the expression is the same as `&(library.float)`.

C99 and C11 provide *designated* initializers for structures. For example, to initialize just the value member of a book structure, you would do this:

	struct book surprise = { .value = 10.99};

You can use designated initializers in any order:

    struct book gift = { .value = 25.99,
        .author = "James Broadfool",
        .title = "Rue for the Toad"};

Just as with arrays, a regular initializer following a designated initializer provides a value for the member following the designated member. Also, the last value supplied for a particular member is the value it gets. For example, consider this declaration:

    struct book gift= { .value = 18.90,
        .author = "Philionna Pestle",
        0.25};

### 结构数组

The `manybook.c` program uses an array of 100 structures. Because the array is an auto- matic storage class object, the information is typically placed on the stack. Such a large array requires a good-sized chunk of memory, which can cause problems. If you get a runtime error, perhaps complaining about the stack size or stack overflow, your compiler probably uses a default size for the stack that is too small for this example. To fix things, you can use the com- piler options to set the stack size to 10,000 to accommodate the array of structures, or you can make the array static or external (so that it isn’t placed in the stack), or you can reduce the array size to 16. Why didn’t we just make the stack small to begin with? Because you should know about the potential stack size problem so that you can cope with it if you run into it on your own.

声明结构数组：

	struct book library[MAXBKS];

访问成员，如`library[0].value`。

### 嵌套结构

    struct names {
		char first[LEN];
        char last[LEN];
    };
	struct guy {
    	struct names handle;
        char favfood[LEN];
        char job[LEN];
        float income;
    };

    struct guy fellow = {
    	{ "Ewen", "Villard" },
        "grilled salmon",
        "personality coach",
    	68112.00
    };

### 指向结构的指针

    struct names {
    	char first[LEN];
        char last[LEN];
    };
    struct guy {
    	struct names handle;
        char favfood[LEN];
        char job[LEN];
        float income;
    };
    struct guy fellow[2] = {
    	{{ "Ewen", "Villard"},
    		"grilled salmon", "personality coach", 68112.00
    	},
        {{"Rodney", "Swillbelly"},
        	"tripe", "tabloid editor", 232400.00
        }
    };
    struct guy * him; /* here is a pointer to a structure */

    him = &fellow[0];

    printf("him->income is $%.2f: (*him).income is $%.2f\n", him->income, (*him).income);
    him++;
    printf("him->favfood is %s: him->handle.last is %s\n", him->favfood, him->handle.last);

Declaration is as easy as can be:

	struct guy * him;

使用指针访问成员有两种方法。第一种方法，比较常见，使用`->`运算符。`him->income` is `barney.income` if `him == &barney`. 第二种方法，形如：`fellow[0].income == (*him).income`。The parentheses are required because the `.` operator has higher precedence than `*`.


In summary, if `him` is a pointer to a type `guy` structure named barney, the following are all equivalent:

	barney.income == (*him).income == him->income // assuming him == &barney

### 函数与结构

之前C实现不允许结构用于函数参数。ANSI C允许。

传结构指针：

    struct funds {
    	char bank[FUNDLEN];
        double bankfund;
        char save[FUNDLEN];
        double savefund;
    };

    double sum(const struct funds * money)
    {
    	return(money->bankfund + money->savefund);
    }

结构直接做参数：


    double sum(struct funds moolah); /* argument is a structure */

此时，形参是实参的拷贝。

Modern C allows you to assign one structure to another, something you can’t do with arrays. That is, if `n_data` and `o_data` are both structures of the same type, you can do the following:

	o_data = n_data; // assigning one structure to another

This works even if a member happens to be an array. Also, you can initialize one struc ture to another of the same type:

    struct names right_field = {"Ruthie", "George"};
    struct names captain = right_field; // initialize a structure to another

Under modern C, including ANSI C, not only can structures be passed as function arguments, they can be returned as function return values. 而利用指针可以实现双向数据交换。

#### 字符数组与字符指针

使用字符数组一般没有问题：创建结构变量时，会分配存储字符数组的空间。但使用字符指针容易出问题，如：

    #define LEN 20
    struct names {
        char first[LEN];
        char last[LEN];
    };

    struct pnames {
        char * first;
        char * last;
    };

    struct pnames attorney;
    scanf("%s", attorney.last); /* here lies the danger */

读取字符串时，`attorney.last`是一个未被初始化的指针，因此读入的字符串存放的位置是一个非法位置，可能导致程序崩溃。

#### 结构，指针和`malloc()`

使用`malloc()`的好处是为字符串分配恰好的大小。方法是先读入一个临时数组。

    void getinfo (struct namect * pst) {
    	char temp[SLEN];
    	printf("Please enter your first name.\n");
        s_gets(temp, SLEN);
    	// allocate memory to hold name
    	pst->fname = (char *) malloc(strlen(temp) + 1);
        strcpy(pst->fname, temp);
        ...

#### （未）Compound Literals and Structures (C99)

#### （未）Flexible Array Members (C99)

#### （未）Anonymous Structures (C11)

### （未）Saving the Structure Contents in a File

### （未）Unions: A Quick

### Enumerated Types













