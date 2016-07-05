[toc]

## 2. 构建与运行模块

本章构建并运行一个完整的模块，学习所有模块共用的基础概念。

### 2.1 搭建测试系统

我们推荐你从kernel.org获得的一个mainline内核，并安装到你的系统中。厂商的内核的补丁可能改变驱动使用的内核API。

之前版本的内核只需要一些头文件。但构建2.6.x模块需要你的系统中有一个已配置、构建好的内核树。2.6模块链接到内核源码树中的对象文件；the result is a more robust module loader, but also the requirement that those object files be available. 因此第一件事是先获得内核源码（如从kernel.org），构建一个新内核，并安装到你的系统中。For reasons we’ll see later, life is generally easiest if you are actually running the target kernel when you build your modules, though this is not required.

### 2.2 Hello World模块

```c
    #include <linux/init.h>
    #include <linux/module.h>
    MODULE_LICENSE("Dual BSD/GPL");
    static int hello_init(void)
    {
        printk(KERN_ALERT "Hello, world\n");
        return 0;
    }
    static void hello_exit(void)
    {
    	printk(KERN_ALERT "Goodbye, cruel world\n");
    }
    module_init(hello_init);
    module_exit(hello_exit);
```

模块定义了两个函数，分别在模块加载和移除时调用。`MODULE_LICENSE`声明模块的许可证；没有它加载模块时会警告。

`printk`定义在Linux内核里；它与标准C库函数`printf`类似。内核使用自己的打印函数，因为它运行时没有C库。`insmod`加载了模块后，模块被链接都内核，可以访问内核的符号（函数、变量），因此模块内可以调用`printk`。`KERN_ALERT`是消息的优先级。We’ve specified a high priority in this module, because a message with the default priority might not show up anywhere useful, depending on the kernel version you are running, the version of the **klogd** daemon, and your configuration. You can ignore this issue for now; we explain it in Chapter 4.

作为超级用户，可以利用`insmod`和`rmmod`测试模块。

    % make
    make[1]: Entering directory `/usr/src/linux-2.6.10'
        CC [M]  /home/ldd3/src/misc-modules/hello.o
        Building modules, stage 2.
        MODPOST
        CC      /home/ldd3/src/misc-modules/hello.mod.o
        LD [M]  /home/ldd3/src/misc-modules/hello.ko
        make[1]: Leaving directory `/usr/src/linux-2.6.10'
    % su
    root# insmod ./hello.ko
    Hello, world
    root# rmmod hello
    Goodbye, cruel world

注意，上述操作的前提，是有一个配置好的内核源代码树，且makefile能找到它（上面的例子，内核在/usr/src/linux-2.6.10）。We get into the details of how modules are built in the section “Compiling and Loading.”

In particular, the previous screen dump was taken from a text console; if you are running `insmod` and `rmmod` from a **terminal emulator** running under the window system, you won’t see anything on your screen. The message goes to one of the system log files, such as `/var/log/messages` (the name of the actual file varies between Linux distributions). The mechanism used to deliver kernel messages is described in Chapter 4.

### 2.3 内核模块与应用的对比

应用可以调用不是它定义的函数：链接阶段解析外部引用。如`printf`定义在库**libc**。

模块只链接到内核，于是只能调用内核暴露的函数。模块不链接到任何库。

用于库不链接到模块，源文件不应包括常见的头文件。一个例外是`<stdarg.h>`。内核模块只能使用内核自己定义的函数。所有这些函数都在内核源码中的头文件中。多数在文件**include/linux**和**include/asm**中。

Another important difference between kernel programming and application programming is in how each environment handles faults: whereas a segmentation fault is harmless during application development and a debugger can always be used to trace the error to the problem in the source code, a kernel fault kills the current process at least, if not the whole system. We see how to trace kernel errors in Chapter 4.

#### 2.3.1 用户空间与内核空间

模块运行在内核空间，应用运行在用户空间。

#### 2.3.2 内核中的并发

As a result, Linux kernel code, including driver code, must be **reentrant**—it must be capable of running in more than one context at the same time. Every sample driver in this book has been written with concurrency in mind.

