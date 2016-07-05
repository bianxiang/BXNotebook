[toc]

## 8. 设备驱动基础

### 8.1 设备驱动概念

#### 8.1.1 可加载的模块

第一种方法，运行时加载。第二种方法，设备驱动还可以被静态编译进内核。对于某些驱动，这样做非常合适。例如，若内核要挂载一个NFS文件系统。此时，网络相关的驱动编译进内核，让它们在启动时可用。第三种方法，利用initial ramdisk，模块和相关脚本放入initial ramdisk镜像。

可加载的模块在内核启动后安装。可由启动脚本加载设备驱动，或按需加载。Linux can request a module when a service is requested that requires a particular module. 内核模块，设备驱动，loadable kernel module (LKM)，可加载的模块等等，这些词由于历史原因，其实指的都是内核设备驱动模块。

#### 8.1.3 最小设备驱动的例子

LISTING 8-1 Minimal Device Driver

    #include <linux/module.h>
    static int __init hello_init(void)
    {
        printk(KERN_INFO “Hello Example Init\n”);
        return 0;
    }
    static void __exit hello_exit(void)
    {
    	printk(“Hello Example Exit\n”);
    }
    module_init(hello_init);
    module_exit(hello_exit);
    MODULE_AUTHOR(“Chris Hallinan”);
    MODULE_DESCRIPTION(“Hello World Example”);
    MODULE_LICENSE(“GPL”);

上面代码已经足够内核加载和卸载该模块。

设备驱动一个特殊的二进制模块。但与普通可执行应用程序不同，设备驱动不能直接在命令行执行。2.6内核要求设备驱动二进制是一个特殊的“内核对象”的格式。正确构建后，设备驱动二进制模块以`.ko`结尾。The build steps and compiler options required to create the .ko module object can be complex. Here we outline a set of steps to harness the power of the Linux kernel build system without requiring you to become an expert in it, which is beyond the scope of this book.

#### 8.1.4 模块构建的基础设施

设备驱动必须针对将来要执行它的那个内核编译。最简单的办法是在内核自己的源码树中构建模块。这样可以确保，当开发者改变内核配置后，它定制的驱动能自动重新构建。虽然，在讷河源码树之外构建驱动也是可能的。但此时要确保你的设备驱动构建配置与内核的同步。This typically includes compiler switches, the location of kernel header files, and kernel configuration options.

要构建上一节的样例驱动，我们需要对Linux内核源码树做下面的修改：

1、在内核源码根目录，创建目录`.../drivers/char/examples`。
2、在该目录下，创建源码文件`hello1.c`。然后创建一个Makefile。内容很简单：

	obj-$(CONFIG_EXAMPLES) += hello1.o

3、在内核配置中增加一个菜单项，启用构建`examples`，决定是内建还是做成可加载的内核模块。Listing 8-2 contains a patch that, when applied to the `.../drivers/char/Kconfig` file
from a recent Linux release, adds the configuration menu item to enable our examples
configuration option. In case you’re unfamiliar with the unified diff format, each line
in Listing 8-2 preceded by a single plus character (+) is inserted in the file between the
indicated lines (those without the leading +). 应用这个patch后，产生一个新的内核配置选项，称为`CONFIG_EXAMPLES`。

LISTING 8-2 Kconfig Patch for examples

    diff --git a/drivers/char/Kconfig b/drivers/char/Kconfig
    index 6f31c94..0805290 100644
    --- a/drivers/char/Kconfig
    +++ b/drivers/char/Kconfig
    @@ -4,6 +4,13 @@
    menu “Character devices”
    +config EXAMPLES
    +   tristate “Enable Examples”
    +   default M
    +   ---help---
    +     Enable compilation option for Embedded Linux Primer
    +     driver examples
    +
    config VT
    	bool “Virtual terminal” if EMBEDDED
    	depends on !S390

As a reminder from our discussion on building the Linux kernel in Chapter 4, “The Linux Kernel: A Different Perspective,” the configuration utility is invoked as follows (this example assumes the ARM architecture):

	$ make ARCH=arm CROSS_COMPILE=xscale_be- gconfig

After the configuration utility is invoked using a command similar to this one, our new Enable Examples configuration option appears under the Character devices menu. Because it is defined as type `tristate`, the kernel developer has three choices:

    (N) No. Do not compile examples.
    (Y) Yes. Compile examples and link with the final kernel image.
    (M) Module. Compile examples as a dynamically loadable module.

4、Now that we have added the configuration option to enable compiling our examples device driver module, 接下来修改`.../drivers/char/Makefile`，告诉构建系统，向下进到新的`examples`目录，如果配置选项`CONFIG_EXAMPLES`出现在配置中。Listing 8-3 contains the patch for this against the makefile in a recent Linux release.

