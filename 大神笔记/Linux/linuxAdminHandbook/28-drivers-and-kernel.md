[toc]

## 28 驱动与内核

内核绝大部分是C编写的，少量汇编用于与硬件或芯片相关的功能交互。

### 28.1 内核的适配（adpataion）

### 28.2 设备与设备文件

设备驱动是内核的一部分；不是用户空间程序；但可以被内核或用户空间程序访问。用户空间的访问一般通过`/dev`目录下的特殊文件。内核将对这些文件的操作转换为驱动的调用。

#### 28.2.1 设备文件与设备号

很多设备在`/dev`下都有响应的文件；网络设备等没有。`/dev`下的设备文件每个都有一个主设备号和一个次设备号。内核使用这些好像将设备文件引用映射到相应的驱动。

The major device number identifies the **driver** with which the **file** is associated (in other words, the type of device). The minor device number usually identifies which particular instance of a given device type is to be addressed. 次设备号有时称为unit number。可以通过`ls -l`观察设备的主次设备号：

    $ ls -l /dev/sda
    brw-rw---- 1 root disk 8, 0 Jan 5 2005 /dev/sda

主设备号为8，次设备号为0。驱动使用次设备号选择设备的特定特性。For example, a single tape drive can have several files in /dev representing it in various configurations of recording density and rewind characteristics. 驱动可以对次设备号自由解析。Look up the man page for the driver to determine what convention it’s using.

伪设备，没有实际设备，但有驱动。例如，通过网络登录的用户会被分配一个PTY（pseudo-TTY）——从上层软件看像一个串口。This trick allows programs written in the days when everyone used a TTY to continue to function in the world of windows and networks.

When a program performs an operation on a device file, the kernel automatically catches the reference, looks up the appropriate function name in a table, and transfers control to it. 要做文件系统模型中没有的操作（如弹出软盘），程序可以通过`ioctl`系统调用，从用户空间向驱动发消息。

#### 28.2.2 创建设备文件

可以通过`mknod`命令手工创建设备文件，

	mknod filename type major minor

`type`取`c`或`b`，分别表示字符和块设备。If you are manually creating a device file that refers to a driver that’s already present in your kernel, check the man page for the driver to find the appropriate major and minor device numbers.

历史上，`/dev`下的设备文件由系统管理员手工创建。多数系统在`/dev`下有一个脚本`MAKEDEV`帮助完成该项工作。MAKEDEV sometimes, but not always, knew how to create the correct device files for a particular component. It was a tedious process at best.

从内核版本2.6开始，udev系统动态管理设备文件的创建和移除，根据设备的出现和移除。The **udevd** daemon listens for messages from the kernel regarding device status changes. Based on configuration information in `/etc/udev/udev.conf` and subdirectories, **udevd** can take a variety of actions when a device is discovered or disconnected. By default, **udevd** creates device files in /dev. It also attempts to run network configuration scripts when new network interfaces are detected.

#### 28.2.3 sysfs：设备的窗口

2.6内核的另一个新特性是**sysfs**。它是一个虚拟文件系统，提供可用设备的详细信息，包括配置，状态等。这些信息可以在内核和用户空间访问。

sysfs一般挂载在`/sys`目录，to find out everything from what IRQ a device is using to how many blocks have been queued for writing on a disk controller. One of the guiding principles of sysfs is that each file in /sys should represent only one attribute of the underlying device. This convention imposes a certain amount of structure on an otherwise chaotic data set.

过去，设备配置信息通过`/proc`文件系统获取。尽管`/proc`将继续保留进程和内核的运行时信息，但今后设备特定的信息将转到`/sys`。

sysfs刚出现不就。最终，期望通过它可以实时配置设备。最终，它甚至会替代`/dev`的部分或全部。

#### 28.2.4 设备的命名约定

Serial device files are named `ttyS` followed by a number that identifies the specific interface to which the port is attached. TTYs are sometimes represented by more than one device file; the extra files usually afford access to alternative flow control methods or locking protocols.

