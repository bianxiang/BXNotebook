[toc]

## 5. Linux文件系统与设备文件系统

由于字符设备和块设备都良好地体现了“一切都是文件”的设计思想，掌握Linux文件系统、设备文件系统的知识就显得相当重要了。首先，驱动工程师编写的驱动最终通过操作系统的文件操作系统调用或C库函数（本质也基于系统调用）被访问，而设备驱动的结构最终也是为了迎合提供给应用程序员的API。其次，驱动工程师在设备驱动中不可避免地会与设备文件系统打交道，从Linux 2.4内核的**devfs**文件系统到目前Linux 2.6基于sysfs的**udev**文件系统。

### 5.1 Linux文件操作

#### 5.1.1 文件操作系统调用

1、创建

	int creat(const char *filename, mode_t mode);

参数`mode`指定新建文件的存取权限，它同 umask 一起决定文件的最终权限（mode&umask），其中 umask 代表了文件在创建时需要去掉的一些存取权限。 umask 可通过系统调用`umask()`来改变：

	int umask(int newmask);

该调用将 umask 设置为 newmask，然后返回旧的 umask，它只影响读、写和执行权限。

2、打开

	int open(const char *pathname, int flags);
	int open(const char *pathname, int flags, mode_t mode);

`open()`函数有两个形式，其中 pathname 是我们要打开的文件名（包含路径名称，缺省是认为在当前路径下面），flags 可以是如表 5.1 所示的一个值或者是几个值的组合。

O_RDONLY 以只读的方式打开文件
O_WRONLY 以只写的方式打开文件
O_RDWR 以读写的方式打开文件
O_APPEND 以追加的方式打开文件
O_CREAT 创建一个文件
O_EXEC 如果使用了O_CREAT而且文件已经存在，就会发生一个错误
O_NOBLOCK 以非阻塞的方式打开一个文件
O_TRUNC 如果文件已经存在，则删除文件的内容

O_RDONLY、O_WRONLY、O_RDWR 三个标志只能使用任意的一个。

如果使用了 O_CREATE 标志，则使用的函数是`int open(const char *pathname,int flags,mode_t mode);`这个时候我们还要指定 mode 标志，用来表示文件的访问权限。 mode 可以是如表 5.2 所列值的组合。

S_IRUSR 用户可以读
S_IWUSR 用户可以写
S_IXUSR 用户可以执行
S_IRWXU 用户可以读、写、执行
S_IRGRP 组可以读
S_IWGRP 组可以写
S_IXGRP 组可以执行
S_IRWXG 组可以读、 写、 执行
S_IROTH 其他人可以读
S_IWOTH 其他人可以写
S_IXOTH 其他人可以执行
S_IRWXO 其他人可以读、写、执行
S_ISUID 设置用户执行ID
S_ISGID 设置组的执行ID

除了可以通过上述宏进行OR逻辑产生标志以外，我们也可以自己用数字来表示，Linux用5个数字来表示文件的各种权限：第一位表示设置用户ID；第二位表示设置组 ID；第三位表示用户自己的权限位；第四位表示组的权限；最后一位表示其他人的权限。每个数字可以取1（执行权限）、2（写权限）、4（读权限）、0（无）或者是这些值的和。例如，要创建一个用户可读、可写、可执行，但是组没有权限，其他人可以读、可以执行的文件，并设置用户ID位。那么，我们应该使用的模式是1（设置用户ID）、0（不设置组ID）、7（1+2+4，读、写、执行）、0（没有权限）、5（1+4，读、执行）即10705：

	open("test", O_CREAT, 10705);

上述语句等价于：

	open("test", O_CREAT, S_IRWXU | S_IROTH | S_IXOTH | S_ISUID );

如果文件打开成功，open函数会返回一个文件描述符，以后对该文件的所有操作就可以通过对这个文件描述符进行操作来实现。

3、读写
在文件打开以后，我们才可对文件进行读写，Linux中提供文件读写的系统调用是 read、write函数：

    int read(int fd, const void *buf, size_t length);
    int write(int fd, const void *buf, size_t length);

函数read()实现从文件描述符 fd 所指定的文件中读取 length 个字节到 buf 所指向的缓冲区中，返回值为实际读取的字节数。函数 write 实现将把 length 个字节从 buf 指向的缓冲区中写到文件描述符 fd 所指向的文件中，返回值为实际写入的字节数。

以 O_CREAT 为标志的 open 实际上实现了文件创建的功能，因此，下面的函数等同 creat()函数：

	int open(pathname, O_CREAT | O_WRONLY | O_TRUNC, mode);

4、定位

