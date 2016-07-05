[toc]

# 4.Android for Game Developers

Android提供两大API用于绘图。一个用于简单的2D图形编程，一个用于硬件加速的3D图形。本章和下一章关注2D图形，使用Canvas API。Canvas API是对*Skia*库的一个包装，适于中等复杂度的2D图形。

## （未） 4.1 Defining an Android Application: The Manifest File

## 4.2 Android API基础

### 4.2.1 创建测试工程

工程名`ch04–android-basics`。单个活动`AndroidBasicsStarter`。Minimum SDK设为3 (Android 1.5)，目标SDK设为9 (Android 2.3)。

	package com.badlogic.androidgames; 
	import android.app.ListActivity; 
	import android.content.Intent; 
	import android.os.Bundle; 
	import android.view.View; 
	import android.widget.ArrayAdapter; 
	import android.widget.ListView; 
	public class AndroidBasicsStarter extends ListActivity { 
		String tests[] = { "LifeCycleTest", "SingleTouchTest",
			"MultiTouchTest", 
			"KeyTest", "AccelerometerTest", "AssetsTest", 
			"ExternalStorageTest", "SoundPoolTest", "MediaPlayerTest",
			"FullScreenTest", "RenderViewTest", "ShapeTest", "BitmapTest", 
			"FontTest", "SurfaceViewTest" }; 
		public void onCreate(Bundle savedInstanceState) { 
			super.onCreate(savedInstanceState); 
			setListAdapter(newArrayAdapter<String>(this, 
				android.R.layout.simple_list_item_1, tests)); 
		} 
		@Override 
		protected void onListItemClick(ListView list, View view, int position,
			long id) { 
			super.onListItemClick(list, view, position, id); 
			String testName = tests[position]; 
			try{ 
				Class clazz = Class.forName("com.badlogic.androidgames." + testName); 
				Intent intent = newIntent(this, clazz); 
				startActivity(intent); 
			} catch(ClassNotFoundException e) { 
				e.printStackTrace(); 
			} 
		} 
	} 

### 4.2.3 输入设备

讨论触屏、键盘、加速度计。

多点触控是Android 2.0 (SDK version 5)引入的。The multitouch event reporting was tagged onto the single-touch API, with some mixed results in usability. 

#### 单点触摸事件

触屏事件传给`OnTouchListener`接口。该接口只有一个方法：

	public abstract boolean onTouch (View v, MotionEvent event)

任何View都可以注册该事件，通过`View.setOnTouchListener()`。

`OnTouchListener`在`MotionEvent`分派到View自己之前被调用。如果我们在`onTouch`中返回true，是告诉View不要再处理。如果返回false，View自己将处理事件。

* `MotionEvent.getX()`和`MotionEvent.getY()`返回相对于View的坐标。原点在View的左上角。x轴向右，y轴向下。返回值是浮点型。
* `MotionEvent.getAction()`：返回触摸事件的类型。取值`MotionEvent.ACTION_DOWN`，`MotionEvent.ACTION_MOVE`，`MotionEvent.ACTION_CANCEL`和 `MotionEvent.ACTION_UP`。  
`MotionEvent.ACTION_MOVE`事件总是产生，因为屏幕很敏感。`MotionEvent.ACTION_CANCEL`表示用户取消动作，但实际中，几乎从不会被触发。

触摸事件的一些问题：

* Touch event flood: 驱动会尽可能多的报告触摸事件。在一些设备上可能每秒有几百次。We can fix this issue by putting a `Thread.sleep(16)` call into our onTouch()method, which will put the UI thread on which those events are dispatched to sleep for 16 milliseconds. With this we’ll get 60 events per second at most, which is more than enough to have a responsive game. This is only a problem on devices with Android version 1.5. 
* 触摸事件可能占据太多CPU。Even if we sleep in our onTouch() method, 系统仍要在内核处理驱动的报告。在老的设备上如Hero、G1，可能占到50%的CPU。As a consequence, our perfectly fine frame rate will drop considerably, sometimes to the point where the game becomes unplayable. 在第二代设备上，该问题不重要了，基本可以忽略。不过，在老设备上依然无法解决。

#### （未，可能过时）多点触摸事件

The multitouch API has been tagged onto the MotionEvent class, which originally only handled single touches. This makes for some major confusion when trying to decode multitouch events.

> NOTE 在SDK version 8 (Android 2.2)中有了新方法、新常量。

#### 按键事件

