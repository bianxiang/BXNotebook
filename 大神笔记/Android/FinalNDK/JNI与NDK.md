[toc]

## JNI

JNI 使得 Java 跟 C/Cpp 可以**相互**调用。

用 Android SDK 开发 Java 程序。使用 NDK 开发 C/Cpp 程序。通过 JNI 集成在一起。

### 在 Java 中调用 C库函数

> 注意，本节展示的是普通 Java 程序，不是 Android 应用。

步骤：1、编写、编译 Java 代码；2、**根据 Java 代码生成C语言头文件**；3、编写C代码；4、产生共享库。

HelloJNI.java

```java
class HelloJNI {

    native void printHello();
    native void printString(String str);

	// 加载库
    static { System.loadLibrary("hellojni");}
    public static void main(String[] args) {
        HelloJNI myJNI = new HelloJNI();
        myJNI.printHello();
        myJNI.printString("Hello world");
    }
}
```

加注 `native` 的方法是本地方法。该方法与用 C/Cpp 编写的本地函数相对应。

`System.loadLibrary("hellojni");` 加载了库 `hellojni`。在 Windows 和 Linux 下，库文件分别是 hellojni.ddl 和 libhellojni.so。

利用JDK工具 **javah**（在bin目录下）生成本地方法的头文件：

```
javah HelloJNI
```

注意，上述调用假设类的包为空（注意上面的类代码最上面没有 `pacakge` 语句）。但如果类有报名，例如类 `cn.a.b.Test`，要在生成 .class 文件的根目录 —— 注意不是在源码目录，而是在 .class 文件根目录，`cn` 在该目录下 —— 调用：

```
javah -cp . cn.a.b.Test
```

将在 .class 文件的根目录生成 `cn_a_b_Test.h`。

在相同目录生成 `HelloJNI.h`。不要手工修改该文件。

```c
	#include <jni.h>
    /* Header for class HelloJNI */
    #ifndef _Included_HelloJNI
    #define _Included_HelloJNI
    #ifdef __cplusplus
    extern "C" {
    #endif
    /*
     * Class: HelloJNI
     * Method: printHello
     * Signature: ()V
     */
	JNIEXPORT void JNICALL Java_HelloJNI_printHello(JNIEnv *, jobject);
    /*
     Class: HelloJNI
     Method: printString
     Signature: (Ljava/lang/String;)V
     */
	JNIEXPORT void JNICALL Java_HelloJNI_printString(JNIEnv *, jobject, jstring);
	#ifdef __cplusplus
    }
    #endif
    #endif
```

`JNIEXPORT` 和 `JNICALL` 都是宏，定义在 JDK 的 jni_md.h 中。

生成的函数原型，第一个、第二个参数类型分别是 `JNIEnv *` 和 `jobject`（如果是静态方法，第二个参数类型是 `jclass`）。第一个参数是 JNI 接口的指针，用来调用 JNI 表中各种 JNI函数。这里 JNI 函数（非 JNI 本地函数）指 JNI 提供的基本函数集，用来在 JNI 本地函数中创建 Java 对象或调用相应方法（具体下一节介绍）。

第二个参数 `jobject` 指向调用本地方法的对象，例如这里是 `myJNI` 对象。

为了解决 Java 和 C/CPP 中数据类型大小不一致的问题，JNI 提供了一套与 Java 数据类型相对应的 Java 本地类型。使得本地语言可以使用 Java 数据类型。

Java 类型 byte、short、int、long、float、double、char、boolean 对应的本地类型是 Java 类型名前加 `j`，如 jbyte、jshort、jint 等。void 的本地类型为 void。类、对象、字符串的本地类型分别是 jclass、jobject 和 jstring。这些本地类型定义在 JDK 的 `include/jni.h` 和 `include/platform/jni_md.h` 中。此外还有一些类型，参见官方JNI文档。

有了函数原型后，可以编写实现：hellojin.c。

```c
	#include "HelloJNI.h"
    #include <stdio.h>
    JNIEXPORT void JNICALL Java_Hello_printHello(JNIEnv *env, jobject obj) {
    	printf("Hello\n");
    }
    JNIEXPORT void JNICALL Java_Hello_printString(JNIEnv *env, jobject obj, jstring string) {
    	// 将Java String转换为C字符串
        const char * str = (*env)->GetStringUTFChars(env, string, 0);
        printf("%s\n", str);
    }
```

最后，生成平台上的共享库（见后面的NDK一节）。

### 本地函数中使用Java代码

访问静态成员和实例成员。

```c
    public class JniFuncMain {
        private static int staticIntField = 300;
        static { System.loadLibrary("jnifunc"); }
        public static native JniTest createJniObject();
        public static void main(String[] args) {
            JniTest jniObj = createJniObject(); // 实现调用本地方法创建Java对象
            jniObj.callTest();
        }
    }
```

JniTest类。

