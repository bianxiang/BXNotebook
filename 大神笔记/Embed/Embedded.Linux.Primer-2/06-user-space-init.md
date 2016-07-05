[toc]

## 6. 用户空间初始化

Linux内核只是嵌入式系统中很小一部分。内核初始化后，它**必须挂载一个根文件系统**，再执行开发者定义的初始化。

### 6.1 根文件系统

#### 6.1.1 FHS: File System Hierarchy Standard

Several kernel developers authored a standard governing the organization and layout of a UNIX file system. The File System Hierarchy Standard (FHS) establishes a minimum baseline of compatibility between Linux distributions and application programs. You are encouraged to review the FHS for a better background of the layout and rationale of UNIX file system organization.

Many Linux distributions have directory layouts closely matching that described in the FHS standard. The standard exists to provide one element of a common base between different UNIX and Linux distributions. The FHS standard allows your application software (and developers) to predict where certain system elements, including files and directories, can be found in the file system.

#### 6.1.2 文件系统布局

考虑到空间大小，很多嵌入式开发者在启动设备上创建一个较小的跟文件系统。然后从其他设备上挂载更大的文件系统，如硬盘、网络文件系统（NFS）。甚至在原来的**根文件系统**之上挂载一个更大的根文件系统。一个简单的Linux跟文件系统包含以下顶层目录：

    .
    |
    |--bin
    |--dev
    |--etc
    |--home
    |--lib
    |--sbin
    |--usr
    |--var
    |--tmp

目录用途：

- `bin`：二进制可执行文件，供系统中所有用户使用。
- `dev`：设备节点。
- `etc`：本地系统配置文件。
- `home`：用户账户文件。
- `lib`：系统库，如标准C库等
- `sbin`：二进制可执行文件，一般用于超级用户
- `tmp`：临时文件
- `usr`：二级文件系统，包含应用程序，一般是只读的
- `var`：包含可变文件，如系统日志

#### 6.1.3 最小文件系统

下面是一个最小根文件系统的组织：

    .
    |-- bin
    |     |-- busybox
    |     `-- sh -> busybox
    |-- dev
    |     `-- console
    |-- etc
    |    `-- init.d
    |         `-- rcS
    `-- lib
        |-- ld-2.3.2.so
        |-- ld-linux.so.2 -> ld-2.3.2.so
        |-- libc-2.3.2.so
         `-- libc.so.6 -> libc-2.3.2.so

This tiny root file system boots and provides the user with a fully functional command prompt **on the serial console**. 用户可以使用busybox中的任何命令。

`/bin`下有一个`busybox`可执行文件，或指向它的一个软链接`sh`。`/dev/console`是一个设备节点，用于打开一个console设备读写。`/etc/init.d/rcS`是默认的初始化脚本（不必需），在启动时由busybox处理。`rcS`的存在可以让busybox启动时不报相关的警告信息。

最后是目录和两个必需的库，glibc (libc-2.3.2.so) 和Linux动态加载器 (ld-2.3.2.so)。**glibc**包含标准C库函数，如`printf()`。Linux动态加载器负责加载二进制可执行文件到内存，并动态链接（应用到需要的共享库函数）。目录中还有两个软链接。这两个链接为了向后兼容，**是所有Linux系统都有的**。

在ARM/XScale板上，这个根文件系统只有1.7MB。其中80%是C库。If you need to reduce its size for your embedded system, you might want to investigate the Library Optimizer Tool at http://libraryopt.sourceforge.net/.

#### 6.1.6 自动构建文件系统