`OnKeyListener`接口只有一个方法
 
	public boolean onKey(View view, int keyCode, KeyEvent event) 

`keyCode`是定义在`KeyEvent`类中的常量。

`KeyEvent`类有两个方法：

* `KeyEvent.getAction()`：返回值`KeyEvent.ACTION_DOWN`、`KeyEvent.ACTION_UP`、`KeyEvent.ACTION_MULTIPLE`。For our purposes we can ignore the last key event type. The other two will be sent when a key is either pressed or released. 
* `KeyEvent.getUnicodeChar()`: This returns the Unicode character the key would produce in a text field. Say we hold down the Shift key and press the A key. This would be reported as an event with a key code of `KeyEvent.KEYCODE_A`, but with a Unicode character A. We can use this method if we want to do text input ourselves.

要接收键盘事件，View必须拥有焦点。

	View.setFocusableInTouchMode(true); 
	View.requestFocus(); 

#### （未）读取加速度计状态

### 4.2.4 文件处理

`assets/`和`res/`都可以放置资源。我们不用`res/`因为它限制太多。

`assets/`文件夹下的类通过`AssetManager`暴露。
 
	AssetManager assetManager = context.getAssets(); 

打开文件：

	InputStream inputStream = assetManager.open("dir/dir2/filename.txt"); 

记得关闭流。

### xxx 4.2.6 播放音效

### xxx 4.2.7 Streaming Music


### 4.2.8 基本图形编程

#### 使用Wake Locks

要使屏幕一直亮着，利用所谓的wake lock。首先要加<uses-permission>，名字`android.permission.WAKE_LOCK`。

	PowerManager powerManager = 
		(PowerManager)context.getSystemService(Context.POWER_SERVICE); 
	WakeLock wakeLock = powerManager.newWakeLock(PowerManager.FULL_WAKE_LOCK, "My Lock");

The `PowerManager.newWakeLock()` method takes two arguments: the type of the lock and a tag we can freely define.

Usually we instantiate the WakeLock instance on the `Activity.onCreate()` method, call `WakeLock.acquire()` in the `Activity.onResume()` method, and call the `WakeLock.release()` method in the `Activity.onPause()` method.

#### 全屏

隐藏标题条和通知条：

	requestWindowFeature(Window.FEATURE_NO_TITLE); // 隐藏标题
	getWindow().setFlags(WindowManager.LayoutParams.FLAG_FULLSCREEN,
		WindowManager.LayoutParams.FLAG_FULLSCREEN);

上述方法要在设置活动的View之前调用。

#### 在UI线程中做连续渲染

创建定制View，在屏幕上绘图。还想尽可能频繁的重绘。

继承`View`类，实现`View.onDraw()`。Android系统会在每次需要重绘时调用该方法。

```java
class RenderView extends View {
	public RenderView(Context context) {
		super(context);
	}
	protected void onDraw(Canvas canvas) {
		// all drawing goes here
		invalidate();
	}
}
```
`View.invalidate()`告诉 Android，一旦系统有时间，尽可能早的重绘`RenderView`。所有这些操作都发生在UI线程。

`Canvas`类是底层 Skia 库的封装，使用 CPU 绘制 2D 图形。`Canvas`可以绘制形状、bitmaps甚至文字。

绘制到哪里？不一定。`Canvas`可以绘到一个`Bitmap`实例上。而在上面的例子中，是绘到View所占的屏幕上。不过，这是个过度简化的说法。实际上，不是直接绘制到屏幕上，而是某种bitmap上，接下来系统会将其与活动中其他 View 的bitmap组合，构成最后的输出图像。此图形会被交给GPU。

只要系统允许，`onDraw()`会被以尽可能快的速度调用。它跟游戏的主循环非常类似。但不要把游戏逻辑放在该方法内——至少出于性能。

下面编写一个代码，看系统重绘的有多快。方法是随机颜色填充屏幕。

`Canvas`用某个单色填充目标的方法是：`Canvas.drawRGB(int r, int g, int b);`。
```java
protected voidonDraw(Canvas canvas) {
	canvas.drawRGB(rand.nextInt(256), rand.nextInt(256),
		rand.nextInt(256));
	invalidate();
}
```

#### 获取屏幕分辨率（和坐标系统）

返回`Canvas`的绘图目标的大小：

	int width = canvas.getWidth(); 
	int height = canvas.getHeight(); 

#### 绘制简单的形状

