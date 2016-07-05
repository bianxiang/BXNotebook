## Canvas and Drawables

http://developer.android.com/guide/topics/graphics/2d-graphics.html

利用Android的 2D 绘图 API，可以在 Canvas 上绘自己的图形，或修改已存在的View，定制它们的外观。绘制 2D 图形有两种方式：
- 将图形或动画绘制到布局中的`View`对象。In this manner, the drawing of your graphics is handled by the system's normal View hierarchy drawing process — 你只要定义View中的图形。
- 直接在 Canvas 上绘图。此时，需要你去调相应类的`onDraw()`（`View.onDraw`）方法。(passing it your Canvas), 或 Canvas 的某个`draw...()`方法。这种方式还可以让你自己控制各种动画。

不需要图形动态改变，或不是性能敏感的游戏，应选择第一种，绘制到View上。可以在View上显示静态图形或预定义动画。

如果应用需要定期重绘自己，则适合用第二个选择，绘制到Canvas。例如游戏。有两种重绘方式：
- 在Activity所在的线程，对创建的自定义View，调用`invalidate()`，然后处理`onDraw()`回调。
- 在另一个线程，即管理`SurfaceView`的线程，以这个线程最大可能的速度向 Canvas 绘制，不需要调用`invalidate()`。

### Draw with a Canvas

Canvas 是实际的绘图表面的一个接口——it **holds** all of your "draw" calls. 利用Canvas，实际是在一个底层的`Bitmap`上绘制，which is placed into the window.

`onDraw()`方法会向你提供一个 Canvas 实例。
若使用`SurfaceView`，可以通过`SurfaceHolder.lockCanvas()`方法获取 Canvas。

如果你要定义一个新的Canvas，必须先定义一个Bitmap（绘制发生在其上）。**Canvas总是需要一个Bitmap**。创建一个新的Canvas：
```java
Bitmap b = Bitmap.createBitmap(100, 100, Bitmap.Config.ARGB_8888);
Canvas c = new Canvas(b);
```

利用新的Canvas完成绘制后，需要将底层的`Bitmap`传给其他Canvas（通过`Canvas.drawBitmap(Bitmap,...)`）。因为，推荐最终要使用`View.onDraw()`或`SurfaceHolder.lockCanvas()`提供的Canvas。｛｛图形才能出现的屏幕上｝｝

在Canvas上绘制可以用上两类方法。Canvas类有一组绘制方法，如`drawBitmap(...)`、`drawRect(...)`、`drawText(...)`。其他可能用到的类也有一些`draw()`方法。如，若想向Canvas添加一个`Drawable`，可以调用`Drawable.draw()`方法，传入`Canvas`对象。

#### On a View

如果你的应用对帧率没有高要求，应该考虑自定义一个View，在`View.onDraw()`方法中利用Canvas绘制。使用此方法最便利的地方是，Android系统会提供给你一个预定义的 Canvas，用于绘制。

首先，继承`View`类（或子类），定义`onDraw()`方法。该方法由Android框架调用。只会在必要时调用。当你的应用打算重绘时，需要调用`invalidate()`，令View失效。然后Android会负责调用`onDraw()`。

> 在非UI线程中请求invalidate，调用`postInvalidate()`。

参见[Building Custom Components](http://developer.android.com/guide/topics/ui/custom-components.html)。

For a sample application, see the Snake game, in the SDK samples folder: `/samples/Snake/`.

#### On a SurfaceView

`SurfaceView`是`View`的一个子类，提供一个专用的绘图表面。目标是使用单独线程访问此绘图表面。

首先创建一个`SurfaceView`的子类。子类还需要实现`SurfaceHolder.Callback`。该接口会通知你底层Surface的变化，如被创建、改变、销毁。利用这些事件，决定何时开始绘制，是否需要调整，以及何时停止绘制，杀死一些任务。独立的`Thread`类一般定义在`SurfaceView`类中，在该线程中进行所有的绘制。

不要直接与 Surface 对象交互，通过 `SurfaceHolder` 与之交互。当 `SurfaceView` 初始化后，调用 `getHolder()` 获取 `SurfaceHolder`。然后通过`addCallback(this)`，表示本类想要接收`SurfaceHolder`的回调（`SurfaceHolder.Callback`）。最后在`SurfaceView`类实现`SurfaceHolder.Callback`所有的回调方法。

In order to draw to the Surface Canvas from within your second thread, you must pass the thread your `SurfaceHandler`，并通过`lockCanvas()`获取`Canvas`。You can now take the Canvas given to you by the `SurfaceHolder` and do your necessary drawing upon it. Once you're done drawing with the Canvas, call `unlockCanvasAndPost()`, passing it your Canvas object. The Surface will now draw the Canvas as you left it. Perform this sequence of locking and unlocking the canvas each time you want to redraw.

> Note: On each pass you retrieve the Canvas from the SurfaceHolder, the previous state of the Canvas will be retained. In order to properly animate your graphics, you must re-paint the entire surface. For example, you can clear the previous state of the Canvas by filling in a color with `drawColor()` or setting a background image with `drawBitmap()`. Otherwise, you will see traces of the drawings you previously performed.

For a sample application, see the Lunar Lander game, in the SDK samples folder: `/samples/LunarLander/`.

### Drawables