#### 2.3.3 当前进程

内核的多数操作都是代表一个特定进程进行的。内核代码可以通过全局指针（定义在`<asm/current.h>`）访问；指针指向`struct task_struct`（定义在`<linux/sched.h>`）。The `current` pointer refers to the process that is currently executing. During the execution of a system call, such as `open` or `read`, the current process is the one that invoked the call. Kernel code can use process-specific information by using `current`, if it needs to do so. An example of this technique is presented in Chapter 6.

Actually, `current` is not truly a global variable. The need to support SMP systems forced the kernel developers to develop a mechanism that finds the current process on the relevant CPU. This mechanism must also be fast, since references to `current` happen frequently. The result is an architecture-dependent mechanism that, usually, hides a pointer to the `task_struct` structure on the kernel stack. The details of the implementation remain hidden to other kernel subsystems though, and a device driver can just include `<linux/sched.h>` and refer to the current process. For example, the following statement prints the process ID and the command name of the current process by accessing certain fields in struct `task_struct`:

	printk(KERN_INFO "The process is \"%s\" (pid %i)\n", current->comm, current->pid);

#### 2.3.4 其他细节

Applications are laid out in virtual memory with a very large stack area. The stack, of course, is used to hold the function call history and all automatic variables created by currently active functions. The kernel, instead, has a very small stack; it can be as small as a single, 4096-byte page. Your functions must share that stack with the entire kernel-space call chain. Thus, it is never a good idea to declare large automatic variables; if you need larger structures, you should allocate them dynamically at call time.

Often, as you look at the kernel API, you will encounter function names starting with a double underscore (`__`). Functions so marked are generally a low-level component of the interface and should be used with caution. Essentially, the double underscore says to the programmer: “If you call this function, be sure you know what you are doing.”

内核代码不能进行浮点运算。Enabling floating point would require that the kernel save and restore the floating point processor’s state on each entry to, and exit from, kernel space—at least, on some architectures. Given that there really is no need for floating point in kernel code, the extra overhead is not worthwhile.

### 2.4 编译与加载

#### 2.4.1 编译模块

新的构建系统比之前的版本有很大变化，但更易用。要阅读Documentation/kbuild。

一些前置条件，包括正确版本的编译器等，内核文档 Documentation/Changes 列出了需要的工具版本。一般来说太老太新都是不行的。

下面，是创建makefile。对于上一章的Hello World，只需要一行：

	obj-m := hello.o

上面这行代码与常见的Makefile不太一样，这些差异会由内核的构建系统负责处理。上面的赋值用到了GNU make的扩展语法。产生的模块命名为**hello.ko**。

若需要创建模块`module.ko`，模块来自两个源文件（file1.c和file2.c），写法是：

    obj-m := module.o
    module-objs := file1.o file2.o

上述Makefile必须在内核构建系统的上下文中执行。假设你的内核源代码位于`~/kernel-2.6`目录，则在模块源码及Makefile所在目录下，执行以下make命令。

	make -C ~/kernel-2.6 M=`pwd` modules

该命令先将目录改到`-C`选项指定的目录（内核源码目录），在那里它找到内核的顶级Makefile文件。`M=`选项令Make在执行构建*modules*（`modules`是make的目标）前，先回到模块源码目录。

在内核树外构建模块，有一套习惯的写法可以减少一些麻烦：

    # If KERNELRELEASE is defined, we've been invoked from the
    # kernel build system and can use its language.
    ifneq ($(KERNELRELEASE),)
    	obj-m := hello.o
    # Otherwise we were called directly from the command
    # line; invoke the kernel build system.
    else
    	KERNELDIR ?= /lib/modules/$(shell uname -r)/build
    	PWD  := $(shell pwd)
    default:
    	$(MAKE) -C $(KERNELDIR) M=$(PWD) modules
    endif

