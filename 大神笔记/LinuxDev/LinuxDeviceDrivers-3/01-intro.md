[toc]

## 1. 设备驱动介绍

We introduce new ideas gradually, starting off with very simple drivers and building on them; every new concept is accompanied by sample code that **doesn’t need special hardware** to be tested.

### 1.1 设备驱动的角色

**设备驱动的角色是提供机制（mechanism），而非策略（policy）。**

机制与策略的区别，是Unix设计的最佳思想之一。多数编程问题可以被分为两部分：“要提供什么功能呢”（机制）和“如何使用这些功能”（策略）。If the two issues are addressed by different parts of the program, or even by different programs altogether, the software package is much easier to develop and to adapt to particular needs.

例如，Unix对图像显示的管理，分别交给X server（了解硬件，向用户程序提供统一的接口）和窗口、会话管理器（实现特定策略，不需要了解硬件）。People can use the same window manager on different hardware, and different users can run different configurations on the same workstation. Even completely different desktop environments, such as KDE and GNOME, can coexist on the same system. Another example is the layered structure of TCP/IP networking: the operating system offers the socket abstraction, which implements no policy regarding the data to be transferred, while different servers are in charge of the services (and their associated policies). Moreover, a server like *ftpd* provides the file transfer mechanism, while users can use whatever client they prefer; both command-line and graphic clients exist, and anyone can write a new user interface to transfer files.

分离同样作用于驱动。例如软盘驱动不管策略：它的角色只是将盘片作为连续的数据块展示。系统更高层提供策略，如访问权限控制，直接访问还是通过文件系统。Since different environments usually need to use hardware in different ways, it’s important to be as policy free as possible.

When writing drivers, a programmer should pay particular attention to this fundamental concept: write kernel code to access the hardware, but don’t force particular policies on the user, since different users have different needs. The driver should deal with making the hardware available, leaving all the issues about how to use the hardware to the applications. A driver, then, is flexible if it offers access to the hardware capabilities without adding constraints. Sometimes, however, some policy decisions must be made. For example, a digital I/O driver may only offer byte-wide access to the hardware in order to avoid the extra code needed to handle individual bits.

You can also look at your driver from a different perspective: it is a software layer that lies between the applications and the actual device. This privileged role of the driver allows the driver programmer to choose exactly how the device should appear: different drivers can offer different capabilities, even for the same device. The actual driver design should be a balance between many different considerations. For instance, a single device may be used concurrently by different programs, and the driver programmer has complete freedom to determine how to handle concurrency. You could implement memory mapping on the device independently of its hardware capabilities, or you could provide a user library to help application programmers implement new policies on top of the available primitives, and so forth. One major consideration is the trade-off between the desire to present the user with as many options as possible and the time you have to write the driver, as well as the need to keep things simple so that errors don’t creep in.

Policy-free drivers have a number of typical characteristics. 包括支持同步和异步操作，能被多次打开，充分利用硬件的全部能力，and the lack of software layers to “simplify things” or provide policy-related operations. Drivers of this sort not only work better for their end users, but also turn out to be easier to write and maintain as well. Being policy-free is actually a common target for software designers.

Many device drivers, indeed, are released together with user programs to help with configuration and access to the target device. Those programs can range from simple utilities to complete graphical applications. Examples include the tunelp program, which adjusts how the parallel port printer driver operates, and the graphical cardctl utility that is part of the PCMCIA driver package. Often a client library is provided as well, which provides capabilities that do not need to be implemented as part of the driver itself.

The scope of this book is the kernel, so we try not to deal with policy issues or with application programs or support libraries. Sometimes we talk about different policies and how to support them, but we won’t go into much detail about programs using the device or the policies they enforce. You should understand, however, that user programs are an integral part of a software package and that even policy-free packages are distributed with configuration files that apply a default behavior to the underlying mechanisms.

### 1.2 分解内核

内核的角色可变分解为几个部分：

**进程管理**：内核负责创建和销毁进程，处理其输入和输出；负责进程间通信。

**内存管理**：内核为所有进程搭建一个虚拟地址空间。The different parts of the kernel interact with the memory-management subsystem through a set of function calls, ranging from the simple **malloc/free** pair to much more complex functionalities.

**文件系统**：Unix非常依赖文件系统概念。

**设备控制**：The kernel must have embedded in it a device driver for every peripheral present on a system. This aspect of the kernel’s functions is our primary interest in this book.

**网络**：网络必须有操作系统管理，因为多数网络操作不是特定进程才有的：incoming packets are asynchronous events. The packets must be collected, identified, and dispatched before a process takes care of them. 而且，所有的路由和解析都是由内核实现的。

#### 1.2.1 可加载的模块

Linux允许在运行时向内核添加（和删除）功能。它们称为模块（`module`）。设备驱动就是模块的一种。Each module is made up of object code (not linked into a complete executable) that can be dynamically linked to the running kernel by the `insmod` program and can be unlinked by the `rmmod` program.

### 1.3 设备类型与模块

有三种基本的设备类型。模块一般实现其中某个，因此模块有char module, block module和network module三种。

**字符设备**：以字节流的形式访问（类似于文件）；此类驱动一般至少实现`open`、`close`、`read`和`write`几个系统调用。字符设备有/dev/console、串口（/dev/ttyS0等）。字符设备通过文件系统节点访问，如`/dev/tty1`和`/dev/lp0`。

**块设备**：块设备也是通过`/dev`下的文件系统节点访问。块设备可以容纳文件系统。块设备一般只能一次读取一个块，如512字节。但Linux允许像字符设备一样逐个字节读取块设备。因此对于用户来说块设备与字符设备区别不大。

