[toc]

## 3. Linux内核及内核编程

本章为读者打下 Linux 驱动编程的软件基础。由于 Linux 驱动编程本质属于 Linux 内核编程，因此我们有必要熟悉 Linux 内核及内核编程的基础知识。

### （未）3.1 Linux内核的发展与演变

针对嵌入式系统的应用，一些改进内核的Linux被开发出来，如改进实时性的Hard Hat Linux和 RTLinux、**支持不含MMU CPU的μClinux**(日前 Linux mainline 已经支持 MMU-less 系统)。

### （未）3.2 Linux 2.6内核的特点

### 3.3 Linux内核的组成

#### 3.3.2 Linux内核的组成部分

Linux 内核主要由进程调度(SCHED)、内存管理(MM)、虚拟文件系统(VFS)、网络接口(NET)和进程间通信(IPC)5个子系统组成。

1、进程调度

进程调度控制系统中的多个进程对CPU￼的访问，使得多个进程能在CPU中“微观串行、宏观并行”地执行。进程调度处于系统的中心位置,内核中其他的子系统都依赖它。因
为每个子系统都需要挂起或恢复进程。

Linux的进程在几个状态间进行切换。在设备驱动编程中，当请求的资源不能得到满足时，驱动一般会调度其他进程执行，并使本进程进入睡眠状态，直到它请求的资源被释放，才会被唤醒而进入就绪态。睡眠分成**可被打断的睡眠**和**不可被打断的睡眠**。

在设备驱动编程中，当请求的资源不能得到满足时，驱动一般会调度其他进程执行，其对应进程进入睡眠状态，直到它请求的资源被释放，才会被唤醒而进入就绪态。

设备驱动中，如果需要几个并发执行的任务，可以启动内核线程，启动内核线程的函数为:

	pid_t kernel_thread(int (*fn)(void *), void *arg, unsigned long flags);

2、内存管理

内存管理的主要作用是控制多个进程安全地共享主内存区域。当CPU提供内存管理单元 (MMU)时，Linux内存管理完成为每个进程进行虚拟内存到物理内存的转换。Linux 2.6引入了对无MMU CPU的支持。

一般而言Linux的每个进程享有4GB的内存空间。0~3GB属于用户空间，3~4GB属于内核空间。内核空间对常规内存、I/O设备内存以及高端内存存在不同的处理方式。

3、虚拟文件系统

Linux虚拟文件系统(VFS)隐藏各种了硬件的具体细节，为所有的设备提供了统一的接口。它独立于各个具体的文件系统，是对各种文件系统的一个抽象。它使用**超级块super block**存放文件系统相关信息，使用**索引节点inode**存放文件的物理信息，使用目录项dentry存放文件的逻辑信息。

4、网络接口

网络接口提供了对各种网络标准的存取和各种网络硬件的支持。在Linux中网络接口可分为网络协议和网络驱动程序。网络协议部分负责实现每一种可能的网络传输协议，网络设备驱动程序负责与硬件设备通信。每一种可能的硬件设备都有相应的设备驱动程序。

5、进程通信

Linux支持进程间的多种通信机制，包含信号量、共享内存、管道等。这些机制可协助多个进程、多资源的互斥访问、进程间的同步和消息传递。

#### 3.3.3 Linux内核空间与用户空间

ARM处理器分为7种工作模式。

- 用户模式(usr)：大多数的应用程序运行在用户模式下。当处理器运行在用户模式下时,某些被保护的系统资源是不能被访问的。
- 快速中断模式(fiq)：用于高速数据传输或通道处理。
- 外部中断模式(irq)：用于通用的中断处理。
- 管理模式(svc)：操作系统使用的保护模式。
- 数据访问终止模式(abt)：当数据或指令预取终止时进入该模式，可用于虚拟存储及存储保护。
- 系统模式(sys)：运行具有特权的操作系统任务。
- 未定义指令中止模式(und)：当未定义的指令执行时进入该模式，可用于支持硬件协处理器的软件仿真。

ARM Linux的系统调用实现原理是采用swi软中断从用户态usr模式陷入内核态svc模式。又如，X86处理器包含4个不同的特权级，称为Ring0~Ring3。Ring0下，可以执行特权级指令，对任何I/O设备都有访问权等。而Ring3则被限制很多操作。

Linux系统充分利用CPU的这一硬件特性。但它只使用了两级。在Linux系统中内核可进行任何操作；而应用程序则被禁止对硬件的直接访问和对内存的未授权访问。

内核空间和用户空间，使用两种不同的地址空间。Linux只能通过**系统调用**和**硬件中断**完成从用户空间到内核空间的控制转移。

### 3.4 Linux内核的编译及加载

