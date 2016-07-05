[toc]

## 3. 可移植性

本章分为两部分。第一个部分讲纯软件的可移植性。即你写的软件可以在各种Linux版本和分发上运行。第二是为不同的硬件平台写软件。You’ll learn about big and little endian computers, writing code that is 32/64-bit clean and briefly touch upon the need for the Linux kernel as a form of portable hard- ware abstraction layer in Linux systems.

### 3.1 可移植性的需求

Unix市场份额下降，出现其他类似Unix系统。为标准化，使得程序可移植。The standardization effort directly resulted in the Portable Operating System Interface (POSIX) standard. POSIX was later complemented by the (freely available) Single UNIX Specification (SUS).

### 3.2 Linux的可移植性

跨平台、跨分发的可移植性。Well-designed Linux software, that relies upon portable libraries and that is written to take full advantage of the variety of automated configuration tools available will stand up to each of these requirements.

#### 抽象的层

Linux系统的核心是Linux内容。It supports more than 24 different architectures of hardware platform in the standard “stock” release alone. When you think that a regular PC workstation is just one variant of the i386 architecture, you start to see the scale involved. In addition to amazing levels of hardware portability, Linux also provides numerous device drivers that allow a wide range of peripheral devices to be attached to a system running Linux. Linux内核的主要目的是在不同的硬件之上提供一个可移植的软件抽象。

When you write software for Linux, you will use standardized system libraries that rely upon specific features of the Linux kernel. The libraries will take care of providing commonly used software routines, while the kernel handles the pesky hardware details. In this way, you don’t need to concern yourself with the underlying hardware of the machine for many regular applications. Of course, if you are build- ing customized devices or need to get every ounce of performance out of the machine, then you might choose to voluntarily break this abstraction in your application, to further some greater goal.

不同的软件库版本也可能造成问题。本章后续讨论GNU Autotools时会讨论。

#### The Linux Standard Base

Although there is no such thing as a truly standard Linux system (yet), there are de facto standards (and a few more official standards besides) that determine, among other issues, the normal filesystem location and availability of libraries, configuration files, and applications. These Linux standards include the popular Linux Standard Base (LSB). The LSB is actually based upon the POSIX and Single UNIX specifi- cations, but it also adds additional considerations for computer graphics, the desktop, and many other areas besides. LSB favors the adoption of existing standards as a base rather than reinventing the wheel.You can obtain the latest version of the LSB (with examples) from: www.freestandards.org.

The Linux Standard Base aims to offer true binary compatibility between different vendor distributions — and between vendor releases — on a given host architecture conforming to a particular version of the LSB. The LSB is not Linux-specific, since it is possible (in theory) for any modern operating system to implement a particular level of the LSB (though in practice none do). Since the LSB is a binary ABI (Application Binary Interface) standard, your LSB-compliant packages should (in theory) be wildly portable between many systems and everything should just work — reality is somewhat different.

LSB uses RPM packages as the basis for its notion of a portable packaging format and provides various stipulations upon the naming and internal content of LSB-compliant packages. This chapter aims to be broadly in conformance with the standard (you’ll find a section later on testing LSB compliance in your application), but for a full understanding of the dos and don’ts of LSB-compliant packaging, you’ll want to check out one of the books specifically devoted to the current standard — check out the LSB website.

**LSB as a Panacea**

Alas, life is never as simple as one may desire it to be, and there are a number of stumbling blocks along the road to achieving universal portability through the LSB. Not least of these is the fact that the LSB is an evolving specification that is supported to varying extents by the existing Linux distributions. Each vendor supports particular levels of the LSB and may have been certified as meeting the various LSB requirements in particular configurations of a distribution. Thus, the LSB is not a panacea that will cure all of your inter-distribution ailments and make your life as a developer forever a life free from pain. What the LSB does offer in the long term, however, is the chance to make portability simpler.

The fact that some level of standardization exists doesn’t mean that you won’t have to deal with petty differences when they do arise. As a goal, aim to keep your assumptions as simple as you can. Avoid relying upon distribution-specific system capabilities that you don’t provide and that aren’t available on other Linux systems. If you must support a specific Linux distribution, you can usually detect which flavor of Linux is installed through a vendor-supplied release or version file and act accordingly.

The LSB is covered in a little more detail later in this chapter when building example packages with the RPM package management tool. An effort has been made to discuss certain criteria for LSB compliance and to explain these criteria as the examples progress. However, this book does not aim to replace the up-to-date documentation and examples available from the LSB website.

#### 基于RPM的分发

RPM is the standard packaging format used by the LSB, is widely adopted by anyone working with Linux systems and as a result is supported on some level by various distributions that don’t use it for regular system pack- age management. Debian and Ubuntu are good examples of this.

一些更高层的工具位于RPM之上。例如Fedora上的**yum**。yum isn’t a packaging system per se, but is rather an automated front-end that is capable of resolving other RPM dependencies that may arise during installation.

#### Debian派生

Those distributions that are based upon Debian (including Ubuntu and all of its variants) don’t use RPM natively. Debian has never decided to make any switchover to RPM, largely because they prefer some of the advanced capabilities of their own package management system, **dpkg**. Debian derivatives rely upon dpkg, the higher-level package resolution tool apt and graphical package managers like synaptic. There is a brief description of the build process for Debian packages included in this chapter but for more information (including detailed HOWTO guides), check out www.debian.org.

It is worth noting that much of the Debian project documentation is truly of excellent quality. Even though RPM may be a prevalent “de facto standard,” Debian makes it easy to use dpkg, too.

#### （未）构建包

到P97

#### 可移植的源代码











