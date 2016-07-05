[toc]

## 2 启动和关闭

### 2.1 启动

启动脚本常被称为**rc**文件。**rc**的意思是`runcom`或`run command`。

Linux系统可以被自动或手动启动。手工启动时，启动一部分，然后将控制权交给操作员，此时系统处于“单用户“模式；多数初始化脚本尚未运行，多数进程尚未启动，其他用户不能登录。

典型Linux启动过程包含6个阶段：加载和初始化内核、设备侦测和配置、创建内核线程、操作员参与（手工启动时）、执行系统启动脚本、多用户操作。

Linux内核自身也是程序，第一步是将其放入内存并执行。内核一般位于`/vmlinuz` 或 `/boot/vmlinuz`。

内核进行内存测试找出有多少RAM可用。内部部分内部数据结构是固定大小的，因此从真实内存中分出一部分供自己使用。这些内存不能用于用户空间进程。内核会打印消息，报告内存总量及用户空间可以用多少。

#### 内核线程

Once basic initialization is complete, the kernel creates several “spontaneous” processes in user space. They’re called spontaneous processes because they are not created through the normal system **fork** mechanism

The number and nature of the spontaneous processes vary from system to system. Under Linux, there is no visible PID 0. init (always process 1) is accompanied by several memory and kernel handler processes, including those shown in Table 2.1 on the next page. These processes all have low-numbered PIDs and can be identified by the brackets around their names in ps listings (e.g., [kacpid]). Sometimes the process names have a slash and a digit at the end, such as [kblockd/0]. The number indicates the processor on which the thread is running, which may be of interest on a multiprocessor system.

- **kjournald**：Commits ext3 journal updates to disk. There is one **kjournald** for each mounted ext3 filesystem.
- **kswapd**：Swaps processes when physical memory is low
- **kreclaimd**：Reclaims memory pages that haven’t been used recently
- **ksoftirqd**：Handles multiple layers of soft interrupts
- **khubd**：Configures USB devices

Among these processes, only init is really a full-fledged user process. The others are actually portions of the kernel that have been dressed up to look like processes for scheduling or architectural reasons.

创建完这些进程后，内核在启动期的角色就完成了。此时，负责基本操作（如登录）的进程，或多数Linux守护进程都未启动。这些任务都是由**init**负责。

#### Operator intervention (manual boot only)

If the system is to be brought up in single-user mode, a command-line flag (the word “single”) passed in by the kernel notifies init of this fact as it starts up. init eventually turns control over to sulogin, a special neutered-but-rabid version of login that prompts for the root password. If you enter the right password, the system spawns a root shell. You can type `<Control-D>` instead of a password to bypass single-user mode and continue to multiuser mode. See page 31 for more details.

From the single-user shell, you can execute commands in much the same way as when logged in on a fully booted system. However, on SUSE, Debian, and Ubuntu systems, only the root partition is usually mounted; you must mount other filesystems by hand to use programs that don’t live in /bin, /sbin, or /etc. In many single-user environments, the filesystem root directory starts off being mounted read-only. If /tmp is part of the root filesystem, a lot of commands that use temporary files (such as vi) will refuse to run. To fix this problem, you’ll have to begin your single-user session by remounting / in read/write mode. The command

	# mount -o rw,remount /

usually does the trick.

The fsck command is normally run during an automatic boot to check and repair filesystems. When you bring the system up in single-user mode, you may need to run fsck by hand. See page 131 for more information about fsck.

When the single-user shell exits, the system attempts to continue booting into multiuser mode.

#### 多用户操作

After the initialization scripts have run, the system is fully operational, except that no one can log in. For logins to be accepted on a particular terminal (including the console), a **getty** process must be listening on it. init spawns these **getty** processes directly, completing the boot process. init is also responsible for spawning graphical login systems such as **xdm** or **gdm** if the system is set up to use them.

Keep in mind that **init** continues to perform an important role even after bootstrapping is complete. **init** has one single-user and several multiuser “run levels” that determine which of the system’s resources are enabled. Run levels are described later in this chapter, starting on page 33.