上面的Makefile也用到了一些扩展的GNU make语法。一次典型的构建会两次读取该文件。第一次调用时，它发现`KERNELRELEASE`变量未定义。It locates the kernel source directory by taking advantage of the fact that the **symbolic link build in the installed modules directory points back at the kernel build tree**. If you are not actually running the kernel that you are building for, you can supply a `KERNELDIR=` option on the command line, 设置`KERNELDIR`环境变量，或重写Makefile中`KERNELDIR`定义的一行。找到内核源码树后，Makefile调用`default:`目标，执行第二个make命令（`$(MAKE)`）、调用内核构建系统。第二次读取Makefile时，makefile设置`obj-m`，内核的Makefile负责实际构建该模块。

注意上面的Makefile其实不是完整的。完整的还需要清理、安装模块等目标。完整的例子参见本书源码例子。

#### 2.4.2 加载与卸载模块

模块构建好后，下一步是将模块加载到内核。使用`insmod`；与`ld`类似，it links any unresolved symbol in the module to the symbol table of the kernel. 与链接器不同的是，内核不会修改模块的磁盘文件，只会修改内存中的拷贝。

`insmod`接受一些命令行参数；可以在模块被链接到内核前，向模块赋参数。因此如果模块正确设计，它可以到加载时才配置。

内核是如何支持`insmod`的？它依赖定义在**kernel/module.c**中的一个系统调用。The function `sys_init_module` allocates kernel memory to hold a module (this memory is allocated with `vmalloc`; see the section “vmalloc and Friends” in Chapter 8); it then copies the module text into that memory region, resolves kernel references in the module via the kernel symbol table, and calls the module’s initialization function to get everything going.

如果你看过内核源代码，你会发现系统调用的名字都有前缀`sys_`，且其他函数都不会用此命名。

The `lsmod` program produces a list of the modules currently loaded in the kernel. Some other information, such as any other modules making use of a specific mod ule, is also provided. lsmod works by reading the /proc/modules virtual file. Information on currently loaded modules can also be found in the sysfs virtual filesystem under /sys/module.

#### 2.4.3 版本依赖

每次模块被链接到一个新版本的内核，模块都一定要重新编译。模块的版本更多是开发者分发的一个标记。模块与特定内核版本的数据结构和函数原型是紧密绑定的；每个内核给模块开发的接口可能差别很大。内核开发就是如此。

内核不仅要求模块根据正确的内核版本编译；One of the steps in the build process is to link your module against a file (called vermagic.o) from the current kernel tree; this object contains a fair amount of information about the kernel the module was built for, including the target kernel version, compiler version, and the settings of a number of important configuration variables. When an attempt is made to load a module, this information can be tested for compatibility with the running kernel. If things don’t match, the module is not loaded; instead, you see something like:

    # insmod hello.ko
    Error inserting './hello.ko': -1 Invalid module format

A look in the system log file (/var/log/messages or whatever your system is configured to use) will reveal the specific problem that caused the module to fail to load. 当你需要为特定内核版本编译一个模块，你需要那个版本内核的构建系统和源码。可以修改一下上面样例Makefile的`KERNELDIR`变量。

Kernel interfaces often change between releases. If you are writing a module that is intended to work with multiple versions of the kernel (especially if it must work across major releases), you likely have to make use of macros and #ifdef constructs to make your code build properly. 本书只涉及一个版本的内核，因此本书的样例代码中几乎没有版本测试。

But the need for them does occasionally arise. In such cases, you want to make use of
the definitions found in **linux/version.h**. This header file defines the following macros:

- `UTS_RELEASE`：This macro expands to a string describing the version of this kernel tree. For example, `"2.6.10"`.
- `LINUX_VERSION_CODE`：This macro expands to the binary representation of the kernel version, one byte for each part of the version release number. For example, the code for 2.6.10 is 132618 (i.e., 0x02060a). With this information, you can (almost) easily determine what version of the kernel you are dealing with.
- `KERNEL_VERSION(major,minor,release)`：This is the macro used to build an integer version code from the individual numbers that build up a version number. For example, KERNEL_VERSION(2,6,10) expands to 132618. This macro is very useful when you need to compare the current version and a known checkpoint.

