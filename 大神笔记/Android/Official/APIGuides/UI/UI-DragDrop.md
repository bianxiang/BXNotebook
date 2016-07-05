[toc]

## （未）拖放

利用 Android 拖放框架，用户可以通过拖放手势，在当前布局的视图之间转移数据。框架包括一个“拖动事件”类，一个监听器及一些辅助方法。

尽管该框架注意用语数据移动，它也可以用作其他UI操作。但下面主要描述数据的移动。

### 概述

A drag and drop operation starts when the user makes some gesture that you recognize as a signal to start dragging data. In response, your application tells the system that the drag is starting. The system calls back to your application to get a representation of the data being dragged. As the user's finger moves this representation (a "drag shadow") over the current layout, the system sends drag events to the drag event listener objects and drag event callback methods associated with the View objects in the layout. Once the user releases the drag shadow, the system ends the drag operation.

You create a drag event listener object ("listeners") from a class that implements `View.OnDragListener`. You set the drag event listener object for a View with the View object's `setOnDragListener()` method. Each View object also has a `onDragEvent()` callback method.

When you start a drag, you include both the data you are moving and metadata describing this data as part of the call to the system. During the drag, the system sends drag events to the drag event listeners or callback methods of each View in the layout. The listeners or callback methods can use the metadata to decide if they want to accept the data when it is dropped. If the user drops the data over a View object, and that View object's listener or callback method has previously told the system that it wants to accept the drop, then the system sends the data to the listener or callback method in a drag event.

Your application tells the system to start a drag by calling the `startDrag()` method. This tells the system to start sending drag events. The method also sends the data that you are dragging.

You can call `startDrag()` for any attached View in the current layout. The system only uses the View object to get access to global settings in your layout.

Once your application calls `startDrag()`, the rest of the process uses events that the system sends to the View objects in your current layout.

#### 拖放的过程

在一个拖放的过程中有四个基本步骤（或状态）：

**Started**

In response to the user's gesture to begin a drag, your application calls `startDrag()` to tell the system to start a drag. The arguments `startDrag()` provide the data to be dragged, metadata for this data, and a callback for drawing the drag shadow.

The system first responds by calling back to your application to get a drag shadow. It then displays the drag shadow on the device.

Next, the system sends a drag event with action type `ACTION_DRAG_STARTED` to the drag event listeners for all the View objects in the current layout. To continue to receive drag events, including a possible drop event, a drag event listener must return `true`. This registers the listener with the system. Only registered listeners continue to receive drag events. At this point, listeners can also change the appearance of their View object to show that the listener can accept a drop event.

If the drag event listener returns `false`, then it will not receive drag events for the current operation until the system sends a drag event with action type `ACTION_DRAG_ENDED`. By sending false, the listener tells the system that it is not interested in the drag operation and does not want to accept the dragged data.

**Continuing**

The user continues the drag. As the drag shadow intersects the bounding box of a View object, the system sends one or more drag events to the View object's drag event listener (if it is registered to receive events). The listener may choose to alter its View object's appearance in response to the event. For example, if the event indicates that the drag shadow has entered the bounding box of the View (action type `ACTION_DRAG_ENTERED`), the listener can react by highlighting its View.

**Dropped**

The user releases the drag shadow within the bounding box of a View that can accept the data. The system sends the View object's listener a drag event with action type `ACTION_DROP`. The drag event contains the data that was passed to the system in the call to `startDrag()` that started the operation. The listener is expected to return boolean `true` to the system if code for accepting the drop succeeds.

Note that this step only occurs if the user drops the drag shadow within the bounding box of a View whose listener is registered to receive drag events. If the user releases the drag shadow in any other situation, no `ACTION_DROP` drag event is sent.

**Ended**

After the user releases the drag shadow, and after the system sends out (if necessary) a drag event with action type `ACTION_DROP`, the system sends out a drag event with action type `ACTION_DRAG_ENDED` to indicate that the drag operation is over. This is done regardless of where the user released the drag shadow. The event is sent to every listener that is registered to receive drag events, even if the listener received the `ACTION_DROP` event.


