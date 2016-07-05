[toc]

## 4. Linux内核模块

Linux设备驱动会以内核模块的形式出现。因此学会编写Linux内核模块编程是学习Linux设备驱动的先决条件。

### 4.1 Linux内核模块简介

Linux提供了这样的一种机制，这种机制被称为模块（Module）。模块具有这样的特点：

- 模块本身不被编译入内核映像，从而控制了内核的大小。
- 模块一旦被加载，它就和内核中的其他部分完全一样。

我们先来看一个最简单的内核模块“Hello World”。

    #include <linux/init.h>
    #include <linux/module.h>

    static int hello_init(void)
    {
        printk(KERN_INFO " Hello World enter\n");
        return 0;
    }

    static void hello_exit(void)
    {
        printk(KERN_INFO " Hello World exit\n ");
    }

    module_init(hello_init);
    module_exit(hello_exit);

    MODULE_AUTHOR("Barry Song <21cnbao@gmail.com>");
    MODULE_LICENSE("Dual BSD/GPL");
    MODULE_DESCRIPTION("A simple Hello World Module");
    MODULE_ALIAS("a simplest module");

这个最简单的内核模块只包含内核模块加载函数、卸载函数和对Dual BSD/GPL许可权限的声明以及一些描述信息，位于本书配套光盘 VirtualBox 虚拟机映像的/home/lihacker/develop/svn/ldd6410-read-only/training/kernel/drivers/hello 目录。编译它会产生`hello.ko`目标文件，通过`insmod ./hello.ko`命令可以加载它，通过`rmmod hello`命令可以卸载它。加载时输出Hello World enter。卸载时输出Hello World exit。

内核模块中用于输出的函数是内核空间的`printk()`而非用户空间的`printf()`，`printk()`的用法和`printf()`基本相似，但前者可定义输出级别。printk()可作为一种最基本的内核调试手段，在Linux驱动的调试章节中将详细讲解这个函数。

在Linux中，使用`lsmod`命令可以获得系统中加载了的所有模块以及模块间的依赖关系。lsmod 命令实际上读取并分析/proc/modules文件。

内核中已加载模块的信息也存在于/sys/module目录下，加载hello.ko后，内核中将包含/sys/module/hello目录，该目录下又包含一个refcnt文件和一个 sections目录，在/sys/module/hello目录下运行`tree –a`得到如下目录树：

`modprobe`命令比insmod命令要强大，它在加载某模块时，会同时加载该模块所依赖的其他模块。使用 modprobe 命令加载的模块若以`modprobe -r filename`的方式卸载将同时卸载其依赖的模块。

使用`modinfo <模块名>`命令可以获得模块的信息，包括模块作者、模块的说明、模块所支持的参数以及vermagic：

    lihacker@lihacker-laptop: ~/develop/svn/ldd6410-read-only/training/kernel/drivers/
    hello$ modinfo hello.ko
        filename: hello.ko
        alias: a simplest module
        description: A simple Hello World Module
        license: Dual BSD/GPL
        author: Barry Song <21cnbao@gmail.com>
        srcversion: 3FE9B0FBAFDD565399B9C05
        depends:
        vermagic: 2.6.28-11-generic SMP mod_unload modversions 586

### 4.2 Linux内核模块程序结构

一个 Linux 内核模块主要由如下几个部分组成。

1. 模块加载函数（一般需要）。当通过 insmod 或 modprobe 命令加载内核模块时，模块的加载函数会自动被内核执行，完成本模块的相关初始化工作。
2. 模块卸载函数（一般需要）。当通过 rmmod 命令卸载某模块时，模块的卸载函数会自动被内核执行，完成与模块卸载函数相反的功能。
3. 模块许可证声明（必须）。许可证声明描述内核模块的许可权限，如果不声明 LICENSE，模块被加载时，将收到内核被污染（kernel tainted）的警告。Linux 2.6内核可接受的 LICENSE 包括GPL、GPL v2、GPL and additional rights、Dual BSD/GPL、Dual MPL/GPL和Proprietary。大多数情况下，内核模块应遵循 GPL 兼容许可权。
4. 模块参数（可选）。模块参数是模块被加载的时候可以被传递给它的值，它本身对应模块内部的全局变量。
5. 模块导出符号（可选）。内核模块可以导出符号（symbol，对应于函数或变量），这样其他模块可以使用本模块中的变量或函数。
6. 模块作者等信息声明（可选）。

### 4.3 模块加载函数

Linux内核模块加载函数一般以`__init`标识声明，典型的模块加载函数的形式如下：

    static int __init initialization_function(void)
    {
    	/* 初始化代码 */
    }
    module_init(initialization_function);

