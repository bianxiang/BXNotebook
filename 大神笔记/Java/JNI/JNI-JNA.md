[toc]

JNI与JNA

http://docs.oracle.com/javase/7/docs/technotes/guides/jni/spec/jniTOC.html
https://github.com/twall/jna

## JNI

Java Native Interface (JNI). Java代码与其他语言（如C、Cpp等）的应用、库互操作。

利用JNI，你的本地方法可以：

- 创建、查看、更新Java对象（包括数组和字符串）。
- 调用Java方法
- 捕获和抛出异常
- 加载类，获取类的信息
- 进行运行时的类型检查

JNI与Invocation API一起，可以让本地应用嵌入Java虚拟机。

除了JNI，还有一些遗留的技术，如：

- JDK 1.0 native method interface
- Netscape’s Java Runtime Interface
- Microsoft’s Raw Native Interface and Java/COM interface
- JDK 1.0 Native Method Interface

As of Java SE 6.0, the deprecated structures `JDK1_1InitArgs` and `JDK1_1AttachArgs` have been removed, instead `JavaVMInitArgs` and `JavaVMAttachArgs` are to be used.

### 设计概述

#### JNI接口函数和指针

本地代码通过JNI函数访问Java虚拟机。JNI函数通过一个接口指针获取。接口指针是一个指向指针的指针。它指向一个指针数组（an array of pointers），数组中每个指针指向一个接口函数。Every interface function is at a predefined offset inside the array.

JNI接口的组织类似于C++的虚函数表。The advantage to using an interface table, rather than hard-wired function entries, is that the JNI name space becomes separate from the native code. 一个虚拟机可以轻松提供多个版本的JNI函数表。例如，VM可以支持两个JNI函数表：

- one performs thorough illegal argument checks, and is suitable for debugging;
- the other performs the minimal amount of checking required by the JNI specification, 因此更高效.

JNI接口指针只在当前线程有效。本地方法不要将接口指针在线程间传递。A VM implementing the JNI may allocate and store thread-local data in the area pointed to by the JNI interface pointer.

JNI接口指针会作为参数传给本地函数。只要在同一个Java线程，调用多次本地方法，虚拟机保证传过去的接口指针是同一个。若一个本地方法被多个Java线程调用，会收到不同的JNI接口指针。

#### 编译、加载和链接本地方法

因为Java虚拟机是多线程的，编译本地库时也要使用懂多线程的本地编译器。For example, the `-mt` flag should be used for C++ code compiled with the Sun Studio compiler. 若使用GNU gcc编译器，要使用`-D_REENTRANT`或`-D_POSIX_C_SOURCE`。

本地方法通过`System.loadLibrary`加载。

```java
    package pkg;
    class Cls {
         native double f(int i, String s);
         static {
             System.loadLibrary(“pkg_Cls”);
         }
    }
```

`System.loadLibrary`是库的名字。库名到本地库实际名字的转换取决于系统。For example, a Solaris system converts the name `pkg_Cls` to `libpkg_Cls.so`, while a Win32 system converts the same `pkg_Cls` name to `pkg_Cls.dll`.

一个库中可以包含了几个类的所有本地方法，只要这些类使用同一个类加载器加载。虚拟机内部维护每个类加载已加载的库。

If the underlying operating system does not support dynamic linking, all native methods must be prelinked with the VM. In this case, the VM completes the `System.loadLibrary` call without actually loading the library.

程序还可以通过JNI函数`RegisterNatives()`注册关联的本地方法。` RegisterNatives()`在静态链接函数的情况下很有用。

##### 解析本地方法的名字

Dynamic linkers resolve entries based on their names. A native method name is concatenated from the following components:

- 前缀`Java_`
- 分隔的全限类名
- 下划线
- 分隔的方法名
- 对于重载（overloaded）的本地方法，两个下划线，后面是分隔的参数签名。