### 2.2 启动PC

When a machine boots, it begins by executing code stored in ROMs. The exact location and nature of this code varies, depending on the type of machine you have. On a machine designed explicitly for UNIX or another proprietary operating system, the code is typically firmware that knows how to use the devices connected to the machine, how to talk to the network on a basic level, and how to understand disk-based filesystems. Such intelligent firmware is convenient for system administrators. For example, you can just type in the filename of a new kernel, and the firmware will know how to locate and read that file.

PC中，初始启动代码一般称为BIOS，and it is extremely simplistic compared to the firmware of a proprietary machine. Actually, a PC has several levels of BIOS: one for the machine itself, one for the video card, one for the SCSI card if the system has one, and sometimes for other peripherals such as network cards.
The built-in BIOS knows about some of the devices that live on the motherboard, typically the IDE controller (and disks), network interface, keyboard, serial ports, and parallel ports. SCSI cards are usually only aware of the devices that are connected to them. Thankfully, the complex interactions required for these devices to work together has been standardized in the past few years, and little manual intervention is required.

Modern BIOSes are a little smarter than they used to be. They usually allow you to enter a configuration mode at boot time by holding down one or two special keys; most BIOSes tell you what those special keys are at boot time so that you don’t have　to look them up in the manual.

The BIOS normally lets you select which devices you want to try to boot from, which sounds more promising than it actually is. You can usually specify something like “Try to boot off the floppy, then try to boot off the CD-ROM, then try to boot off the hard disk.” Unfortunately, some BIOSes are limited to booting from the first IDE CD-ROM drive or the first IDE hard disk. If you have been very, very good over the previous year, Santa might even bring you a BIOS that acknowledges the existence of SCSI cards.

Once your machine has figured out what device to boot from, it will try to load the first 512 bytes of the disk. This 512-byte segment is known as the master boot record or **MBR**. The MBR contains a program that tells the computer from which disk partition to load a secondary boot program (the “boot loader”). For more information on PC-style disk partitions and the MBR, refer to Chapter 7, Adding a Disk.

The default MBR contains a simple program that tells the computer to get its boot loader from the first partition on the disk. Linux offers a more sophisticated MBR that knows how to deal with multiple operating systems and kernels.
Once the MBR has chosen a partition to boot from, it tries to load the boot loader specific to that partition. The boot loader is then responsible for loading the kernel.

### （未）2.3 启动加载器：LILO和GRUB

Two boot loaders are used in the Linux world: LILO and GRUB. LILO is the traditional boot loader. It is very stable and well documented but is rapidly being eclipsed by GRUB, which has become the default boot loader on Red Hat, SUSE, and Fedora systems. In fact, current Red Hat and Fedora distributions do not include LILO at all. On the other hand, Debian still uses LILO as its boot loader of choice.

#### GRUB: The GRand Unified Boot loader

Unlike LILO, which must be reinstalled into the boot record or MBR every time it is reconfigured, GRUB reads its configuration file at boot time, eliminating an easy-to-forget administrative step.

You install GRUB on your boot drive by running `grub-install`. This command takes the name of the device from which you’ll be booting as an argument. The way GRUB names the physical disk devices differs from the standard Linux convention (although GRUB can use standard Linux names as well). A GRUB device name looks like this: `(hd0,0)`. The first numeric value indicates the physical drive number (starting from zero), and the second numeric value represents the partition number (again, starting from zero). In this example, (hd0,0) is equivalent to the Linux device `/dev/hda1`. Ergo, if you wanted to install GRUB on your primary drive, you would use the command

	# grub-install '(hd0,0)'

The quotes are necessary to prevent the shell from trying to interpret the parentheses in its own special way. By default, GRUB reads its default boot configuration from `/boot/grub/grub.conf`. Here’s a sample grub.conf file:

    default=0
    timeout=10
    splashimage=(hd0,0)/boot/grub/splash.xpm.gz
    title Red Hat Linux (2.6.9-5)
            root (hd0,0)
            kernel /boot/vmlinuz-2.6.9-5 ro root=/dev/hda1

