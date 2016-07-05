[toc]

修改了一个头文件，可能有两个问题：1、忘记重新编译依赖该头文件的某个源文件；2、不依赖该头文件的源文件，不必编译，否则浪费时间。**make** 可以解决这两个问题：确保受修改影响的文件被重新编译。

**make** 不仅可以用于编译程序。它可以用于任何从输入文件产生输出文件的地方。

通过一个文件 **makefile**，告诉 **make** 应用的结构。

makefile 一般与源代码放在一个目录。一个项目可以有多个 makefile。若工程很大，可能需要几个 makefile 管理工程的不同部分。

**make** 不仅用于编译代码，还可以用于准备 man 及安装应用。

### Makefiles 语法

makefile 由一组依赖及规则构成。依赖包含一个目标（要创建的文件）和目标依赖的一些源代码。规则描述如何根据依赖的文件创建目标。目标一般是单个可执行的文件。

make 读取 makefile，发现需要构建的目标（可能有多个），比较源文件的日期时间，判定调用哪个规则来构建目标。一般来说，最终目标之前需要先构建一些中间目标。make 利用 makefile 来决定目标的构建顺序、规则的调用顺序。

单行注释，以 `#` 开头。

### make 的选项和参数

**make** 命令自身有几个选项。三个最常见的是：

- `-k`，make 默认遇到错误立即停止。该选项让 make 遇到错误后仍然执行。
- `-n`，让 make 打印出它打算做什么，但不实际执行。
- `-f <filename>`，指定 makefile。若不指定，**标准** make 先从当前目录中寻找名叫 makefile 的文件。若不存在，找 Makefile。但如果使用 **GNU Make**（在Linux中很常见），先寻找 GNUmakefile 文件，然后才是 makefile 和 Makefile。多数Linux开发者 Makefile，因为它在一堆小写字母的文件中出现在最前。

要构建一个特定的目标，可以将其传给 **make**。否则 make 将构建 makefile 中的**第一个**目标。很多开发者将 **all** 作为 makefile 的第一个目标，让其他目标都成为 **all** 的依赖。

#### 依赖

先是目标名，冒号，空格或tab，然后是空格或tab分隔的依赖文件列表。例子：

	myapp: main.o 2.o 3.o
    main.o: main.c a.h
    2.o: 2.c a.h b.h
    3.o: 3.c b.h c.h

假如你的应用包括可执行文件 myapp 和一个 manual page，myapp.1。可以加这样一行：

	all: myapp myapp.1

#### 规则

makefile 的第二部分指定规则，描述如何构建一个目标。例如，用什么命令构建 `2.o` 目标？可能用 `gcc -c 2.c` 就足够了，也可能你需要指定额外的 **include** 目录等。你可以显式指定构建规则。

**规则**所在的行必须前置一个 tab，不能是空格！makefile 中行尾的空格可能导致 make 失败。例子：

	myapp: main.o 2.o 3.o
    	gcc -o myapp main.o 2.o 3.o
    main.o: main.c a.h
    	gcc -c main.c
    2.o: 2.c a.h b.h
    	gcc -c 2.c
    3.o: 3.c b.h c.h
    	gcc -c 3.c

### Makefile 中的宏

定义宏：`MACRONAME=value`。若想设置宏的值为空，则`=`后直接换行。

调用宏：`$(MACRONAME)` 或 `${MACRONAME}`。一些版本的 make 也接受 `$MACRONAME`。

makefiles 中，宏常用于编译器的选项。例如，Makefile 默认假设编译器是 `gcc`。若你想其他平台使用其他编译器，修改每一条规则使用的编译器显然很麻烦。不如将使用的编译器定义为宏，则只修改一个宏就可以实现修改所有规则使用的编译器。

宏除了可以定义在 makefile 中，也可以在调用 `make` 时指定，如 `make CC=c89`。且命令行的定义覆盖 makefile 中的定义。在命令行中指定宏，一个宏只能是一个参数，即不能包含空格或引号，如 `make “CC = c89“`。例子：

    all: myapp
    # Which compiler
    CC = gcc
    # Where are include files kept
    INCLUDE = .
    # Options for development
    CFLAGS = -g -Wall –ansi
    # Options for release
    # CFLAGS = -O -Wall –ansi

    myapp: main.o 2.o 3.o
    	$(CC) -o myapp main.o 2.o 3.o
    main.o: main.c a.h
    	$(CC) -I$(INCLUDE) $(CFLAGS) -c main.c
    2.o: 2.c a.h b.h
    	$(CC) -I$(INCLUDE) $(CFLAGS) -c 2.c
    3.o: 3.c b.h c.h
    	$(CC) -I$(INCLUDE) $(CFLAGS) -c 3.c

make 有一些内建的宏。下面列出常用的一些，后面会详细解释。这些宏要到使用前才展开，因此随着 makefile 处理，这些宏的含义可能变化。

- `$?`：List of prerequisites (files the target depends on) changed more recently than the current target
- `$@`：当前目标的名字
- `$<`：当前预备条件的名字{{只有一个？}}
- `$*`：当前预备条件的名字，无后缀

