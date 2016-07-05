## 文件

[toc]

A [File](http://developer.android.com/reference/java/io/File.html) object is suited to reading or writing large amounts of data in start-to-finish order without skipping around. For example, it's good for image files or anything exchanged over a network.

> Tip: If you want to save a static file in your application at compile time, save the file in your project `res/raw/` directory. You can open it with `openRawResource()`, passing the `R.raw.<filename>` resource ID. This method returns an InputStream that you can use to read the file (but you cannot write to the original file).

### 选择内部外部存储

所有的Android设备都有两块存储区：内部和外部。一些设备，即使没有SD卡，只有永久存储，也把存储区划分为内部和外部两部分。

下面总结它们的特征：

- 内部存储：
	- 总是可用
	- 此区域的文件默认只可被自己应用访问
	- 卸载App时，系统删除内部存储的所有文件
- 外部存储：
	- 不总是可用。外部存储可能挂载成USB存储，或从设备上移除。
	- 全局可读。
	- 卸载App后，只删除`getExternalFilesDir()`目录下的文件。

### 存储权限

读写内部存储不需要任何权限。

要向外部存储写必须请求`WRITE_EXTERNAL_STORAGE`权限。

	<manifest ...>
	    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
	    ...
	</manifest>

注意：目前，**读取**外部存储不需要特殊的权限。但将来可能改变。因此，如果需要只读，最好也申请`READ_EXTERNAL_STORAGE`。

	<manifest ...>
	    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
	    ...
	</manifest>

如果申请了`WRITE_EXTERNAL_STORAGE`权限，会隐式获取读权限。

### 内部存储

API：

- `getFilesDir()` 应用的内部目录，返回File对象
- `getCacheDir()` 应用临时缓存文件的内部目录。文件不再使用时记得删除。注意限制使用量，如1MB。如果系统内存变少，**在不加警告的情况下删除文件**。
- `getDir()` 在内部存储中创建（和打开已存在的）目录。
- `deleteFile()` 删除内部存储中的一个文件
- `fileList()` Returns an array of files currently saved by your application.

在获取目录后，创建文件引用：

	File file = new File(context.getFilesDir(), filename);

如果是写，可以通过`openFileOutput()`获取一个 `FileOutputStream`：

	String filename = "myfile";
	String string = "Hello world!";
	FileOutputStream outputStream;
	
	try {
	  outputStream = openFileOutput(filename, Context.MODE_PRIVATE);
	  outputStream.write(string.getBytes());
	  outputStream.close();
	} catch (Exception e) {
	  e.printStackTrace();
	}

若想缓存文件，使用 `createTempFile()`。

	public File getTempFile(Context context, String url) {
	    File file;
	    try {
	        String fileName = Uri.parse(url).getLastPathSegment();
	        file = File.createTempFile(fileName, null, context.getCacheDir());
	    catch (IOException e) {
	        // Error while creating file
	    }
	    return file;
	}

> App的内部存储目录位于Android文件系统下一个特殊位置，由包名定义。Technically, another app can read your internal files if you set the file mode to be readable. 此时，它仍需要知道你的包名和文件名。Other apps cannot browse your internal directories and do not have read or write access unless you explicitly set the files to be readable or writable. So as long as you use `MODE_PRIVATE` for your files on the internal storage, they are never accessible to other apps.

### 外部存储

> 向Media Scanner隐藏你的文件
 Include an empty file named .nomedia in your external files directory (note the dot prefix in the filename). This prevents media scanner from reading your media files and providing them to other apps through the MediaStore content provider. 但如果想让你的文件完全私有，应该将它们存放在*app-private*目录。

外部存储可以被用户或其他App需改。文件可分为两类：

- 公开目录：自有共享；删除应用后，文件不删除。如`Music/`、`Pictures/`、`Ringtones/`。By saving your files to the corresponding media-type directory, the system's media scanner can properly categorize your files in the system (for instance, ringtones appear in system settings as ringtones, not as music).
- 私有文件：文件权利上属于你的App，在用户卸载后应删除。

#### 可用性检查

由于外部存储可能无法使用，使用前先查询状态。调用` Environment.getExternalStorageState()`，如果返回`MEDIA_MOUNTED`，则可读写。

	/* Checks if external storage is available for read and write */
	public boolean isExternalStorageWritable() {
	    String state = Environment.getExternalStorageState();
	    if (Environment.MEDIA_MOUNTED.equals(state)) {
	        return true;
	    }
	    return false;
	}
	
	/* Checks if external storage is available to at least read */
	public boolean isExternalStorageReadable() {
	    String state = Environment.getExternalStorageState();
	    if (Environment.MEDIA_MOUNTED.equals(state) ||
	        Environment.MEDIA_MOUNTED_READ_ONLY.equals(state)) {
	        return true;
	    }
	    return false;
	}

#### 公有目录

保存公开文件。用`getExternalStoragePublicDirectory()`。传入参数指定文件类型，如`DIRECTORY_MUSIC` 或 `DIRECTORY_PICTURES`。

	public File getAlbumStorageDir(String albumName) {
	    // Get the directory for the user's public pictures directory. 
	    File file = new File(Environment.getExternalStoragePublicDirectory(
	            Environment.DIRECTORY_PICTURES), albumName);
	    if (!file.mkdirs()) {
	        Log.e(LOG_TAG, "Directory not created");
	    }
	    return file;
	}

#### 私有目录

> Note: Also, the system media scanner does not read files in these directories, so they are not accessible from the MediaStore content provider. As such, you should not use these directories for media that ultimately belongs to the user, such as photos captured or edited with your app, or music the user has purchased with your app—those files should be saved in the public directories. 注意拥有`READ_EXTERNAL_STORAGE`的其他应用仍可以访问你的私有外部存储。如果想完全禁止，只能写到内部存储。

If you want to save files that are private to your app, you can acquire the appropriate directory by calling` getExternalFilesDir()` and passing it a name indicating the type of directory you'd like. Each directory created this way is added to a parent directory that encapsulates **all your app's** external storage files, 卸载后系统会删除该目录。

有时，设备会将部分内部内存用作外部存储，同时提供SD卡槽。若这种设备使用Android 4.3及之前版本，`getExternalFilesDir()`只能访问内部分区，你的App不能读写SD卡。从Android 4.4你可以通过`getExternalFilesDirs()`访问两个位置，返回一个File数组。第一项被看组是主外部存储，应该被首先使用，除非它已满或不可用。如果想在Android 4.3及之前访问这些位置，使用支持库的静态方法`ContextCompat.getExternalFilesDirs()`。它也返回一个File数组。

For example, here's a method you can use to create a directory for an individual photo album:

	public File getAlbumStorageDir(Context context, String albumName) {
	    // Get the directory for the app's private pictures directory. 
	    File file = new File(context.getExternalFilesDir(
	            Environment.DIRECTORY_PICTURES), albumName);
	    if (!file.mkdirs()) {
	        Log.e(LOG_TAG, "Directory not created");
	    }
	    return file;
	}

If none of the pre-defined sub-directory names suit your files, you can instead call `getExternalFilesDir()` and pass `null`. This returns the root directory for your app's private directory on the external storage.

无论使用`getExternalStoragePublicDirectory()`还是`getExternalFilesDir()`，记得传入正确的文件类型（如`DIRECTORY_PICTURES`）。这使得系统能够正确处理文件。For instance, files saved in `DIRECTORY_RINGTONES` are categorized by the system media scanner as ringtones instead of music.

#### 缓存
To open a File that represents the external storage directory where you should save cache files, call getExternalCacheDir(). 用户卸载应用后，这些文件会被自动删除。

Similar to ContextCompat.getExternalFilesDirs(), mentioned above, you can also access a cache directory on a secondary external storage (if available) by calling ContextCompat.getExternalCacheDirs().


### 查询空闲空间

`getFreeSpace()`获取当前可以空间。`getTotalSpace()`获取总空间。

However, the system does not guarantee that you can write as many bytes as are indicated by getFreeSpace(). If the number returned is a few MB more than the size of the data you want to save, or if the file system is less than 90% full, then it's probably safe to proceed. Otherwise, you probably shouldn't write to storage.

Note: 可以不检查直接写。会报出IOException。

### 删除文件

The most straightforward way to delete a file is to have the opened file reference call delete() on itself.

	myFile.delete();

如果文件位于内部存储，可以通过`Context.deleteFile()`删除：

	myContext.deleteFile(fileName);

卸载应用后，Android系统删除以下文件：

- 内部存储中的所有文件
- 使用`getExternalFilesDir()`存储的所有文件

However, you should manually delete all cached files created with `getCacheDir()` on a regular basis and also regularly delete other files you no longer need.


























