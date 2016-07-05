[toc]

## 1. Linux设备驱动概述及开发环境构建

### 1.5 Linux设备驱动开发环境构建

#### 1.5.1 PC上的Linux环境

本书配套光盘提供了一个Ubuntu的 **VirtualBox** 虚拟机映像，该虚拟机上安装了所有本书涉及的源代码、工具链和各种开发工具，读者无需再安装和配置任何环境。该虚拟机可运行于Windows等操作系统中，运行方法如下。

解压缩安装盘内的虚拟机磁盘映像 virtual-disk.rar 到本地硬盘得到 virtual-disk.vdi（至少需要 16GB 的空闲磁盘空间）。

启动虚拟机，账号和密码都是 **lihacker**。本书配套源代码都位于 lihacker 主目录的 develop 目录下，几个主要项目针对/home/lihacker/develop/的子目录如下:
LDD6410开发板内核源代码：svn/ldd6410-2-6-28-read-only/linux-2.6.28-samsung。
LDD6410开发板 U-BOOT 源代码：svn/ldd6410-read-only/s3c-u-boot-1.1.6。
LDD6410开发板文件系统用的 busybox、jpegview、mplayer、appweb等： svn/ldd6410-readonly/utils。
LDD6410开发板及常用Linux用户空间驱动测试程序：svn/ldd6410-read-only/tests。
书中globalmem、globalfifo等驱动实例：svn/ldd6410-read-only/training/kernel。
Android 的源代码： git/myandroid。
NDK： android-ndk-r3。
eclipse：单击桌面上的android-eclipse图标，即可运行附带 ADT 的 eclipse 开发工具。

#### 1.5.2 LDD6410开发板

ARM11处理器开发板。采用三星公司最新推出 S3C6410 处理器。

配套电路板提供了如下软件：

- 工具链：提供了arm-linux-gcc、arm-linux-gdb、gdbserver、strace 用于 Android 开发的 eclipse（带 ADT 插件）、JDK 和 NDK。
- U-BOOT：U-BOOT 源代码包含独立的 LDD6410 文件，支持从 SD 卡、 NAND 启动，支持 DM9000 网卡引导。
- Linux 内核、BSP和驱动：Linux 2.6.28 内核、源代码，包含独立的 LDD6410 BSP 和完整的设备驱动。
- 文件系统：基于新版 Busybox 1.15.1，文件系统集成 jpegview、mplayer、appweb 等大量应用，集成了按键、鼠标、触摸屏、LCD 等测试程序，作为驱动的用户应用案例。
- Android：提供 Android 源代码和文件系统、内核电源管理补丁源代码、内核 Android 驱动源代码。LDD6410 的 Android 系统支持按键、触摸屏和鼠标操作，支持使用 LCD 和 **VGA** 进行显示。

#### 1.5.3 工具链安装

本书配套光盘的虚拟机映像中已经安装好了 LDD6410 的工具链，读者如果想在其他环境中安装，只需要从 http://ldd6410.googlecode.com/files/cross-4.2.2-eabi.tar.bz2 下载。 LDD6410 开发板
工具链为 S3C6410X-ToolChain4.2.2-EABI-V0.0-cross-4.2.2-eabi.tar。安装步骤如下。

1. 解压上述工具链获得文件夹：4.2.2-eabi/。
1. 在/usr/local/下面创建目录arm/（注意，最好是放到这个目录，不然在以后的编译过程中可能出现一些错误）。
1. 将目录4.2.2-eabi/移动到/usr/local/arm/下面。
1. 设置环境变量。编辑/etc/profile文件，在文件末尾添加：

        PATH="$PATH:/usr/local/arm/4.2.2-eabi/usr/bin"
        export PATH
1. 测试交叉编译工具链。在终端输入`arm-linux-gcc –v`”，显示如下：

        Using built-in specs.
        Target: arm-unknown-linux-gnueabi
        Configured with:
        /home/scsuh/workplace/coffee/buildroot-20071011/toolchain_build_arm
        /gcc-4.2.2/configure --prefix=/usr --build=i386-pc-linux-gnu --host=i386-pc-linux-gnu
        --target=arm-unknown-linux-gnueabi --enable-languages=c,c++ --with-sysroot=/usr/local
        /arm/4.2.2-eabi/ --with-build-time-tools=/usr/local/arm/4.2.2-eabi//usr/arm-unknown-linuxgnueabi/bin --disable-cxa_atexit --enable-target-optspace --with-gnu-ld --enable-shared
        --with-gmp=/usr/local/arm/4.2.2-eabi/gmp --with-mpfr=/usr/local/arm/4.2.2-eabi//mpfr
        --disable-nls --enable-threads --disable-multilib --disable-largefile --with-arch=armv4t
        --with-float=soft --enable-cxx-flags=-msoft-float
        Thread model: posix gcc version 4.2.2

说明交叉编译工具链已经安装成功。

ldd6410-debug-tools.tar.gz 调试工具包包含了 strace、 gdbserver 和 arm-linux-gdb，其中 strace、gdbserver 用于目标板文件系统， arm-linux-gdb 运行于主机端，对目标板上的内核、内核模块应用程序进行调试。

下载地址为 http://ldd6410.googlecode.com/files/ldd6410-debug-tools.tar.gz，光盘目录为 toolchains/ldd6410-debug-tools.tar.gz。

解压 ldd6410-debug-tools.tar.gz，将其中的 arm-linux-gdb 放入主机上 arm-linux-gcc 所在的目录/usr/local/arm/4.2.2-eabi/usr/bin/。

而 strace、 gdbserver 则可根据需要放入目标机根文件系统的/usr/sbin 目录。

### （未）1.6 设备驱动Hello World：LED驱动