还有两个特殊字符，放在命令前面：

- `-`：让 make 忽略所有错误。例如，若你想创建一个目录，忽略所有错误（如目录已存在），则在 `mkdir` 前加一个减号。
- `@`：tells make not to print the command to standard output before executing it. This character is handy if you want to use echo to display some instructions.

### 多个目标

下面的例子，除了目标 **all**，还添加了一个目标 **clean**，删除对象文件；目标 **install**，安装应用到特定目录。

    all: myapp
    # Which compiler
    CC = gcc
    # Where to install
    INSTDIR = /usr/local/bin
    # Where are include files kept
    INCLUDE = .
    # Options for development
    CFLAGS = -g -Wall –ansi
    # Options for release
    # CFLAGS = -O -Wall –ansi

	myapp: main.o 2.o 3.o
    	$(CC) -o myapp main.o 2.o 3.o
    main.o: main.c a.h
    	$(CC) -I$(INCLUDE) $(CFLAGS) -c main.c
    2.o: 2.c a.h b.h
    	$(CC) -I$(INCLUDE) $(CFLAGS) -c 2.c
    3.o: 3.c b.h c.h
    	$(CC) -I$(INCLUDE) $(CFLAGS) -c 3.c
    clean:
    	-rm main.o 2.o 3.o
    install: myapp
        @if [ -d $(INSTDIR) ]; \
            then \
            cp myapp $(INSTDIR);\
            chmod a+x $(INSTDIR)/myapp;\
            chmod og-w $(INSTDIR)/myapp;\
            echo “Installed in $(INSTDIR)“;\
        else \
        	echo “Sorry, $(INSTDIR) does not exist”;\
        fi

目标 **all** 仍只把 **myapp** 作为依赖。因此不指定任何额外参数执行 make 的任务就是构建目标 **myapp**。

目标 **clean** 没有任何依赖，所以 `clean:` 后面是空的。注意冒号仍然在！

目标 **install** 由一些shell脚本组成。make调用shell执行规则，且每个规则会使用一个新的shell调用。因此上面的脚本命令每行后必须放反斜杠使得所有的脚本命令称为一个逻辑行，被整体传给一个shell执行。注意到命令前的 `@` 符号，表示不要将该命令打印到输出。

