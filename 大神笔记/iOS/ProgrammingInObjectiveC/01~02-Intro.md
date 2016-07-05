[toc]

## 1 介绍

In 1994, NeXT Computer and Sun Microsystems released a standardized specification of the NEXTSTEP system, called OPENSTEP. The FSF’s implementation of OPENSTEP is called GNUStep. A Linux version, which also includes the Linux kernel and the GNUStep development environment, is called, appropriately enough, LinuxSTEP.

Objective-C 是 C 的扩展并不表示要先学 C。前者是面向对象的语言。后者是面向过程的语言。
本书不会分开教 C 和 Objective-C，而是在面向对象的视角下，以统一一门语言教。

## 2 Objective-C 编程

```m
    // First program example
    #import <Foundation/Foundation.h>
    int main (int argc, const char * argv[])
    {
        @autoreleasepool {
        	NSLog (@"Programming is fun!");
        }
        return 0;
    }
```

### 2.1 编译运行

You can both compile and run your program using Xcode, or you can use the **Clang** Objective-C compiler in a Terminal window.

**Xcode**

select Create a New Xcode Project

选 OSX，Application，Command Line Tool。

Objective-C 源文件的扩展名是 `.m`。`.mm` 是 Objective-C++ 源文件的扩展名。

**命令行**

需要 Xcode’s Command Line Tools。

    sh-2.05a$ mkdir Progs Create a directory to store programs in
    sh-2.05a$ cd Progs Change to the new directory
    sh-2.05a$ vi main.m Start up a text editor to enter program

You can use the LLVM Clang Objective-C compiler, which is called clang, to compile and link your program. This is the general format of the `clang` command:

	clang -fobjc-arc files -o program

`files` is the list of files to be compiled.

We’ll call the program prog1 ; here, then, is the command line to compile your first Objective-C program:

	$ clang -fobjc-arc main.m -o prog1 Compile main.m & call it prog1

测试执行：

	$ . /prog1 Execute prog1
	2012-09-03 18:48:44.210 prog1[7985:10b] Programming is fun!

### 2.2 第一个程序的解释

注释支持 `//` 和 `/* */`。

```
	#import <Foundation/Foundation.h>
```

`#import` says to import or include the information from that file into the program, exactly as if the contents of the file were typed into the program at that point.

```
	@"Programming is fun!"
```

Here, the `@` sign immediately precedes a string of characters enclosed in a pair of double quotes. Collectively, this is known as a constant `NSString` object. Without that leading `@` character, you are writing a constant **C-style** string; with it, you are writing an `NSString` string object.

语句结尾必须有分号。

### 2.3 显示变量的值

```m
    #import <Foundation/Foundation.h>
    int main (int argc, const char *argv[])
    {
        @autoreleasepool {
            int sum;
            sum = 50 + 25;
            NSLog (@"The sum of 50 and 25 is %i", sum);
        }
    return 0;
    }
```

变量必须先定义再使用。










