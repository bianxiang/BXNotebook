[toc]

# 24 基本网络

参见官方文档 URL Loading System Programming Guide。参见官方示例代码：SimpleURLConnections, AdvancedURLConnections, SimpleNetworkStreams, SimpleFTPSample, MVCNetworking。

之前章节已经涉及到了一些网络相关的接口和框架。

## 24.1 HTTP 请求

HTTP 请求通过 `NSURLSession` 对象发出。多数情况下只需要一个 `NSURLSession` 对象：对于非常简单的使用，可以用单例对象 `sharedSession`。通过 `init(configuration:)` 或 `init(configuration:delegate:delegateQueue:)` 产生 `NSURLSession`。`configuration:` 是一个 `NSURLSessionConfiguration` 对象。

从 `NSURLSession` 获得一个 `NSURLSessionTask` 对象进行网络请求，包括上传或下载。`NSURLSessionTask` 是一个抽象类，包含以下属性：

- `taskDescription` 和 `taskIdentifier`：前者你来定；后者是 `NSURLSession` 中的唯一标示
- `originalRequest` 和 `currentRequest`：因为重定向，请求可能改变
- An initial `response` from the server
- Various `countOfBytes...` properties allowing you to track progress
- `state`，值取 `NSURLSessionTaskState`，包括：.Running、 .Suspended、 .Canceling、 .Completed
- An `error` if the task failed

你可以让任务 `resume`、 `suspend` 或 `cancel`。任务开始处于挂起状态，直到调用 `resume` 才启动。

iOS 8 支持设置 `NSURLSessionTask` 的属性 `priority`，取值 0 到 1；此外还定义有三个常量：

- NSURLSessionTaskPriorityLow (0.25)
- NSURLSessionTaskPriorityDefault (0.5)
- NSURLSessionTaskPriorityHigh (0.75)

实际有三种任务：

- `NSURLSessionDataTask`：`NSURLSessionTask` 的子类。随着数据的到达，数据增量提供给应用。
- `NSURLSessionDownloadTask`：`NSURLSessionTask` 的子类。数据下载到文件，保存文件的 URL 在下载完成后传给你。The file is outside your sandbox and will be destroyed, so preserving it (or its contents) is up to you.
- `NSURLSessionUploadTask`：`NSURLSessionDataTask` 的子类。上传一个文件，可以放手不管，或监听进度。

从 `NSURLSession` 获取的任务，你可以保留对它的引用；也可以不保留，`NSURLSession` 会维护它。`NSURLSession` 维护正在进行的任务的列表；call `getTasksWithCompletionHandler:`. The completion handler is handed three arrays, one for each type of task. 任务被取消或完成后后 `NSURLSession` 会将其释放。

向 `NSURLSession` 请求一个新的 `NSURLSessionTask` 有三种方法：

- 调用便利方法：所有便利方法都有一个 `completionHandler:` 参数。任务完成后调用这个处理器。
- 调用一个基于代理的方法：创建 `NSURLSession` 时传入一个代理，在任务的不同阶段会回调这个代理。

### 24.1.1 简单HTTP请求

使用共享的 `NSURLSession`。使用一个下载任务。调用 `resume` 开始任务。下载在后台线程执行，因此界面不会被阻塞。完成后，会调用“完成处理器”，回调发生在后台线程！本例我们获取一个图片，想要显示在界面上，必须先回到主线程（第25章）。

```swift
let s = "http://www.someserver.com/somefolder/someimage.jpg"
let url = NSURL(string:s)!
let session = NSURLSession.sharedSession()
let task = session.downloadTaskWithURL(url) {
    (loc:NSURL!, response:NSURLResponse!, error:NSError!) in
    if error != nil {
        println(error)
        return
    }
    let status = (response as NSHTTPURLResponse).statusCode
    if status != 200 {
        println("response status: \(status)")
        return
    }
    let d = NSData(contentsOfURL:loc)!
    let im = UIImage(data:d)
        dispatch_async(dispatch_get_main_queue()) {
            self.iv.image = im
        }
    }
task.resume()
```

### 24.1.2 正儿八经的网络请求

通过 `NSURLSessionConfiguration` 对象配置 `NSURLSession`，给它一个代理。Instead of a mere URL, we’ll start with an `NSURLRequest`.

通过 `NSURLSessionConfiguration` 可以配置一系列选项：

- 是否允许使用蜂窝网络，还是需要 Wi-Fi
- 到服务器的最大同步连接数
- 超时：
  - `timeoutIntervalForRequest`：The maximum time you’re willing to wait between pieces of data.
  - `timeoutIntervalForResource`：整个下载完成最多时间
