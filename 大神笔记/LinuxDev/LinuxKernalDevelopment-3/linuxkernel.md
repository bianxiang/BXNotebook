[toc]

The learning curve for the kernel is getting longer and steeper. The system is becoming increasingly complex, and it is very large.And as the years pass, the current members of the kernel development team gain deeper and broader knowledge of the kernel’s internals, which widens the gap between them and newcomers.

First and foremost, this is a book about the design and implementation of the Linux kernel.

More important, however, is the decision made by the Linux kernel community to not proceed with a 2.7 development kernel in the near to mid-term.(This decision was made in the summer of 2004 at the annual Linux Kernel Developers Summit.) Instead, kernel developers plan to continue developing and stabilizing the 2.6 series. This decision has many implications, but the item of relevance to this book is that there is quite a bit of staying power in a contemporary book on the 2.6 Linux kernel.As the Linux kernel matures, there is a greater chance of a snapshot of the kernel remaining representative long into the future. This book functions as the canonical documentation for the kernel, documenting it with both an understanding of its history and an eye to the future.

**内核版本**

**This book is based on the 2.6 Linux kernel series.** It does not cover older kernels, except for historical relevance. Specifically, this book is up to date as of Linux kernel version 2.6.34.

Although this book discusses the 2.6.34 kernel, I have made an effort to ensure the material is factually correct with respect to the 2.6.32 kernel as well. That latter version is sanctioned as the “enterprise” kernel by the various Linux distributions, ensuring we will continue to see it in production systems and under active development for many years. (2.6.9, 2.6.18, and 2.6.27 were similar “long-term” releases.)

**读者**

Nor is it a guide to developing drivers or a reference on the kernel API. Instead, the goal of this book is to provide enough information on the design and implementation of the Linux kernel that a sufficiently accomplished programmer can begin developing code in the kernel.