Most dependencies based on the kernel version can be worked around with preprocessor conditionals by exploiting `KERNEL_VERSION` and `LINUX_VERSION_CODE`. Version dependency should, however, not clutter driver code with hairy `#ifdef` conditionals; the best way to deal with incompatibilities is by confining them to a specific header file. As a general rule, code which is explicitly version (or platform) dependent should be hidden behind a low-level macro or function. High-level code can then just call those functions without concern for the low-level details. Code written in this way tends to be easier to read and more robust.

#### 2.4.4 平台依赖

Each computer platform has its peculiarities, and kernel designers are free to exploit all the peculiarities to achieve better performance in the target object file. Unlike application developers, who must link their code with precompiled libraries and stick to conventions on parameter passing, kernel developers can dedicate some processor registers to specific roles, and they have done so. Moreover, kernel code can be optimized for a specific processor in a CPU family to get the best from the target platform: unlike applications that are often distributed in binary format, a custom compilation of the kernel can be optimized for a specific computer set.

Clearly, if a module is to work with a given kernel, it must be built with the same understanding of the target processor as that kernel was. Once again, the vermagic.o object comes in to play. When a module is loaded, the kernel checks the processorspecific configuration options for the module and makes sure they match the running kernel. If the module was compiled with different options, it is not loaded.


### 2.5 内核符号表

We’ve seen how insmod resolves undefined symbols against the table of public kernel symbols. The table contains the addresses of global kernel items—functions and
variables—that are needed to implement modularized drivers. 模块加载后，模块暴露的符号称为内核符号表的一部分。

Module stacking is useful in complex projects. If a new abstraction is implemented in
the form of a device driver, it might offer a plug for hardware-specific implementations. For example, the video-for-linux set of drivers is split into a generic module that
exports symbols used by lower-level device drivers for specific hardware. According to
your setup, you load the generic video module and the specific module for your
installed hardware.

When using stacked modules, it is helpful to be aware of the `modprobe` utility. As we
described earlier, modprobe functions in much the same way as insmod, but it also
loads any other modules that are required by the module you want to load. Thus,
one modprobe command can sometimes replace several invocations of `insmod`
(although you’ll still need insmod when loading your own modules from the current
directory, **because `modprobe` looks only in the standard installed module directories**).

The Linux kernel header files provide a convenient way to manage the visibility of
your symbols, thus reducing namespace pollution (filling the namespace with names
that may conflict with those defined elsewhere in the kernel) and promoting proper
information hiding. 若你的模块需要向其他模块暴露符号，需要用到下面的宏：

    EXPORT_SYMBOL(name);
    EXPORT_SYMBOL_GPL(name);

符号必须在模块文件的全局部分暴露——在函数外，because the macros expand to the declaration of a special-purpose variable that is expected to be accessible globally. 这些变量存储在模块的可执行文件的一个特殊部分（ELF section）。

### 2.6 预热

多模块代码需要引入大量头文件。其中有两个必须出现在模块中：

    #include <linux/module.h>
    #include <linux/init.h>

**module.h**包含大量符号和函数定义。**init.h**用于指定初始化和清理函数。

加载时向模块传递参数，需要模块引入**moduleparam.h**。

模块要指定许可证，利用宏：

	MODULE_LICENSE("GPL");

The specific licenses recognized by the kernel are “GPL” (for any version of the GNU General Public License), “GPL v2” (for GPL version two only), “GPL and additional rights,”“Dual BSD/GPL,”“Dual MPL/GPL,” and “Proprietary.” Unless your module is explicitly marked as being under a free license recognized by the kernel, it is assumed to be proprietary, and the kernel is “tainted” when the module is loaded.

其他描述性的宏包括`MODULE_AUTHOR`、`MODULE_DESCRIPTION`、`MODULE_VERSION` (for a code revision number; see the comments in `<linux/module.h>` for the conventions to use in creating version strings), `MODULE_ALIAS`（别名），`MODULE_DEVICE_TABLE`（告诉用户控件模块支持的设备）。The various `MODULE_` declarations can appear anywhere within your source file outside of a function. 最近习惯的做法是放到文件最后。

### 2.7 初始化与关闭

初始化函数的例子：