- Cookie, caching, and credential policies

`init(configuration:delegate:delegateQueue:)` 还需要一个代理，以及一个队列（粗略的说，一个线程，见第25章），代理在该队列（线程）上调用。For each type of task, there’s a delegate protocol, which is itself often a composite of multiple protocols. 

例如，对于数据任务，我们需要一个数据代理：`NSURLSessionDataDelegate`。代理（及父代理）中重要的方法有：

- `URLSession:dataTask:didReceiveData:`：收到部分数据，以 `NSData` 对象的形式。下载过程中可能多次调用该方法，直到数据都到达。
- `URLSession:task:didCompleteWithError:`：有错误，或没有错误的、下载完成都调用该方法。

下载任务，需要代理 `NSURLSessionDownloadDelegate`，重要方法有：

- `URLSession:downloadTask:didResumeAtOffset:expectedTotalBytes:`：该方法仅对一个可恢复的下载有意义，且任务已被暂停又恢复了。
- `URLSession:downloadTask:didWriteData:totalBytesWritten:totalBytesExpectedToWrite:`：周期性的调用，告诉我们下载进度。
- `URLSession:downloadTask:didFinishDownloadingToURL:`：在最后调用；我们必须立即抓取下载的文件，因为它将被销毁。必须实现的代理方法只有这个。

最好也实现 `URLSession:task:didCompleteWithError:`；若有通信错误，`error:` 参数将给出。

Here, then, is my recasting of the same image file download as in the previous example. I’m going to keep a reference both to the `NSURLSession` (`self.session`) and to the current download task (`self.task`). 指定 `self` 做代理。让代理的回调在主线程中执行。

```swift
var task : NSURLSessionTask!
lazy var session : NSURLSession = {
	let config = NSURLSessionConfiguration.ephemeralSessionConfiguration()
	config.allowsCellularAccess = false
	let session = NSURLSession(configuration: config, delegate: self,
		delegateQueue: NSOperationQueue.mainQueue())
    return session
}()
```

准备图片视图，启动下载。这里使用 `NSURLRequest` 替代 `NSURL` 发请求。有时使用 `NSURLRequest` 更方便或是必要的（上传）。

```swift
let s = "http://www.someserver.com/somefolder/someimage.jpg"
let url = NSURL(string:s)!
let req = NSMutableURLRequest(URL:url)
let task = self.session.downloadTaskWithRequest(req)
self.task = task
self.iv.image = nil
task.resume()
```

> 不要设置 `NSURLRequest` 的属性，如果这些属性可以通过 `NSURLSession` 设置。这些属性是 `NSURLSession` 出现前发明的。如 `timeoutInterval`。现在应使用 `NSURLSession` 的配置。

以下是相关大力方法：

```swift
func URLSession(session: NSURLSession, downloadTask: NSURLSessionDownloadTask,
	didWriteData bytesWritten: Int64, totalBytesWritten writ: Int64,
	totalBytesExpectedToWrite exp: Int64) {
	println("downloaded \(100*writ/exp)%")
}
func URLSession(session: NSURLSession, task: NSURLSessionTask,
	didCompleteWithError error: NSError?) {
	println("completed: error: \(error)")
}
func URLSession(session: NSURLSession, downloadTask: NSURLSessionDownloadTask,
	didFinishDownloadingToURL location: NSURL) {
	self.task = nil
    let response = downloadTask.response as NSHTTPURLResponse
	let stat = response.statusCode
	println("status \(stat)")
    if stat != 200 {
	    return
    }
    let d = NSData(contentsOfURL:location)!
    let im = UIImage(data:d)
    dispatch_async(dispatch_get_main_queue()) {
        self.iv.image = im
    }
}
```

下面展示数据任务。数据任务的主要区别是需要你自己拼装数据。这里通过 `NSMutableData` 组装数据：

```swift
var data = NSMutableData()
```

创建任务时，设置 `self.data` 为0：

```swift
let s = "http://www.someserver.com/somefolder/someimage.jpg"
let url = NSURL(string:s)!
let req = NSMutableURLRequest(URL:url)
let task = self.session.dataTaskWithRequest(req) // *
self.task = task
self.iv.image = nil
self.data.length = 0 // *
task.resume()
```

As the chunks of data arrive, I keep appending them to `self.data`. When all the data has arrived, it is ready for use:

```swift
func URLSession(session: NSURLSession, dataTask: NSURLSessionDataTask,
	didReceiveData data: NSData) {
	self.data.appendData(data)
}
func URLSession(session: NSURLSession, task: NSURLSessionTask,
	didCompleteWithError error: NSError?) {
	self.task = nil
	if error == nil {
		self.iv.image = UIImage(data:self.data)
	}
}
```

Some delegate methods provide a `completionHandler:` parameter. These are delegate methods that require a response from you. For example, in the case of a data task, `URLSession:dataTask:didReceiveResponse:completionHandler:` arrives when we first connect to the server. Here, we could check the status code of the initial `NSHTTPURLResponse`. 我们需要返回一个响应表达是否要继续（或是否将一个数据任务转换为一个下载任务）。But because of the multithreaded nature of networking, we do this, not by returning a value directly, but by calling the `completionHandler:` that we’re handed as a parameter and passing our response into it. Several of the delegate methods are constructed in this way.

使用 `NSURLSession` 和代理时要考虑内存问题。`NSURLSession` 会 retain 代理。在上面的例子中，我们导致了一个 **retain cycle**：We have an `NSURLSession` instance variable `self.session`, but that `NSURLSession` is retaining `self` as its delegate.

As with an `NSTimer`, the solution is to invalidate the `NSURLSession` at some appropriate moment. 有两种方法：

- `finishTasksAndInvalidate`：允许已存在的任务继续。然后 `NSURLSession` 释放代理；其自身也不可再用了。
- `invalidateAndCancel`：立即中断所有任务。`NSURLSession` 释放代理；其自身也不可再用了。

若 `self` 是一个视图控制器，则在 `viewWillDisappear:` 方法中我们可以令 `NSURLSession` 失效。(We cannot use `deinit`, because `deinit` won’t be called until after we have invalidated the `NSURLSession`; that’s what it means to have a retain cycle.) So, for example:

```swift
override func viewWillDisappear(animated: Bool) {
    super.viewWillDisappear(animated)
    self.session.finishTasksAndInvalidate()
}
```

### 24.1.3 封装 Session, Task, and Delegate

The methods for attaching a property to an `NSMutableURLRequest` and retrieving it again later are:

- `setProperty:forKey:inRequest:`
- `propertyForKey:inRequest:`

第一个方法的 `property:` 属性以及第二个方法的返回值都是 `AnyObject`。但 Swift 闭包不是 `AnyObject`。解决办法是（Appendix B）定义一个泛型的包裹类：

```swift
class Wrapper<T> {
	let p:T
	init(_ p:T){self.p = p}
}
```

### 24.1.4 多任务

假设有一个 `UITableView` 显示大量图片。图片应是延迟加载的：当用户滚动到某一单元格再加载。但如果用户快速滚动，则来不及显示的单元格最好停止加载。我们可以实现 `tableView:didEndDisplayingCell:forRowAtIndexPath:`，在其中取消掉 `NSURLSessionTask`。

先定义模型类，表示每个单元格的数据：

```swift
class Model {	var text : String!	var im : UIImage!	var picurl : String!	var task : NSURLSessionTask!}
```

取消：

```swift
override func tableView(tableView: UITableView,	didEndDisplayingCell cell: UITableViewCell,	forRowAtIndexPath indexPath: NSIndexPath) {	let m = self.model[indexPath.row]	if let task = m.task {		if task.state == .Running {			task.cancel()			m.task = nil		}	}}
```

正向代码。Moreover, each task is retaining a closure that refers to `self`; that’s a potential retain cycle, which we break through a `weak` reference to `self`:

```swift
override func tableView(tableView: UITableView,
	cellForRowAtIndexPath indexPath: NSIndexPath) -> UITableViewCell {	let cell = tableView.dequeueReusableCellWithIdentifier(		"Cell", forIndexPath:indexPath) as UITableViewCell	let m = self.model[indexPath.row]	cell.textLabel.text = m.text	// have we got a picture?	if let im = m.im {		cell.imageView.image = im	} else {		if m.task == nil { // no task? start one!			cell.imageView.image = nil			m.task = self.downloader.download(m.picurl) { // *				[weak self] url in // *				m.task == nil // *				if url == nil {					return				}				let data = NSData(contentsOfURL: url)!				let im = UIImage(data:data)				m.im = im				dispatch_async(dispatch_get_main_queue()) {					self!.tableView.reloadRowsAtIndexPaths(						[indexPath], withRowAnimation: .None)				}			}		}	}	return cell}
```

### （未）24.1.5 后台下载

## （未）24.2 Background App Refresh

## （未）24.3 In-App Purchases

## （未）24.4 Bonjour




