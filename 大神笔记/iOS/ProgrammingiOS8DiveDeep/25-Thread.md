[toc]

# 25 线程

关于线程的更多知识，特别是让代码线程安全，超出本章范围。参考官方 Concurrency Programming Guide and Threading Programming Guide。

## 25.1 主线程

主线程有一些特殊的特点：

- 主线程自动获得一个运行循环（run loop）。运行循环是一个事件的接收器。没有运行循环，线程无法接收事件。Cocoa 事件一般落在主线程的运行循环上；因此被这些事件调用的你的代码，运行在主线程。
- 主线程是界面的线程。与界面交互的代码必须在主线程执行。

## （未）25.3 阻塞主线程

绘制 Mandelbrot set。计算量较大，会引起显著的延迟。

## （未）25.4 NSOperation
`NSOperation` 封装的不是任务，而是线程。`NSOperation` 描述的操作可以在后台线程中执行。You describe the operation and add the NSOperation to an `NSOperationQueue` to set it going. You arrange to be notified when theoperation ends, typically by the NSOperation posting a notification. Youcan query both the queue and its operations from outside with regard totheir state.

We’ll rewrite `MyMandelbrotView` to use `NSOperation`. We need a property, an `NSOperationQueue`; we’ll call it `queue`, and we’ll create and configure it in its initializer:

```swiftlet queue : NSOperationQueue = {	let q = NSOperationQueue()	// ... further configurations can go here ...	return q}()
```

We also have a new class, `MyMandelbrotOperation`, an NSOperationsubclass. (It is possible to take advantage of a built-in NSOperationsubclass such as `NSBlockOperation`, but I’m deliberately illustrating themore general case by subclassing `NSOperation` itself.) Ourimplementation of `drawThatPuppy` thus creates an instance of`MyMandelbrotOperation`, configures it, registers for its notification, andadds it to the queue:

```swiftfunc drawThatPuppy () {	let center = CGPointMake(self.bounds.midX, self.bounds.midY)
	let op = MyMandelbrotOperation(size: self.bounds.size, center: center, zoom: 1)
	NSNotificationCenter.defaultCenter().addObserver(self,
		selector: "operationFinished:",
		name: "MyMandelbrotOperationFinished", object: op)	self.queue.addOperation(op)}
```