#### 3.4.1 Linux内核的编译

Linux 驱动工程师需要牢固地掌握 Linux 内核的编译方法以为嵌入式系统构建可运行的 Linux 操作系统映像。在编译 LDD6410 的内核时,需要配置内核,可以使用下面命令中的一个:

    ￼#make config(基于文本的最为传统的配置界面,不推荐使用)
    #make menuconfig(基于文本菜单的配置界面)
    #make xconfig(要求 QT 被安装)
    #make gconfig(要求 GTK+被安装)

最值得推荐的是`make menuconfig`，它不依赖于 QT 或 GTK+，且非常直观。

内核配置包含的项目相当多。arch/arm/configs/ldd6410lcd_defconfig 文件包含了 LDD6410 的默认配置。因此只需要运行`make ldd6410lcd_defconfig`就可以为 LDD6410 开发板配置内核。

编译内核和模块的方法是:

	make zImage
    make modules

执行完上述命令后，在源代码的根目录下会得到未压缩的内核映像 **vmlinux** 和内核符号表文件System.map，在arch/arm/boot/目录会得到压缩的内核映像**zImage**，在内核各对应目录得到选中的内核模块。

Linux 2.6 内核的配置系统由以下3部分组成。

- Makefile：分布在Linux内核源代码中的Makefile
- 配置文件(Kconfig)：给用户提供配置选择的功能。
- 配置工具：包括配置命令解释器（对配置脚本中使用的配置命令进行解释）和配置用户界面（提供基于字符界面和图形界面）。这些配置工具都是使用脚本语言，如 Tcl/TK、Perl等编写。

使用`make config`、`make menuconfig`等命令后，会生成一个`.config`配置文件，记录哪些部分被编译入内核，哪些部分被编译为内核模块。运行`make menuconfig`等时，配置工具首先分析与体系结构对应的/arch/xxx/Kconfig文件(xxx即为传入的 ARCH 参数)，/arch/xxx/Kconfig文件中除本身包含一些与体系结构相关的配置项和配置菜单以外，还通过 source 语句引入了一系列 Kconfig 文件，而这些 Kconfig 又可能再次通过 source 引入下一层的 Kconfig。配置工具依据这些 Kconfig 包含的菜单和项目即可描绘出一个分层结构。

#### 3.4.2 Kconfig 和 Makefile

在 Linux 内核中增加程序需要完成以下3项工作。

- 将编写的源代码拷入Linux内核源代码的相应目录。
- 在目录的Kconfig文件中增加关于新源代码对应项目的编译配置选项。
- 在目录的Makefile文件中增加对新源代码的编译条目。

**1、实例引导：S3C6410 处理器的 RTC 驱动配置**

在讲解 Kconfig 和 Makefile 的语法之前,我们先利用两个简单的实例引导读者建立初步的认识。首先在linux-2.6.28-samsung/drivers/rtc目录中包含了S3C6410处理器的RTC设备驱动源代码`rtc-s3c.c`。

而在该目录的 Kconfig 文件中包含关于 RTC_DRV_S3C 的配置项目:

    config RTC_DRV_S3C
     tristate "Samsung S3C series SoC RTC"
     depends on ARCH_S3C2410 || ARCH_S3C64XX || ARCH_S5PC1XX || ARCH_S5P64XX
     help
        RTC (Realtime Clock) driver for the clock inbuilt into the Samsung S3C24XX series of SoCs. This can provide periodic interrupt rates from 1Hz to 64Hz for user programs, and wakeup from Alarm.
        The driver currently supports the common features on all the S3C24XX range, such as the S3C2410, S3C2412, S3C2413, S3C2440 and S3C2442.
        This driver can also be build as a module. If so, the module will be called rtc-s3c.

上述 Kconfig 文件的这段脚本意味着只有在 ARCH_S3C2410、ARCH_S3C64XX、ARCH_S5PC1XX 或 ARCH_S5P64XX 项目之一被配置的情况下，才会出现 RTC_DRV_S3C 配置项目。这个配置项目为三态（可编译入内核、可不编译、也可编译为内核模块，选项分别为“Y”、“N”和“M”）。菜单上显示的字符串为“Samsung S3C series SoC RTC”。“help”后面的内容为帮助信息。

还存在一种布尔型(bool)配置选项。它意味着要么编译入内核要么不编译，选项为“Y”或“N”。

在目录的 Makefile 中关于 RTC_DRV_S3C 的编译脚本为：

	obj-$(CONFIG_RTC_DRV_S3C) += rtc-s3c.o

