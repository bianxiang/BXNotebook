[toc]


## 1. Linux内核介绍

### （未）1.1 History of Unix

### （未）1.2 Along Came Linus: Introduction to Linux

### 1.3 操作系统与内核概述

内核典型的组件有：中断处理器处理中断请求，调度器在多个进程之间共享处理器时间，内存管理系统管理进程的地址空间，系统服务，如网络和跨进程通信。在现代的带有受保护的内存管理单元的系统中，内核一般具有更高权限，包括一个受保护的内存空间和对硬件的完全访问。这个系统状态与内存空间一起称为内核空间。与此相反，用户应用在用户空间执行。

应用与内核通过系统调用通讯。应用调用的库函数（如C库）靠系统调用通知内核代码应用执行任务。库调用提供很多系统调用不具备的功能，或者说，系统调用只是库方法中一小步。例如`printf()`函数，提供格式化和缓冲，但只需要一步系统调用`write()`。

硬件通过中断与系统通讯。例如当你输入时，键盘控制器发出中断，让系统知道在键盘缓冲中有新数据。中端处理器不在进程上下文中运行。它们在一个特殊的、称为中断上下文中运行；中断上下文不予任何进程关联；该上下文的存在只是为了让中断处理器能够快速响应中断。

**在任一时刻，处理器做的事情是下面三者之一**：

- 在用户空间，执行用户代码
- 在内核空间，在进程的上下文，代表一个进程执行
- 在内核空间，在中断上下文，不与任何进程关联，处理中断

处理器不可能处于三者之外的状态。例如即使空闲时，内核在进程上下文中执行一个空闲进程。

### （未）1.4 Linux Versus Classic Unix Kernels

### 1.5 Linux内核版本

Linux内核有两种：稳定版和开发版。二者命名方式很简单：第2位（minor）还表示内核是稳定还是开发版：偶数表示稳定版，奇数表示开发版。

For example, the first version of the 2.6 kernel series was 2.6.0. The next was 2.6.1. These revisions contain bug fixes, new drivers, and new features, but the difference between two revisions—say, 2.6.3 and 2.6.4—is minor.

This is how development progressed until 2004, when at the invite-only Kernel Developers Summit, the assembled kernel developers decided to prolong the 2.6 kernel series and postpone the introduction of a 2.7 development series. The rationale was that the 2.6 kernel was well received, stable, and sufficiently mature such that new destabilizing features were unneeded. This course has proven wise, as the ensuing years have shown 2.6 is a mature and capable beast. As of this writing, a 2.7 development series is not on the table and seems unlikely. Instead, the development cycle of each 2.6 revision has grown longer, each release incorporating a mini-development series. Andrew Morton, Linus’s second-in-command, has repurposed his 2.6-mm tree—once a testing ground for memory management-related changes—into a general-purpose test bed. Destabilizing changes thus flow into 2.6-mm and, when mature, into one of the 2.6 mini-development series. Thus, over the last few years, each 2.6 release—for example, 2.6.29—has taken several months, boasting significant changes over its predecessor. This “development series in miniature” has proven rather successful, maintaining high levels of stability while still introducing new features and appears unlikely to change in the near future. Indeed, the consensus among kernel developers is that this new release process will continue indefinitely.

To compensate for the reduced frequency of releases, the kernel developers have introduced the aforementioned stable release. This release (the 8 in 2.6.32.8) contains crucial bug fixes, often back-ported from the under-development kernel (in this example, 2.6.33). In this manner, the previous release continues to receive attention focused on stabilization.

### 1.6 内核开发社区

The main forum for this community is the Linux Kernel Mailing List (oft-shortened to lkml). Subscription information is available at http://vger.kernel.org.