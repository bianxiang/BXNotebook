## android.view.View

All of the views in a window are arranged in a single tree.

> 注意：测量、布局和绘制的方法，由Android框架负责调用。开发者不要调用，除非你要实现的是一个android.view.ViewGroup。

### 实现自定义View

选择性的重写下面的方法。最简单的情况只要重写`onDraw(android.graphics.Canvas)`。

- 创建类方法：
	- 构造器
	- `onFinishInflate()`：在View及所有孩子从XML充气完成后调用
- 布局类方法：
	- `onMeasure(int, int)`：决定此View及其所有孩子所需要的大小
	- `onLayout(boolean, int, int, int, int)`：当View需要为所有孩子分配大小和位置时调用，
	- `onSizeChanged(int, int, int, int)`：当此View的大小改变时调用。
- 绘制类方法：
	- `onDraw(android.graphics.Canvas)`：当View渲染自己内容时调用
- 事件处理方法：`onKeyDown(int, KeyEvent)`等。
- 焦点类方法：
	- `onFocusChanged(boolean, int, android.graphics.Rect)`：当View自身获得或失去焦点
	- `onWindowFocusChanged(boolean)`：当View所在窗口获得或失去焦点。
- Attaching类方法：
	- `onAttachedToWindow()`：当View附到窗口上时调用
	- `onDetachedFromWindow`
	- `onWindowVisibilityChanged(int)`：View所在窗口的可见性发生变化

### IDs

每个View可以有一个整数ID。

View IDs need not be unique throughout the tree, but it is good practice to ensure that they are at least unique within the part of the tree you are searching. 

### 位置

View的位置由左上角位置决定。

位置和大小的单位是像素。 

通过`getLeft()`和`getTop()`方法获取View的位置。这两个方法返回的都是相对于父容器的位置。

相关方法：`getRight()`和`getBottom()`。例如`getRight()`等价于`getLeft() + getWidth()`。

### 大小、padding和margins

View实际有两组宽高值。

第一组称为 measured width 和 measured height。这两个值是View**想要**在父容器内占用的空间。The measured dimensions can be obtained by calling getMeasuredWidth() and getMeasuredHeight().

第二组成为 width 和 height，或 drawing width 和 drawing height。这两个值定义View在屏幕上的实际大小（布局后，绘制期）。这两个值与 measured width 和 height可能不同。 The width and height can be obtained by calling getWidth() and getHeight(). 

To measure its dimensions, a view takes into account its padding. padding单位是像素。 Padding可以通过`setPadding(int, int, int, int)`或`setPaddingRelative(int, int, int, int)`设置。

**View不支持Margin**。Viewer支持Margin。Refer to android.view.ViewGroup and android.view.ViewGroup.MarginLayoutParams for further information.

### 布局

布局分两个阶段：测量阶段和布局阶段。测量阶段由`measure(int, int)`实现，沿view树从上到下遍历。Each view pushes dimension specifications down the tree during the recursion. At the end of the measure pass, every view has stored its measurements. 第二阶段发生在`layout(int, int, int, int)`，也是自上而下对的。在该阶段，根据测量阶段的值，父容器定位所有孩子。

当View的`measure()`方法返回后，`getMeasuredWidth()`和`getMeasuredHeight()`的值必须被设置；View所有后代的这两个值也会被设置。A view's measured width and measured height values must respect the constraints imposed by the view's parents. This guarantees that at the end of the measure pass, all parents accept all of their children's measurements. A parent view may call measure() more than once on its children. For example, the parent may measure each child once with unspecified dimensions to find out how big they want to be, then call measure() on them again with actual numbers if the sum of all the children's unconstrained sizes is too big or too small. 

测量阶段使用两个类携带大小值。View通过`MeasureSpec`类告诉父容器它们想如何被测量和定位。The base `LayoutParams` class just describes how big the view wants to be for both width and height. For each dimension, it can specify one of: 