#### 点

	Canvas.drawPoint(float x, float y, Paint paint); 

Paint封装颜色和样式。对于点来说，只有颜色。

	Paint paint = new Paint(); 
	paint.setARGB(alpha, red, green, blue);

或者可以调用

	Paint.setColor(0xff00ff00); 

We pass a 32-bit integer to this method. It again encodes an ARGB8888 color; in this case it’s the color green with alpha set to full opacity. The Color class defines some static constants that encode some standard colors like Color.RED, Color.YELLOW, and so on.

##### 线

	Canvas.drawLine(float startX, float startY, float stopX, float stopY, Paint paint); 

指定线宽：
	
	Paint.setStrokeWidth(float widthInPixels); 

##### 矩形

	Canvas.drawRect(float topleftX, float topleftY, float bottomRightX, float bottomRightY,
		Paint paint);

设置样式：

	Paint.setStyle(Style style); 

Styleis an enumeration that has the values `Style.FILL`, `Style.STROKE`, and `Style.FILL_AND_STROKE`. If we specify Style.FILL, the rectangle will be filled with the color of the Paint. If we specify Style.STROKE, only the outline of the rectangle will be drawn, again with the colorand stroke width of the Paint. If Style.FILL_AND_STROKE is set, the rectangle will be filled, and the outline will be drawn with the given color and stroke width.

##### 圆

```java
Canvas.drawCircle(float centerX, float centerY, float radius, Paint paint);
```

圆也可以有样式。

#### 使用Bitmaps

##### 加载

利用`BitmapFactory`加载`Bitmap`。

因为我们的照片在`assets/`中：

	InputStream inputStream = assetManager.open("bob.png");
	Bitmap bitmap = BitmapFactory.decodeStream(inputStream);

获取大小：

	int width = bitmap.getWidth();
	int height = bitmap.getHeight();

`Bitmap`使用的颜色格式：

	Bitmap.Config config = bitmap.getConfig();

`Bitmap.Config`取值：`Config.ALPHA_8`、`Config.ARGB_4444`、`Config.ARGB_8888`、`Config.RGB_565`。

Interestingly there’s no RGB888 color format. PNG only supports ARGB8888, RGB888, and palettized colors. What color format would an RGB888 PNG be loaded to? `BitmapConfig.RGB_565` is the answer. This happens automatically for any RGB888 PNG we load via the BitmapFactory. The reason for this is that the actual framebuffer of most Android devices works with that color format. It would be a waste of memory to load an image with a higher bit depth per pixel, as the pixels would need to be converted to
RGB565 anyway for final rendering.

So why is there the Config.ARGB_8888 configuration then? Because image composition can be done on the CPU prior to actually drawing the final image to the framebuffer. In the case of the alpha component, we also have a lot more bit depth than with Config.ARGB_4444, which might be necessary for some high-quality image processing.

An ARGB8888 PNG image would be loaded to a Bitmap with a Config.ARGB_8888 configuration. The other two color formats are barely used. 我们也可以要求`BitmapFactory`加载特定颜色格式的图像，即使与图像本身的格式不同。

	InputStream inputStream = assetManager.open("bob.png"); 
	BitmapFactory.Options options = new BitmapFactory.Options(); 
	options.inPreferredConfig = Bitmap.Config.ARGB_4444; 
	Bitmap bitmap = BitmapFactory.decodeStream(inputStream, null, options); 

还可以创建一个空`Bitmap`：

	Bitmap bitmap = Bitmap.createBitmap(int width, int height, Bitmap.Config config); 

This might come in handy if you want to do custom image compositing yourself on the fly. `Canvas`类也能工作在bitmaps上： 

	Canvas canvas = new Canvas(bitmap); 

You can then modify your bitmaps in the same way you modify the contents of a View. 

##### Disposing of Bitmaps 
We should thus always dispose of any `Bitmap` instance we no longer need via the following method: 

	Bitmap.recycle(); 


##### Drawing Bitmaps 

加载好bitmaps后可以用`Canvas`绘制。最简单的方法：
 
	Canvas.drawBitmap(Bitmap bitmap, float topLeftX, float topLeftY, Paint paint);

`topLeftX`和`topLeftY`指定bitmap左上角的放置位置。最后一个参数可以为null。

另一个方法：

	Canvas.drawBitmap(Bitmap bitmap, Rect src, Rect dst, Paint paint);