LISTING 8-3 Makefile Patch for examples

	diff --git a/drivers/char/Makefile b/drivers/char/Makefile
    index f957edf..f1b373d 100644
    --- a/drivers/char/Makefile
    +++ b/drivers/char/Makefile
    @@ -102,6 +102,7 @@
    obj-$(CONFIG_MWAVE) += mwave/
    obj-$(CONFIG_AGP) += agp/
    obj-$(CONFIG_PCMCIA) += pcmcia/
    obj-$(CONFIG_IPMI_HANDLER) += ipmi/
    +obj-$(CONFIG_EXAMPLES) += examples/
    obj-$(CONFIG_HANGCHECK_TIMER) += hangcheck-timer.o
    obj-$(CONFIG_TCG_TPM) += tpm/

上面的补丁只增加了一行。这一行放在这个文件中的位置，其实不重要，但尽量逻辑归类一下吧。

此时，所有构建新模块的基础设施已就绪。整个机制的好处时，驱动会随着内核的构建而自动构建。实际构建的命令形如：

	$ make ARCH=arm CROSS_COMPILE=xscale_be- modules

Listing 8-4 shows the build after a typical editing session on the module (all other modules have already been built in this kernel source tree).

LISTING 8-4 Module Build Output

    $ make ARCH=arm CROSS_COMPILE=xscale_be- modules
    CHK include/linux/version.h
    make[1]: ‘include/asm-arm/mach-types.h’ is up to date.
    CHK include/linux/utsrelease.h
    SYMLINK include/asm -> include/asm-arm
    CALL scripts/checksyscalls.sh
    CC [M] drivers/char/examples/hello1.o
    Building modules, stage 2.
    MODPOST 76 modules
    LD [M] drivers/char/examples/hello1.ko

#### 8.1.5 安装设备驱动

构建好后，加载前，我们需要先将模块拷贝到目标系统上一个合适的位置。尽管其实可以放在任何位置，a convention is in place for kernel modules and where they are populated on a running Linux system. 最久的办法是让内核构建系统帮我们做。利用Make目标`modules_install`自动places modules in the system in a logical layout。You simply need to supply the desired location as a prefix to the default path.

标准Linux工作站，设备驱动模块位于`/lib/modules/<kernel-version>/...`，排布类似于Linux内核源码树中的设备驱动目录。If you do not provide an installation prefix to the kernel build system, by default your modules are installed in your own workstation’s `/lib/modules/...` directory. 因为我们做的是嵌入式开发，我们做的是交叉编译，这可能不是我们想要的。You can point to a temporary location in your home directory and manually copy the modules to your target’s file system. 若你的开发版使用NFS根，你可以直接将模块安装到目标文件系统。The following example assumes the latter:

    $ make ARCH=arm CROSS_COMPILE=xscale_be- \
        INSTALL_MOD_PATH=/home/chris/sandbox/coyote-target \
        modules_install

This places **all** your modules in the directory `coyote-target`, which on this sample system is exported via NFS and mounted as root on the target system.

#### 8.1.6 加载一个模块

Having completed all the necessary steps, we are now in a position to load and test the device driver module. Listing 8-5 shows the output resulting from loading and subsequently unloading the device driver on the embedded system.

LISTING 8-5 Loading and Unloading a Module

    # modprobe hello1 <<< Load the driver, must be root
    Hello Example Init
    # modprobe -r hello1 <<< Unload the driver, must be root
    Hello Example Exit
    #

#### （未）8.1.7 模块参数

### （未）8.2 模块工具命令

### （未）8.3 Driver Methods

### （未）8.4 Bringing It All Together

### 8.5 在内核源码树外构建驱动

It is often convenient to build device drivers outside of the kernel source tree. Using a simple makefile patterned after one of the many in the kernel source tree makes this job easy. Driver makefiles in the Linux kernel source tree usually are quite simple. 450个驱动中大半，Makefile不会超过10行。很多只有一行。If we build a makefile for our hello1 example to build it outside the kernel tree, it might look like this:

	obj-$(CONFIG_EXAMPLES) += hello1.o

Create a makefile in a directory of your choice, and place the `hello1.c` source code there. Next, create a new file named Makefile in the same directory. The makefile should contain the single line just shown. Then execute the following build command from this directory (which you just created):

    $ make ARCH=arm CROSS_COMPILE=xscale_be- -C \
    <path/to/your/linux-2.6> SUBDIRS=$PWD modules

将`<path/to/your/linux-2.6>`替换为你的Linux源码树所在位置。This make command, when invoked, switches to your kernel source tree via the `-C` parameter, and instructs the build to build those targets defined in `SUBDIRS`. It’s that simple.

As soon as you understand the concepts, you can build your makefile to have a bit more intelligence. For example, it can define `SUBDIRS` and the path to your kernel if you like. 注意，你的内核配置必须定义`CONFIG_EXAMPLES`，值必须为`=m`或`=y`。You can check this in your `.config` file. 若`CONFIG_EXAMPLES`未定义，`hello1.c`模块将不会被编译。

### （未）8.6 Device Drivers and the GPL