IDE硬盘设备命名规则是`/dev/hdLP`。其中`L`是一个字母，表示unit (with `a` being the master on the first IDE interface, `b` being the slave on that interface, `c` being the master on the second IDE interface, etc.)。`P`是分区号（从1开始）。例如，第一块IDE磁盘的第一个分区是`/dev/hda1`。SCSI磁盘与此类似，只是前缀由`/dev/hd`变成`/dev/sd`。You can drop the partition number on both types of devices to access the entire disk (e.g., `/dev/hda`).

SCSI CD-ROM驱动是`/dev/scdN`，其中`N`用于区别多个CD-ROM驱动。Modern IDE (ATAPI) CD-ROM drives are referred to just like IDE hard disks (e.g., `/dev/hdc`).

### 28.3 为什么/如何配置内核

### （未）28.4 调优内核参数

### 28.5 构建一个Linux内核

之前内核源文件必须放在`/usr/src/linux`目录，由根用户构建。2.6开始，放在哪里都行。普通用户可以构建。

内核配置文档：Documentation/Configure.help。

如果你想把一个旧版本的内核配置迁移到新内核版本，可以利用`make oldconfig`，读取已存在的配置文件，只询问新问题。

### 28.6 添加一个Linux设备驱动

### （未）28.7 Loadable kernel modules

### 28.8 热插拔

The Linux hot-plugging features export information about device availability into user space. This facility lets user processes respond to events such as the connection of a USB-enabled digital camera.

Beginning with kernel version 2.6, hot-plugging is available on buses and drivers that have been designed to use **sysfs**. 热插拔、sysfs和设备驱动注册是紧密联系的。

In the current implementation of hot-plugging, the kernel executes the user process specified by the parameter `/proc/sys/kernel/hotplug` (usually `/sbin/hotplug`) whenever it detects that a device has been added or removed. `/sbin/hotplug` is a shell script that calls a device-type-specific agent from the `/etc/hotplug/` directory to act on the event. For example, if the event was the addition of a network interface, the `/etc/hotplug/net.agent` script would be executed to bring the interface on-line.

You can add or edit scripts in the `/etc/hotplug` directory to customize your system’s hot-plugging behavior. For security and other reasons, you may not want the hot-plugging system to act upon an event. In these instances, you can add devices to the `/etc/hotplug/blacklist` file to prevent their events from triggering actions.

Conversely, you can force hot-plug actions by creating “handmap” files, such as `/etc/hotplug/type.handmap`. (Make sure that type doesn’t conflict with an existing name if you’re creating something new.)

### 28.9 设置启动选项

启动时可以向内核传参数，如指定根设备。由bootloader（LILO或GRUB）负责向内核传参数。

要执行每次启动时都使用的参数，可以配置`/etc/lilo.conf`或`/boot/grub/grub.conf`。若无法编辑配置文件，可以手工传入。例如，在LILO提示符下输入：

	LILO: linux root=/dev/hda1 ether=0,0,eth0 ether=0,0,eth1

to tell LILO to load the kernel specified by the “linux” tag, to use the root device /dev/hda1, and to probe for two Ethernet cards.

A similar example using GRUB would look like this:

    grub> kernel /vmlinuz root=/dev/hda1 ether=0,0,eth0 ether=0,0,eth1
    grub> boot

Another common situation in which it’s helpful to use boot-time options is when probing logical unit numbers (LUNs) on a storage area network (SAN). By default, the Linux kernel probes for LUN 0 only, which may not be adequate if your environment presents logical storage areas as different LUNs. (Contact your SAN administrator or vendor to determine if this is the case.) In this situation, you need to tell the kernel how many LUNs to probe since the probing occurs during bootstrapping.

For example, if you wanted a 2.4.x kernel to probe the first 8 LUNs, you might use a boot line like this:

    grub> kernel /vmlinuz root=/dev/hda1 max_scsi_luns=8
    grub> boot

In kernels 2.6.x and later, the parameter name has changed:

    grub> kernel /vmlinuz root=/dev/hda1 max_luns=8
    grub> boot









































































































































































































