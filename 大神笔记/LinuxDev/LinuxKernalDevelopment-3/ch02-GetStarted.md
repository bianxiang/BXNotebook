[toc]

## 2. 内核入门

本章介绍获取、编译、安装。We then go over the differences between the kernel and user-space programs and common programming constructs used in the kernel.

### 2.1 获取内核源代码

http://www.kernel.org

You can use Git to obtain a copy of the latest “pushed” version of Linus’s tree:

	$ git clone git://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux-2.6.git

When checked out, you can update your tree to Linus’s latest:

	$ git pull

#### 安装内核源代码

The Linux kernel tarball in bzip2 format is named linux-x.y.z.tar.bz2.

	$ tar xvjf linux-x.y.z.tar.bz2

内核源码一般安装到 `/usr/src/linux`。You should not use this source tree for development because the kernel version against which your C library is compiled is often linked to this tree. 构建内核也不要用root用户。在你的主目录下操作，仅在需要安装时用root用户。即便是安装新内核，也不要动 `/usr/src/linux`。

#### 使用Patches

You will distribute your code changes in patches and receive code from others as patches. Incremental patches provide an easy way to move from one kernel tree to the next. Instead of downloading each large tarball of the kernel source, you can simply apply an incremental patch to go from one version to the next. This saves everyone bandwidth and you time. To apply an incremental patch, from inside your kernel source tree, simply run

	$ patch –p1 < ../patch-x.y.z

Generally, a patch to a given version of the kernel is applied against the previous version. Generating and applying patches is discussed in much more depth in later chapters.

### 2.2 内核源码树

根目录：

- `arch` Architecture-specific source
- `block` Block I/O layer
- `crypto` Crypto API
- `Documentation` Kernel source documentation
- `drivers` Device drivers
- `firmware` Device firmware needed to use certain drivers
- `fs` The VFS and the individual filesystems
- `include` Kernel headers
- `init` Kernel boot and initialization
- `ipc` Interprocess communication code
- `kernel` Core subsystems, such as the scheduler
- `lib` Helper routines
- `mm` Memory management subsystem and the VM
- `net` Networking subsystem
- `samples` Sample, demonstrative code
- `scripts` Scripts used to build the kernel
- `security` Linux Security Module
- `sound` Sound subsystem
- `usr` Early user-space code (called initramfs)
- `tools` Tools helpful for developing Linux
- `virt` Virtualization infrastructure

### 2.3 构建内核

2.6内核引入了一个新的配置和构建系统，使得构建内核更简单了。

#### 2.3.1 配置内核

构建前要先配置。内核配置由配置选项控制，which are prefixed by `CONFIG` in the form `CONFIG_FEATURE`. 例如，对称多处理器（SMP）的配置选项是`CONFIG_SMP`。

配置选项是布尔值，或是三态的（tristate）。布尔选项即取值是或否。三态选项取值是、否或模块（module）；此时“模块”表示作为模块编译，而“是”表示将代码编译到主内核镜像，而不是作为模块。驱动一般表示成三态的。

配置选择还可以是字符串或整数。这些选项不控制构建进程。内核源码可以通过宏访问这些设定的值。

配置工具有多个：

	$ make config
	$ make menuconfig
	$ make gconfig

This command creates a configuration based on the defaults for your architecture:

	$ make defconfig

Although these defaults are somewhat arbitrary (on i386, they are rumored to be Linus’s configuration!), they provide a good start if you have never configured the kernel. To get off and running quickly, run this command and then go back and ensure that configuration options for your hardware are enabled.

配置选项存储在内核源码根目录，名叫 `.config`。直接编辑他也是很容易的。After making changes to your configuration file, or when using an existing configuration file on a new kernel tree, you can validate and update the configuration:

	$ make oldconfig

You should always run this before building a kernel.

The configuration option `CONFIG_IKCONFIG_PROC` places the complete kernel configuration file, compressed, at `/proc/config.gz`. This makes it easy to clone your current configuration when building a new kernel. If your current kernel has this option enabled, you can copy the configuration out of `/proc` and use it to build a new kernel:

	$ zcat /proc/config.gz > .config
    $ make oldconfig

内核配置好后，就可以直接构建了：

	$ make

Unlike kernels before 2.6, you no longer need to run `make dep` before building the kernel—the dependency tree is maintained automatically. 也不再需要指定特定的构建类型，如`bzImage`。The default Makefile rule will handle everything.

#### 2.3.2 最小化构建的噪音

A trick to minimize build noise, but still see warnings and errors, is to redirect the output from make:

	$ make > ../detritus

If you need to see the build output, you can read the file. Because the warnings and errors are output to standard error, however, you normally do not need to. In fact, I just do

	$ make > /dev/null

#### 2.3.3 并行构建

To build the kernel with multiple make jobs, use

	$ make -jn

Here, `n` is the number of jobs to spawn. Usual practice is to spawn one or two jobs per processor. For example, on a 16-core machine, you might do

	$ make -j32 > /dev/null

Using utilities such as the excellent **distcc** or **ccache** can also dramatically improve kernel build time.

#### 2.3.4 安装新内核

安装取决于架构和bootloader。

例如，x86和grub，拷贝 `arch/i386/boot/bzImage` 到 `/boot`。将其命名为 `vmlinuz-version`，然后编辑 `/boot/grub/grub.conf`，为新内核添加一项。

