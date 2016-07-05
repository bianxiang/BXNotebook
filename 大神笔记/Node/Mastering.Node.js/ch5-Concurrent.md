[toc]


## 5 处理多个客户端同时连接

构建高并发系统的两个选择的争论：线程还是事件。

> Threading allows programmers to write straight-line code and rely on the operating system to overlap computation and I/O by transparently switching across threads. The alternative, events, allows programmers to manage concurrency explicitly by structuring code as a single-threaded handler that reacts to events (such as non-blocking I/O completions, application-specific messages, or timer events).
	— "A Design Framework for Highly Concurrent Systems" (Welsh, Gribble, Brewer & Culler, 2000), p. 2. http://www.eecs.harvard.edu/~mdw/papers/events.pdf

上面一段话的两个关键点：
- Developers prefer to write structuredcode (straight-line; single-threaded) that hides the complexity of multiple simultaneous operations where possible
- 高效 I/O 是高并发的关键

Node尝试组合线程和事件的优势。使用单个线程服务所有客户端（一个事件循环，包装一个Javascript运行时）。将阻塞操作 (I/O) 代理给一个优化了的线程池。通过事件通知告知主线程状态。

The claim implicit in Node's design is this: it is easier to reason about highly concurrent software when program flow is organized along a single thread, and that decreasing I/O latency increases the number of simultaneous clients that can be supported even in a single-threaded execution model. 第二点将在后面证明。