对于随机文件，我们可以随机地指定位置读写，使用如下函数进行定位：

	int lseek(int fd, offset_t offset, int whence);

`lseek()`将文件读写指针相对 whence 移动 offset 个字节。操作成功时，返回文件指针相对于文件头的位置。参数 whence 可使用下述值：

    SEEK_SET：相对文件开头
    SEEK_CUR：相对文件读写指针的当前位置
    SEEK_END：相对文件末尾

offset 可取负值，例如下述调用可将文件指针相对当前位置向前移动 5 个字节：

	lseek(fd, -5, SEEK_CUR);

由于 lseek 函数的返回值为文件指针相对于文件头的位置，因此下列调用的返回值就是文件的长度：

	lseek(fd, 0, SEEK_END);

5、关闭

当我们操作完成以后，我们要关闭文件了，只要调用 close 就可以了，其中 fd 是我们要关闭的文件描述符：

	int close(int fd);

例程：编写一个程序，在当前目录下创建用户可读写文件hello.txt，在其中写入“ Hello, software weekly”，关闭该文件。再次打开该文件，读取其中的内容并输出在屏幕上。

    #include <sys/types.h>
    #include <sys/stat.h>
    #include <fcntl.h>
    #include <stdio.h>
    #define LENGTH 100
    main()
    {
    	int fd, len;
    	char str[LENGTH];

		fd = open("hello.txt", O_CREAT|O_RDWR, S_IRUSR|S_IWUSR);
        if (fd) {
            write(fd, "Hello World", strlen("Hello World"));
            close(fd);
        }
        fd = open("hello.txt", O_RDWR);
        len = read(fd, str, LENGTH); /* 读取文件内容 */
        str[len] = '\0';
        printf("%s\n", str);
        close(fd);
    }

#### 5.1.2 C库文件操作

C库函数的文件操作实际上是独立于具体的操作系统平台的。

1、创建和打开

	FILE *fopen(const char *path, const char *mode);

`fopen()`实现打开指定文件，其中的mode为打开模式，C库函数中支持的打开模式如表 5.3 所示。

r、rb 以只读方式打开
w、wb 以只写方式打开。如果文件不存在，则创建该文件，否则文件被截断
a、ab 以追加方式打开。如果文件不存在，则创建该文件
r+、r+b、rb+ 以读写方式打开
w+、w+b、wh+ 以读写方式打开。如果文件不存在时，创建新文件，否则文件被截断
a+、a+b、ab+ 以读和追加方式打开。如果文件不存在， 则创建新文件

其中**b**用于区分二进制文件和文本文件，这一点在DOS、Windows系统中是有区分的，但Linux不区分二进制文件和文本文件。

2、读写

C库函数支持以字符、字符串等为单位，支持按照某种格式进行文件的读写，这一组函数为：

    int fgetc(FILE *stream);
    int fputc(int c, FILE *stream);
    char *fgets(char *s, int n, FILE *stream);
    int fputs(const char *s, FILE *stream);
    int fprintf(FILE *stream, const char *format, ...);
    int fscanf (FILE *stream, const char *format, ...);
    size_t fread(void *ptr, size_t size, size_t n, FILE *stream);
    size_t fwrite (const void *ptr, size_t size, size_t n, FILE *stream);

fread()实现从流 stream 中读取加 n 个字段，每个字段为 size 字节，并将读取的字段放入 ptr 所指的字符数组中，返回实际已读取的字段数。在读取的字段数小于 num 时，可能是在函数调用时出现错误，也可能是读到文件的结尾。所以要通过调用 feof()和 ferror()来判断。

write()实现从缓冲区 ptr 所指的数组中把 n 个字段写到流 stream 中，每个字段长为 size 个字节，返回实际写入的字段数。

另外，C 库函数还提供了读写过程中的定位能力，这些函数包括：

    int fgetpos(FILE *stream, fpos_t *pos);
    int fsetpos(FILE *stream, const fpos_t *pos);
    int fseek(FILE *stream, long offset, int whence);

3、关闭

利用 C 库函数关闭文件依然是很简单的操作：

	int fclose (FILE *stream);

例程：将第5.1.1节中的例程用C库函数来实现，如代码清单5-2所示。

    #include <stdio.h>
    #define LENGTH 100
    main()
    {
        FILE *fd;
        char str[LENGTH];

        fd = fopen("hello.txt", "w+"); /* 创建并打开文件 */
        if (fd) {
            fputs("Hello World", fd); /* 写入字符串 */
            fclose(fd);
        }

        fd = fopen("hello.txt", "r");
        fgets(str, LENGTH, fd); /* 读取文件内容 */
        printf("%s\n", str);
        fclose(fd);
    }