模块加载函数必须以`module_init(函数名)`的形式被指定。它返回整型值。初始化成功应返回0。初始化失败应该返回错误码。在Linux内核里错误编码是一个负值，在`<linux/errno.h>`中定义，包含-ENODEV、-ENOMEM之类的符号值。总是返回相应的错误编码是种非常好的习惯，因为只有这样，用户程序才可以利用 perror 等方法把它们转换成有意义的错误信息字符串。

在Linux 2.6内核中，可以使用`request_module(const char *fmt, …)`函数加载内核模块，驱动开发人员可以通过调用

	request_module(module_name);

或

	request_module("char-major-%d-%d", MAJOR(dev), MINOR(dev));

这种灵活的方式加载其他内核模块。

在Linux中所有标识为`__init`的函数在连接的时候都放在`.init.text`这个区段内。此外，所有的`__init`函数在区段`.initcall.init`中还保存了一份函数指针，在初始化时内核会通过这些函数指针调用这些`__init`函数，并在初始化完成后释放init区段（包括.init.text、.initcall.init等）。

### 4.4 模块卸载函数

Linux内核模块加载函数一般以`__exit`标识声明，典型的模块卸载函数的形式如下：

	static void __exit cleanup_function(void)
	{
    	/* 释放代码 */
    }
	module_exit(cleanup_function);

模块卸载函数在模块卸载的时候执行，不返回任何值，必须以`module_exit(函数名)`的形式来指定。

通常来说，模块卸载函数要完成与模块加载函数相反的功能，如下所示。

- 若模块加载函数注册了XXX，则模块卸载函数应该注销XXX。
- 若模块加载函数动态申请了内存，则模块卸载函数应释放该内存。
- 若模块加载函数申请了硬件资源（中断、DMA 通道、I/O端口和I/O内存等）的占用，则模块卸载函数应释放这些硬件资源。
- 若模块加载函数开启了硬件，则卸载函数中一般要关闭之。

和`__init`一样，`__exit`也可以使对应函数在运行完成后自动回收内存。实际上`__init`和`__exit`都是宏，其定义分别为：

    #define __init __attribute__ ((__section__(".init.text")))

和

    #ifdef MODULE
    #define __exit __attribute__ ((__section__(".exit.text")))
    #else
    #define __exit __attribute_used__attribute__ ((__section__(".exit.text")))
	#endif

数据也可以被定义为`__initdata`和`__exitdata`，这两个宏分别为：

	#define __initdata __attribute__ ((__section__(".init.data")))

和

	#define __exitdata __attribute__ ((__section__(".exit.data")))

### 4.5 模块参数

我们可以用`module_param(参数名,参数类型,参数读/写权限)`为模块定义一个参数，例如下
列代码定义了1个整型参数和1个字符指针参数：

    static char *book_name = " dissecting Linux Device Driver ";
    static int num = 4000;
    module_param(num, int, S_IRUGO);
    module_param(book_name, charp, S_IRUGO);

在装载内核模块时，用户可以向模块传递参数，形式为`insmod/modprobe 模块名 参数名=参数值`，如果不传递，参数将使用模块内定义的缺省值。

参数类型可以是byte、short、ushort、int、uint、long、ulong、charp（字符指针）、bool 或invbool（布尔的反），在模块被编译时会将`module_param`中声明的类型与变量定义的类型进行比较，判断是否一致。

模块被加载后，在/sys/module/目录下将出现以此模块名命名的目录。当“参数读/写权限”为0时，表示此参数不存在sysfs文件系统下对应的文件节点，如果此模块存在“参数读/写权限”不为0的命令行参数，在此模块的目录下还将出现`parameters`目录，包含一系列以参数名命名的文件节点，这些文件的权限值就是传入`module_param()`的“参数读/写权限”，而文件的内容为参数的值。

除此之外，模块也可以拥有参数数组，形式为`module_param_array(数组名,数组类型,数组长,参数读/写权限)`。从2.6.0～2.6.10版本，需将数组长变量名赋给“数组长”，从2.6.10版本开始， 需将数组长变量的指针赋给“数组长”，当不需要保存实际输入的数组元素个数时，可以设置“数组长” 为NULL。