```c
    static int __init initialization_function(void)
    {
    /* Initialization code here */
    }
    module_init(initialization_function);
```
初始化函数应声明为`static`，因为它们不必对其他文件可见。`__init`是给内核的一个标志，表示该函数只在初始化的时候使用。The module loader drops the initialization function after the module is loaded, making its memory available for other uses. 有一个类似的标记`__initdata`，用于标记只在初始化阶段使用的数据。`__init`和`__initdata`不是一定要用的，但值得用。You may also encounter `__devinit` and `__devinitdata` in the kernel source; these translate to `__init` and `__initdata` only if the kernel has not been configured for hotpluggable devices. We will look at hotplug support in Chapter 14.

`module_init`是必须使用的。该宏向模块的对象文件中添加一个特殊的节，指出从哪里寻找模块的初始化函数。

Modules can register many different types of facilities, including different kinds of
devices, filesystems, cryptographic transforms, and more. For each facility, there is a
specific kernel function that accomplishes this registration. The arguments passed to
the kernel registration functions are usually pointers to data structures describing the
new facility and the name of the facility being registered. The data structure usually
contains pointers to module functions, which is how functions in the module body
get called.

The items that can be registered go beyond the list of device types mentioned in
Chapter 1. They include, among others, serial ports, miscellaneous devices, sysfs
entries, /proc files, executable domains, and line disciplines. Many of those registrable items support functions that aren’t directly related to hardware but remain in the
“software abstractions” field. Those items can be registered, because they are integrated into the driver’s functionality anyway (like /proc files and line disciplines for
example).

There are other facilities that can be registered as add-ons for certain drivers, but
their use is so specific that it’s not worth talking about them; they use the stacking
technique, as described in the section “The Kernel Symbol Table.” If you want to
probe further, you can grep for `EXPORT_SYMBOL` in the kernel sources, and find the
entry points offered by different drivers. Most registration functions are prefixed with
`register_`, so another possible way to find them is to grep for `register_` in the kernel source.

#### 清理函数

清理函数的例子：

```c
    static void __exit cleanup_function(void)
    {
    /* Cleanup code here */
    }
    module_exit(cleanup_function);
```

清理函数没有返回值。`__exit`标记表示代码只用于模块卸载（告诉编译器将其放在ELF特殊部分）。

`module_exit`也是必须调用的。

若模块没有定义清理函数，内核将不允许它被卸载。

#### 初始化阶段的错误处理

注册可能失败。如申请内存，但分配不到。

若模块遇到错误，不能继续，必须将之前已注册的设施回滚。若不能回滚获得的资源，内核将处于不稳定的状态；it contains internal pointers to code that no longer exists。此时，只能重启了。

初始化函数的返回值是错误码。在Linux内核中，错误用负数表示。预定义参见`<linux/errno.h>`，如`-ENODEV`。

#### 模块加载的竞争条件

在注册一个设施之前，要确保支持工作全部完成。因为一旦注册，内核的其他部分就有可能立即使用该设施。

You must also consider what happens if your initialization function decides to fail, but some part of the kernel is already making use of a facility your module has registered. If this situation is possible for your module, you should seriously consider not failing the initialization at all. After all, the module has clearly succeeded in exporting something useful. If initialization must fail, it must carefully step around any possible operations going on elsewhere in the kernel until those operations have completed.

### 2.8 模块参数

These parameter values can be assigned at load time by `insmod` or `modprobe`; the latter can also read parameter assignment from its configuration file (/etc/modprobe.conf). 例子：

	insmod hellop howmany=10 whom="Mom"

参数通过宏`module_param`声明（定义在头**moduleparam.h**）。`module_param`取三个参数：变量名，类型，and a permissions mask to be used for an accompanying sysfs entry. 宏应放在所有函数外，一般在源文件上部。例子：

    static char *whom = "world";
    static int howmany = 1;
    module_param(howmany, int, S_IRUGO);
    module_param(whom, charp, S_IRUGO);

模块参数的支持的类型有：

- 布尔类：`bool`、`invbool`。关联的变量应是`int`型。`invbool`类型反转了真假。
- 字符指针：`charp`。可以用于字符或字符串。
- 整数：`int`、`long`、`short`、`uint`、`ulong`、`ushort`。前缀u表示无符号数。