### 5.2 Linux文件系统

应用程序和 VFS 之间的接口是系统调用，而 VFS 与磁盘文件系统以及普通设备之间的接口是 file_operations 结构体成员函数，这个结构体包含对文件进行打开、关闭、读写、控制的一系列成员函数。

由于字符设备的上层没有磁盘文件系统，所以字符设备的 file_operations 成员函数就直接由设备驱动提供了。在稍后的第6章，将会看到 file_operations 正是字符设备驱动的核心。

而对于块存储设备而言，ext2、fat、jffs2 等文件系统中会实现针对 VFS 的 file_operations 成员函数，设备驱动层将看不到 file_operations 的存在。磁盘文件系统和设备驱动会将对磁盘上文件的访问最终转换成对磁盘上柱面和扇区的访问。

在设备驱动程序的设计中，一般而言，会关心 `file` 和 `inode` 这两个结构体。

1、file 结构体

file结构体代表一个打开的文件（设备对应于设备文件），系统中每个打开的文件在内核空间都有一个关联的`struct file`。它由内核在打开文件时创建，并传递给在文件上进行操作的任何函数。

在文件的所有实例都关闭后，内核释放这个数据结构。在内核和驱动源代码中，struct file的指针通常被命名为 file 或 filp。代码清单 5.3 给出了文件结构体的定义。

    struct file
    {
        union {
            struct list_head fu_list;
            struct rcu_head fu_rcuhead;
        } f_u;
        struct dentry *f_dentry; /*与文件关联的目录入口(dentry)结构*/
        struct vfsmount *f_vfsmnt;
        struct file_operations *f_op; /* 和文件关联的操作*/
        atomic_t f_count;
        unsigned int f_flags; /*文件标志，如O_RDONLY、O_NONBLOCK、 O_SYNC*/
        mode_t f_mode; /*文件读/写模式，FMODE_READ和FMODE_WRITE*/
        loff_t f_pos; /* 当前读写位置*/
        struct fown_struct f_owner;
        unsigned int f_uid, f_gid;
        struct file_ra_state f_ra;

        unsigned long f_version;
        void *f_security;

        /* tty 驱动需要，其他的也许需要 */
        void *private_data; /*文件私有数据*/
        ...
        struct address_space *f_mapping;
    };

文件读/写模式 mode、标志f_flags都是设备驱动关心的内容，而私有数据指针 `private_data`在设备驱动中被广泛应用，大多被指向设备驱动自定义用于描述设备的结构体。

驱动程序中经常会使用如下类似的代码来检测用户打开文件的读写方式：

    if (file->f_mode & FMODE_WRITE) {/* 用户要求可写 */
    }
    if (file->f_mode & FMODE_READ) {/* 用户要求可读 */
    }

下面的代码可用于判断以阻塞还是非阻塞方式打开设备文件：

    if (file->f_flags & O_NONBLOCK) /* 非阻塞 */
    	pr_debug("open: non-blocking\n");
    else /* 阻塞 */
    	pr_debug("open: blocking\n");

（未）2、inode结构体

VFS inode 包含文件访问权限、属主、组、大小、生成时间、访问时间、最后修改时间等信息。它是 Linux 管理文件系统的最基本单位，也是文件系统连接任何子目录、文件的桥梁，inode结构体的定义如代码清单 5.4 所示。

### 5.3 devfs设备文件系统

devfs（设备文件系统）是由Linux 2.4内核引入的，引入时被许多工程师给予了高度评价。它的出现使得设备驱动程序能自主地管理它自己的设备文件。具体来说，devfs具有如下优点：

1. 可以通过程序在设备初始化时在/dev目录下创建设备文件，卸载设备时将它删除。
2. 设备驱动程序可以指定设备名、所有者和权限位，用户空间程序仍可以修改所有者和权限位。
3. 不再需要为设备驱动程序分配主设备号以及处理次设备号，在程序中可以直接给 `register_chrdev()`传递 `0` 主设备号以获得可用的主设备号，并在`devfs_register()`中指定次设备号。

驱动程序应调用下面这些函数来进行设备文件的创建和删除工作。

    /*创建设备目录*/
    devfs_handle_t devfs_mk_dir(devfs_handle_t dir, const char *name, void *info);
    /*创建设备文件*/
    devfs_handle_t devfs_register(devfs_handle_t dir, const char *name, unsigned
    int flags, unsigned int major, unsigned int minor, umode_t mode, void *ops,
    void *info);
    /*撤销设备文件*/
    void devfs_unregister(devfs_handle_t de);

