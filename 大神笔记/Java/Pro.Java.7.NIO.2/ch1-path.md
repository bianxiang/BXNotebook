[toc]

“JSR 203: More New I/O APIs for the Java Platform” (NIO.2)

## 1. `Path`类

推荐从新的抽象类`java.nio.file.Path`开始探索NIO.2 API。实际中它是NIO.2最常用的类，很多I/O操作都基于它。

`Path`类支持两种类型的操作：语法（syntactic）操作（不需要访问文件系统的路径操作，属于在内存中进行的逻辑操作），and operations over files referenced by paths. 本章介绍第一种操作。第四章介绍第二种操作。The concepts presented in this chapter will be very useful in the rest of the book.

### 1.1 `Path`类介绍

可以通过`java.nio.file.FileSystems`访问文件系统；用于获取`java.nio.file.FileSystem`对象。`FileSystems`包含下面两个重要的方法，即一些`newFileSystem()`方法：

- `getDefault()`：静态方法，返回JVM默认的`FileSystem`，一般是操作系统默认的文件系统。
- `getFileSystem(URI uri)`：静态方法，从一组可以的文件系统中，根据给定的URI选出一个文件系统。`Path`可以操纵任意文件系统（`FileSystem`）中的文件；文件系统可以使用任何底层存储（`java.nio.file.FileStore`）。默认（一般），`Path`表示默认文件系统（计算机的文件系统）中的文件；但NIO.2是完全模块化的——an implementation of `FileSystem` for data in memory, on the network, or on a virtual file system is perfectly agreeable to NIO.2。NIO.2 provides us with all file system functionalities that we may need to perform over a file, a directory, or a link.

`Path`类是`java.io.File`的升级版本；但`File`类仍保留了一些特殊的操作，因此不能将其视为已过时（deprecated, obsolete）。二者可以转换。

之前的写法：

    File file = new File("index.html");

现在的写法：

    import java.nio.file.Path;
    import java.nio.file.Paths;

    Path path = Paths.get("index.html");


`Path`中可以包含文件名、目录名，也可以包含OS特定的路径分隔符（“\” on Microsoft Windows and forward slash “/” on Solaris and Linux）。即`Path`不是平台独立的。由于`Path`基本上知识个字符串，因此其引用的资源可以不存在。