数组参数（实参逗号分隔）声明：

	module_param_array(name,type,nump,perm);

其中，`type`是元素类型。若运行时提供了该参数的实参，`nump`是数据元素的实际个数。The module loader refuses to accept more values than will fit in the array.

If you really need a type that does not appear in the list above, there are hooks in the module code that allow you to define them; see **moduleparam.h** for details on how to do that.

The final `module_param` field is a permission value; you should use the definitions found in `<linux/stat.h>`. This value controls who can access the representation of the module parameter in sysfs. If perm is set to 0, there is no sysfs entry at all; otherwise, it appears under /sys/module* with the given set of permissions. Use `S_IRUGO` for a parameter that can be read by the world but cannot be changed; `S_IRUGO|S_IWUSR` allows root to change the parameter. Note that if a parameter is changed by sysfs, the value of that parameter as seen by your module changes, but your module is not notified in any other way. You should probably not make module parameters writable, unless you are prepared to detect the change and react accordingly.

### 2.9 用户空间的驱动

但有人认为在用户空间写驱动比hack内核更好。本节讨论在用户空间写驱动。

用户空间的驱动的优势有：

- 可以链接完整的C库。The driver can perform many exotic tasks without resorting to external programs (the utility programs implementing usage policies that are usually distributed along with the driver itself).
- The programmer can run a conventional debugger on the driver code without having to go through contortions to debug a running kernel.
- 若用户控件的驱动挂住了，可以直接杀死它。Problems with the driver are unlikely to hang the entire system, unless the hardware being controlled is really misbehaving.
- 用户内存是可交换的，不像内核内存。An infrequently used device with a huge driver won’t occupy RAM that other programs could be using, except when it is actually in use.
- A well-designed driver program can still, like kernel-space drivers, allow concurrent access to a device.
- If you must write a closed-source driver, the user-space option makes it easier for you to avoid ambiguous licensing situations and problems with changing kernel interfaces.

例如，USB驱动可以在用户控件编写；see the (still young) libusb project at libusb.sourceforge.net and “gadgetfs” in the kernel source. Another example is the X server: it knows exactly what the hardware can do and what it can’t, and it offers the graphic resources to all X clients. Note, however, that there is a slow but steady drift toward frame-buffer-based graphics environments, where the X server acts only as a server based on a real kernel-space device driver for actual graphic manipulation.

Usually, the writer of a user-space driver implements a server process, taking over from the kernel the task of being the single agent in charge of hardware control. Client applications can then connect to the server to perform actual communication with the device; therefore, a smart driver process can allow concurrent access to the device. This is exactly how the X server works.

用户空间驱动的缺点：

- 用户控件无法用中断。There are workarounds for this limitation on some platforms, such as the vm86 system call on the IA32 architecture.
- Direct access to memory is possible only by mmapping /dev/mem, and only a privileged user can do that.
- Access to I/O ports is available only after calling **ioperm** or **iopl**. Moreover, not all platforms support these system calls, and access to /dev/port can be too slow to be effective. Both the system calls and the device file are reserved to a privileged user.
- Response time is slower, because a context switch is required to transfer information or actions between the client and the hardware.
- Worse yet, if the driver has been swapped to disk, response time is unacceptably long. Using the **mlock** system call might help, but usually you’ll need to lock many memory pages, because a user-space program depends on a lot of library code. mlock, too, is limited to privileged users.
- The most important devices can’t be handled in user space, including, but not limited to, network interfaces and block devices.

As you see, user-space drivers can’t do that much after all. Interesting applications nonetheless exist: for example, support for SCSI scanner devices (implemented by the SANE package) and CD writers (implemented by cdrecord and other tools). In both cases, user-level device drivers rely on the “SCSI generic” kernel driver, which exports low-level SCSI functionality to user-space programs so they can drive their own hardware.

One case in which working in user space might make sense is when you are beginning to deal with new and unusual hardware. This way you can learn to manage your hardware without the risk of hanging the whole system. Once you’ve done that, encapsulating the software in a kernel module should be a painless operation.