This example configures only a single operating system, which GRUB boots automatically (default=0) if it doesn’t receive any keyboard input within 10 seconds (timeout=10). The root filesystem for the “Red Hat Linux” configuration is the GRUB device (hd0,0). GRUB loads the kernel from `/boot/vmlinuz-2.6.9-5` and displays a splash screen from the file `/boot/grub/splash.xpm.gz` when it is loaded.

GRUB supports a powerful command-line interface as well as facilities for editing configuration file entries on the fly. To enter command-line mode, type `c` from the GRUB boot image. From the command line you can boot operating systems that aren’t in `grub.conf`, display system information, and perform rudimentary filesystem testing. You can also enjoy the command line’s shell-like features, including command completion and cursor movement. Anything that can be done through the `grub.conf` file can be done through the GRUB command line as well. Press the `<Tab>` key to obtain a quick list of possible commands. Table 2.2 on the next page lists some of the more useful commands.

- `reboot`：Soft-reboot the system
- `find`：Find a file on all mountable partitions
- `root`：Specify the root device (a partition)
- `kernel`：Load a kernel from the root device
- `help`：Get interactive help for a command
- `boot`：Boot the system from the specified kernel image

#### （未）LILO: The traditional Linux boot loader

#### 内核选项

LILO and GRUB allow command-line options to be passed to the kernel. These options typically modify the values of kernel parameters, instruct the kernel to probe for particular devices, specify the path to init, or designate a specific root device.

- `init=/sbin/init`：Tells the kernel to use /sbin/init as its init program
- `init=/bin/bash`：Starts only the bash shell; useful for emergency recovery
- `root=/dev/foo`：Tells the kernel to use /dev/foo as the root device
- `single`：Boots to single-user mode

### （未）2.4 BOOTING SINGLE-USER MODE

### 2.5 启动脚本

**init**执行系统启动脚本。启动脚本的常见任务有：

- 设置计算名、时区
- 用fsck检查磁盘
- 挂载磁盘
- 删除/tmp中的旧文件
- 配置网络接口
- 启动守护进程和网络服务

#### init与run levels

**init**定义了7个运行级别。每个表示一组服务。

- Level 0 is the level in which the system is completely shut down.
- Level 1 or S represents single-user mode.
- Levels 2 through 5 are multiuser levels.
- Level 6 is a “reboot” level.

通用的多用户运行级别是2或3。运行级别5常用于X Windows登录。运行级别4很少使用，运行级别1和S在不同系统上定义是不同的。单用户模式过去是运行级别1。It brought down all multiuser and remote login processes and made sure the system was running a minimal complement of software. 单用户模式允许根权限；但管理员希望获取根权限时一定要验证密码；因此创造了运行级别S：它产生一个进程，询问用户根密码。Linux中S级别只有这一个目的。

Linux actually supports up to 10 run levels, but levels 7 through 9 are undefined.

`/etc/inittab`文件定义在每个运行级别`init`要做什么。它的格式在不同系统中是不同的。共同点是`inittab`定义了，当系统进入一个级别时要执行哪些命令。

**机器启动后，init从运行级别0执行到默认的运行级别**。默认的运行级别也定义在`/etc/inittab`。相邻级别转换要执行的操作也定义在`/etc/inittab`. **系统关闭时，则反向操作**。

现在多数Linux分发默认启动运行级别5。The default run level is easy to change. This excerpt from a SUSE machine’s inittab defaults to run level 5:

	id:5:initdefault:

为了提高灵活性，Linux系统实现了另一层抽象：“改变运行级别”（change run levels）脚本（一般是`/etc/init.d/rc`），供inittab调用。要进入某个级别，这个脚本执行该运行级别的目录中的脚本。一般不需要直接编辑`/etc/inittab`，脚本就足够多数应用需求了。