```c
    class JniTest {
        private int intField;
        public JniTest(int num) {
            intField = num;
        }
        // 此方法会被JNI本地函数调用
        public int callByNative(int num) {
            return num;
        }
        public void callTest() {
            System.out.println("intField="+intField);
        }
    }
```

接下来用 javah 生成 `JniFuncMain.h` 头文件。

junifunc.cpp 文件：

```c
    JNIEXPORT jobject JNICALL Java_JniFuncMain_createJniObject(JNIEnv * env, jclass clazz) {
        jclass targetClass;
        jmethodId mid;
        jobject newObject;
        jstring helloStr;
        jfieldID fid;
        jint staticIntField;
        jint result;
        // 获取J niFuncMain 类的 staticIntField 变量值
        fid = env->GetStaticFieldID(clazz, "staticIntField", "I");
        staticIntField = env->GetStaticIntField(clazz, fid);
        // 查找目标对象的类
        targetClass = env->FindClass("JniTest");
        // 查找构造方法
        mid = env->GetMethodID(targetClass, "<init>", "(I)V");
        // 实例化JniTest对象
        newObject = env->NewObject(targetClass, mid, 100);
        // 调用对象的方法
        mid = env->GetMethodID(targetClass, "callByNative", "(I)I");
        result = env->CallIntMethod(newObject, mid, 200);
        // 设置对象的字段
        fid = env->GetFieldID(targetClass, "intField", "I");
        env->SetIntField(newObject, fid, result);
		// 返回新创建的对象
        return newObject;
    }
```

以上大量用到成员函数和变量的签名。通过 JDK 的 javap 命令可以获取成员变量或方法的签名。

### C 代码启动 Java 虚拟机并调用 Java 类（的 main()函数）

Java 必须运行在 Java 虚拟机上。在 C/CPP 中运行 Java 也必须使用 Java 虚拟机。为此 JNI 提供了一套 Invocation API，他允许本地代码在自身内存区域内加载Java虚拟机。

运行 Java 程序的 java.exe 命令是由 C 编写的。该命令通过 Invocation API 接收命令参数，并执行命令参数指定的 Java 类。Android 系统的 dalvikvm 虚拟机的 Launcher 程序也是通过 Invocation API 启用的。

InvocationApiTest.java

```java
    public class InvocationApiTest {
        public static void main(String[] args) {
            System.out.println(args[0]);
        }
    }
```

invocationApi.c

```c
    #include <jni.h>
    int main() {
        JNIEnv * env;
        JavaVM * vm;
        JavaVMInitArgs vm_args;
        JavaVMOption options[1];
        jint res;
        jclass cls;
        jmethodID mid;
        jstring jstr;
        jclass stringClass;
        jobjectArray args;
        // 生成Java虚拟机选项
        options[0].optionString = "-Djava.class.path=."
        vm_args.version = 0x00010002;
        vm_args.options = options;
        vm_args.nOptions = 1;
        vm_args.ignoreUnrecognized = JNI_TRUE;
        // 生成Java虚拟机
        res = JNI_CreateJavaVM(&vm, (void**)&env, &vm_args);
        // 查找并加载类
        cls = (*env)0>FindClass(env, "InvocationApiTest");
        // 获取main()方法的ID
        mid = (*env)->GetStaticMethodID(env, cls, "main", "([LJava/lang/String;)V");
        // 生成字符串对象，用作main()方法的参数
        jstr = (*env)->NewStringUTF(env, "Hello Invocation API!!");
        stringClass = (*env)->FindCLass(env, "java/lang/String");
        args = (*env)->NewObjectArray(env, 1, stringClass, jstr);
        // 调用main()方法
        (*env)->CallStaticVoidMethod(env, cls, mid, args);
        // 销毁Java虚拟机
        (*vm)->DestroyJavaVM(vm);
    }
```

`JavaVMInitArgs`和`JavaVMOption`结构体放在头文件**jni.h**。

### 直接注册JNI本地函数

Java 虚拟机在运行包含本地方法的 Java 代码时，要经过以下两个步骤：1、调用 `System.loadLibrary()` 方法，将包含本地方法实现的库加载到内存。2、Java 虚拟机检索加载进来的库函数符号，在其中查找与Java本地方法拥有相同签名的JNI本地函数符号。

这种自动检索、映射的方式要花费不少时间。为减少这些时间，JNI 提供 RegisterNatives() 函数，让开发者将 JNI 本地函数与Java 类本地方法直接映射在一起。而且此时不再需要 JNI 本地函数与 Java 本地函数签名完全一致。

调用 `System.loadLibrary()` 加载共享库后，Java 虚拟机会检索共享库内的函数符号，检查 `JNI_OnLoad()` 函数是否被实现，若有，则调用。否则，Java 虚拟机会会将 Java 本地方法与 JNI 本地函数进行比较匹配（耗时）。`RegisterNatives()` 函数适合在 `JNI_OnLoad()` 函数内调用。

下面是 hellojnimap.cpp：

