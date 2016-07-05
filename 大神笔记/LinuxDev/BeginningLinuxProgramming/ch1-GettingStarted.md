本书基于x86。

[toc]

## 1. 入门

### 1.2 Linux程序设计

#### 1.2.3 C语言编译器

POSIX兼容系统中C语言编译器被称为c89。历史上C语言编译器简称为cc。

第一个程序

    #include <stdio.h>
    #include <stdlib.h>
    int main()
    {
    	printf("Hello World\n");
    	exit(0);
    }

#### 1.2.4 开发系统导引

##### 1、应用程序

系统提供的通用程序，包括程序开发工具，放在`/usr/bin`。Applications added by system administrators for a specific host computer or local network are often found in `/usr/local/bin` or `/opt`.

管理员更喜欢`/opt`和`/usr/local`，因为这样可以将系统提供的应用与后续添加的区分开。有利于更新系统：只有`/opt`和`/usr/local`需要保留。We recommend that you compile your applications to run and access required files from the `/usr/local` hierarchy for system-wide applications. 开发过程中的应用及个人应用应用最好只放在`home`目录下。

Additional features and programming systems may have their own directory structures and program directories. 例如X Window System，一般安装在`/usr/X11`或`/usr/bin/X11`目录。Linux distributions typically use the X.Org Foundation version of the X Window System, based on Revision 7 (X11R7).

**gcc**一般位于`/usr/bin`或`/usr/local/bin`。但它的依赖程序在其他位置。这些位置是编译编译器时决定的。对于Linux系统，这个可能是`/usr/lib/gcc/`下的一个子目录（不同版本不同子目录），如`/usr/lib/gcc/i586-suse-linux/4.1.3`。The separate passes of the GNU C/C++ compiler, and GNU-specific header files, are stored here.

##### 2、头文件

对于C语言来说，头文件几乎总是位于`/usr/include`目录及其子目录。Linux的头文件一般放在`/usr/include/sys`和`/usr/include/linux`。

Other programming systems will also have header files that are stored in directories that get searched automatically by the appropriate compiler. Examples include `/usr/include/X11` for the X Window System and `/usr/include/c++` for GNU C++.

可以通过`-I`选项显式指定头文件的位置：

	$ gcc -I/usr/openwin/include fred.c

在`/usr/include`目录下，利用grep命令查找某个定义或函数原型：

    $ grep EXIT_ *.h
    ...
    stdlib.h:#define EXIT_FAILURE 1 /* Failing exit status. */
    stdlib.h:#define EXIT_SUCCESS 0 /* Successful exit status. */
    ...

##### 3、库文件

标准系统库位于`/lib`和`/usr/lib`。C编译器默认只搜索标准C库。你要告诉C编译器还需要搜索哪些库。

库文件名**总是以`lib`开头**。扩展名指定库类型：`.a`表示静态库。`.so`表示共享库。

库常常同时以静态和动态两种形式存在（查看`/usr/lib`）。告诉编译器搜索库，可以给出完整路径，或使用`-l`选项。例如：

	$ gcc -o fred fred.c /usr/lib/libm.a

大致等价的写法：

	$ gcc -o fred fred.c -lm

`-lm`中`l`与`m`之间没有空格。使用`-l`的好处是，如果有共享库，编译器会优先选择共享库。

通过`-L`选项增加库的搜索路径。如：

	$ gcc -o x11fred -L/usr/openwin/lib x11fred.c -lX11

在`/usr/openwin/lib`下寻找名叫`libX11`的库。

##### 4、静态库

当程序需要库中的一个函数时，程序需要include声明函数的头文件。编译器和连接器负责将程序代码和库组合成一个可执行文件。如果库不是标准C运行时库，需要通过`–l`选择指出是哪个库。

静态库，也称为archives，传统上以`.a`结尾。如`/usr/lib/libc.a`是C库。

利用`ar`（即archive）可以创建我们自己的静态库。And compiling functions separately with `gcc -c`. 尽量将多个函数放入独立的源文件。若多个函数需要访问共用的数据，可以将它们放入一个源文件，使用静态变量。

下面的例子创建一个库，包含两个函数。函数叫`fred`和`bill`。