The VM checks for a method name match for methods that reside in the native library. 先查看短名，及没有参数签名的方法名。再查看长名，带参数签名。只有在本地方法与另外的本地方法充满（重载）时才需要使用长名。However, this is not a problem if the native method has the same name as a nonnative method. A nonnative method (a Java method) does not reside in the native library. 例如下面的例子中，本地方法`g`不需要使用长名链接，因为另一个`g`方法不是本地方法，不在本地库中。

```java
    class Cls1 {
      int g(int i);
      native int g(double d);
    }
```

We adopted a simple name-mangling scheme to ensure that all Unicode characters translate into valid C function names. We use the underscore (“_”) character as the substitute for the slash (“/”) in fully qualified class names. Since a name or type descriptor never begins with a number, we can use `_0`, ..., `_9` for escape sequences, as Table 2-1 illustrates:

Table 2-1 Unicode Character Translation

- `_0XXXX`：a Unicode character XXXX. Note that lower case is used to represent non-ASCII Unicode characters, e.g., `_0abcd` as opposed to `_0ABCD`.
- `_1`：字符“_”
- `_2`：签名中的字符“;”
- `_3`：签名中的字符 “[“

Both the native methods and the interface APIs follow the standard library-calling convention on a given platform. For example, UNIX systems use the C calling convention, while Win32 systems use `__stdcall`.

##### 本地方法参数

本地方法的第一个参数是JNI接口指针。JNI接口指针的类型是`JNIEnv`。第二个参数取决于本地方法是静态还是非静态的：非静态的本地方法第二个参数是对象引用，静态方法的第二个参数是Java类的引用。后面的参数是常规Java方法参数。

例子：

```java
    package pkg;
    class Cls {
         native double f(int i, String s);
         ...
    }
```

C函数的长名版本是：`Java_pkg_Cls_f_ILjava_lang_String_2`：

```c
    jdouble Java_pkg_Cls_f__ILjava_lang_String_2 (
         JNIEnv *env,        /* interface pointer */
         jobject obj,        /* "this" pointer */
         jint i,             /* argument #1 */
         jstring s)          /* argument #2 */
    {
         /* Obtain a C-copy of the Java string */
         const char *str = (*env)->GetStringUTFChars(env, s, 0);
         /* process the string */
         ...
         /* Now we are done with str */
         (*env)->ReleaseStringUTFChars(env, s, str);
         return ...
    }
```

CPP版本更简洁：

```cpp
    extern "C" /* specify the C calling convention */
    jdouble Java_pkg_Cls_f__ILjava_lang_String_2 (
         JNIEnv *env,        /* interface pointer */
         jobject obj,        /* "this" pointer */
         jint i,             /* argument #1 */
         jstring s)          /* argument #2 */
    {
         const char *str = env->GetStringUTFChars(s, 0);
         ...
         env->ReleaseStringUTFChars(s, str);
         return ...

    }
```

With CPP, the extra level of indirection and the interface pointer argument disappear from the source code. However, the underlying mechanism is exactly the same as with C. In CPP, JNI functions are defined as inline member functions that expand to their C counterparts.

#### 引用Java对象

Primitive types, such as integers, characters, and so on, are copied between Java and native code. Arbitrary Java objects, on the other hand, are passed by reference. 虚拟机必须记住所有传给本地代码的对象，防止它们被GC。本地代码需要能够告诉虚拟机它已经不需要某对象。In addition, the garbage collector must be able to move an object referred to by the native code.

##### 全局和局部引用

本地代码使用的对象引用分为两类：局部和全局引用。局部引用在本地方法执行期间有效，在本地方法返回后自动释放。全局引用一直有效直到被显式释放。

对象作为局部引用传给本地方法。JNI函数返回的所有Java对象都是局部引用。JNI允许通过局部引用创建全局引用。JNI functions that expect Java objects accept both global and local references. 本地方法可以返回给虚拟机一个局部或全局引用。

In most cases, the programmer should rely on the VM to free all local references after the native method returns. However, there are times when the programmer should explicitly free a local reference. Consider, for example, the following situations:

- 本地方法访问一个很大的Java对象，thereby creating a local reference to the Java object. The native method then performs additional computation before returning to the caller. The local reference to the large Java object will prevent the object from being garbage collected, even if the object is no longer used in the remainder of the computation.
- 本地方法创建了很多局部引用，although not all of them are used at the same time. Since the VM needs a certain amount of space to keep track of a local reference, creating too many local references may cause the system to run out of memory. For example, a native method loops through a large array of objects, retrieves the elements as local references, and operates on one element at each iteration. After each iteration, the programmer no longer needs the local reference to the array element.

The JNI allows the programmer to manually delete local references at any point within a native method. To ensure that programmers can manually free local references, JNI functions are not allowed to create extra local references, except for references they return as the result.

Local references are only valid in the thread in which they are created. 本地代码不用将局部引用在线程间传递。

##### 实现局部引用

To implement local references, the Java VM creates a registry for each transition of control from Java to a native method. A registry maps nonmovable local references to Java objects, and keeps the objects from being garbage collected. All Java objects passed to the native method (including those that are returned as the results of JNI function calls) are automatically added to the registry. The registry is deleted after the native method returns, allowing all of its entries to be garbage collected.

There are different ways to implement a registry, such as using a table, a linked list, or a hash table. Although reference counting may be used to avoid duplicated entries in the registry, a JNI implementation is not obliged to detect and collapse duplicate entries.

Note that local references cannot be faithfully implemented by conservatively scanning the native stack. The native code may store local references into global or heap data structures.

#### 访问Java对象

The JNI provides a rich set of accessor functions on global and local references. This means that the same native method implementation works no matter how the VM represents Java objects internally. This is a crucial reason why the JNI can be supported by a wide variety of VM implementations.

The overhead of using accessor functions through opaque references is higher than that of direct access to C data structures. We believe that, in most cases, Java programmers use native methods to perform nontrivial tasks that overshadow the overhead of this interface.

##### 访问原生数组

This overhead is not acceptable for large Java objects containing many primitive data types, such as integer arrays and strings. (Consider native methods that are used to perform vector and matrix calculations.) It would be grossly inefficient to iterate through a Java array and retrieve every element with a function call.

One solution introduces a notion of “pinning” so that the native method can ask the VM to pin down the contents of an array. The native method then receives a direct pointer to the elements. This approach, however, has two implications:

The garbage collector must support pinning.
The VM must lay out primitive arrays contiguously in memory. Although this is the most natural implementation for most primitive arrays, boolean arrays can be implemented as packed or unpacked. Therefore, native code that relies on the exact layout of boolean arrays will not be portable.
We adopt a compromise that overcomes both of the above problems.

First, we provide a set of functions to copy primitive array elements between a segment of a Java array and a native memory buffer. Use these functions if a native method needs access to only a small number of elements in a large array.

Second, programmers can use another set of functions to retrieve a pinned-down version of array elements. Keep in mind that these functions may require the Java VM to perform storage allocation and copying. Whether these functions in fact copy the array depends on the VM implementation, as follows:

If the garbage collector supports pinning, and the layout of the array is the same as expected by the native method, then no copying is needed.
Otherwise, the array is copied to a nonmovable memory block (for example, in the C heap) and the necessary format conversion is performed. A pointer to the copy is returned.
Lastly, the interface provides functions to inform the VM that the native code no longer needs to access the array elements. When you call these functions, the system either unpins the array, or it reconciles the original array with its non-movable copy and frees the copy.

Our approach provides flexibility. A garbage collector algorithm can make separate decisions about copying or pinning for each given array. For example, the garbage collector may copy small objects, but pin the larger objects.

A JNI implementation must ensure that native methods running in multiple threads can simultaneously access the same array. For example, the JNI may keep an internal counter for each pinned array so that one thread does not unpin an array that is also pinned by another thread. Note that the JNI does not need to lock primitive arrays for exclusive access by a native method. Simultaneously updating a Java array from different threads leads to nondeterministic results.

##### 访问字段和方法

The JNI allows native code to access the fields and to call the methods of Java objects. The JNI identifies methods and fields by their symbolic names and type signatures. A two-step process factors out the cost of locating the field or method from its name and signature. For example, to call the method f in class cls, the native code first obtains a method ID, as follows:


jmethodID mid =  env->GetMethodID(cls, “f”, “(ILjava/lang/String;)D”); 

The native code can then use the method ID repeatedly without the cost of method lookup, as follows:

jdouble result = env->CallDoubleMethod(obj, mid, 10, str);

A field or method ID does not prevent the VM from unloading the class from which the ID has been derived. After the class is unloaded, the method or field ID becomes invalid. The native code, therefore, must make sure to:

keep a live reference to the underlying class, or
recompute the method or field ID
if it intends to use a method or field ID for an extended period of time.

The JNI does not impose any restrictions on how field and method IDs are implemented internally.

#### Reporting Programming Errors

The JNI does not check for programming errors such as passing in NULL pointers or illegal argument types. Illegal argument types includes such things as using a normal Java object instead of a Java class object. The JNI does not check for these programming errors for the following reasons:

Forcing JNI functions to check for all possible error conditions degrades the performance of normal (correct) native methods.
In many cases, there is not enough runtime type information to perform such checking.
Most C library functions do not guard against programming errors. The printf() function, for example, usually causes a runtime error, rather than returning an error code, when it receives an invalid address. Forcing C library functions to check for all possible error conditions would likely result in such checks to be duplicated--once in the user code, and then again in the library.

The programmer must not pass illegal pointers or arguments of the wrong type to JNI functions. Doing so could result in arbitrary consequences, including a corrupted system state or VM crash.

#### Java异常

The JNI allows native methods to raise arbitrary Java exceptions. The native code may also handle outstanding Java exceptions. The Java exceptions left unhandled are propagated back to the VM.

##### Exceptions and Error Codes

Certain JNI functions use the Java exception mechanism to report error conditions. In most cases, JNI functions report error conditions by returning an error code and throwing a Java exception. The error code is usually a special return value (such as NULL) that is outside of the range of normal return values. Therefore, the programmer can:

quickly check the return value of the last JNI call to determine if an error has occurred, and
call a function, ExceptionOccurred(), to obtain the exception object that contains a more detailed description of the error condition.
There are two cases where the programmer needs to check for exceptions without being able to first check an error code:

The JNI functions that invoke a Java method return the result of the Java method. The programmer must call ExceptionOccurred() to check for possible exceptions that occurred during the execution of the Java method.
Some of the JNI array access functions do not return an error code, but may throw an ArrayIndexOutOfBoundsException or ArrayStoreException.
In all other cases, a non-error return value guarantees that no exceptions have been thrown.

##### Asynchronous Exceptions

In cases of multiple threads, threads other than the current thread may post an asynchronous exception. An asynchronous exception does not immediately affect the execution of the native code in the current thread, until:

the native code calls one of the JNI functions that could raise synchronous exceptions, or
the native code uses ExceptionOccurred() to explicitly check for synchronous and asynchronous exceptions.
Note that only those JNI function that could potentially raise synchronous exceptions check for asynchronous exceptions.

Native methods should insert ExceptionOccurred()checks in necessary places (such as in a tight loop without other exception checks) to ensure that the current thread responds to asynchronous exceptions in a reasonable amount of time.

##### Exception Handling

There are two ways to handle an exception in native code:

The native method can choose to return immediately, causing the exception to be thrown in the Java code that initiated the native method call.
The native code can clear the exception by calling ExceptionClear(), and then execute its own exception-handling code.
After an exception has been raised, the native code must first clear the exception before making other JNI calls. When there is a pending exception, the JNI functions that are safe to call are:


  ExceptionOccurred()
  ExceptionDescribe()
  ExceptionClear()
  ExceptionCheck()
  ReleaseStringChars()
  ReleaseStringUTFChars()
  ReleaseStringCritical()
  Release<Type>ArrayElements()
  ReleasePrimitiveArrayCritical()
  DeleteLocalRef()
  DeleteGlobalRef()
  DeleteWeakGlobalRef()
  MonitorExit()
  PushLocalFrame()
  PopLocalFrame()