```cc
    #include "jni.h"
    #include <stdio.h>
    // JNI本地函数原型
    void printHelloNative(JNIEnv *env, jobject obj);
    void printStringNative(JNIEnv *env, jobject obj, jstring string);

    JNIEXPORT jint JNICALL JNI_OnLoad(JavaVM *vm, void *reserved)
    {
        JNIEnv* env = null;
        JNINativeMethod nm[2];
        jclass cls;
        jint result = -1;
        if (vm->GetEnv((void**)&env, JNI_VERSION_1_4) != JNI_OK) {
            printf("Error");
            return JNI_ERR;
        }
        cls = env->FindClass("HelloJNI");
        nm[0].name = "printHello";
        nm[0].signature = "()V";
        nm[0].fnPtr = (void *)printHelloNative;

        nm[1].name  = "printString";
        nm[1].signature = "(Ljava/lang/String;)V";
        nm[1].fnPtr = (void *)printStringNative;

        env->RegisterNatives(cls, nm, 2);

        return JNI_VERSION_1_4;
    }

    // 实现JNI本地函数
    void printHelloNative(JNIEnv * env, jobject obj) {
        printf("Hello World\n");
    }
    void printStringNative(JNIEnv * env, jobject obj, jstring string) {
        const char *str = env->GetStringUTFChars(string, 0);
        printf("%s\n", str);
    }
```

## NDK

利用 Android NDK 能轻松地开发出基于JNI机制的应用。
本书使用版本是 r4。
功能如下:

- 将C/C++源代码编译成本地库的工具（编译器、链接器等）
- 将编译好的本地库插入apk
- 在生成本地库时，Android平台可支持的系统头文件与库

Android NDK 提供了一些稳定可靠的系统头文件和库。这些库在新版 Android 会一直被支持。每次 NDK 版本更新会加入一些新的库。如 OpenGL ES 2.0 库是 NDK r3 时添加进来的。

- libc：C库
- libm：数学库
- JNI接口头文件
- libz：zlib压缩
- liblog：Android日志
- OpenGL ES 2.0
- libjnigraphics
- CPP的支持库

把 `NDK_HOME` 添加到 `PATH`。

配置好后，尝试运行 `ndk-build` 命令。

在 NDK_HOME\apps 下有一些 NDK 的示例程序。

例子：求两数只和。libndk-exam.so 实现本地方法 `add()`。它由 first.c 和 second.c 两个源文件构成。本地方法 `add()` 与 second.c 的 `Java_org_example_ndk_NDKExam_add()` 链接在一起。

```java
    package org.example.ndk;
    import ...
    public class NDKExam extends Activity {
        public void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            TextView tv = new TextView(this);
            int x=1000, y=42;
            System.loadLibrary("ndk-exam");
            int z = add(x, y);
            tv.setText("z="+z);
            setContentView(tv);
        }
        public native int add(int x, int y);
    }
```

生成class文件，利用 javah 生成本地方法的头文件：`org_example_ndk_NDKExample.h`。

在Android工程下，创建jni目录。在jni目录下，创建 second.c。

```c
    #include "first.h"
    #include <jni.h>
    jint Java_org_example_ndk_NDKExam_add(JNIEnv* env, jobject this, jint x, jint y)
    {
        return first(x, y);
    }
```
下面是 first.c：

```c
    #include "first.h"
    int first(int x, int y)
    {
        return x + y;
    }
```

下面是first.h：

```c
    #ifndef FIRST_H
    #define FIRST_H
    extern int first(int x, int y);
    #endif
```

接下来用 NDK 编译。要先编写 Android.mk 文件。然后执行 ndk-build 脚本。Android.mk 文件语法参见doc/Android-Mk.txt。ndk-build 语法参见 docs/ndk-build.text。

NDK 默认存放 Android.mk 的位置是工程的 jni 目录。在 jni 目录下创建 Android.mk：

```
	# 指定源文件的位置
    LOCAL_PATH := $(call my-dir)
    # 初始化与Make相关的环境变量
    include $(CLEAR_VARS)
    # 库编译相关信息
    LOCAL_MODULE := ndk-exam
    LOCAL_SRC_FIELS := first.c second.c
    # 生成共享库
    include $(BUILD_SHARED_LIBRARY)
```

`include $(CLEAR_VARS)` 用来初始化 Android.mk 文件中 `LOCAL_` 开头的变量（`LOCAL_PATH` 除外。由于 Android 编译系统会将 `LOCAL_` 变量用作全局变量，所有需要使用该命令初始化这些变量。

进入工程**根目录**，运行 `ndk-build`。指定 `ndk-build` 命令时，编译系统会在当前目录下查找AndroidManifest.xml 文件。若该文件存在，将把当前目录看做 Android 工程目录。然后转到 jni 目录下，根据 Android.mk 进程 NDK 编译。

编译完成后，会在 libs 目录下找到 libndk-exam.so 文件。