- 一个精确的数组：
- `MATCH_PARENT`, which means the view wants to be as big as its parent (minus padding) 
- `WRAP_CONTENT`, which means that the view wants to be just big enough to enclose its content (plus padding). 

`ViewGroup`的子类们定义了一些`LayoutParams`的子类。For example, AbsoluteLayout has its own subclass of LayoutParams which adds an X and Y value. 

`MeasureSpec`s are used to push requirements down the tree from parent to child. 一个`MeasureSpec`可以处于三种模式之一：

- `UNSPECIFIED`: This is used by a parent to determine the desired dimension of a child view. For example, a LinearLayout may call measure() on its child with the height set to `UNSPECIFIED` and a width of `EXACTLY` 240 to find out how tall the child view wants to be given a width of 240 pixels. 
- `EXACTLY`: This is used by the parent to impose an exact size on the child. The child must use this size, and guarantee that all of its descendants will fit within this size. 
- `AT_MOST:` This is used by the parent to impose a maximum size on the child. The child must gurantee that it and all of its descendants will fit within this size. 

To intiate a layout, call `requestLayout`. This method is typically called by a view on itself when it believes that is can no longer fit within its current bounds. 

### 绘制

Drawing is handled by walking the tree and rendering each view that intersects the invalid region. Because the tree is traversed in-order, this means that parents will draw before (i.e., behind) their children, with siblings drawn in the order they appear in the tree. If you set a background drawable for a View, then the View will draw it for you **before** calling back to its `onDraw()` method. 

Note that the framework will not draw views that are not in the invalid region. 

To force a view to draw, call `invalidate()`. 

### 事件处理

The basic cycle of a view is as follows: 

1. An event comes in and is dispatched to the appropriate view. The view handles the event and notifies any listeners. 
1. If in the course of processing the event, the view's bounds may need to be changed, the view will call `requestLayout()`. 
1. Similarly, if in the course of processing the event the view's appearance may need to be changed, the view will call `invalidate()`. 
1. If either `requestLayout() `or `invalidate()` were called, the framework will take care of measuring, laying out, and drawing the tree as appropriate. 

### 焦点

框架会根据用户输入处理焦点移动。包括当View被移除或隐藏时，当新View被加入时。Views indicate their willingness to take focus through the `isFocusable` method. To change whether a view can take focus, call `setFocusable(boolean)`. When in touch mode (see notes below) views indicate whether they still would like focus via `isFocusableInTouchMode` and can change this via `setFocusableInTouchMode(boolean)`. 

焦点移动的算法基于在某个方向上的最近邻。若此算法与期望不符，可以提供显式对的覆盖，通过XML特性：

- nextFocusDown
- nextFocusLeft
- nextFocusRight
- nextFocusUp

To get a particular view to take focus, call `requestFocus()`.

### 触摸模式

如果设备具备触发能力，则不再需要让View能够获得焦点（获得焦点后保持高亮）。这种模式称为触摸模式。

对于一个可触摸的设备，**用户触摸屏幕后**，设备进入触摸模式。此后，只有`isFocusableInTouchMode`返回true的View可以获得焦点（如文本框）。

任何时候，用户触摸方向键，设备退出触摸模式。

触摸模式在Activity之间会保持。通过`isInTouchMode`判断设备是否处于触摸模式。

### 滚动

The framework provides basic support for views that wish to internally scroll their content. This includes keeping track of the X and Y scroll offset as well as mechanisms for drawing scrollbars. See `scrollBy(int, int)`, `scrollTo(int, int)`, and `awakenScrollBars()` for more details. 

### Tags

Unlike IDs, tags are not used to identify views. Tags are essentially an extra piece of information that can be associated with a view. They are most often used as a convenience to store data related to views in the views themselves rather than by putting them in a separate structure. 

### Properties