`src`指定只绘制Bitmap的一部分。传null表示绘制全部。`dst`指定绘制目标位置。两个`Rect`不必一样大。如果`dst`比`src`小，Canvas will automatically scale for us。The same is true for specifying a larger destination 
rectangle, of course. 最后一个参数一般是null。注意，缩放操作非常昂贵。只有绝对必要时才能做。

So, you might wonder, if we have Bitmapinstances with different color formats, do we need to convert them to some kind of standard format before wecan draw them via a Canvas? The answer is no. The Canvaswill do this for us automatically. Of course, it will be a bit faster if we use color formats that are equal to the native framebuffer format. Usually we just ignore this, though.

	public class BitmapTest extends Activity { 
		class RenderView extends View { 
			Bitmap bob565; 
			Bitmap bob4444; 
			Rect dst = newRect(); 
			public RenderView(Context context) { 
				super(context); 
				try{ 
					AssetManager assetManager = context.getAssets(); 
					InputStream inputStream = assetManager.open("bobrgb888.png"); 
					bob565 = BitmapFactory.decodeStream(inputStream); 
					inputStream.close(); // !!! close !!!
					Log.d("BitmapText", 
						"bobrgb888.png format: " + bob565.getConfig()); 
					
					inputStream = assetManager.open("bobargb8888.png"); 
					BitmapFactory.Options options = newBitmapFactory.Options(); 
					options.inPreferredConfig = Bitmap.Config.ARGB_4444; 
					bob4444 = BitmapFactory.decodeStream(inputStream, null, options); 
					inputStream.close(); 
					Log.d("BitmapText", 
						"bobargb8888.png format: " + bob4444.getConfig()); 
				} catch(IOException e) { 
					// silently ignored, bad coder monkey, baaad! 
				} finally{ 
					// we should really close our input streams here. 
				} 
			} 
			protected voidonDraw(Canvas canvas) { 
				dst.set(50, 50, 350, 350); 
				canvas.drawBitmap(bob565, null, dst, null); 
				canvas.drawBitmap(bob4444, 100, 100, null); 
				invalidate();
			} 
		} 
		@Override 
		public voidonCreate(Bundle savedInstanceState) { 
			super.onCreate(savedInstanceState); 
			requestWindowFeature(Window.FEATURE_NO_TITLE); 
			getWindow().setFlags(WindowManager.LayoutParams.FLAG_FULLSCREEN, 
				WindowManager.LayoutParams.FLAG_FULLSCREEN); 
			setContentView(newRenderView(this)); 
		} 
	} 

#### 渲染文本

如何绘制TrueType字体的文字。

首先加载`assets/`下的字体文件。

##### 加载字体
 
The Android API provides us with a class called `Typeface` that encapsulates a `TrueType` 
font. It provides a simple static method to load such a font file from the `assets/`
directory: 

	Typeface font = Typeface.createFromAsset(context.getAssets(), "font.ttf"); 

如果字体不能加载，抛出运行时异常。

##### 使用字体绘制文字

Once we have our font, we set it as the Typeface of a Paint instance: 
	
	paint.setTypeFace(font); 

用`Paint`设置字体大小：

	paint.setTextSize(30); 

The documentation of this method is again a little sparse. It doesn’t tell whether the text 
size is given in points or pixels. We just assume the latter. 
Finally, we can draw text with this font via the following Canvas method: 

	canvas.drawText("This is a test!", 100, 100, paint); 

第二三个参数是绘制位置坐标。

##### 文本对齐和边界

The Paint instance has an attribute called the align setting. It can be set via this method of the Paint class: 

	Paint.setTextAlign(Paint.Align align); 

`Paint.Align`有三个值`Paint.Align.LEFT`, `Paint.Align.CENTER`, `Paint.Align.RIGHT`。根据对齐方式的设置，传给`Canvas.drawText()`方法的坐标被解释为矩形的左上角，矩形的上中点和右上角。默认是`Paint.Align.LEFT`。

调用下面的方法，可以获取特定字符串（`text`）中的子串（`start`开始，`end`之前）的边界位置。字符串的边界的单位是像素。

	Paint.getTextBounds(String text, int start, int end, Rect bounds);  

该方法会把边界矩形的宽度和高度放入`Rect.right`和`Rect.bottom`字段。For convenience we can call `Rect.width()` and `Rect.height()` to get the same values.

#### 使用SurfaceView做连续绘制

##### 动机

不要阻塞UI线程。脏活在其他线程中做。`SurfaceView`提供使用非UI线程绘制的方法。 

