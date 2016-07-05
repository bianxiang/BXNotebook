[toc]

## 前言

### 本书使用的硬件

For this book, we’ve used a number of embedded systems to test the various procedures. Some of these systems, such as the OpenMoko-based NEO 1973, are commercial products available in the mainstream market. We included these intentionally, to demonstrate that any willing reader can find the materials to support learning how to build embedded Linux systems. You can, of course, still use an old x86 PC for experimenting, but you are likely to miss much of the fun, given the resemblance between such systems and most development hosts.

To illustrate the range of target architectures on which Linux can be used, we varied the target hardware we used in the examples between chapters. Though some chapters are based on different architectures, the commands given in each chapter apply readily to other architectures as well. If, for instance, an example in a chapter relies on the arm-linux-gcc command, which is the gcc compiler for ARM, the same example would work for a PPC target by using the powerpc-linux-gcc command instead. Whenever more than one architecture is listed for a chapter, the main architecture discussed is the first one listed.

Unless specific instructions are given to the contrary, the host’s architecture is always different from the target’s. In Chapter 4, for example, we used a PPC host to build tools for an x86 target. The same instructions could, nevertheless, be carried out on a SPARC or an S/390 with little or no modification. Note that most of the content of the early chapters is architecture-independent, so there is no need to provide any architecture-specific commands.

### 软件版本

This book concentrates on version 2.6 of the Linux kernel, and 2.6.22 in particular.

## 1. 介绍






