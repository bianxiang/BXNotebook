[toc]

## （未）JNI Tips

JNI is the Java Native Interface. 先阅读[JNI规范](http://docs.oracle.com/javase/7/docs/technotes/guides/jni/spec/jniTOC.html)。

### JavaVM and JNIEnv

JNI defines two key data structures, "JavaVM" and "JNIEnv". Both of these are essentially pointers to pointers to function tables. (In the C++ version, they're classes with a pointer to a function table and a member function for each JNI function that indirects through the table.) The `JavaVM` provides the "invocation interface" functions, 用于创建和销毁JavaVM。理论上一个进程可以有多个JavaVM，但Android只允许一个。

`JNIEnv`提供多数JNI函数。本地函数的第一个参数都是`JNIEnv`。

`JNIEnv`用于thread-local存储。因此，不能在多个线程之间共享`JNIEnv`。If a piece of code has no other way to get its `JNIEnv`, you should share the `JavaVM`, and use `GetEnv` to discover the thread's `JNIEnv`. (Assuming it has one; see `AttachCurrentThread` below.)

C和CPP版的`JNIEnv`、`JavaVM`声明不同。The "jni.h" include file provides different typedefs depending on whether it's included into C or C++. For this reason it's a bad idea to include `JNIEnv` arguments in header files included by both languages. (Put another way: if your header file requires `#ifdef __cplusplus`, you may have to do some extra work if anything in that header refers to `JNIEnv`.)

### 线程

所有的线程都是Linux线程，由内核调度。They're usually started from managed code (using `Thread.start`), but they can also be created elsewhere and then attached to the JavaVM. For example, a thread started with `pthread_create` can be attached with the JNI `AttachCurrentThread` or `AttachCurrentThreadAsDaemon` functions. Until a thread is attached, it has no JNIEnv, and cannot make JNI calls.

Attaching a natively-created thread causes a `java.lang.Thread` object to be constructed and added to the "main" `ThreadGroup`, making it visible to the debugger. Calling `AttachCurrentThread` on an already-attached thread is a no-op.

Android不会挂起正在执行本地代码的线程。If garbage collection is in progress, or the debugger has issued a suspend request, Android will pause the thread the next time it makes a JNI call.

Threads attached through JNI must call `DetachCurrentThread` before they exit. If coding this directly is awkward, in Android 2.0 (Eclair) and higher you can use `pthread_key_create` to define a destructor function that will be called before the thread exits, and call `DetachCurrentThread` from there. (Use that key with pthread_setspecific to store the JNIEnv in thread-local-storage; that way it'll be passed into your destructor as the argument.)

### jclass, jmethodID, and jfieldID

If you want to access an object's field from native code, you would do the following:

- Get the class object reference for the class with `FindClass`
- Get the field ID for the field with `GetFieldID`
- Get the contents of the field with something appropriate, such as `GetIntField`

Similarly, to call a method, you'd first get a class object reference and then a method ID. The IDs are often just pointers to internal runtime data structures. Looking them up may require several string comparisons, but once you have them the actual call to get the field or invoke the method is very quick.

If performance is important, it's useful to look the values up once and cache the results in your native code. Because there is a limit of one JavaVM per process, it's reasonable to store this data in a static local structure.

The class references, field IDs, and method IDs are guaranteed valid until the class is unloaded. Classes are only unloaded if all classes associated with a ClassLoader can be garbage collected, which is rare but will not be impossible in Android. Note however that the `jclass` is a class reference and must be protected with a call to `NewGlobalRef` (see the next section).

If you would like to cache the IDs when a class is loaded, and automatically re-cache them if the class is ever unloaded and reloaded, the correct way to initialize the IDs is to add a piece of code that looks like this to the appropriate class:

```java
    /*
     * We use a class initializer to allow the native code to cache some
     * field offsets. This native function looks up and caches interesting
     * class/field/method IDs. Throws on failure.
     */
    private static native void nativeInit();

    static {
        nativeInit();
    }
```

Create a `nativeClassInit` method in your C/C++ code that performs the ID lookups. The code will be executed once, when the class is initialized. If the class is ever unloaded and then reloaded, it will be executed again.

### 局部与全局引用

Every argument passed to a native method, and almost every object returned by a JNI function is a "local reference". This means that it's valid for the duration of the current native method in the current thread. Even if the object itself continues to live on after the native method returns, the reference is not valid.

This applies to all sub-classes of `jobject`, including `jclass`, `jstring`, and `jarray`. (The runtime will warn you about most reference mis-uses when extended JNI checks are enabled.)

The only way to get non-local references is via the functions `NewGlobalRef` and `NewWeakGlobalRef`.

If you want to hold on to a reference for a longer period, you must use a "global" reference. The `NewGlobalRef` function takes the local reference as an argument and returns a global one. The global reference is guaranteed to be valid until you call `DeleteGlobalRef`.

This pattern is commonly used when caching a jclass returned from `FindClass`, e.g.:

```cpp
    jclass localClass = env->FindClass("MyClass");
    jclass globalClass = reinterpret_cast<jclass>(env->NewGlobalRef(localClass));
```

All JNI methods accept both local and global references as arguments. It's possible for references to the same object to have different values. For example, the return values from consecutive calls to `NewGlobalRef` on the same object may be different. To see if two references refer to the same object, you must use the `IsSameObject` function. 不用在本地代码中使用`==`比较引用。

One consequence of this is that you must not assume object references are constant or unique in native code. The 32-bit value representing an object may be different from one invocation of a method to the next, and it's possible that two different objects could have the same 32-bit value on consecutive calls. Do not use `jobject` values as keys.

Programmers are required to "not excessively allocate" local references. In practical terms this means that if you're creating large numbers of local references, perhaps while running through an array of objects, you should free them manually with `DeleteLocalRef` instead of letting JNI do it for you. The implementation is only required to reserve slots for 16 local references, so if you need more than that you should either delete as you go or use `EnsureLocalCapacity`/`PushLocalFrame` to reserve more.

Note that jfieldIDs and jmethodIDs are opaque types, not object references, and should not be passed to `NewGlobalRef`. The raw data pointers returned by functions like GetStringUTFChars and GetByteArrayElements are also not objects. (They may be passed between threads, and are valid until the matching Release call.)

One unusual case deserves separate mention. If you attach a native thread with AttachCurrentThread, the code you are running will never automatically free local references until the thread detaches. Any local references you create will have to be deleted manually. In general, any native code that creates local references in a loop probably needs to do some manual deletion.