**install** 执行多条命令，且上面的写法忽略每个命令的结果，即调用下一条命令时不在乎上一条命令是否成功。如果要求上一次命令必须成功才执行下一条命令，可以用 `&&` 连接行：

    ￼@if [ -d $(INSTDIR) ]; \
    	then \
    	cp myapp $(INSTDIR) &&\
    	chmod a+x $(INSTDIR)/myapp && \
        chmod og-w $(INSTDIR/myapp && \
        echo “Installed in $(INSTDIR)“ ;\
    ￼￼￼￼￼￼else \
    	echo “Sorry, $(INSTDIR) does not exist” ; false ; \
    fi

### 内建规则

make 内建了大量规则可以显著简化 makefile。

实验，创建一个 **foo.c** 文件：

```c
    #include <stdlib.h>
    #include <stdio.h>
    int main() {
    	printf(“Hello World\n”);
    	exit(EXIT_SUCCESS);
    }
```

直接使用make编译：

    $ make foo
    cc foo.c -o foo

make 知道调用什么编译器，虽然这里用了 **cc** 而不是 **gcc**。实际上，内建的规则大量基于宏，可以通过宏修改规则的默认行为，如：

    $ rm foo
    $ make CC=gcc CFLAGS=”-Wall -g” foo
    gcc -Wall -g foo.c -o foo

`make -p` 可以列出所有内建的规则。下面是 GNU make 的部分内建规则：

    OUTPUT_OPTION = -o $@
    COMPILE.c = $(CC) $(CFLAGS) $(CPPFLAGS) $(TARGET_ARCH) -c %.o: %.c
    # commands to execute (built-in):
    $(COMPILE.c) $(OUTPUT_OPTION) $<

利用内建宏，makefile 可以简化为只列出依赖，省略规则：

	main.o: main.c a.h
    2.o: 2.c a.h b.h
    3.o: 3.c b.h c.h

### 后缀与模式规则

内建规则是依赖文件后缀的。最常见的是从 `.c` 文件创建 `.o` 文件。

下面介绍如何定义通用的基于后缀的规则。Linux能识别 `.cc` 结尾的**C++**文件，但不能识别 `.cpp` 结尾的（MS-DOS风格）。我们要创建一条规则，将 `.cpp` 结尾的文件按**C++**编译为 `.o` 文件。

为此需要先添加一行告诉 make 新后缀 **.cpp**，然后制定一条特殊的规则。

    .SUFFIXES: .cpp
    .cpp.o:
		$(CC) -xc++ $(CFLAGS) -I$(INCLUDE) -c $<

`.cpp.o` 的作用是，将 **.cpp** 文件转换为 **.o** 文件。该规则只负责处理 **.cpp** 文件，make 已经知道如何将对象文件变成二进制文件。The extra `-xc++` flag is to tell **gcc** that this is a **C++** source file.

> 最近版本的 make 已经原生支持 **.cpp** 扩展名了。

更新版本的 make 支持一种叫模式规则的语法。例如，模式规则使用 `%` 作为通配符，用于匹配文件名，而不仅仅依赖文件的扩展名。等效写法：

	%.cpp: %o
		$(CC) -xc++ $(CFLAGS) -I$(INCLUDE) -c $<

### make 产生库

开发大型工程时，利用库管理多个编译产品会很方便。库也是文件，习惯上以 `.a` 为扩展名。The make command has a special syntax for dealing with libraries that makes them very easy to manage. The syntax is `lib(file.o)`, which means the object file `file.o`, as stored in the library `lib.a`. The make command has a built-in rule for managing libraries that is usually equivalent to something like this:

    .c.a:
        $(CC) -c $(CFLAGS) $<
        $(AR) $(ARFLAGS) $@ $*.o

宏 `$(AR)` 和 `$(ARFLAGS)` 一般默认为命令 `ar` 和选项 `rv`。The rather terse syntax tells make that to get from a .c file to an .a library it must apply two rules:

- 第一条规则表示先把源文件编译为对象。
- 第二条规则表示，用 `ar` 命令修订库，添加新的对象文件。

Here you change your application so that the files `2.o` and `3.o` are kept in a library called `mylib.a`. The makefile needs very few changes, and Makefile5 looks like this:

	all: myapp
    # Which compiler
    CC = gcc
    # Where to install
    INSTDIR = /usr/local/bin
    # Where are include files kept
    INCLUDE = .
    # Options for development
    CFLAGS = -g -Wall –ansi
    # Options for release
    # CFLAGS = -O -Wall –ansi
    ￼# Local Libraries
    MYLIB = mylib.a
    myapp: main.o $(MYLIB)
    	$(CC) -o myapp main.o $(MYLIB)
    $(MYLIB): $(MYLIB)(2.o) $(MYLIB)(3.o)
    main.o: main.c a.h
    2.o: 2.c a.h b.h
    3.o: 3.c b.h c.h

### 高级主题：Makefiles与子目录

如果工程大，最好把构成一个库的文件放入一个子目录。There are two ways of doing this with make.

First, you can have a second makefile in the subdirectory to compile the files, store them in a library, and then copy the library up a level into the main directory. The main makefile in the higher-level directory then has a rule for making the library, which invokes the second makefile like this:

    mylib.a:
	    (cd mylibdirectory;$(MAKE))

This says that you must always try to `make mylib.a`. When make invokes the rule for building the library, it changes into the subdirectory mylibdirectory and then invokes a new make command to manage the library. Because a new shell is invoked for this, the program using the makefile doesn’t execute the cd. However, the shell invoked to carry out the rule to build the library is in a different directory. 括号为了确保它们被一个shell处理。

The second way is to use some additional macros in a single makefile. The extra macros are generated by appending a `D` for directory or an `F` for filename to those macros we’ve already discussed. You could then override the built-in `.c.o` suffix rule with

    .c.o:
	    $(CC) $(CFLAGS) -c $(@D)/$(<F) -o $(@D)/$(@F)

for compiling files in a subdirectory and leaving the objects in the subdirectory. You then update the library in the current directory with a dependency and rule something like this:

	mylib.a: mydir/2.o mydir/3.o ar -rv mylib.a $?

You need to decide which approach you prefer for your own project. Many projects simply avoid having subdirectories, but this can lead to a source directory with a ridiculous number of files in it. As you can see from the preceding brief overview, you can use make with subdirectories with only slightly increased complexity.

### GNU make 与 gcc

If you’re using GNU make and the GNU gcc compiler, there are two interesting extra options:

- The first is the `-jN` (“jobs”) option to make. This allows make to execute N commands at the same time. Where several different parts of the project can be compiled independently, make will invoke several rules simultaneously. Depending on your system configuration, this can give a significant improvement in the time to recompile. If you have many source files, it may be worth trying this option. In general, smaller numbers such as `-j3` are a good starting point. If you share your computer with other users, use this option with care. Other users may not appreciate your starting large numbers of processes every time you compile!
- The other useful addition is the `-MM` option to gcc. This produces a dependency list in a form suitable for make. On a project with a significant number of source files, each including different combinations of header files, it can be quite difficult (but very important) to get the dependencies correct. If you make every source file depend on every header file, sometimes you’ll compile files unnecessarily. If, on the other hand, you omit some dependencies, the problem is even worse because you’re now failing to compile files that need recompiling.

In this Try It Out, you use the -MM option to gcc to generate a dependency list for your example project:

    $ gcc -MM main.c 2.c 3.c
    main.o: main.c a.h
    2.o: 2.c a.h b.h
    3.o: 3.c b.h c.h
    $

If you really feel confident about **makefiles**, you might try using the makedepend tool, which performs a function similar to the `-MM` option but actually appends the dependencies at the end of the specified makefile.