`SurfaceView`是一个持有`Surface`的View。`Surface`也是Android一个类。What is a Surface? It’s an abstraction of a raw buffer that is used by the screen compositor for rendering that specific View. The screen compositor is the master mind behind all rendering on Android, 最终负责将所有像素放入GPU。`Surface`可被硬件加速。

##### SurfaceHolder和Locking 

为了在另一个线程绘制，要先获取`SurfaceHolder`：

	SurfaceHolder holder = surfaceView.getHolder(); 

`SurfaceHolder`是`Surface`的一个包装，and does some bookkeeping for us. 提供两个方法：

	Canvas SurfaceHolder.lockCanvas(); 
	SurfaceHolder.unlockAndPost(Canvas canvas); 

The first method locks the Surface for rendering and returns a nice `Canvas` instance we 
can use. The second method unlocks the Surface again and makes sure that what 
we’ve drawn via the `Canvas` gets displayed on the screen. 在我们的绘制线程中，先获得一个`Canvas`，用它渲染，最后令渲染好的图像显示在屏幕上。传给`SurfaceHolder.unlockAndPost()`方法的`Canvas`必须是`SurfaceHolder.lockCanvas()`方法返回的。

The Surface is not immediately created when the SurfaceView is instantiated. Instead it 
is created asynchronously. The surface will be destroyed each time the activity is 
paused and recreated when the activity is resumed again.

##### Surface Creation and Validity 

Surface有效之前，不能从`SurfaceHolder`获取`Canvas`。可以检查：

	boolean isCreated = surfaceHolder.getSurface().isValid(); 

如果该方法返回true，我们可以安全的锁住surface。最后一定一定要调用`SurfaceHolder.lockCanvas()`。否则活动可能锁住手机。

	public class SurfaceViewTest extends Activity {  
		FastRenderView renderView;
		public void onCreate(Bundle savedInstanceState) {
			super.onCreate(savedInstanceState);
			requestWindowFeature(Window.FEATURE_NO_TITLE);
			getWindow().setFlags(WindowManager.LayoutParams.FLAG_FULLSCREEN,
				WindowManager.LayoutParams.FLAG_FULLSCREEN);
			renderView = new FastRenderView(this);
			setContentView(renderView);
		}
		protected void onResume() {
			super.onResume();
			renderView.resume();
		}
		protected void onPause() {
			super.onPause(); 
			renderView.pause();
		}

		class FastRenderView extends SurfaceView implements Runnable {
			Thread renderThread = null;
			SurfaceHolder holder;
			volatile boolean running = false;
			public FastRenderView(Context context) {
				super(context);
				holder = getHolder();    
			}
			public void resume() { 
				running = true; 
				renderThread = newThread(this); 
				renderThread.start(); 
			} 
			public void run() { 
				while(running) { 
					if(!holder.getSurface().isValid()) 
						continue; 
					Canvas canvas = holder.lockCanvas(); 
					canvas.drawRGB(255, 0, 0); 
					holder.unlockCanvasAndPost(canvas); 
				} 
			} 
			public void pause() { 
				running = false; 
				while(true) { 
					try{ 
						renderThread.join(); 
					} catch(InterruptedException e) { 
						// retry 
					} 
				} 
			} 
		} 
	}

## 4.3 最佳实践

* The garbage collector is your biggest enemy. Once it gets CPU time to 
do its dirty work, it will stop the world for up to 600 ms. That’s half a 
second that our game will not update or render. The user will 
complain. Avoid object creation as much as possible, especially in 
your inner loops. 
* Objects can get created in some not-so-obvious places, which you’ll 
want to avoid. Don’t use iterators,as they create new objects. Don’t 
use any of the standard Setor Mapcollection classes,as they create 
new objects on each insertion; use the SparseArrayclass provided by 
the Android API instead. Use StringBuffers instead of concatenating 
strings with the +operator. That will create a new StringBuffereach 
time. And for the love of all that’s good in this world, don’t use boxed 
primitives! 
* Method calls have a larger associated cost in Dalvik than in other VMs. 
Use static methods if you can, as those perform best. Static methods 
are generally regarded as evil, much like static variables, as they 
promote bad design. So try to keepyour design as clean as possible. 
You should maybe avoid getters andsetters as well. Direct field 
access is about three times faster than method invocations without the 
JIT, and about seven times faster with the JIT. Again, think of your 
design before removing all yourgetters and setters, though. 


