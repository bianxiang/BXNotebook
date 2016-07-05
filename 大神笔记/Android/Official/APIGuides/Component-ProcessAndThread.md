# 进程与线程

[toc]

[来源](http://developer.android.com/guide/components/processes-and-threads.html)

When an application component starts and the application does not have any other components running, the Android system starts a new Linux process for the application with a single thread of execution. 默认一个应用的所有组件运行在同一个进程和线程中（称为主线程）。如果应用组件启动时，已有应用的一个进程存在（因为应用的其他组件存在），则组件在这个进程的同一个线程执行。但你可以安排*应用内的组件运行在自己的进程*，也可以在进程中创建附加的线程。

## 进程

默认，一个应用的所有组件运行在同一进程，且多数应用不要改变这一点。但是可以manifest改变这一点。

主要组件`<activity>`, `<service>`, `<receiver>`, `<provider>`都支持`android:process`特性，指定组件运行的进程。可以让一些组件运行在某个进程，另一些运行在另一个进程。还可以用`android:process`让另一个应用的组件运行在相同进程，但要求the applications share the same Linux user ID and are signed with the same certificates。

The `<application>` element also supports an android:process attribute, to set a default value that applies to all components.

当内存下降时，Android可能关闭一个进程。导致，运行在被杀死的进程中的应用组件被销毁。A process is started again for those components when there's again work for them to do.

杀死进程的决定会考虑它们对用户的重要性。进程的重要性受进程中组件的重要性影响。For example, it more readily shuts down a process hosting activities that are no longer visible on screen, compared to a process hosting visible activities. 

### 进程生命周期

Android会尽可能的维持进程的运行。But eventually needs to remove old processes to reclaim memory for new or more important processes. 

进程重要性有五级，从高到低依次是：

1. 前台进程  
当满足以下条件时，进程被认为在前台。
	* 之上的Activity正在与用户交互（`onResume()`方法已执行）。
	* 之上的Service绑定一个与用户交互的Activity。
	* 之上的Service在前台：调用过`startForeground()`。
	* 之上的Service正在执行以下生命周期回调：onCreate(), onStart(), onDestroy()。
	* 之上的BroadcastReceiver长在执行`onReceive()`方法。  

	杀死前台进程是万不得已的。Generally, at that point, the device has reached a memory paging state, so killing some foreground processes is required to keep the user interface responsive.
2. 可见进程  
进程随没有前台组件，但仍能影响用户所见的屏幕。当满足以下条件时，进程被认为是可见的：
	* 之上的Activity不是前台的，但仍对用户可见（`onPause()`已被调用过。例如前台Activity是个对话框。
	* 之上的服务绑定到一个可见（或前台）Activity。
3. 服务进程  
启程正在运行一个被`startService()`启动的服务。且非前两种情况。它们做的事情，用户也是关心的（如播放音乐）。
4. 后台进程  
之上的活动已不可见（`onStop()`方法已被调用）。一般会有很多后台进程存在着，它们保存在LRU (least recently used)中。如果活动正确实现了其生命周期回调，保存了当前状态，杀死进程对用户没有什么影响。当用户导航会Activity，Activity会恢复所有可视状态。保存状态参见[link](http://developer.android.com/guide/components/activities.html#SavingActivityState)。
5. 空进程
A process that doesn't hold any active application components. 让这些进程存货的唯一原因是缓存，下次当有组件需要在它上面运行时可以减少启动时间。

如果进程之前有依赖。则被依赖的进程的优先级不能低于依赖它的进程的优先级。

由于运行服务的进程比持有后台活动的进程优先级高，因此活动启动一个新服务比启动线程有时更好。特别是操作持续比活动久。如退出活动后，仍继续上传图片。*This is the same reason that **broadcast receivers** should employ services* rather than simply put time-consuming operations in a thread.

## 线程

When an application is launched, the system creates a thread of execution for the application, called "main." 该线程负责分发事件到相应的UI组件，包括drawing事件。It is also the thread in which your application interacts with components from the Android UI toolkit （`android.widget`和`android.view`包）。因此主线程也被称为UI线程。

The system does not create a separate thread for each instance of a component. All components that run in the same process are instantiated in the UI thread, and system calls to each component are dispatched from that thread. 因此响应系统调用的方法（如`onKeyDown()`或生命周期回调）总是运行在UI线程。

For instance, when the user touches a button on the screen, your app's UI thread dispatches the touch event to the widget, which in turn sets its pressed state and posts an **invalidate** request to the event queue. The UI thread dequeues the request and notifies the widget that it should redraw itself.

Specifically, if everything is happening in the UI thread, performing long operations such as network access or database queries will block the whole UI. When the thread is blocked, no events can be dispatched, including drawing events. From the user's perspective, the application appears to hang. Even worse, if the UI thread is blocked for more than a few seconds (about 5 seconds currently) the user is presented with the infamous "[application not responding](http://developer.android.com/guide/practices/responsiveness.html)" (ANR) dialog. The user might then decide to quit your application and uninstall it if they are unhappy.

Andoid UI toolkit不是线程安全的。	So, you must not manipulate your UI from a worker thread—you must do all manipulation to your user interface from the UI thread. 

### 后台线程

Android提供了几种从其他线程访问UI线程的方法。

* Activity.runOnUiThread(Runnable)
* View.post(Runnable)
* View.postDelayed(Runnable, long)

例子：

	public void onClick(View v) {
	    new Thread(new Runnable() {
	        public void run() {
	            final Bitmap bitmap = loadImageFromNetwork("http://example.com/image.png");
	            mImageView.post(new Runnable() {
	                public void run() {
	                    mImageView.setImageBitmap(bitmap);
	                }
	            });
	        }
	    }).start();
	}

### 使用AsyncTask

AsyncTask allows you to perform asynchronous work on your user interface. It performs the blocking operations in a worker thread and then publishes the results on the UI thread, without requiring you to handle threads and/or handlers yourself.

To use it, you must subclass AsyncTask and implement the doInBackground() callback method, which runs in a pool of background threads. To update your UI, you should implement onPostExecute(), which delivers the result from doInBackground() and runs in the UI thread, so you can safely update your UI. You can then run the task by calling execute() from the UI thread.

例子：

	public void onClick(View v) {
	    new DownloadImageTask().execute("http://example.com/image.png");
	}
	
	private class DownloadImageTask extends AsyncTask<String, Void, Bitmap> {
	    /** The system calls this to perform work in a worker thread and
	      * delivers it the parameters given to AsyncTask.execute() */
	    protected Bitmap doInBackground(String... urls) {
	        return loadImageFromNetwork(urls[0]);
	    }
	    
	    /** The system calls this to perform work in the UI thread and delivers
	      * the result from doInBackground() */
	    protected void onPostExecute(Bitmap result) {
	        mImageView.setImageBitmap(result);
	    }
	}


> 注意：由于运行时改变（如屏幕方向改变引起）导致的意外重启，这可能导致后台线程被摧毁。To see how you can persist your task during one of these restarts and how to properly cancel the task when the activity is destroyed, see the source code for the [Shelves](http://code.google.com/p/shelves/) sample application.

### 线程安全方法

This is primarily true for methods that can be called remotely—such as methods in a bound service. When a call on a method implemented in an IBinder originates in the same process in which the IBinder is running, the method is executed in the caller's thread. However, when the call originates in another process, the method is executed in a thread chosen from a pool of threads that the system maintains in the same process as the IBinder (it's not executed in the UI thread of the process). For example, whereas a service's onBind() method would be called from the UI thread of the service's process, methods implemented in the object that onBind() returns (for example, a subclass that implements RPC methods) would be called from threads in the pool. Because a service can have more than one client, more than one pool thread can engage the same IBinder method at the same time. IBinder methods must, therefore, be implemented to be thread-safe.

Similarly, a content provider can receive data requests that originate in other processes. Although the ContentResolver and ContentProvider classes hide the details of how the interprocess communication is managed, ContentProvider methods that respond to those requests—the methods query(), insert(), delete(), update(), and getType()—are called from a pool of threads in the content provider's process, not the UI thread for the process. Because these methods might be called from any number of threads at the same time, they too must be implemented to be thread-safe.

## 跨进程通讯

Android利用RPC实现进程间通讯（interprocess communication (IPC)）：方法被活动等组件调用，但在远端执行（另一个进程），最后结果返回给调用者。This entails decomposing a method call and its data to a level the operating system can understand, transmitting it from the local process and address space to the remote process and address space, then reassembling and reenacting the call there. Return values are then transmitted in the opposite direction. Android provides all the code to perform these IPC transactions, so you can focus on defining and implementing the RPC programming interface.

要进程进程间通讯，你的应用必须绑定到服务（利用`bindService()`）。