**网络设备**：Not being a stream-oriented device, a network interface isn’t easily mapped to a node in the filesystem. The Unix way to provide access to interfaces is still by assigning a unique name to them (such as `eth0`), but that name doesn’t have a corresponding entry in the filesystem. 内核与网络设备驱动的通讯与字符或块驱动完全不同。Instead of `read` and `write`, the kernel calls functions related to packet transmission.

There are other ways of classifying driver modules that are orthogonal to the above device types. In general, some types of drivers work with additional layers of kernel support functions for a given type of device. For example, one can talk of universal serial bus (USB) modules, serial modules, SCSI modules, and so on. Every USB device is driven by a USB module that works with the USB subsystem, but the device itself shows up in the system as a char device (a USB serial port, say), a block device (a USB memory card reader), or a network device (a USB Ethernet interface).

Other classes of device drivers have been added to the kernel in recent times, including FireWire drivers and I2C drivers. In the same way that they handled USB and SCSI drivers, kernel developers collected class-wide features and exported them to driver implementers to avoid duplicating work and bugs, thus simplifying and strengthening the process of writing such drivers.

### 1.4 安全问题

这里讨论几个普遍的概念。具体问题虽本书讲到时再涉及。

Any security check in the system is enforced by kernel code. If the kernel has secu- rity holes, then the system as a whole has holes. In the official kernel distribution, only an authorized user can load modules; the system call `init_module` checks if the invoking process is authorized to load a module into the kernel. Thus, when running an official kernel, only the superuser, or an intruder who has succeeded in becoming privileged, can exploit the power of privileged code.
When possible, driver writers should avoid encoding security policy in their code. Security is a policy issue that is often best handled at higher levels within the kernel, under the control of the system administrator. There are always exceptions, however.

As a device driver writer, you should be aware of situations in which some types of device access could adversely affect the system as a whole and should provide adequate controls. For example, device operations that affect global resources (such as setting an interrupt line), which could damage the hardware (loading firmware, for example), or that could affect other users (such as setting a default block size on a tape drive), are usually only available to sufficiently privileged users, and this check must be made in the driver itself.

Driver writers must also be careful, of course, to avoid introducing security bugs. The C programming language makes it easy to make several types of errors. Many current security problems are created, for example, by buffer overrun errors, in which the programmer forgets to check how much data is written to a buffer, and data ends up written beyond the end of the buffer, thus overwriting unrelated data. Such errors can compromise the entire system and must be avoided. Fortunately, avoiding these errors is usually relatively easy in the device driver context, in which the interface to the user is narrowly defined and highly controlled.

Some other general security ideas are worth keeping in mind. Any input received from user processes should be treated with great suspicion; never trust it unless you can verify it. Be careful with uninitialized memory; any memory obtained from the kernel should be zeroed or otherwise initialized before being made available to a user process or device. Otherwise, information leakage (disclosure of data, passwords, etc.) could result. If your device interprets data sent to it, be sure the user cannot send anything that could compromise the system. Finally, think about the possible effect of device operations; if there are specific operations (e.g., reloading the firmware on an adapter board or formatting a disk) that could affect the system, those operations should almost certainly be restricted to privileged users.

Be careful, also, when receiving software from third parties, especially when the kernel is concerned: because everybody has access to the source code, everybody can break and recompile things. Although you can usually trust precompiled kernels found in your distribution, you should avoid running kernels compiled by an untrusted friend—if you wouldn’t run a precompiled binary as root, then you’d better not run a precompiled kernel. For example, a maliciously modified kernel could allow anyone to load a module, thus opening an unexpected back door via `init_module`.

Note that the Linux kernel can be compiled to have no module support whatsoever, thus closing any module-related security holes. In this case, of course, all needed drivers must be built directly into the kernel itself. It is also possible, with 2.2 and later kernels, to disable the loading of kernel modules after system boot via the capability mechanism.

### 1.5 版本号

To run the examples we introduce during the discussion, you won’t need particular versions of any tool beyond what the 2.6 kernel requires; any recent Linux distribution can be used to run our examples.

This book covers Version 2.6 of the kernel. Our focus has been to show all the features available to device driver writers in 2.6.10, the current version at the time we are writing.

Kernel programmers should be aware that the development process changed with 2.6. Among other things, that means that internal kernel programming interfaces can change, thus potentially obsoleting parts of this book; for this reason, the sample code accompanying the text is known to work with 2.6.10, but some modules don’t compile under earlier versions. There is also a web page maintained at http://lwn.net/Articles/2.6-kernel-api/, which contains information about API changes that have happened since this book was published.

This book is platform independent as far as possible, and all the code samples have been tested on at least the x86 and x86-64 platforms. As you might expect, the code samples that rely on particular hardware don’t work on all the supported platforms, but this is always stated in the source code.

### 1.6 License Terms

Linux is licensed under Version 2 of the GNU General Public License (GPL).

All the programs are available at ftp://ftp.ora.com/pub/examples/linux/drivers/.

### 1.7 加入内核开发社区

The central gathering point for Linux kernel developers is the linux-kernel mailing list. All major kernel developers, from Linus Torvalds on down, subscribe to this list. Please note that the list is not for the faint of heart: traffic as of this writing can run up to 200 messages per day or more. Nonetheless, following this list is essential for those who are interested in kernel development; it also can be a top-quality resource for those in need of kernel development help.

To join the linux-kernel list, follow the instructions found in the linux-kernel mailing list FAQ: http://www.tux.org/lkml. Read the rest of the FAQ while you are at it; there is a great deal of useful information there. Linux kernel developers are busy people, and they are much more inclined to help people who have clearly done their homework first.

### 1.8 本书概览

[DONE]