首先为每个函数分别创建一个源文件（fred.c和bill.c）。

    #include <stdio.h>
    void fred(int arg)
    {
    	printf("fred: we passed %d\n", arg);
    }

    #include <stdio.h>
    void bill(char *arg)
    {
    	printf("bill: we passed %s\n", arg);
    }

分别编译两个文件。调用编译器时添加`-c`选项，目的是防止编译器创建一个完整程序。这两个文件无法创建完整程序，因为没有`main`函数。

    $gcc -c bill.c fred.c
    $ls *.o
    bill.o fred.o

为库创建一个头文件，声明库中的函数。所有使用库的程序都应该包含这个头文件。最好也在fred.c和bill.c中包含该头文件，让编译器能检查一些基本错误。

    /* This is lib.h. */
    void bill(char *);
	void fred(int);

下面是我们的一个程序program.c，将调用函数`bill`。首先它include库的头文件：

    #include <stdlib.h>
    #include "lib.h"
    int main()
    {
        bill("Hello World");
        exit(0);
    }

显式指出依赖的对象文件bill.o：

    $gcc -c program.c
    $gcc -o program program.o bill.o
    $./program
    bill: we passed Hello World

下面，开始使用库的方式。利用`ar`创建archive。

    $ar crv libfoo.a bill.o fred.o
    a - bill.o
    a - fred.o

一些系统，他不是继承自Berkeley UNIX的系统，要求创建一个库的目录。Do this with the `ranlib` command. Linux下若使用的是GNU的工具，这一步是可选的（虽然是无害的）。

	$ranlib libfoo.a

下面可以这样编译：

    $ gcc -o program program.o libfoo.a
    $ ./program
    bill: we passed Hello World

或者利用`–l`选项访问库。但由于它不在标准位置下，需要告诉编译器它的位置（`–L.`，点表示当前目录）：

	$ gcc –o program program.o –L. –lfoo

`–lfoo`选项表示寻找库`libfoo.a`（或共享库`libfoo.so`）。

若想知道一个对象文件、库或可执行文件中有哪些函数，可以使用`nm`命令。若查看 *program* 和lib.a，会发现库包含`fred`和`bill`两个函数，但 *program* 只包含`bill`。创建程序时，它只包含实际需要的函数。包含的头文件中有多个函数，并不会导致整个库被包含进最终的程序。

Windows和Linux文件的比较：

|Item    |UNIX   |Windows|
|--------|-------|----|
|对象模块 |func.o |FUNC.OBJ|
|静态库   |lib.a  |LIB.LIB|
|program |program|PROGRAM.EXE|

##### 5、共享库

静态库的缺点：如果多个程序都用到同一个静态库的函数，则程序需要重复拷贝，内存中也会有多个拷贝。多占了磁盘空间和内存。

共享库可以解决该问题。A complete discussion of shared libraries and their implementation on different systems is beyond the scope of this book, so we’ll restrict ourselves to the visible implementation under Linux.

共享库与静态库存放在相同位置。在Linux系统中，标准math库的共享版本位于`/lib/libm.so`。

另一个好处是，共享库可以独立于应用更新。符号链接从`/lib/libm.so`到实际的库版本（`/lib/libm.so.N`）。When Linux starts an application, it can take into account the version of a library required by the application to prevent major new versions of a library from breaking older applications.

对于Linux系统，负责加载共享库并解析函数引用的程序（动态加载器）称为`ld.so`（可能是ld-linux.so.2，或ld-lsb.so.2，或ld-lsb.so.3）。共享库的搜索位置配置在`/etc/ld.so.conf`，由`ldconfig`负责维护（如安装X Window System后添加X11共享库）。

利用工具`ldd`可以查看程序依赖的共享库。

    $ ldd program
    linux-gate.so.1 => (0xffffe000)
    libc.so.6 => /lib/libc.so.6 (0xb7db4000)
    /lib/ld-linux.so.2 (0xb7efc000)

从输出看，程序需要的标准C库libc是共享库。程序需要的主版本是6。

共享库在很多情况下类似于Windows的动态链接库。.so库相当于.DLL；.a库相对于.LIB文件。

### （未）1.3 获取帮助




