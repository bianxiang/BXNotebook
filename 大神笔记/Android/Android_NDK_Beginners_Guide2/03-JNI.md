[toc]

## 3 Interfacing Java and C/C++ with JNI

It is a two-way bridge between the Java and native side; the only way to inject the power of C/C++ into your Java application.

### 初始化一个原生 JNI 库

Before accessing their native methods, native libraries must be loaded through a Java call to `System.loadLibrary()`. JNI provides a hook, `JNI_OnLoad()`, to plug your own initialization code. Let's override it to initialize our native store.

Create the **jni/Store.h** file, which defines store data structures:

- The `StoreType` enumeration will reflect the corresponding Java enumeration. Leave it empty for now.
- The `StoreValue` union will contain any of the possible store values. Leave it empty for now too.
- The `StoreEntry` structure contains one piece of data in the store. It is composed of a key (a raw C string made from `char*`), a type (`StoreType`), and a value (`StoreValue`).

`Store` is the main structure that defines a fixed size array of entries and a length (that is, the number of allocated entries):

```c
    #ifndef _STORE_H_
    #define _STORE_H_
    #include <cstdint>
    #define STORE_MAX_CAPACITY 16
    typedef enum {
    } StoreType;
    typedef union {
    } StoreValue;
    typedef struct {
    	char* mKey;
    	StoreType mType;
    	StoreValue mValue;
    } StoreEntry;
    typedef struct {
    	StoreEntry mEntries[STORE_MAX_CAPACITY];
    	int32_t mLength;
    } Store;
    #endif
```