上述脚本意味着如果RTC_DRV_S3C配置选项被选择为“Y”或“M”，即obj-$(CONFIG_RTC_DRV_S3C)等同于obj-y或obj-m时，则编译rtc-s3c.c。选“Y”的情况直接会将生成的目标代码直接连接到内核。为“M”的情况则会生成模块rtc-s3c.ko；如果 RTC_DRV_S3C 配置选项被选择为“N”，即obj-$(CONFIG_RTC_DRV_S3C)等同于 obj-n 时则不编译 rtc-s3c.c。

一般而言驱动工程师只会在内核源代码的 drivers 目录的相应子目录中增加新设备驱动的源代码。并增加或修改 Kconfig 配置脚本和 Makefile 脚本。完全仿照上述过程执行即可。

**2、Makefile**

这里主要对内核源代码各级子目录中的kbuild(内核的编译系统)Makefile进行简单介绍，这部分是内核模块或设备驱动的开发者最常接触到的。Makefile的语法包括如下几个方面。

（1）目标定义

目标定义就是用来定义哪些内容要作为模块编译，哪些要编译并连接进内核。例如:

	obj-y += foo.o

表示要由 foo.c 或者 foo.s 文件编译得到 foo.o 并连接进内核。而 `obj-m` 则表示该文件要作为模块编译。除了y、m以外的obj-x形式的目标都不会被编译。而更常见的做法是根据.config文件的`CONFIG_`变量来决定文件的编译方式，如：

    obj-$(CONFIG_ISDN) += isdn.o
    obj-$(CONFIG_ISDN_PPP_BSDCOMP) += isdn_bsdcomp.o

除了obj-形式的目标以外,还有 lib-y library 库，hostprogs-y 主机程序等目标。但是基本都应用在特定的目录和场合下。

（2）多文件模块的定义

最简单的 Makefile 如上一节一句话的形式就够了。如果一个模块由多个文件组成会稍微复杂一些。这时候应采用模块名加-y或-objs后缀的形式来定义模块的组成文件。如以下例子：

    # Makefile for the linux ext2-filesystem routines.
    obj-$(CONFIG_EXT2_FS) += ext2.o
    ext2-y := balloc.o dir.o file.o fsync.o ialloc.o inode.o \
    ioctl.o namei.o super.o symlink.o
    ext2-$(CONFIG_EXT2_FS_XATTR) += xattr.o xattr_user.o xattr_trusted.o
    ext2-$(CONFIG_EXT2_FS_POSIX_ACL) += acl.o
    ext2-$(CONFIG_EXT2_FS_SECURITY) += xattr_security.o
    ext2-$(CONFIG_EXT2_FS_XIP) += xip.o

模块的名字为ext2。由balloc.o、dir.o、file.o等多个目标文件最终链接生成 ext2.o直至ext2.ko文件。并且是否包括xattr.o、acl.o等则取决于内核配置文件的配置情况。例如如果`CONFIG_ EXT2_FS_POSIX_ACL`被选择则编译acl.c得到acl.o并最终链接进ext2。

（3）目录层次的迭代

如下例：

	obj-$(CONFIG_EXT2_FS) += ext2/

当 CONFIG_EXT2_FS 的值为 y 或 m 时，kbuild将会把ext2目录列入向下递归的目标中。

**（未）3、Kconfig内核配置脚本文件的语法也比较简单，主要包括如下几个方面。**

**（未）4、应用实例:在内核中新增驱动代码目录和子目录**

#### 3.4.3 Linux内核的引导

引导Linux系统的过程包括很多阶段。这里将以引导X86 PC为例来进行讲解。引导X86 PC上的Linux的过程和引导嵌入式系统上的Linux的过程基本类似。不过在X86 PC上有一个从BIOS转移到Bootloader的过程。而嵌入式系统往往复位后就直接运行 **Bootloader**。

从上电/复位到运行Linux用户空间初始进程，流程如下：系统启动（BIOS）、Bootloader的第一阶段（MBR）、Bootloader的第二阶段（LILO、GRUB等）、启动内核、运行init进程。

1、当系统上电或复位时，CPU会将PC指针赋值为一个特定的地址`0xFFFF0`并执行该地址处的指令。在PC机中，该地址位于BIOS中，它保存在主板上的ROM或Flash中。
2、BIOS运行时按照CMOS的设置定义的启动**设备**顺序来搜索处于活动状态并且可以引导的设备。若从硬盘启动，BIOS会将硬盘**MBR（主引导记录）**中的内容加载到RAM。**MBR是一个512字节大小的扇区**，位于磁盘上的第一个扇区中（0道0柱面1扇区）。当MBR被加载到RAM中之后，BIOS就会将控制权交给MBR。
3、主引导加载程序查找并加载次引导加载程序。它在分区表中查找活动分区，当找到一个活动分区时，扫描分区表中的其他分区，以确保它们都不是活动的。当这个过程验证完成之后，就将活动分区的**引导记录**从这个设备中读入RAM中并执行它。
4、次引导加载程序加载Linux内核和可选的初始RAM磁盘，将控制权交给Linux内核源代码。
5、运行被加载的内核，并启动用户空间应用程序。

