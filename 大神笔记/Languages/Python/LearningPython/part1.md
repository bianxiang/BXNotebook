[toc]

## Part I 入门

## 1. Python 问答

## 2. Python 如何运行程序

### 程序执行

Python首先将源码编译为字节码。字节码是低级的、平台独立的。Roughly, Python translates each of your source statements into a group of byte code instructions by decomposing them into individual steps. 字节码比源代码运行速度快很多。

如果Python进程有写权限，它会将字节码保存到文件`.pyc`。Prior to Python 3.2, you will see these files show up on your computer after you’ve run a few programs alongside the corresponding source code files—that is, in the samedirectories.

In 3.2 and later, Python instead saves its `.pyc` byte code files in a subdirectory named `__pycache__` located in the directory where your source files reside, and in files whose names identify the Python version that created them (e.g., `script.cpython-33.pyc`). The new `__pycache__` subdirectory helps to avoid clutter, and the new naming convention for byte code files prevents different Python versions installed on the same computer from overwriting each other’s saved byte code. We’ll study these byte code file models in more detail in Chapter 22, though they are automatic and irrelevant to most Python programs, and are free to vary among the alternative Python implementations described ahead.

Python is happy to run a program if all it can find are `.pyc` files, even if the original `.py` source files are absent.

Finally, keep in mind that byte code is saved in files only for files that are imported, not for the top-level files of a program that are only run as scripts (strictly speaking, it’s an import optimization).

### （未）执行模型差异

## （未）3. 你如何运行程序



