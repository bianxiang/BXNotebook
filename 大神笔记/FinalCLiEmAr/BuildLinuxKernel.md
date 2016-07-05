[toc]

## 编译 Linux2.6 内核

### 系统中相关的目录, 文件及命令

- `/boot`
- `/boot/vmlinuz-`：用于启动的压缩内核镜像，它也就是`/arch/boot`中的压缩镜像。
- `/boot/system.map-`：存储内核符号地址。
- `/boot/initrd.img-`：初始化RAM disk时，用来存储挂载根文件系统所需的模块。
- `/boot/grub/menu.lst`：grub的配置文件。（不同的发行版中它可能位于不同位置）
- `/lib/modules`：该目录包含了内核模块及其他文件。modules中一般会有多个目录：系统自带的内核模块在这里；你编译自己的内核模块后，它们也会被安装到这里。不同的目录由内核版本号来区分。即modules里目录的名称是内核版本号。（使用命令`uname -r`可知当前系统内核所用的模块位于哪个目录）。
- `/lib/modules//build`：储存为该版本的内核编译新模块所需的文件。包括Makefile、.config、module.symVers（模块符号信息）、内核头文件（位于include/、include/asm/中）。
- `/lib/modules//kernel`：储存内核目标文件（以.ko为后缀）。它的目录组织和内核源代码中kernel的目录组织相同。
- `/lib/modules//`中：
  - modules.alias：模块别名定义。模块加载工具使用它来加载相应的模块。
  - modules.dep：定义了模块间的依赖关系。
  - modules.symbols：指定符号属于哪个模块。
  这些文件都是文本文件, 可以查看它们。

uname(1)被用来查看系统信息。这里对我们有用的是它的"-r"选项，它显示内核版本信息。

	$ uname -r

### 下载内核、验证签名

到http://www.kernel.org/下载最新版本的2.6内核。这里以 linux-2.6.17.13 为例。

下载了内核压缩包之后，还可下载对应的sign文件。它被用来验证内核压缩文档的openPGP签名。

    $ wget -c http://www.kernel.org/pub/linux/kernel/v2.6/linux-2.6.17.13.tar.bz2
    $ wget -c http://www.kernel.org/pub/linux/kernel/v2.6/linux-2.6.17.13.tar.bz2.sign

**验证签名**

首先从pgp的服务器获取签名公匙，linux内核包的公匙编号是0x517D0F0E。再利用sign文件来验证.bz2压缩包的签名。如果输出中有类似gpg: Good signature from "Linux Kernel Archives Verification Key "的内容，说明该包是有效的。后面给出的警告信息可以忽略。

    $ gpg --keyserver wwwkeys.pgp.net --recv-keys 0x517D0F0E
    $ gpg --verify linux-2.6.17.13.tar.bz2.sign linux-2.6.17.13.tar.bz2

GPG签名只是保证镜像网站提供的压缩包和kernel.org所提供的是相同的，如果你在kernel.org下载，不需要验证签名。

### 解压缩

解压缩之前，有个问题值得思考：要将压缩包解压到何处？即要在哪个目录进行Linux内核源代码的编译？内核源码树的README中有这样一段话:

    Do NOT use the /usr/src/linux area! This area has a (usually incomplete) set of kernel headers that are used by the library header files. They should match the library, and not get messed up by whatever the kernel-du-jour happens to be.

实际上在我的Ubuntu系统中/usr/src/目录中最初是没有linux目录的。你可以在/usr/src中新建一个目录，用内核版本命名，比如/usr/src/linux-2.6.17.13。这样即便之前在/usr/src中安装了linux的头文件也不会对它们造成影响。我采用的方法是：在/usr/local/src/kernel目录中进行。

编译内核时候,若在`make`后添加`O=`将会使生成的目标文件(包括.config)被放置到指定的目录。否则生成的目标文件默认地被放到内核源码目录。我们就采用默认的方法；这是安全的。