启动脚本的主拷贝都在`/etc/init.d`目录下。每个脚本表示一个守护进程或系统的特定方面。调用脚本时指定`start`或 `stop`作为参数，可以启动或停止服务。多数脚本也可以用`restart`。作为管理员，可以通过运行`init.d`脚本，手工启用或停用服务。例如下面是一个针对**sshd**的简单的脚本：

    #! /bin/sh
    test -f /usr/bin/sshd || exit 0
    case "$1" in
    	start)
            echo -n "Starting sshd: sshd"
            /usr/sbin/sshd
            echo "."
            ;;
        stop)
            echo -n "Stopping sshd: sshd"
            kill `cat /var/run/sshd.pid`
            echo "."
        	;;
        restart)
            echo -n "Stopping sshd: sshd"
            kill `cat /var/run/sshd.pid`
            echo "."
            echo -n "Starting sshd: sshd"
            /usr/sbin/sshd
            echo "."
        	;;
    	*)
    		echo "Usage: /etc/init.d/sshd start|stop|restart"
    		exit 1
    		;;
    esac

进入某个运行级别，init需要知道`/etc/init.d`下的哪些脚本要运行（不是所有的脚本都要运行），以及要传递什么参数（可能不同）。每个运行级别有一个`/etc/rc<level>.d`目录日，如`rc0.d`、`rc1.d`等等。这些`rc<level>.d`目录包含一些符号链接，这些链接指向`init.d`目录下的相应脚本。所有符号链接都以`S`或`K`开头，接着是一个数字，然后是脚本控制的服务，如`S34named`。当init从一个低的运行级别进入一个更高的运行级别，运行所有`S`开头的脚本，按数字顺序依次执行。反过来，当init从一个高运行级别进入较低的运行级别时，运行所有以`K`开头（kill）的脚本，按相反的顺序执行。即，服务启动的顺序是可以指定的。
例子，告诉系统进入运行级别2时启动CUPS，并在关机时停止它：

    # ln -s /etc/init.d/cups /etc/rc2.d/S80cups
    # ln -s /etc/init.d/cups /etc/rc0.d/K80cups

一些系统将关机与重启分开对待，此时我们还要在`/etc/rc6.d`目录下放关闭的符号链接，以确保重启时能关闭该守护进程。

#### Red Hat 和 Fedora 的启动脚本

Red Hat 和 Fedora 的启动脚本由于历史原因比较混乱。在每个运行级别，init调用`/etc/rc.d/rc`，传入新的运行级别作为参数。`/etc/rc.d/rc` usually runs in “normal” mode, in which it just does its thing, but it can also run in “confirmation” mode, in which it asks you before it runs each individual startup script.

Red Hat 和 Fedora 有一个`chkconfig`命令关联服务。该命令添加或移除启动脚本，指定它们的运行级别，列出某脚本关联的所有启动级别。See man `chkconfig` for usage information.

Red Hat also has an `rc.local` script much like that found on BSD systems. `rc.local` is the last script run as part of the startup process. Historically, `rc.local `was overwritten by the **initscripts** package. This has changed, however, and it is now safe to add your own startup customizations here.

Here’s an example of a Red Hat startup session:

    [kernel information]
    INIT: version 2.85 booting
    Setting default font (latarcyrhev-sun16): [ OK ]
    Welcome to Red Hat Linux
    Press 'I' to enter interactive startup.
    Starting udev: [ OK ]
    Initializing hardware... storage network audio done
    Configuring kernel parameters: [ OK ]
    Setting clock (localtime): Tue Mar 29 20:50:41 MST 2005: [ OK ]
    …

看到“Welcome to Red Hat Enterprise Linux”后，可以输入`i`进入confirmation模式。Unfortunately, Red Hat gives you no confirmation that you have pressed the right key. It blithely continues to mount local filesystems, activate swap partitions, load keymaps, and locate its kernel modules. Only after it switches to run level 3 does it actually start to prompt you for confirmation:

    			Welcome to Red Hat Enterprise Linux WS
    		Press 'I' to enter interactive startup.
    Starting udev: [ OK ]
    Initializing hardware... storage network audio done
    Configuring kernel parameters: [ OK ]
    setting clock (localtime): tue mar 29 20:50:41 mst 2005: [ OK ]
    Setting hostname rhel4: [ OK ]
    Checking root filesystem
    /dev/hda1: clean, 73355/191616 files, 214536/383032 blocks [ OK ]
    Remounting root filesystem in read-write mode: [ OK ]
    Setting up Logical Volume Management: [ OK ]
    Checking filesystems
    Mounting local filesystems: [ OK ]
    Enabling local filesystem quotas: [ OK ]
    Enabling swap space: [ OK ]
    INIT: Entering runlevel: 3
    Entering interactive startup
    Start service kudzu (Y)es/(N)o/(C)ontinue? [Y]