在 Linux 2.4 的设备驱动编程中，分别在模块加载和卸载函数中创建和撤销设备文件是被普遍采用并值得大力推荐的好方法。代码清单 5.5 给出了一个使用 devfs 的例子。

    static devfs_handle_t devfs_handle;
    static int __init xxx_init(void)
    {
        int ret;
        int i;
        /*在内核中注册设备*/
        ret = register_chrdev(XXX_MAJOR, DEVICE_NAME, &xxx_fops);
        if (ret < 0) {
            printk(DEVICE_NAME " can't register major number\n");
            return ret;
        }
        /*创建设备文件*/
        devfs_handle = devfs_register(NULL, DEVICE_NAME, DEVFS_FL_DEFAULT,
        XXX_MAJOR, 0, S_IFCHR | S_IRUSR | S_IWUSR, &xxx_fops, NULL);
        ...
        printk(DEVICE_NAME " initialized\n");
        return 0;
    }

    static void __exit xxx_exit(void)
    {
        devfs_unregister(devfs_handle); /*撤销设备文件*/
        unregister_chrdev(XXX_MAJOR, DEVICE_NAME); /*注销设备*/
    }

    module_init(xxx_init);
    module_exit(xxx_exit);

代码中第7行和第23行分别用于注册和注销字符设备，使用的`register_chrdev()`和`unregister_chrdev()`在 Linux 2.6 内核中虽然仍然被支持，但这是**过时的**做法。第13和22行分别用于创建和删除devfs文件节点。

### 5.4 udev设备文件系统

#### 5.4.1 udev与devfs的区别

尽管devfs有这样和那样的优点，但在 Linux 2.6 内核中，devfs被认为**是过时的方法并最终被抛弃**。**udev**取代了它。Linux VFS内核维护者Al Viro指出了几点udev取代devfs的原因：

1. devfs所做的工作被确信可以在用户态来完成。
2. devfs被加入内核之时，大家寄望它的质量可以迎头赶上。
3. devfs被发现了一些可修复和无法修复的bug。
4. 对于可修复的bug，几个月前就已经被修复了，其维护者认为一切良好。
5. 对于后者，同样是相当长一段时间以来没有改观了。
6. devfs的维护者和作者对它感到失望并且已经停止了对代码的维护工作。

Linux设计中强调的一个基本观点是**机制**和**策略**的分离。机制是做某样事情的固定的步奏、方法，而策略就是每一个步奏所采取的不同方式。机制是相对固定的，而每个步奏采用的策略是不固定的。机制是稳定的，而策略则是灵活的。因此在 Linux 内核中，不应该实现策略。Richard Gooch 认为，属于策略的东西应该被移到用户空间。这就是为什么 devfs 位于内核空间，而 udev 确要移到用户空间的原因。

udev完全在用户态工作，利用设备加入或移除时内核所发送的**热插拔事件（hotplug event）**来工作。在热插拔时，设备的详细信息会由内核输出到位于/sys的sysfs文件系统。udev的设备命名策略、权限控制和事件处理都是在用户态下完成的，它利用sysfs中的信息来进行创建设备文件节点等工作。

由于udev根据系统中硬件设备的状态动态更新设备文件，进行设备文件的创建和删除等，因此在使用udev后**/dev**目录下就会只包含系统中真正存在的设备了。

devfs与udev的另一个显著区别在于：采用devfs，当一个并不存在的/dev节点被打开的时候，devfs能自动加载对应的驱动，而udev则不这么做。这是因为udev的设计者认为Linux应该在设备被发现的时候加载驱动模块，而不是当它被访问的时候。udev的设计者认为devfs所提供的打开/dev节点时自动加载驱动的功能对于一个配置正确的计算机是多余的。系统中所有的设备都应该产生热插拔事件并加载恰当的驱动，而udev能注意到这点并且为它创建对应的设备节点。

#### （未）5.4.2 sysfs文件系统与Linux设备模型

Linux 2.6的内核引入了**sysfs**文件系统，**sysfs**被看成是与proc、devfs和devpty同类别的文件系统，该文件系统是一个虚拟的文件系统，它可以产生一个包括所有**系统硬件**的层级视图，与提供进程和状态信息的 proc 文件系统十分类似。

sysfs 把连接在系统上的设备和总线组织成为一个分级的文件，它们可以由用户空间存取，向用户空间导出内核数据结构以及它们的属性。sysfs的一个目的就是展示设备驱动模型中各组件的层次关系，其顶级目录包括block、device、bus、drivers、class、power和firmware。

