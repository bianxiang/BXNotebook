[toc]

## 前言

The purpose of this book is to remedy that situation and to enable you to embed Android in any device. 你将学习Android架构，如何阅读它的源码，how to modify its various components, and how to create your own version for your particular device. In addition, you will learn how Android integrates into the Linux kernel and understand the commonalities and differences it has with its Linux roots. 例如，我们将讨论Android如何利用Linux的驱动模型创建它自己的硬件层，and how to take “legacy” Linux components such as **glibc** and **BusyBox** and package them as part of Android. Along the way, you will learn day-to-day tips and tricks, such as how to use Android’s **repo** tool and how to integrate with or modify Android’s build system.

Little of what I knew about Linux and the packages it’s commonly used with in embedded systems applied to Android. Not only that, but the abstractions built in Android were even weirder still. 作者花了很久才完全理解Android各方面。In parallel, I started exploring the sources made available by Google through the Android Open Source Project (AOSP). In all honesty, it took me about 6 to 12 months before I actually started feeling confident enough to navigate within the AOSP.

尽管Android版本每6个月升级一次。但其底层内核变化较小。So while this book was first written with 2.3/Gingerbread in mind, it’s been relatively straightforward to update it to also cover 4.2/Jelly Bean with references included to other versions, including 4.0/Ice-Cream Sandwich and 4.1/Jelly Bean where relevant.

## 1. 介绍

### (未) Hardware and Compliance Requirements

### 开发准备和工具

At the most basic level, though, you need to have a Linux-based workstation to build the AOSP. In fact, at the time of this writing, Google’s only supported build environment is 64-bit Ubuntu 10.04. That doesn’t mean that another Ubuntu version or even another Linux distribution won’t work or, in the case of Android versions up to Gingerbread, that you won’t be able to build the AOSP on a 32-bit system,8 but essentially that configuration reflects Google’s own Android compile farms configuration.

No matter what your setup is, keep in mind that the AOSP is several gigabytes in size before building, and its final size is much larger. Gingerbread, for example, is about 3GB in size uncompiled and grows to about 10GB once compiled, while 4.2/Jelly Bean is 6GB uncompiled and grows to about 24GB once compiled.