Interactive startup and single-user mode both begin at the same spot in the booting process. When the startup process is so broken that you cannot reach this point safely, you can use a rescue floppy or CD-ROM to boot.

You can also pass the argument `init=/bin/sh` to the kernel to trick it into running a single-user shell before init even starts. If you take this tack, you will have to do all the startup housekeeping by hand, including manually fscking and mounting the local filesystems.

Much configuration of Red Hat’s boot process can be achieved through manipulation of the config files in `/etc/sysconfig`. Table 2.4 summarizes the function of some popular items in the /etc/sysconfig directory.

Table 2.4 Files and subdirectories of Red Hat’s /etc/sysconfig directory

clock Specifies the type of clock that the system has (almost always UTC)a
console A mysterious directory that is always empty
httpd Determines which Apache processing model to use
hwconf Contains all of the system’s hardware info. Used by Kudzu.
i18n Contains the system’s local settings (date formats, languages, etc.)
init Configures the way messages from the startup scripts are displayed
keyboard Sets keyboard type (use “us” for the standard 101-key U.S. keyboard)
mouse Sets the mouse type. Used by X and gpm.
network Sets global network options (hostname, gateway, forwarding, etc.)
network-scripts Contains accessory scripts and network config files
sendmail Sets options for sendmail

Several of the items in Table 2.4 merit additional comments:
• The hwconf file contains all of your hardware information. The Kudzu service checks it to see if you have added or removed any hardware and asks
you what to do about changes. You may want to disable this service on a production system because it delays the boot process whenever it detects
a change to the hardware configuration, resulting in an extra 30 seconds of
downtime for every hardware change made.
• The network-scripts directory contains additional material related to network configuration. The only things you should ever need to change are the
files named ifcfg-interface. For example, network-scripts/ifcfg-eth0 contains the configuration parameters for the interface eth0. It sets the interface’s IP address and networking options. See page 299 for more information about configuring network interfaces.
• The sendmail file contains two variables: DAEMON and QUEUE. If the
DAEMON variable is set to yes, the system starts sendmail in daemon
mode (-bd) when the system boots. QUEUE tells sendmail how long to
wait between queue runs (-q); the default is one hour.

#### （未）SUSE startup scripts

#### Debian and Ubuntu startup scripts

Debian设计的非常差，没有文档，不一致！Sadly, it appears that the lack of a standard way of setting up scripts has resulted in chaos in this case. Bad Debian!

At each run level, init invokes the script `/etc/init.d/rc` with the new run level as an argument. Each script is responsible for finding its own configuration information, which may be in the form of other files in `/etc`, `/etc/default`, another subdirectory of `/etc`, or somewhere in the script itself.

If you’re looking for the hostname of the system, it’s stored in `/etc/hostname`, which is read by the `/etc/init.d/hostname.sh` script. Network interface and default gateway parameters are stored in `/etc/network/interface`s, which is read by the `ifup` command called from `/etc/init.d/networking`. Some network options can also be set in `/etc/network/options`.

Debian and Ubuntu have a sort of clandestine startup script management program in the form of `update-rc.d`. Although its man page cautions against interactive use, we have found it to be a useful, if less friendly, substitute for `chkconfig`. For example, to start **sshd** in run levels 2, 3, 4, and 5, and to stop it in levels 0, 1, and 6, use:

	$ sudo /usr/sbin/update-rc.d sshd start 0123 stop 456

####  （未）2.6 重启和关闭


