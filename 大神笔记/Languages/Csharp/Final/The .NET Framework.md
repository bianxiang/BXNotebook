[toc]

## .NET框架

.NET Framework可以运行在非Windows系统。例如 Mono，一个开源的.NET Framework（包括C#编译器），可以运行在Linux和Mac OS。还有 Mono 的变体可以运行在 iPhone (MonoTouch) 和 Android (Mono for Android, a.k.a. MonoDroid)。

.NET Framework可以被多种语言使用，如C#、C++、VB等。这些语言不仅可以访问.NET Framework，之间还可以相互通讯。例如，C#开发者可以使用VB开发的代码，反之亦然。

### 组成

.NET Framework包含一个巨大的库。

不同的操作系统支持不同的模块。

作为.NET Framework库的一部分，定义了一些基本类型。这些类型在使用.NET Framework的不同语言之间是互操作的。它们称为**Common Type System (CTS)**。

除了提供库，.Net Framework还提供一个**.NET Common Language Runtime (CLR)**，负责执行所有使用.NET开发的库。

### 编译

编译成目标平台的代码在.NET Framework中分两个阶段。

先把代码编译为**Common Intermediate Language(CIL)**代码。这种代码独立于操作系统，独立于语言（如C#）。

> CIL之前也被称为MSIL或IL。

后续工作，由JIT编译器，将CIL编译为本地代码。JIT表示代码是按需编译的。

有多种不同的JIT编译器，针对不同架构。

### Assemblies

When you compile an application, the CIL code is stored in an assembly. Assemblies include both executable application files that you can run directly from Windows without the need for any other programs (these have a .exe file extension) and libraries (which have a .dll extension) for use by other applications.

除了包含CIL，assemblies还包含元数据和可选的资源（CIL使用的其他数据，如音乐和图片）。The meta information enables assemblies to be fully self-descriptive. You need no other information to use an assembly, meaning you avoid situations such as failing to add required data to the system registry and so on, which was often a problem when developing with other platforms.

This means that deploying applications is often as simple as copying the files into a directory on a remote computer. Because no additional information is required on the target systems, you can just run an executable file from this directory and (assuming the .NET CLR is installed) you’re good to go.

Of course, you won’t necessarily want to include everything required to run an application in one place. You might write some code that performs tasks required by multiple applications. In situations like that, it is often useful to place the reusable code in a place accessible to all applications. In the .NET Framework, this is the **global assembly cache (GAC)**. Placing code in the GAC is simple — you just place the assembly containing the code in the directory containing this cache.

### 受管理的代码

使用.NET Framework编写的代码在运行时是受管理的。CLR负责管理应用内存、安全跨语言调试等。不受CLR管理的应用，如用C++编写的应用，能够访问操作系统底层函数。但用C#只能写的代码只能运行在受管理的环境。由.NET处理与操作系统的交互。

### 链接

The C# code that compiles into CIL in step 2 needn’t be contained in a single file. It’s possible to split application code across multiple source-code files, which are then compiled together into a single assembly. This extremely useful process is known as linking. It is required because it is far easier to work with several smaller files than one enormous one. You can separate logically related code into an individual file so that it can be worked on independently and then practically forgotten about when completed. This also makes it easy to locate specific pieces of code when you need them and enables teams of developers to divide the programming burden into manageable chunks, whereby individuals can “check out” pieces of code to work on without risking damage to otherwise satisfactory sections or sections other people are working on.