运行insmod或modprobe命令时，应使用逗号分隔输入的数组元素。现在我们定义一个包含两个参数的模块（位于虚拟机/home/lihacker/develop/svn/ldd6410-read-only/training/kernel/drivers/param 目录），并观察模块加载时被传递参数和不传递参数时的输出。

    #include <linux/init.h>
    #include <linux/module.h>
    MODULE_LICENSE("Dual BSD/GPL");

    static char *book_name = "dissecting Linux Device Driver";
    static int num = 4 000;
    static int book_init(void)
    {
        printk(KERN_INFO " book name:%s\n",book_name);
        printk(KERN_INFO " book num:%d\n",num);
        return 0;
    }
    static void book_exit(void)
    {
        printk(KERN_INFO " Book module exit\n ");
    }
    module_init(book_init);
    module_exit(book_exit);
    module_param(num, int, S_IRUGO);
    module_param(book_name, charp, S_IRUGO);

    MODULE_AUTHOR("Barry Song <21cnbao@gmail.com>");
    MODULE_DESCRIPTION("A simple Module for testing module params");
    MODULE_VERSION("V1.0");

当用户运行

	insmod book.ko book_name='GoodBook' num=5000

命令时，输出的是用户传递的参数：

    [root@localhost driver_study]# tail -n 2 /var/log/messages
    Jul 2 01:06:21 localhost kernel: <6> book name:GoodBook
    Jul 2 01:06:21 localhost kernel: book num:5 000

### 4.6 导出符号

Linux2.6的`/proc/kallsyms`文件对应着内核符号表，它记录了符号以及符号所在的内存地址。
模块可以使用如下宏导出符号到内核符号表：

	EXPORT_SYMBOL(符号名);
	EXPORT_SYMBOL_GPL(符号名);

导出的符号将可以被其他模块使用，使用前声明一下即可。`EXPORT_SYMBOL_GPL()`只适用于包含 GPL 许可权的模块。下面给出了一个导出整数加、减运算函数符号的内核模块的例子（这些导出符号毫无实际意义，仅仅是为了演示）。

    #include <linux/init.h>
    #include <linux/module.h>
    MODULE_LICENSE("Dual BSD/GPL");
    int add_integar(int a, int b)
    {
        return a+b;
    }

    int sub_integar(int a, int b)
    {
	    return a-b;
    }
    EXPORT_SYMBOL(add_integar);
    EXPORT_SYMBOL(sub_integar);

从/proc/kallsyms文件中找出 add_integar、sub_integar 相关信息：

	[root@localhost driver_study]# cat /proc/kallsyms | grep integar
    c886f050 r __kcrctab_add_integar [export]
    c886f058 r __kstrtab_add_integar [export]
    c886f070 r __ksymtab_add_integar [export]
    c886f054 r __kcrctab_sub_integar [export]
    c886f064 r __kstrtab_sub_integar [export]
    c886f078 r __ksymtab_sub_integar [export]
    c886f000 T add_integar [export]
    c886f00b T sub_integar [export]
    13db98c9 a __crc_sub_integar [export]
    e1626dee a __crc_add_integar [export]

### 4.7 模块声明与描述

在Linux内核模块中，我们可以用MODULE_AUTHOR、MODULE_DESCRIPTION、MODULE_VERSION、 MODULE_DEVICE_TABLE、MODULE_ALIAS分别声明模块的作者、描述、版本、设备表和别名，例如：

    MODULE_AUTHOR(author);
    MODULE_DESCRIPTION(description);
    MODULE_VERSION(version_string);
    MODULE_DEVICE_TABLE(table_info);
    MODULE_ALIAS(alternate_name);

对于USB、PCI等设备驱动，通常会创建一个`MODULE_DEVICE_TABLE`，表明该驱动模块所支持的设备，如下所示：

    /* 对应此驱动的设备表 */
    static struct usb_device_id skel_table [] = {
	    { USB_DEVICE(USB_SKEL_VENDOR_ID, USB_SKEL_PRODUCT_ID) },
	    { } /* 表结束 */
    };
    MODULE_DEVICE_TABLE (usb, skel_table);

此时，并不需要读者理解`MODULE_DEVICE_TABLE`的作用，后续相关章节会有详细介绍。

### （未）4.8 模块的使用计数

### 4.9 模块的编译

我们可以为4.1节的模板编写一个简单的Makefile：

    KVERS = $(shell uname -r)
    # Kernel modules
    obj-m += hello.o
    # Specify flags for the module compilation.
    #EXTRA_CFLAGS=-g -O0

	build: kernel_modules
    kernel_modules:
		make -C /lib/modules/$(KVERS)/build M=$(CURDIR) modules
    clean:
    	make -C /lib/modules/$(KVERS)/build M=$(CURDIR) clean

该Makefile文件应该与源代码hello.c位于同一目录，开启其中的`EXTRA_CFLAGS=-g -O0`可以得到包含调试信息的 hello.ko 模块。运行 make 命令得到的模块可直接在 PC 上运行。

如果一个模块包括多个.c文件（如file1.c、file2.c），则应该以如下方式编写Makefile：

    obj-m := modulename.o
    modulename-objs := file1.o file2.o

### （未）4.10 使用模块绕开GPL