嵌入式系统中Linux的引导过程与之类似，但一般更加简洁。不论具体以怎样的方式实现，只要具备如下特征就可以**称其为**Bootloader：

- 可以在系统上电或复位的时候以某种方式执行，这些方式包括被BIOS引导执行、直接在NOR Flash中执行、NAND Flash中的代码被MCU自动拷入内部或外部RAM执行等。
- 能将U盘、磁盘、光盘、NOR/NAND Flash、ROM、SD卡等存储介质，甚或网口、串口中的操作系统加载到RAM并把控制权交给操作系统源代码执行

完成上述功能的 Bootloader的实现方式非常多样化，甚至本身也可以是一个简化版的操作系统。著名的Linux Bootloader包括应用于PC的LILO和GRUB，应用于嵌入式系统的**U-Boot**、**RedBoot**等。

相比较于LILO，GRUB本身能理解EXT2、EXT3文件系统，因此可在文件系统中加载 Linux，而LILO只能识别“裸扇区”。

**U-Boot**的定位为“Universal Bootloader”，其功能比较强大，涵盖了包括PowerPC、ARM、MIPS和X86在内的绝大部分处理器构架，提供网卡、串口、 Flash等外设驱动，提供必要的网络协议（BOOTP、DHCP、TFTP)。能识别多种文件系统，并附带了调试、脚本、引导等工具，应用十分广泛。

**Redboot**是Redhat公司随eCos发布的Bootloader开源项目，除了包含U-Boot类似的强大功能外，它还包含GDB stub，因此能通过串口或网口与GDB进行通信，调试 GCC 产生的任何程序（包括内核）。

我们有必要对上述流程的第5个阶段进行更详细的分析，它完成启动内核并运行用户空间的init进程。当内核映像被加载到RAM之后，Bootloader的控制权被释放，内核阶段就开始了。内核映像并不是完全可直接执行的目标代码，而是一个压缩过的zImage（小内核）或 bzImage（大内核，bzImage中的b是“big”的意思）。

但是，并非zImage和bzImage映像中的一切都被压缩了，否则Bootloader把控制权交给这个内核映像它就“傻”了。实际上映像中包含未被压缩的部分，这部分中包含解压缩程序，解压缩程序会解压映像中被压缩的部分。zImage和bzImage都是用gzip压缩的，它们不仅是一个压缩文件，而且在这两个文件的开头部分内嵌有gzip解压缩代码。

当bzImage（用于i386映像）被调用时，它从 /arch/i386/boot/head.S 的`start`汇编例程开始执行。这个程序执行一些基本的硬件设置，并调用/arch/i386/boot/compressed/head.S 中的`startup_32`例程。`startup_32`程序设置一些基本的运行环境（如堆栈）后，清除BSS段，调用 /arch/i386/boot/compressed/misc.c中的`decompress_kernel()` C 函数解压内核。内核被解压到内存中之后，会再调用/arch/i386/kernel/head.S 文件中的 `startup_32` 例程，这个新的`startup_32`例程（称为清除程序或进程0）会初始化页表，并启用内存分页机制，接着为任何可选的浮点单元（FPU）检测CPU的类型，并将其存储起来供以后使用。这些都做完之后，**/init/main.c**中的`start_kernel()`函数被调用，进入与体系结构无关的Linux内核部分。

`start_kernel()`会调用一系列初始化函数来设置中断，执行进一步的内存配置。之后，/arch/i386/kernel/process.c 中 kernel_thread()被调用以启动第一个核心线程，该线程执行init()函数，而原执行序列会调用cpu_idle()等待调度。

作为核心线程的`init()`函数完成外设及其驱动程序的加载和初始化，挂接根文件系统。init()打开/dev/console设备，重定向stdin、stdout和stderr到控制台。之后它搜索文件系统中的init程序（也可以由“init=”命令行参数指定init程序），并使用`execve()`系统调用执行init程序。搜索init程序的顺序为： /sbin/init、/etc/init、/bin/init和/bin/sh。在嵌入式系统中，多数情况下，可以给内核传入一个简单的 shell 脚本来启动必需的嵌入式应用程序。
至此漫长的Linux内核引导和启动过程就此结束，而init()对应的这个由 `start_kernel()`创建的第一个线程也进入用户模式。

### 3.5 Linux下的C编程特点

#### （未）3.5.2 GNU C与ANSI C