### 打补丁

对于kernel.org中的内核，我个人认为没必要下载patch。直接下载bz2包不就行了。特定的补丁只能针对紧随其前的一个版本。比如你想从2.6.17.1升级到2.6.17.13，你得打12次补丁，忒麻烦了。

但是有时候需要对"官方内核"添加补丁，以支持特定的系统。比如ARMLinux，它往往不是发布完整的内核，而是发布针对特定版本的补丁包。这种情况下就要知道如何打补丁了。方法很简单，把补丁下载、解压。得到patch-。将它放到解压后的内核目录树的**父目录**中（也就是补丁和内核目录在同一目录）。然后cd到内核目录树中运行：

	$ patch -p1

### 配置内核

1、构建编译环境。显然需要make、gcc等工具。在Ubuntu中只需一条命令就可安装所有的源代码编译工具：

	# apt-get install build-essential

当然如果你的内核是要安装到不同体系结构的目标系统中还需要构建**cross**编译环境。

2、内核配置工具介绍

Linux提供了多种内核配置工具。最基础的是`make config`，但一般不用它。

`make menuconfig`是比较主流的配置工具，它需要curse库的支持，在Ubuntu中默认是没有的，先安装它：

	# apt-get install libncurses5-dev

`make xconfig`基于X11，使用qt库，在Ubuntu中先安装qt库：

	# apt-get install libqt3-headers libqt3-mt-dev

### 内核配置相关

在内核树的根目录中，有一个.config文件，它记录了内核的配置选项，可直接对它进行修改再运行。若.config不存在，对内核进行配置后会生成它。这种情况下当然不能开始就运行oldconfig。实际上如果你手头有合适的.config文件，可以运行`make oldconfig`直接按.config的内容来配置

其实可以直接在menuconfig中加载已有的配置文件。不要将它改名为.config，否则完成配置退出menuconfig时会提示你运行 make mrproper。上面提到的方法只是比较适合于oldconfig！

### make相关命令

`make oldconfig`：基于已有的.config进行配置。若有新的符号它将询问用户。

`make defconfig`：按默认选项对内核进行配置（386的默认配置是Linus做的）。
`make allnoconfig`：除必须的选项外其它选项一律不选。（常用于嵌入式系统）。
`make clean`：删除生成的目标文件，往往用它来实现对驱动的重新编译。
`make mrproper`：删除包括.config在内的生成的目标文件。

可以查看内核源码树中的README和Makefile了解上述配置方法。

### 编译内核

配置完成后，就要进行编译了。编译2.6的内核很简单，Makefile自动检测依赖性，产生编译文件(bzImage)，你也不用另外编译 modules！只需运行：

	$ make

### 编译生成的文件介绍

**vmlinux**：未经压缩的原始linux内核镜像。
**/arch//boot/zImage(bzImage)**：使用zlib压缩后的内核镜像。

注意，不同的体系结构对压缩后内核镜像的默认命名不同。比如arm的是zImage，而i386的是bzImage。（z表示zlib，bz表示"big zlib"，而非bzip2。）

### 安装内核

编译完成后, 在arch/i386/boot目录中会有bzImage映象文件。安装内核步骤如下:

1、在/boot目录下新建mynewkernel目录，并将bzImage拷贝到/boot/mynewkernel目录下
2、更改/boot/mynewkernel中bzImage的名字：`mv bzImage vmlinuz-2.6.17.13`
3、备份、修改grub配置文件：

	cp /boot/grub/menu.lst menu.lst.origin

修改menu.list，加入以下内容：

    title zp, make defconfig, 2.6.17.13
    root (hd0,2)
    kernel /boot/mynewkernel/vmlinuz-2.6.17.13 root=/dev/sda3 ro quiet splash
    savedefault
    boot

4、安装模块：$sudo make modules_install

重启。在grub启动菜单中选择新内核启动...