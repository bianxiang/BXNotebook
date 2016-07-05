[toc]

## 4 使用 SWIG 自动产生 JNI 代码

As noted, implementing JNI wrapper code and handling the translation of data types is a cumbersome and time-consuming development task. This chapter will introduce the **Simplified Wrapper and Interface Generator (SWIG)**, a development tool that can simplify this process by automatically generating the necessary JNI wrapper code.

SWIG is not an Android- and Java-only tool. It is a highly extensive tool that can generate code in many other programming languages. As SWIG is rather large, this chapter will only cover the following key SWIG concepts and APIs that will get you started:

- Defining a SWIG interface for native code.
- Generating JNI code based on the interface.
- Integrating SWIG into the Android Build process.
- Wrapping C and C++ code.
- Exception handling.
- Using memory management.
- Calling Java from native code.

As SWIG simplifies the development of JNI code, you will be using SWIG often in the next chapters.

### 4.1 SWIG 是啥？

SWIG is a compile-time software development tool that can produce the code necessary to connect native modules written in C/C++ with other programming languages, including Java. SWIG is an interface compiler, merely a code generator; it does not define a new protocol nor is it a component framework or a specialized runtime library. SWIG takes an interface file as its input and produces the necessary code to expose that interface in Java. SWIG is not a stub generator; it produces code that is ready to be compiled and run.

SWIG was originally developed in 1995 for scientific applications; it has since evolved into a general-purpose tool that is distributed under GNU GPL open source license. More information about SWIG can be found at www.swig.org.

### 4.2 安装

SWIG works on all major platforms, including Windows, Mac OS X, and Linux.

MAC 下用 Homebrew：

```
brew install swig
swig -version
```

### 4.3 通用一个例子实验 SWIG

每个 Linux 用户都有一个ID。通过 POSIX OS API `getuid` 函数可以查询。下面我们将：

- 编写一个 `SWIG` 接口文件，暴露 `getuid` 函数。
- 将 SWIG 继承到 Android 的构建过程。
- 将 SWIG 产生的源文件集成到 Android.mk
- 利用 SWIG 产生的代理类查询 `getuid`

You will be using the **hello-jni** sample project as a testbed. Open Eclipse IDE, and go into the **hello-jni** project. As mentioned earlier, SWIG operates on interface files.

#### 4.3.1 接口文件

SWIG 接口文件的语法与任何 `C/C++` 文件相同；包含函数原型、类、变量声明。In addition to `C/C++` keywords and preprocessor directives, the interface files can also contain SWIG specific preprocessor directives that can enable tuning the generated wrapper code.

In order to expose getuid, an interface file needs to be defined. 在 JNI 目录下创建文件 `Unix.i`。

```
    /* Module name is Unix. */
    %module Unix
    %{
    /* Include the POSIX operating system APIs. */
    #include < unistd.h>
    %}
```

{{文本不全！}}

##### 4.3.1.2 模块名

Each invocation of SWIG requires a module name to be specified. The module name is used to name the resulting wrapper files. The module name is specified using the SWIG specific preprocessor directive `%module`, and it should appear at the beginning of every interface file. Complying with this rule, the `Unix.i` interface file also starts by defining its module name as `Unix`, as shown in Listing 4-3.

```
%module Unix
```

##### 4.3.1.3 用户定义的代码

SWIG only uses the interface file while generating the wrapper code; its content does not go beyond this point. It is often necessary to include user-defined code in the generated files, such as any header file that is required to compile the generated code. When SWIG generates a wrapper code, it is broken up to into five sections. SWIG provides preprocessor directives to enable developers to specify the code snippets that should be included in any of these five sections. The syntax for SWIG preprocessor directive is shown in Listing 4-4.

```
    % < section > %{
    . . .
    This code block will be included into generated code as is.
    . . .
    %}
    . . .
```

The `<section>` portion can take following values:

- `begin`: Places the code block at the beginning of the generated wrapper file. It is mostly used for defining preprocessor macros that are used in the later part of the file.
- `runtime`: Places the code block next to SWIG’s internal type-checking and other support functions.
- `header`: Places the code block into the header section next to header files and other helper functions. This is the default place to inject code into the generated files. The `%{ . . . %}` can also be used the short form.
- `wrapper`: Places the code block next to generated wrapper functions.
- `init`: Places the code block into to function that will initialize the module upon loading.

As shown in Listing 4-5, the `Unix.i` interface file injects a header file into the generated wrapper code using the short form of insert header preprocessor directive.

```
    %{
    /* Include the POSIX operating system APIs. */
    #include < unistd.h>
    %}
```

##### 4.3.1.4 类型定义

SWIG can understand all *C/C++* data types but treats anything else as objects and wraps them as pointers. The declaration of the `getuid` function suggests that its return type is `uid_t`, which is not a standard *C/C++* data type. As is, SWIG will treat it as an object, and it will wrap it as a pointer. This is not the preferred behavior since `uid_t` is nothing more than a simple typedef-name based on unsigned integer, not an object. As shown in Listing 4-6, the `Unix.i` interface file uses a `typedef` to inform SWIG about the actual return type of `getuid` function.

```
/* Tell SWIG about uid_t. */
typedef unsigned int uid_t;
```

##### 4.3.1.5 函数原型

The `Unix.i` interface file ends with the function prototype for the `getuid` function as shown in Listing 4-7.
Listing 4-7. Function Prototype for getuid

```
	/* Ask SWIG to wrap getuid function. */
```

{{代码不全！！！}}

#### 4.3.2 从命令行调用 SWIG