The View class exposes an ALPHA property, as well as several transform-related properties, such as `TRANSLATION_X` and `TRANSLATION_Y`. These properties are available both in the `android.util.Property` form as well as in similarly-named setter/getter methods (such as `setAlpha(float)` for ALPHA). These properties can be used to set persistent state associated with these rendering-related properties on the view. The properties and methods can also be used in conjunction with Animator-based animations, described more in the Animation section.

### 动画

Starting with Android 3.0, the preferred way of animating views is to use the android.animation package APIs. These `Animator`-based classes change actual properties of the View object, such as alpha and translationX. This behavior is contrasted to that of the pre-3.0 Animation-based classes, which instead animate only how the view is drawn on the display. In particular, the `ViewPropertyAnimator` class makes animating these View properties particularly easy and efficient. 

Alternatively, you can use the pre-3.0 animation classes to animate how Views are rendered. You can attach an Animation object to a view using setAnimation(Animation) or startAnimation(Animation). The animation can alter the scale, rotation, translation and alpha of a view over time. If the animation is attached to a view that has children, the animation will affect the entire subtree rooted by that node. When an animation is started, the framework will take care of redrawing the appropriate views until the animation completes.

### 安全

Sometimes it is essential that an application be able to verify that an action is being performed with the full knowledge and consent of the user, such as granting a permission request, making a purchase or clicking on an advertisement. Unfortunately, a malicious application could try to spoof the user into performing these actions, unaware, by concealing the intended purpose of the view. As a remedy, the framework offers a touch filtering mechanism that can be used to improve the security of views that provide access to sensitive functionality. 

To enable touch filtering, call `setFilterTouchesWhenObscured(boolean)` or set the android:filterTouchesWhenObscured layout attribute to true. When enabled, the framework will discard touches that are received whenever the view's window is obscured by another visible window. As a result, the view will not receive touches whenever a toast, dialog or other window appears above the view's window. 

For more fine-grained control over security, consider overriding the onFilterTouchEventForSecurity(MotionEvent) method to implement your own security policy. See also `MotionEvent.FLAG_WINDOW_IS_OBSCURED`. 

## android.view.ViewGroup

The view group is the base class for layouts and views containers. This class also defines the `android.view.ViewGroup.LayoutParams` class which serves as the base class for layouts parameters.

## android.view.ViewGroup.LayoutParams

LayoutParams are used by views to tell their parents how they want to be laid out.

基类`LayoutParams`只描述了View对自己宽高的期望。

## android.util.AttributeSet

A collection of attributes, as found associated with a tag in an XML document. Often you will not want to use this interface directly, instead passing it to `Resources.Theme.obtainStyledAttributes()` which will take care of parsing the attributes for you. In particular, the `Resources` API will convert resource references (attribute values such as `"@string/my_label"` in the original XML) to the desired type for you; if you use `AttributeSet` directly then you will need to manually check for resource references (with `getAttributeResourceValue(int, int)`) and do the resource lookup yourself if needed. Direct use of `AttributeSet` also prevents the application of themes and styles when retrieving attribute values. 

This interface provides an efficient mechanism for retrieving data from compiled XML files, which can be retrieved for a particular `XmlPullParser` through `Xml.asAttributeSet()`. Normally this will return an implementation of the interface that works on top of a generic `XmlPullParser`, however it is more useful in conjunction with compiled XML resources: 

	XmlPullParser parser = resources.getXml(myResouce);
	AttributeSet attributes = Xml.asAttributeSet(parser);

The implementation returned here, unlike using the implementation on top of a generic XmlPullParser, is highly optimized by retrieving pre-computed information that was generated by aapt when compiling your resources. For example, the getAttributeFloatValue(int, float) method returns a floating point number previous stored in the compiled resource instead of parsing at runtime the string originally in the XML file. 

This interface also provides additional information contained in the compiled XML resource that is not available in a normal XML file, such as getAttributeNameResource(int) which returns the resource identifier associated with a particular XML attribute name.