Some open source build tools automate the task of building a working root file system. Some of the more notable include bitbake from the OpenEmbedded project (www.openembedded.org/) and buildroot (http://buildroot.uclibc.org/.) Chapter 16, “Open Source Build Systems,” presents details of some popular build systems.

### 6.2 内核最后的启动步骤

`.../init/main.c`文件最后一部分代码如下：

LISTING 6-2 Final Boot Steps from main.c

    ...
    if (execute_command) {
        run_init_process(execute_command);
        printk(KERN_WARNING “Failed to execute %s. Attempting “
        “defaults...\n”, execute_command);
    }
    run_init_process(“/sbin/init”);
    run_init_process(“/etc/init”);
    run_init_process(“/bin/init”);
    run_init_process(“/bin/sh”);
    panic(“No init found. Try passing init= option to kernel.”);

This is the final sequence of events for the kernel thread called `kernel_init` spawned by the kernel during the final stages of boot. `run_init_process()`是对`execve()`函数的包装；后者是一个系统调用，具有特别的行为：若没有调用错误，`execve()`函数**永远不会返回**。调用（calling）线程的内存空间被被调用者（called）程序的内存镜像覆盖。实际上，被调用（called）的程序直接覆盖调用者（calling）线程，继承其Process ID (PID)。

这个初始化序列的结构在Linux内核中很久没改变了。这是用户空间处理的开始。（In actuality, modern Linux kernels create a userspace-like environment earlier in the boot sequence for specialized activities, which are beyond the scope of this book.） 除非Linux内核成功执行了上述进程中的一个，否则内核将中止，调用系统调用`panic()`。

Notice a key ingredient of these processes: They are all programs that are expected to reside on a root file system. Therefore, we know that we must at least satisfy the kernel’s requirement for an init process that can execute within its own environment.

#### 6.2.1 第一个用户空间程序

在多数Linux系统中，`/sbin/init`由内核在启动时生成（spawned）。因此这是上面代码中第一个被尝试的初始化程序。实际上，它是第一个运行的用户空间程序。To review, this is the sequence:

1. 挂载根文件系统
2. 产生（Spawn）第一个用户空间程序，这里是`/sbin/init`。

用我们的最小根文件系统，前三次`run_init_process`尝试都将失败，因为文件系统中并没有相应的`init`文件。但我们有软链接`sh`（到busybox）；它将得到执行；busybox会作为用户空间初始进程。

#### 6.2.2 解析依赖

仅仅将可执行文件（如`init`）放入文件系统是不够的。还必须满足其依赖。多数进程有两类依赖：一是动态链接的可执行文件需要满足依赖，而是外部配置或数据文件。对于第一种依赖，可以通过工具查出。

例如`init`进程是动态链接的可执行文件。运行它必须先满足其对库的依赖。`ldd`可以列出应用依赖哪些库。运行`ldd`的**交叉版本**：

    $ ppc_4xx-ldd init
    libc.so.6 => /opt/eldk/ppc_4xxFP/lib/libc.so.6
    ld.so.1 => /opt/eldk/ppc_4xxFP/lib/ld.so.1

#### 6.2.3 配置最初的进程

通过内核命令行参数，可以指定启动时第一个进程是哪个。Here is how it might look with a user-specified init process:

	console=ttyS0,115200 ip=bootp root=/dev/nfs init=/sbin/myinit

### 6.3 `init`进程

除非你要做非常不寻常的事情，否则不需要定制初始化进程，因为标准`init`进程是非常灵活的。`init`，及一组启动脚本，组成了所谓**System V Init**。现在我们来探索这个强大的配置和控制工具。

`init`是第一个用户空间进程，由内核在完成启动后产生（spawned）。`init`是所有用户空间进程的父进程。此外，`init`提供一些默认的环境变量，供其他进程继承，包括初始的系统`PATH`。

它的主要任务是依据一个特殊的配置文件产生后续进程。该配置文件一般位于`/etc/inittab`。`init`有一个叫**运行级别（runlevel）**的概念。一个运行级别可以被理解为一个系统状态。进入某个运行级别时，会启动特定服务，启动（spawn）一些程序。任意时刻`init`可以处于某个运行级别。运行级别有`0`到`6`，及一个特殊的`S`。运行级别0指示`init`中止系统，运行级别6导致系统重启。每个级别一般都有对应启动和关闭脚本。每个运行级别进行的操作由`/etc/inittab`配置文件决定。

多数Linux分发中，几个特别的运行级别都被规定了特殊含义。下面是在多数Linux分发中通用的运行级别及含义：

- 0： 系统关闭
- 1：Single-user system configuration for maintenance
- 2：User-defined
- 3：General-purpose multiuser configuration
- 4：User-defined
- 5：Multiuser with graphical user interface on startup
- 6：系统重启

运行级别的脚本一般位于`/etc/rc.d/init.d`目录中。这些脚本多数用于启用、禁用一个服务。服务可以被手工配置：调用脚本，传入参数`start`、`stop`或`restart`。Listing 6-3 displays an example of restarting the NFS service.

    $ /etc/init.d/nfs-kernel-server
    Shutting down NFS mountd: [ OK ]
    Shutting down NFS daemon: [ OK ]
    Shutting down NFS quotas: [ OK ]
    Shutting down NFS services: [ OK ]
    Starting NFS services: [ OK ]
    Starting NFS quotas: [ OK ]
    Starting NFS daemon: [ OK ]
    Starting NFS mountd: [ OK ]

当启动一个桌面Linux时，也可能看到上面的消息。

A runlevel is defined by the services that are enabled at that runlevel. Most Linux distributions contain a directory structure under `/etc` that contains symbolic links to the service scripts in `/etc/rc.d/init.d`. These runlevel directories typically are rooted at `/etc/rc.d`. Under this directory, you will find a series of runlevel directories that contain startup and shutdown specifications for each runlevel. `init` simply executes these scripts upon entry and exit from a runlevel. The scripts define the system state, and `inittab` instructs `init` which scripts to associate with a given runlevel. 下面是`/etc/rc.d`下的目录结构，that drives the runlevel startup and shutdown behavior upon entry to or exit from the specified runlevel, respectively.

    $ ls -l /etc/rc.d
    total 96
    drwxr-xr-x 2 root root 4096 Oct 20 10:19 init.d
    -rwxr-xr-x 1 root root 2352 Mar 16 2009 rc
    drwxr-xr-x 2 root root 4096 Mar 22 2009 rc0.d
    drwxr-xr-x 2 root root 4096 Mar 22 2009 rc1.d
    drwxr-xr-x 2 root root 4096 Mar 22 2009 rc2.d
    drwxr-xr-x 2 root root 4096 Mar 22 2009 rc3.d
    drwxr-xr-x 2 root root 4096 Mar 22 2009 rc4.d
    drwxr-xr-x 2 root root 4096 Mar 22 2009 rc5.d
    drwxr-xr-x 2 root root 4096 Mar 22 2009 rc6.d
    -rwxr-xr-x 1 root root 943 Dec 31 16:36 rc.local
    -rwxr-xr-x 1 root root 25509 Jan 11 2009 rc.sysinit

每个运行级别由对应`rcN.d`中的脚本定义。在每个`rcN.d`目录中，有一些按特定顺序排列的链接。这些符号链接以`K`或`S`开头。`S`开头的指向服务脚本，which are invoked with startup instructions. Those starting with K point to service scripts that are invoked with shutdown instructions. An example with a very small number of services might look like Listing 6-5.

LISTING 6-5 Sample Runlevel Directory

    lrwxrwxrwx 1 root root 17 Nov 25 2009 S10network -> ../init.d/network
    lrwxrwxrwx 1 root root 16 Nov 25 2009 S12syslog -> ../init.d/syslog
    lrwxrwxrwx 1 root root 16 Nov 25 2009 S56xinetd -> ../init.d/xinetd
    lrwxrwxrwx 1 root root 16 Nov 25 2009 K50xinetd -> ../init.d/xinetd
    lrwxrwxrwx 1 root root 16 Nov 25 2009 K88syslog -> ../init.d/syslog
    lrwxrwxrwx 1 root root 17 Nov 25 2009 K90network -> ../init.d/network

This code instructs the startup scripts to start three services upon entry to this fictitious runlevel: network, syslog, and xinetd. Because the `S*` scripts are ordered with a numeric tag, they will be started in this order. In a similar fashion, when exiting this runlevel, three services will be terminated: xinetd, syslog, and network. In a similar fashion, these services will be terminated in the order presented by the two-digit number following the K in the symlink filename. In an actual system, there would undoubtedly be many more entries. You can include your own entries for your own custom applications as well.

The top-level script that executes these service startup and shutdown scripts is defined in the init configuration file, which we now examine.

#### 6.3.1 inittab

当`init`启动后，它读取系统配置文件`/etc/inittab`。This file contains directives for each runlevel, as well as directives that apply to all runlevels. This file and init’s behavior are well documented in man pages on most Linux workstations, as well as by several books covering system administration. We do not attempt to duplicate those works; 我们只解释为一个嵌入式系统要如何配置`inittab`。For a detailed explanation of how inittab and init work together, view the man page on most Linux workstations by typing `man init` and `man inittab`.

Let’s look at a typical `inittab` for a simple embedded system. Listing 6-6 contains a simple `inittab` example for a system that supports a single runlevel as well as shutdown and reboot.

LISTING 6-6 Simple inittab

    # /etc/inittab
    # The default runlevel (2 in this example)
    id:2:initdefault:
    # This is the first process (actually a script) to be run.
    si::sysinit:/etc/rc.sysinit
    # Execute our shutdown script on entry to runlevel 0
    l0:0:wait:/etc/init.d/sys.shutdown
    # Execute our normal startup script on entering runlevel 2
    l2:2:wait:/etc/init.d/runlvl2.startup
    # This line executes a reboot script (runlevel 6)
    l6:6:wait:/etc/init.d/sys.reboot
    # This entry spawns a login shell on the console
    # Respawn means it will be restarted each time it is killed
    con:2:respawn:/bin/sh

This very simple `inittab` script describes three individual runlevels. Each runlevel is associated with a script, which must be created by the developer for the desired actions in each runlevel. When this file is read by `init`, the first script to be executed is `/etc/rc.sysinit`. This is denoted by the `sysinit` tag. Then `init` enters runlevel 2 and executes the script defined for runlevel 2. From this example, this would be `/etc/init.d/runlvl2.startup`. As you might guess from the `:wait:` tag shown in Listing 6-6, `init` waits until the script completes before continuing. When the runlevel 2 script completes, `init` spawns a shell on the console (through the /bin/sh symbolic link), as shown in the last line of Listing 6-6. The `respawn` keyword instructs `init` to restart the shell each time it detects that it has exited. Listing 6-7 shows what it looks like during boot.

LISTING 6-7 Sample Startup Messages

    ...
    VFS: Mounted root (nfs filesystem).
    Freeing init memory: 304K
    INIT: version 2.78 booting
    This is rc.sysinit
    INIT: Entering runlevel: 2
    This is runlvl2.startup
    #

The startup scripts in this example do nothing except announce themselves for illustrative purposes. Of course, in an actual system, these scripts enable features and services that do useful work! Given the simple configuration in this example, you would enable the services and applications for your particular widget in the `/etc/init.d/runlvl2.startup` script. You would do the reverse—disable your applications, services, and devices—in your shutdown and/or reboot scripts. The next section looks at some typical system configurations and the required entries in the startup scripts to enable these configurations.

#### 6.3.2 例子：Web服务器启动脚本

This example is based on `busybox`, which has a slightly different initialization behavior than `init`. These differences are covered in detail in Chapter 11. In a typical embedded appliance that contains a web server, you might want several servers available for maintenance and remote access. In this example, we enable servers for HTTP and Telnet access (via `inetd`). Listing 6-8 contains a simple `rc.sysinit` script for our hypothetical web server appliance.

LISTING 6-8 Web Server rc.sysinit

    #!/bin/sh
    echo “This is rc.sysinit”
    busybox mount -t proc none /proc
    # Load the system loggers
    /sbin/syslogd
    /sbin/klogd
    # Enable legacy PTY support for telnetd
    busybox mkdir /dev/pts
    busybox mknod /dev/ptmx c 5 2
    busybox mount -t devpts devpts /dev/pts

In this simple initialization script, we first enable the `proc` file system. The details of this useful subsystem are covered in Chapter 9. Next we enable the system loggers so that we can capture system information during operation. This is especially useful when things go wrong. The last entries enable support for the UNIX PTY subsystem, which is required for the implementation of the Telnet server used for this example.

Listing 6-9 contains the commands in the runlevel 2 startup script. This script contains the commands to enable any services we want to have operational for our appliance.

LISTING 6-9 Sample Runlevel 2 Startup Script

    #!/bin/sh
    echo “This is runlvl2.startup”
    echo “Starting Internet Superserver”
    inetd
    echo “Starting web server”
    webs &

Notice how simple this runlevel 2 startup script is. First we enable the so-called Internet superserver `inetd`, which intercepts and spawns services for common TCP/IP requests. In our example, we enabled Telnet services through a configuration file called `/etc/inetd.conf`. Then we execute the web server, here called `webs`. That’s all there is to it. Although minimal, this is a working configuration for Telnet and web services.

To complete this configuration, you might supply a shutdown script (refer to Listing 6-6), which, in this case, would terminate the web server and the Internet superserver before system shutdown. In our sample scenario, that is sufficient for a clean shutdown.

### 6.4 Initial RAM Disk

Linux内核包含两个机制，挂载早期的根文件系统，进行某些启动相关的初始化和配置。首先我们讨论遗留的方法，initial ramdisk，或称`initrd`。下一节讲新方法，称为**initramfs**。

The legacy method for enabling early user space processing is known as the initial RAM disk, or simply initrd. 对该功能的支持必须被编译进内核。This kernel configuration option is found under General Setup, RAM disk support in the kernel configuration utility.

initial ramdisk是一个小巧的、自包含的根文件系统。一般包含一些指令，在完成启动前，加载特定设备驱动。Red Hat、Ubuntu等系统，initial ramdisk一般用于为EXT3文件系统加载设备驱动，然后再挂载真实的根文件系统。`initrd`常用于加载一个设备驱动，**有这个驱动才能访问真实的根文件系统**。

#### 6.4.1 用initrd启动

To use the initrd functionality, the bootloader gets involved on most architectures to pass the `initrd` image to the kernel. 一个场景的场景是，bootloader加载一个压缩的内核镜像到内存，然后加载`initrd`镜像到内存的另一个区域。此时，bootloader需要负责在控制权交给内核前，把`initrd`的地址｛｛内存地址？Flash上的地址｝｝传给内核。具体的机制取决于架构、bootloader和平台实现。但内核必须知道`initrd`镜像的位置，才能加载它。

若由于bootloader不支持加载`initrd`镜像，或其他原因，内核和`initrd`被压入一个镜像。You will find reference to this type of composite image in the kernel makefiles as `bootpImage`. 目前只有ARM架构用它。

让内核知道`initrd`镜像位置（和大小），最简单的方法是通过内核命令行参数。Here is an example of a kernel command line for a popular ARM-based reference board containing the TI OMAP 5912 processor:

    console=ttyS0,115200 root=/dev/nfs \
    	nfsroot=192.168.1.9:/home/chris/sandbox/omap-target \
    	initrd=0x10800000,0x14af47

解释：

- Specify a single console on device ttyS0 at 115 kilobaud.
- 挂载一个网络文件系统做根文件系统
- NFS在192.168.1.9的/home/chris/sandbox/omap-target目录
- 加载并挂载一个初始化ramdisk，从物理内存0x10800000，大小是0x14AF47 (1,355,591 bytes)。

`initrd`镜像基本都是压缩的。上面的大小是压缩后的大小。

#### 6.4.2 Bootloader对initrd的支持

Let’s look at a simple example based on the popular U-Boot bootloader running on an ARM processor. This bootloader was designed with support for directly booting the Linux kernel. Using U-Boot, it is easy to include an initrd image with the kernel image. Listing 6-10 shows a typical boot sequence containing an initial ramdisk image.

LISTING 6-10 Booting the Kernel with Ramdisk Support

    [uboot]> tftp 0x10000000 kernel-uImage
    ...
    Load address: 0x10000000
    Loading: ############################ done
    Bytes transferred = 1069092 (105024 hex)

    [uboot]> tftp 0x10800000 initrd-uboot
    ...
    Load address: 0x10800000
    Loading: ########################################### done
    Bytes transferred = 282575 (44fcf hex)

    [uboot]> bootm 0x10000000 0x10800040
    Uncompressing kernel.................done.
    ...
    RAMDISK driver initialized: 16 RAM disks of 16384K size 1024 blocksize
    ...
    RAMDISK: Compressed image found at block 0
    VFS: Mounted root (ext2 filesystem).
    Greetings: this is linuxrc from Initial RAMDisk
    Mounting /proc filesystem
    BusyBox v1.00 (2005.03.14-16:37+0000) Built-in shell (ash)
    Enter ‘help’ for a list of built-in commands.
    # (<<<< Busybox command prompt)

首先用`tftp`下载内核镜像。The kernel image is downloaded and placed into the base of this target system’s memory at the 256MB address (0x10000000 hex). 然后下载初始ramdisk镜像，at a higher memory address (256MB + 8MB in this example). 最后发出U-Boot的`bootm`命令，即从内存启动。`bootm`命令的两个参数，即Linux内核镜像的地址，及初始ramdisk镜像的地址。

U-Boot除了支持通过以太网下载内核和ramdisk镜像外，还可以通过Flash、串口。但由于这些镜像较大（内核大约1M，ramdisk大约10M），网络下载才是较快的。

#### 6.4.3 initrd Magic: linuxrc

内核启动时，它首先侦测`initrd`是否存在。Then it copies the compressed binary file from the specified physical location in RAM into a proper kernel ramdisk and mounts it as the root file system. `initrd`的魔法来自于`initrd`镜像内一个特殊的文件：linuxrc。内核挂载好initial ramdisk，即寻找该文件。它将该文件作为脚本执行。Listing 6-11 shows a sample linuxrc file.

LISTING 6-11 Sample linuxrc File

    #!/bin/sh
    echo ‘Greetings: this is ‘linuxrc’ from Initial Ramdisk’
    echo ‘Mounting /proc filesystem’
    mount -t proc /proc /proc
    busybox sh

实际中，该文件在挂载真实的根文件系统去，将包含一些必需的指令。比如，先加载CompactFlash驱动，然后才能读取CompactFlash设备，从中获取根文件系统。上面的例子，最后只是创建（spawn）了一个busybox shell，中止 and halt the boot process for examination. 如果你输入`exit`命令，内核将继续其启动进程直到完成。

当内核把ramdisk从物理内存拷贝到kernel ramdisk后，占据的物理内存返回给内存池。你可以把这个过程看做，将initrd镜像从物理内存硬编码的地址转移到内核的虚拟内存中（以kernel ramdisk设备的形式）。

One last comment about Listing 6-11: The `mount` command in which the `/proc` file system is mounted seems redundant in its use of the word `proc`. 下面的命令也是可行的：

	mount -t proc none /proc

注意到`mount`命令的`device`字段被改成了`none`。`mount`命令忽略`device`字段，因为实际没有任何物理设备与`proc`文件系统关联。`-t proc`已经足够让`mount`知道将`/proc`文件系统挂载到`/proc`挂载点。之前那种写法只是在听我们，将内核的伪设备（`/proc`文件系统）挂载到`/proc`。哪种方法都行。

#### 6.4.4 The initrd Plumbing

As part of the Linux boot process, the kernel must locate and mount a root file system. Late in the boot process, the kernel decides what and where to mount in a function called `prepare_namespace()`, which is found in `.../init/do_mounts.c`. If `initrd` support is enabled in the kernel, as illustrated in Figure 6-1, and the kernel command line is so configured, the kernel decompresses the compressed `initrd` image from physical memory and eventually copies the contents of this file into a ramdisk device (`/dev/ram`). At this point, we have a proper file system on a kernel ramdisk. After the file system has been read into the ramdisk, 内核最终将该ramdisk设备挂载为根文件系统。Finally, the kernel spawns a kernel thread to execute the `linuxrc` file on the `initrd` image.

当`linuxrc`脚本执行完成后，内核卸载（unmount）`initrd`，然后继续系统启动的最后一个阶段。如果真实的根设备有目录`/initrd`，Linux将`initrd`文件系统挂载到该路径。若最终的根文件系统中没有该目录，`initrd`镜像最终被丢弃。

若内核命令行包含参数`root=`，指定一个ramdisk（如`root=/dev/ram0`），则之前描述的`initrd`将有两方面的改变。首先，processing of the linuxrc executable is skipped. 不会尝试将另一个文件系统挂载到根。即你的系统只能让`initrd`作为根文件系统。This is useful for minimal system configurations in which the only root file system is the ramdisk. Placing `/dev/ram0` on the kernel command line allows the full system initialization to complete with `initrd` as the final root file system.

#### 6.4.5 构建一个`initrd`镜像

Constructing a suitable root file system image is one of the more challenging aspects of embedded systems. 创建一个合适的`initrd`镜像则是更具有挑战的事情。它不能太大。This section examines initrd requirements and file system contents.

Listing 6-12 was produced by running the tree utility on our sample `initrd` image from this chapter.

LISTING 6-12 Contents of a Sample initrd

    .
    |-- bin
    |     |-- busybox
    |     |-- echo -> busybox
    |     |-- mount -> busybox
    |     `-- sh -> busybox
    |-- dev
    |     |-- console
    |     |-- ram0
    |     `-- ttyS0
    |-- etc
    |-- linuxrc
    `-- proc
    4 directories, 8 files

它很小，只有500KB（未压缩）。因为它是基于busybox，因此它的功能还是很丰富的。由于这里`busybox`是静态链接的，因此它不依赖任何系统库。

### 6.5 使用`initramfs`

`initramfs`是执行早期用户空间程序更好的机制。它与`initrd`概念类似。使用它也需要激活内核配置中对应项（参见上一节）。它的目的也很类似：加载一些驱动，这些驱动是挂载真实跟文件系统需要的。

但它与`initrd`的机制非常不同。它们的技术实现细节非常不同。例如，`initramfs`在`do_basic_setup()`调用之前加载，which provides a mechanism for loading firmware for devices before its driver has been loaded. For more details, see the Linux kernel documentation for this subsystem at `.../Documentation/filesystems/ramfs-rootfs-initramfs.txt`.

> `do_basic_setup` is called from `.../init/main.c` and calls `do_initcalls()`. This causes driver module initialization routines to be called. This was described in detail in Chapter 5 and shown in Listing 5-10.

从实践的角度看`initramfs`更易用。`initramfs`是一个cpio归档文件，而`initrd`是gzip压缩的文件系统镜像。This simple difference contributes to the ease of use of `initramfs` and removes the requirement that you must be root to create it. It is integrated into the Linux kernel source tree, and a small default (nearly empty) image is built automatically when you build the kernel image. 修改它比构建和加载一个新的`initrd`镜像简单的多。

下面是Linux内核源码的`.../usr`目录，是`initramfs`镜像构建的地方。下面的内容是内核构建完成后出现的。

LISTING 6-13 Kernel `initramfs` Build Directory

    $ ls -l usr
    total 72
    -rw-r--r-- 1 chris chris 1146 2009-12-16 12:36 built-in.o
    -rwxr-xr-x 1 chris chris 15567 2009-12-16 12:36 gen_init_cpio
    -rw-r--r-- 1 chris chris 12543 2009-12-16 12:35 gen_init_cpio.c
    -rw-r--r-- 1 chris chris 1024 2009-06-24 10:57 initramfs_data.bz2.S
    -rw-r--r-- 1 chris chris 512 2009-12-16 12:36 initramfs_data.cpio
    -rw-r--r-- 1 chris chris 1023 2009-06-24 10:57 initramfs_data.gz.S
    -rw-r--r-- 1 chris chris 1025 2009-06-24 10:57 initramfs_data.lzma.S
    -rw-r--r-- 1 chris chris 1158 2009-12-16 12:36 initramfs_data.o
    -rw-r--r-- 1 chris chris 1021 2009-06-24 10:57 initramfs_data.S
    -rw-r--r-- 1 chris chris 4514 2009-06-24 10:57 Kconfig
    -rw-r--r-- 1 chris chris 2154 2009-12-16 12:35 Makefile

脚本`.../scripts/gen_initramfs_list.sh`定义了包含在`initramfs`归档文件的一组默认的文件。最近的内核版本默认包含的文件如下：

LISTING 6-14 Sample `initramfs` File Specification

    dir /dev 0755 0 0
    nod /dev/console 0600 0 0 c 5 1
    dir /root 0700 0 0

这是一个很小的目录结构，包含`/root`和`/dev`两个顶级的目录，及一个表示console的设备节点。The details of how to specify items for `initramfs` file systems are described in the kernel documentation at `.../Documentation/filesystems/ramfs-rootfs-initramfs.txt`. In summary, the preceding listing produces a directory entry (dir) called `/dev`, with 0755 file permissions and a user-id and group-id of 0 (root.) The second line defines a device node (nod) called `/dev/console`, with file permissions of 0600, user and group IDs of 0 (root), being a character device (c) with major number 5 and minor number 1. The third line creates another directory called `/root` similar to the `/dev` specifier.

#### 6.5.1 定制`initramfs`

定制`initramfs`有两种方式。或根据你要求的文件创建一个cpio归档文件，或指定一组目录、文件，合并到`gen_initramfs_list.sh`创建的默认文件结构中。You specify a source for your `initramfs` files via the kernel-configuration facility. 在内核配置中启用`INITRAMFS_SOURCE`，将其指向你开发环境中的一个位置。内核构建系统将使用这些文件作为`initramfs`镜像的源。Let’s see what this looks like using a minimal file system similar to the one built in Listing 6-1.

First, we will build a file collection containing the files we want for a minimal system. Because `initramfs` is supposed to be small and lean, 我们的构建将以一个静态编译的`busybox`为中心。静态编译`busybox`使得它不依赖任何系统库。We need very little beyond `busybox`: a device node for the console in a directory called `/dev` and a symlink pointing back to busybox called `init`. Finally, we’ll include a `busybox` startup script to spawn a shell for us to interact with after booting into this `initramfs`. Listing 6-15 details this minimal file system.

LISTING 6-15 Minimal initramfs Contents

    $ tree ./usr/myinitramfs_root/
    .
    |-- bin
    |    |-- busybox
    | ‘-- sh -> busybox
    |-- dev
    | ‘-- console
    |-- etc
    | ‘-- init.d
    | ‘-- rcS
    `-- init -> /bin/sh
    4 directories, 5 files

配置好`INITRAMFS_SOURCE`，自动将`initramfs`构建成一个cpio归档，链接进内核镜像。

符号链接`init`的意义如下：当内核配置了`initramfs`，它自动搜索`initramfs`镜像下的可执行文件`/init`。如有，作为`init`进程执行（PID为1）。若没有，it skips initramfs and proceeds with normal root file system processing. 这段逻辑来自`.../init/main.c`。A character pointer called `ramdisk_execute_command` contains a pointer to this initialization command. By default it is set to the string `"/init"`.

内核命令行参数`rdinit=`，若设置，覆盖上面默认`init`位置；类似于`init=`的作用。For example, we could have set `rdinit=/bin/sh` on our kernel command line to directly call the busybox shell applet.

### 6.6 关闭

嵌入式系统的关闭常被忽视。不合理的关闭会影响启动时间，设置破坏某些文件类型。对EXT2文件系统的一大抱怨是，若突然断电后启动，做**fsck** (file system check)的时间非常长。Servers with large disk systems can take many hours to properly **fsck** through a collection of large EXT2 partitions.

每个嵌入式工程的关闭策略几乎都应该不太相同。The scale of shutdown can range from a full System V shutdown scheme, to a simple script, to halt or reboot. Several Linux utilities are available to assist in the shutdown process, including the `shutdown`, `halt`, and `reboot` commands. Of course, these must be available for your chosen architecture.

A shutdown script should terminate all user space processes, which results in closing any open files used by those processes. If `init` is being used, issuing the command `init 0` halts the system. In general, the shutdown process first sends all processes the `SIGTERM` signal to notify them that the system is shutting down. A short delay ensures that all processes have the opportunity to perform their shutdown actions, such as closing files, saving state, and so on. Then all processes are sent the `SIGKILL` signal, which results in their termination. The shutdown process should attempt to unmount any mounted file systems and call the architecture-specific halt or reboot routines. The Linux shutdown command in conjunction with init exhibits this behavior.





