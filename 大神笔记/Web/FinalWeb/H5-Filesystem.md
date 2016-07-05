[toc]

## Filesystem

> Filesystem更像是一个本地存储的API。它只能访问一个私有的沙箱。不能访问用户文件系统的任何部分。不能实现读取指定路径下的文件，获取其内容。沙箱内的文件需要你先创建，才会存在。总之，该API更像一个本地文件缓存存储。

参考：

- 《Javascript权威指南》 22.7 The Filesystem API

The `File` and `Blob` types are defined by a draft specification known as the **File API**. 另一份草案，比File API更加新，让Web应用可以访问一个私有的、本地文件系统沙箱，在其中，可以读写文件，创建目录，列出目录等。目前，Filesystem API仅被Chrome实现。This section covers basic filesystem tasks but does not demonstrate all features of the API. Because the API is new and unstable, it is not documented in the reference section of this book.

Working with files in the local filesystem is a multistep process. First, you must obtain an object that represents the filesystem itself. There is a synchronous API for doing this in worker threads and an asynchronous API for use on the main thread:

```js
    // 同步获取文件系统。Pass filesystem lifetime and size.
    // Returns a filesystem object or raises an exception.
    var fs = requestFileSystemSync(PERSISTENT, 1024*1024);

    // 异步版本
    requestFileSystem(TEMPORARY, // lifetime
        50*1024*1024, // size: 50Mb
		function(fs) {
        }, function(e) {
        	console.log(e);
        });
```

You specify the lifetime and the size of the filesystem you want. A `PERSISTENT` filesystem is suitable for web apps that want to store user data permanently. The browser won’t delete it except at the user’s explicit request. A `TEMPORARY` filesystem is appropriate for web apps that want to cache data but can still operate if the web browser deletes the filesystem. The size of the filesystem is specified in bytes and should be a reasonable upper bound on the amount of data you need to store. A browser may enforce this as a quota.

The filesystem obtained with these functions depends on the origin of the containing document. All documents or web apps from the same origin (host, port, and protocol) share a filesystem. Two documents or applications from different origins have completely distinct and disjoint filesystems. 该文件系统与用户磁盘上的其他文件是分离的，没有办法从该文件系统去到磁盘的其他部分。

Note that these functions have “request” in their names. 第一次调用时，浏览器会询问用户是否允许。Once permission has been granted, subsequent calls to the request method should simply return an object that represents the already existing local filesystem.

The filesystem object you obtain with one of the methods above has a root property that refers to the root directory of the filesystem. This is a `DirectoryEntry` object, and it may have nested directories that are themselves represented by `DirectoryEntry` objects. Each directory in the file system may contain files, represented by `FileEntry` objects. The `DirectoryEntry` object defines methods for obtaining DirectoryEntry and FileEntry objects by pathname (they will optionally create new directories or files if you specify a name that doesn’t exist). DirectoryEntry also defines a createReader() factory method that returns a DirectoryReader for listing the contents of a directory.

The FileEntry class defines a method for obtaining the File object (a Blob) that represents the contents of a file. You can then use a FileReader object (as shown in §22.6.5) to read the file. FileEntry defines another method to return a FileWriter object that you can use to write content into a file.

Reading or writing a file with this API is a multistep process. First you obtain the filesystem object. Then you use the root directory of that object to look up (and optionally create) the FileEntry object for the file you’re interested in. Then you use the FileEntry object to obtain the File or FileWriter object for reading or writing. This multistep process is particularly complex when using the asynchronous API:

```js
    // Read the text file "hello.txt" and log its contents.
    // This example doesn't include any error callbacks.
    requestFileSystem(PERSISTENT, 10*1024*1024, function(fs) {
    	fs.root.getFile("hello.txt", {}, function(entry) {
        	entry.file(function(file) {
                var reader = new FileReader();
                reader.readAsText(file);
                reader.onload = function() {
                		console.log(reader.result);
                };
            });
        });
    });
```

Example 22-13 is a more fully fleshed-out example. It demonstrates how to use the asynchronous API to read files, write files, delete files, create directories, and list directories.

（略，见书）

Working with files and the filesystem is quite a bit easier in worker threads, where it is okay to make blocking calls and we can use the synchronous API. Example 22-14 defines the same filesystem utility functions as Example 22-13 does, but it uses the synchronous API and is quite a bit shorter.

（略，见书）