安装模块，是自动的、架构独立的。直接运行：

	% make modules_install

This installs all the compiled modules to their correct home under `/lib/modules`.
The build process also creates the file `System.map` in the root of the kernel source tree. It contains a symbol lookup table, mapping kernel symbols to their start addresses. This is used during debugging to translate memory addresses to function and variable names.

### 2.4 A Beast of a Different Nature

Linux内核与普通用户空间应用相比有一些特别的地方。最重要的区别是：

- 内核不能用C库或C头文件。
- 内核代码是GNU C。
- The kernel lacks the memory protection afforded to user-space.
- The kernel cannot easily execute floating-point operations.
- The kernel has a small per-process fixed-size stack.
- Because the kernel has asynchronous interrupts, is preemptive, and supports SMP, synchronization and concurrency are major concerns within the kernel.
- Portability is important.

Let’s briefly look at each of these issues because all kernel developers must keep them in mind.

#### 2.4.1 没有 libc 或标准头

Unlike a user-space application, the kernel is not linked against the standard C library—or any other library, for that matter. 原因有多个，如先有鸡还是先有蛋的问题，打死你更重要的原因是速度和大小。The full C library—or even a decent subset of it—is too large and too inefficient for the kernel.

Do not fret: Many of the usual libc functions are implemented inside the kernel. For example, the common string manipulation functions are in `lib/string.c`. Just include the header file `<linux/string.h>` and have at them.

本书提到头文件时，我指的都是内核源码树种的内核头文件。Kernel source files cannot include outside headers, just as they cannot use outside libraries.

The base files are located in the `include/` directory in the root of the kernel source tree. For example, the header file `<linux/inotify.h>` is located at `include/linux/inotify.h` in the kernel source tree.
A set of architecture-specific header files are located in `arch/<architecture>/include/asm` in the kernel source tree. For example, if compiling for the x86 architecture, your architecture-specific headers are in `arch/x86/include/asm`. Source code includes these headers via just the `asm/` prefix, for example `<asm/ioctl.h>`.

少掉的函数，如 `printf()`。但内核提供了 `printk()`。The `printk()` function copies the formatted string into the kernel log buffer, which is normally read by the syslog program. Usage is similar to `printf()`:

```c
printk("Hello world! A string '%s' and an integer '%d'\n", str, i);
```

One notable difference between `printf()` and `printk()` is that `printk()` enables you to specify a priority flag. This flag is used by `syslogd` to decide where to display kernel messages. Here is an example of these priorities:

```c
printk(KERN_ERR "this is an error!\n");
```

Note there is no comma between `KERN_ERR` and the printed message. This is intentional; the priority flag is a preprocessor-define representing a string literal, which is concatenated onto the printed message during compilation. We use `printk()` throughout this book.

#### 2.4.2 GNU C

Like any self-respecting Unix kernel, the Linux kernel is programmed in C. Perhaps surprisingly, the kernel is not programmed in strict ANSI C. Instead, where applicable, the kernel developers make use of various language extensions available in gcc (the GNU Compiler Collection, which contains the C compiler used to compile the kernel and most everything else written in C on a Linux system).

The kernel developers use both ISO C99 and GNU C extensions to the C language. These changes wed the Linux kernel to gcc, although recently one other compiler, the Intel C compiler, has sufficiently supported enough gcc features that it, too, can compile the Linux kernel. The earliest supported gcc version is 3.2; gcc version 4.4 or later is recommended. The ISO C99 extensions that the kernel uses are nothing special and, because C99 is an official revision of the C language, are slowly cropping up in a lot of other code. The more unfamiliar deviations from standard ANSI C are those provided by GNU C. Let’s look at some of the more interesting extensions that you will see in the kernel; these changes differentiate kernel code from other projects with which you might be familiar.

##### 内联函数

C99 和 GNU C 都支持内联函数。

An inline function is declared when the keywords `static` and `inline` are used as part of the function definition. For example

```c
static inline void wolf(unsigned long tail_size)
```

Common practice is to place inline functions in header files. Because they are marked static, an exported function is not created. If an inline function is used by only one file, it can instead be placed toward the top of just that file.
In the kernel, using inline functions is preferred over complicated macros for reasons of type safety and readability.

##### Inline Assembly

The gcc C compiler enables the embedding of assembly instructions in otherwise normal C functions. This feature, of course, is used in only those parts of the kernel that are unique to a given system architecture.

The `asm()` compiler directive is used to inline assembly code. For example, this inline assembly directive executes the x86 processor’s `rdtsc` instruction, which returns the value of the timestamp (`tsc`) register:

```c
unsigned int low, high;
asm volatile("rdtsc" : "=a" (low), "=d" (high));
/* low and high now contain the lower and upper 32-bits of the 64-bit tsc */
```

The Linux kernel is written in a mixture of C and assembly, with assembly relegated to low-level architecture and fast path code.The vast majority of kernel code is programmed in straight C.

##### Branch Annotation

The gcc C compiler has a built-in directive that optimizes conditional branches as either very likely taken or very unlikely taken.The compiler uses the directive to appropriately optimize the branch.The kernel wraps the directive in easy-to-use macros, likely() and unlikely().

For example, consider an if statement such as the following: