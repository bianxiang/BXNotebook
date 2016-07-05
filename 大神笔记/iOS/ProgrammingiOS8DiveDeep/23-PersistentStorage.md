[toc]

## （未）23 持久化存储

### 23.1 沙箱

存储设备上只有一块区域对应用可见，对其他应用不可见，即沙箱。

#### 23.1.1 标准目录

沙箱内有一些标准目录，如获取 Documents 目录。下面的方法返回 Documents 目录的路径字符串：

```swift
let docs = NSSearchPathForDirectoriesInDomains(.DocumentDirectory, .UserDomainMask, true).last as String
```

但引用目录和文件更好的办法是使用 URL。You can obtain this from an `NSFileManager` instance:

```swift
let fm = NSFileManager()
var err : NSError?
if let docsurl = fm.URLForDirectory(.DocumentDirectory, inDomain: .UserDomainMask, appropriateForURL: nil, create: true, error: &err) {
	// use docsurl here
} else {
	// error, examine err here
}
```

用户可以通过 iTunes 查看和修改应用的 Documents 目录。如果不想让用户看见某些文件或目录，则不要放在 Documents 目录。Personally, I favor the Application Support directory for most purposes. iOS 应用在沙箱中有自己私有的 Application Support 目录。这个目录开始不能传在，可以在获取时同时创建它：

```swift
let fm = NSFileManager()
var err : NSError?
if let suppurl = fm.URLForDirectory(.ApplicationSupportDirectory, inDomain: .UserDomainMask, appropriateForURL: nil, create: true, error: &err) {
	// use suppurl here
} else {
	// error, examine err here
}
```

临时文件，丢失也不要紧的文件，可以放在 Caches 目录（`NSCachesDirectory`）或 Temporary 目录（`NSTemporaryDirectory`）。如果将临时文件写到 Application Support 文件夹，默认它们会被 iTunes 或 iCloud 备份；若不想某个文件被备份，可以对其URL设置：

```swift
myDocumentURL.setResourceValue(true, forKey: NSURLIsExcludedFromBackupKey, error: &err)
```

如果某个地方一定要一个路径，而你手上有一个 `NSURL` 对象，可以调用其 `path` 方法获取路径。

#### 23.1.2 查看沙箱内容

The Simulator’s sandbox is a folder on your Mac that you can inspect visually. This is more difficult with Xcode 6 than in previous versions, because the sandbox folder layout has changed in iOS 8. In your user ~/Library/Developer/CoreSimulator/Devices folder, you’ll find mysteriously named folders representing the different simulators. The device.plist file inside each folder can help you identify which simulator a folder represents; so can `xcrun simctl list` at the command line.

Inside a simulator’s data/Containers/Data/Application folder are more mysteriously named folders representing apps on that simulator. I don’t know how to identify the different apps, but one of them is the app you’re interested in, and inside it is that app’s sandox. In Figure 23-1, I’ve drilled down from my user Library to an app that I’ve run in the Simulator. My app’s Documents folder is visible, and I’ve opened it to show a folder and a couple of files that I’ve created programmatically (the code that created that folder and files will appear later in this chapter). Also visible is the app’s Library folder; the app’s Application Support folder is inside it.

也可以查看设上上的沙箱，这反而比看模拟器上的更容易。When the device is connected and no app is being run from Xcode, choose *Window → Devices*. Select your device on the left; on the right, under Installed Apps, select your app. Click the Gear icon and choose *Show Container* to view your app’s sandbox hierarchy in a modal sheet. Alternatively, choose *Download Container* to copy your app’s sandbox to your computer; the sandbox arrives on your computer as an .xcappdata package, and you can open it in the Finder with *Show Package Contents*.

#### 23.1.3 基本的文件操作

假如我们要在 Documents 目录下创建文件夹 MyFolder。在上面几节的基础上，我们先构造到 MyFolder 的医用，然后利用 NSFileManager 创建它：

```swift
var err : NSError?
let myfolder = docsurl.URLByAppendingPathComponent("MyFolder")
	var ok = fm.createDirectoryAtURL(myfolder, withIntermediateDirectories: true, attributes: nil, error: &err)
    // ... error-checking omitted
```

列出给定文件夹下的文件和目录。`contentsOfDirectoryAtURL:...` 返回一个数组，数组元素是
 `NSURL`。
```swift
var err : NSError?
let arr = fm.contentsOfDirectoryAtURL(docsurl, includingPropertiesForKeys: nil, options: nil, error: &err)
// ... error-checking omitted
(arr! as [NSURL]).map{$0.lastPathComponent}.map(println)
```

若想列出文件夹下所有目录和文件，包括任意深度下的，可以穷举整个目录。`enumeratorAtURL` 方法返回一个 enumerator，一次只能获取一项文件或目录：

```swift
let dir = fm.enumeratorAtURL(docsurl, includingPropertiesForKeys: nil, options: nil, errorHandler: nil)!
	while let f = dir.nextObject() as? NSURL {
		if f.pathExtension == "txt" {
			println(f.lastPathComponent)
	}
}
```

enumerator 允许你跳过一个子目录（`skipDescendants`）。

Consult the `NSFileManager` class documentation for more about what you can do with files, and see also Apple’s File System Programming Guide.

#### 23.1.4 读写文件

利用特定数据类型的方法，如 `NSString`、 `NSData`、 `NSArray` 和 `NSDictionary` 都提供方法 `writeToURL:...` 和 `init(contentsOfURL:...)`，分别用于写和读。

`NSString` and `NSData` objects map directly between their own contents and the contents of the file. Here, I’ll generate a text file directly from a string:

```swift
var err : NSError?
ok = "howdy".writeToURL(myfolder.URLByAppendingPathComponent("file1.txt"), atomically: true, encoding: NSUTF8StringEncoding, error: &err)
// ... error-checking omitted
```

> You can also read and write an attributed string using a file in a standard format, as I mentioned in Chapter 10.

`NSArray` 和 `NSDictionary` 文件其实是属性列表；因此需要数组或字典的内容必须是属性列表类型（NSString, NSData, NSDate, NSNumber, NSArray, and NSDictionary）。对于其他类型若想也能被存储，需要实现 `NSCoding` 协议，利用 `NSKeyedArchiver` 和 `NSKeyedUnarchiver` 与 `NSData` 转换。一个实现 `NSCoding` 的类，如果有一个属性，其类型也实现了 `NSCoding` 协议，则通过此属性关联的对象也会被存储。

下面是一个简单的例子。更多知识参见 Apple’s Archives and Serializations Programming Guide。

```swift
class Person: NSObject, NSCoding {
    @NSCopying var firstName : NSString
    @NSCopying var lastName : NSString
    override var description : String {
    	return self.firstName + " " + self.lastName
    }
    init(firstName:String, lastName:String) {
        self.firstName = firstName
        self.lastName = lastName
        super.init()
    }
}
```

为满足 `NSCoding` 协议，需要实现两个方法，`encodeWithCoder:` 和 `init(coder:)`，分别用于归档和解档。在 `encodeWithCoder:` 方法中，如果父类也实现了 `NSCoding` 协议，我们必须先调用 `super`。但这里不同：

```swift
func encodeWithCoder(coder: NSCoder) {
    // do not call super in this case
    coder.encodeObject(self.lastName, forKey: "last")
    coder.encodeObject(self.firstName, forKey: "first")
}
```

In `init(coder:)`, we call the appropriate `decode...` method for each property stored earlier, thus restoring the state of our object. We must also call super, using either `init(coder:)` if the superclass adopts the NSCoding protocol or the designated initializer if not:

```swift
required init(coder: NSCoder) {
    self.lastName = coder.decodeObjectForKey("last")! as String
    self.firstName = coder.decodeObjectForKey("first")! as String
    // do not call super init(coder:) in this case
    super.init()
}
```

下面测试将 Person 对象存储到文件中：

```swift
let moi = Person(firstName: "Matt", lastName: "Neuburg")
let moidata = NSKeyedArchiver.archivedDataWithRootObject(moi)
let moifile = docsurl.URLByAppendingPathComponent("moi.txt")
moidata.writeToURL(moifile, atomically: true)
```

再读取出来：

```swift
let persondata = NSData(contentsOfURL: moifile)!
let person = NSKeyedUnarchiver.unarchiveObjectWithData(persondata) as Person
println(person) // Matt Neuburg
```

If the `NSData` object is itself the entire content of the file, as here, then
instead of using `archivedDataWithRootObject:` and `unarchiveObjectWithData:`, you can skip the intermediate `NSData` object and use `archiveRootObject:toFile:` and `unarchiveObjectWithFile:`.

Saving a single Person as an archive may seem like overkill; why didn’t we just make a text file consisting of the first and last names? But imagine that a Person has a lot more properties, or that we have an array of hundreds of Persons, or an array of hundreds of dictionaries where one value in each dictionary is a Person; now the power of an archivable Person is evident.

尽管 `Person` 实现了 `NSCoding` 协议，但包含 `Person` 的 `NSArray` 仍不能直接通过实例方法 `writeToFile...` 或 `writeToURL...` 将自身写到文件，因为 `Person` 仍不是属性列表类型。But the array can be archived and written to disk with `NSKeyedArchiver`.

#### 23.1.5 文件协调器（Coordinators）

iOS 8 开始，捏可以将自己应用的文件暴露给其他应用读写。此时需要保证两个应用操作同一个文件不会冲突。The solution is to use an `NSFileCoordinator`.

To read or write through an `NSFileCoordinator`, instantiate `NSFileCoordinator` along with an `NSFileAccessIntent` appropriate for reading or writing, to which you have handed the URL of your target file. Then call a `coordinate...` method. I’ll demonstrate the use of `coordinateAccessWithIntents(intents:queue:byAccessor:)`. The `accessor:` is a closure where you do your actual reading or writing in the normal way, except that the URL for reading or writing now comes from the `NSFileAccessIntent` object. So, to write a `Person` out to a file under the auspices of an
`NSFileCoordinator`:

```swift
let fc = NSFileCoordinator()
let intent = NSFileAccessIntent.writingIntentWithURL(moifile, options: nil)
    fc.coordinateAccessWithIntents([intent], queue: NSOperationQueue.mainQueue()) {
    (err:NSError!) in
    let ok = moidata.writeToURL(intent.URL, atomically: true)
}
```

And to read a Person from a file using an `NSFileCoordinator:`

```swift
let fc = NSFileCoordinator()
let intent = NSFileAccessIntent.readingIntentWithURL(moifile, options: nil)
	fc.coordinateAccessWithIntents([intent], queue: NSOperationQueue.mainQueue()) {
    	(err:NSError!) in
		let persondata = NSData(contentsOfURL: intent.URL)!
		let person = NSKeyedUnarchiver.unarchiveObjectWithData(persondata) as Person
}
```

### 23.2 User Defaults

User defaults (NSUserDefaults) 是用户偏好的持久化存储。They are little more, really, than a special case of an NSDictionary property list file. You talk to the NSUserDefaults `standardUserDefaults` object much as if it were a dictionary; it has keys and values. And the only legal values are property list values; thus, for example, to store a `Person` in user defaults, you’d have to archive it first to an `NSData` object. Unlike `NSDictionary`, `NSUserDefaults` provides convenience methods for converting between a simple data type such as a `Float` or a `Bool` and the object that is stored in the defaults (`setFloat:forKey:`, `floatForKey:`, and so forth). But the defaults themselves are still a dictionary.

Meanwhile, somewhere on disk, this dictionary is being saved for you automatically as a property list file. 你只需要从这个字典读写值，假设该字典会在适当的时间被读入或写出。Your chief concern is to make sure that you’ve written everything needful into user defaults before your app terminates; in the multitasking world (Appendix A), this will usually mean when the app delegate receives `applicationDidEnterBackground:` at the latest. If you’re worried that your app might crash, you can tell the `standardUserDefaults` object to synchronize as a way of forcing it to save right now, but this is rarely necessary.

To provide the value for a key before the user has had a chance to do so — the default default, as it were — use `registerDefaults:`. What you’re supplying here is a dictionary whose key–value pairs will each be written into the user defaults, but only if there is no such key already. For example:

```swift
NSUserDefaults.standardUserDefaults().registerDefaults([
    "cardMatrixRows" : 4,
    "cardMatrixColumns" : 3
])
```

The idea is that we call registerDefaults: extremely early as the app launches. Either the app has run at some time previously and the user has set these preferences, in which case this call has no effect and does no harm, or not, in which case we now have initial values for these preferences with which to get started.

Alternatively, you can provide a *settings bundle*, consisting mostly of one or more property list files describing an interface and the corresponding user default keys and their initial values; the **Settings** app is then responsible for translating your instructions into an actual interface, and for presenting it to the user.

Using a settings bundle has some obvious disadvantages: the user has to leave your app to access preferences, and you don’t get the kind of control over the interface that you have within your own app. Also, the user can set your preferences while your app is backgrounded or not running; you’ll need to register for `NSUserDefaultsDidChangeNotification` in order to hear about this.

In some situations, though, a settings bundle has some clear advantages. Keeping the preferences interface out of your app can make your app’s own interface cleaner and simpler. You don’t have to write any of the “glue” code that coordinates the preferences interface with the user default values. And it may be appropriate for the user to be able to set at least certain preferences for your app when your app isn’t running.

In iOS 7 and before, another objection to a settings bundle was that the user might not think to look in the Settings app for your preferences. New in iOS 8, however, this is less of an issue, because you can transport your user directly from your app to your app’s preferences in the Settings app:

```swift
let url = NSURL(string:UIApplicationOpenSettingsURLString)!
UIApplication.sharedApplication().openURL(url)
```

Writing a settings bundle is described in Apple’s Preferences and Settings Programming Guide.

It is common practice to misuse NSUserDefaults ever so slightly for various purposes. For example, every method in your app can access the `standardUserDefaults` object, so it often serves as a global “drop” where one instance can deposit a piece of information for another instance to pick up later, when those two instances might not have ready communication with one another or might not even exist simultaneously.

NSUserDefaults is also a lightweight alternative to the built-in view controller–based state saving and restoration mechanism discussed in Chapter 6.

Yet another use of NSUserDefaults, new in iOS 8, is as a way to communicate data between your app and an extension provided by your app. For example, let’s say you’ve written a today extension (Chapter 13) whose interface details depend upon some data belonging to your app. After configuring your extension and your app to constitute an app group, both the extension and the app can access the NSUserDefaults associated with the app group (call init(suiteName:) instead of standardUserDefaults). For more information, see the “Handling Common Scenarios” chapter of Apple’s App Extension Programming
Guide.