block目录包含所有的块设备；devices目录包含系统所有的设备，并根据设备挂接的总线类型组织成层次结构；bus目录包含系统中所有的总线类型；drivers目录包括内核中所有已注册的设备驱动程序；class目录包含系统中的设备类型（如网卡设备、声卡设备、输入设备等）。在/sys目录运行 tree 会得到一个相当长的树型目录，下面摘取一部分：

    |-- block
    | |-- fd0
    | |-- md0
    | |-- ram0
    | |-- ram1
    | |-- ...
    |-- bus
    | |-- eisa
    | | |-- devices
    | | '-- drivers
    | |-- ide
    | |-- ieee1394
    | |-- pci
    | | |-- devices
    | | | |-- 0000:00:00.0 -> ../../../devices/pci0000:00/0000:00:00.0
    | | | |-- 0000:00:01.0 -> ../../../devices/pci0000:00/0000:00:01.0
    | | | |-- 0000:00:07.0 -> ../../../devices/pci0000:00/0000:00:07.0
    | | '-- drivers
    | | |-- PCI_IDE
    | | | |-- bind
    | | | |-- new_id
    | | | '-- unbind
    | | '-- pcnet32
    | | |-- 0000:00:11.0 -> ../../../../devices/pci0000:00/0000:00:11.0
    | | |-- bind
    | | |-- new_id
    | | '-- unbind
    | |-- platform
    | |-- pnp
    | '-- usb
    | |-- devices
    | '-- drivers
    | |-- hub
    | |-- usb
    | |-- usb-storage
    | '-- usbfs
    |-- class
    | |-- graphics
    | |-- hwmon
    | |-- ieee1394
    | |-- ieee1394_host
    | |-- ieee1394_node
    | |-- ieee1394_protocol
    | |-- input
    | |-- mem
    | |-- misc
    | |-- net
    | |-- pci_bus
    | | |-- 0000:00
    | | | |-- bridge -> ../../../devices/pci0000:00
    | | | |-- cpuaffinity
    | | | '-- uevent
    | | '-- 0000:01
    | | |-- bridge -> ../../../devices/pci0000:00/0000:00:01.0
    | | |-- cpuaffinity
    | | '-- uevent
    | |-- scsi_device
    | |-- scsi_generic
    | |-- scsi_host
    | |-- tty
    | |-- usb
    | |-- usb_device
    | |-- usb_host
    | '-- vc
    |-- devices
    | |-- pci0000:00
    | | |-- 0000:00:00.0
    | | |-- 0000:00:07.0
    | | |-- 0000:00:07.1
    | | |-- 0000:00:07.2
    | | |-- 0000:00:07.3
    | |-- platform
    | | |-- floppy.0
    | | |-- host0
    | | |-- i8042
    | |-- pnp0
    | '-- system
    |-- firmware
    |-- kernel
    | '-- hotplug_seqnum
    |-- module
    | |-- apm
    | |-- autofs
    | |-- cdrom
    | |-- eisa_bus
    | |-- i8042
    | |-- ide_cd
    | |-- ide_scsi
    | |-- ieee1394
    | |-- md_mod
    | |-- ohci1394
    | |-- parport
    | |-- parport_pc
    | |-- usb_storage
    | |-- usbcore
    | | |-- parameters
    | | |-- refcnt
    | | '-- sections
    | |-- virtual_root
    | | '-- parameters
    | | '-- force_probe
    | '-- vmhgfs
    | |-- refcnt
    | '-- sections
    | '-- _ _versions
    '-- power
    '-- state

在/sys/bus的pci等子目录下，又会再分出 drivers 和 devices 目录，而 devices目录中的文件是对/sys/devices目录中文件的符号链接。同样地，/sys/class目录下也包含许多对/sys/devices下文件的链接。如图5.2所示，这与设备、驱动、总线和类的现实状况是直接对应的，也正符合Linux 2.6的设备模型。

随着技术的不断进步，系统的拓扑结构越来越复杂，对智能电源管理、热插拔以及即插即用的支持要求也越来越高，Linux 2.4内核已经难以满足这些需求。为适应这种形势的需要，Linux 2.6内核开发了上述全新的设备、总线、类和驱动环环相扣的设备模型。

大多数情况下，Linux 2.6内核中的设备模型代码会作为“幕后黑手”处理好这些关系，内核中的总线级和其他内核子系统会完成与设备模型的交互，这使得驱动工程师几乎不需要关心设备模型。

在Linux内核中，分别使用bus_type、device_driver和device来描述总线、驱动和设备，这3个结构体定义于`include/linux/device.h`头文件中。

#### （未）5.4.3 udev的组成

#### （未）5.4.4 udev规则文件

#### （未）5.4.5 创建和配置mdev

### （未）5.5 LDD6410的SD和NAND文件系统








