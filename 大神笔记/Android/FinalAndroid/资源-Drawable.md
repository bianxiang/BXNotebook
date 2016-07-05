[toc]

### Drawable 资源

一个 Drawable 资源是一个可以绘制在屏幕上的图形。有以下几种类型：

- **Bitmap文件**：一个 bitmap 图形文件(.png, .jpg, or .gif)。创建一个 `BitmapDrawable`。
- **Nine-Patch文件**：一个PNG图片(.9.png)，指定可拉伸区域，允许图片根据内容调整大小。创建一个 `NinePatchDrawable`。
- **层列表**：一个 Drawable 数组。在数组后面的元素绘制在上面。创建一个 `LayerDrawable`。
- **状态列表**：不同的状态引用不同的 bitmap（例如，当按钮被按下后使用不同的图）。创建一个 `StateListDrawable`。
- **Level List**：一个XML文件，定义一个 drawable，管理多个可替换的 Drawables，每个分配一个最大值。Creates a `LevelListDrawable`.
- **Transition Drawable**：两个 drawable 资源淡入淡出（cross-fade）。创建一个 `TransitionDrawable`。
- **Inset Drawable** 缩进一个 drawable 产生一个新的 drawable。This is useful when a View needs a background drawble that is smaller than the View's actual bounds.
- **Clip Drawable** An XML file that defines a drawable that clips another Drawable based on this Drawable's current *level* value. Creates a `ClipDrawable`.
- **Scale Drawable** An XML file that defines a drawable that changes the size of another Drawable based on its current *level* value. Creates a `ScaleDrawable`
- **Shape Drawable** 一个XML文件，定义一个几何形状，包括颜色和渐变。Creates a `ShapeDrawable`.

一个颜色资源也可以被当作一个 drawable。For example, when creating a state list drawable, you can reference a color resource for the `android:drawable` attribute (`android:drawable="@color/green"`).

#### Bitmap

Android支持三种格式：.png (preferred), .jpg (acceptable), .gif (discouraged)。

在构建阶段，aapt 工具（无损压缩）会自动优化 Bitmap 文件。例如一个支持颜色不超过256色的真彩色 PNG 会被转换成带色盘（color palette）的8-bit PNG。这样图片内存减少但指令不变。If you plan on reading an image as a bit stream in order to convert it to a bitmap, put your images in the `res/raw/` folder instead, where they will not be optimized.

##### Bitmap 文件

文件位置 `res/drawable/filename.png`。文件名做为资源名。编译后的资源类型是 `BitmapDrawable`。 引用方式 `R.drawable.filename`、 `@[package:]drawable/filename`。

例子：`res/drawable/myimage.png`

```xml
	<ImageView
	    android:layout_height="wrap_content"
	    android:layout_width="wrap_content"
	    android:src="@drawable/myimage" />
```

```java
Resources res = getResources();
Drawable drawable = res.getDrawable(R.drawable.myimage);
```

参见：

- http://developer.android.com/guide/topics/graphics/2d-graphics.html
- http://developer.android.com/reference/android/graphics/drawable/BitmapDrawable.html

##### XML Bitmap

XML bitmap 定义在XML中，指向一个 bitmap 文件。可以作为一个原始 bitmap 文件的别名。还可以指定 bitmap 的附加属性，如 dithering 和 tiling。

`<bitmap>` 可以作为 `<item>` 的子元素。例如创建 *state list* 时，可以不用 `<item>` 的 `android:drawable` 特性，而是嵌入一个 `<bitmap>`。

例如，文件位置` res/drawable/filename.xml`，文件名作为资源名。编译后的资源类型是 `BitmapDrawable`。 引用：`R.drawable.filename`、`@[package:]drawable/filename`。

语法：

```
	<?xml version="1.0" encoding="utf-8"?>
	<bitmap
	    xmlns:android="http://schemas.android.com/apk/res/android"
	    android:src="@[package:]drawable/drawable_resource"
	    android:antialias=["true" | "false"]
	    android:dither=["true" | "false"]
	    android:filter=["true" | "false"]
	    android:gravity=["top" | "bottom" | "left" | "right" | "center_vertical" |
	                      "fill_vertical" | "center_horizontal" | "fill_horizontal" |
	                      "center" | "fill" | "clip_vertical" | "clip_horizontal"]
	    android:mipMap=["true" | "false"]
	    android:tileMode=["disabled" | "clamp" | "repeat" | "mirror"] />
```

特性：

- `android:src`：必需。引用另一个 drawable 资源。
- `android:antialias`：布尔。antialiasing.
- `android:dither`：布尔。Enables or disables dithering of the bitmap if the bitmap does not have the same pixel configuration as the screen (for instance: a ARGB 8888 bitmap with an RGB 565 screen).
- `android:filter`：布尔。Enables or disables bitmap filtering. Filtering is used when the bitmap is shrunk or stretched to smooth its apperance.
- `android:gravity`：The gravity indicates where to position the drawable in its container if the bitmap is smaller than the container. 可以用'|'分隔多个常量值：
  - `top` 放在容器顶部。不改变大小。
  - `bottom`
  - `left`
  - `right`
  - `center_vertical` 不改变大小。
  - `fill_vertical` 如果需要，增加对象的高度。填满容器。
  - `center_horizontal` 不改变大小。
  - `fill_horizontal` 如果需要，增加对象的水平长度。填满容器。
  - `center`水平和垂直居中，不改变大小。
  - `fill` **默认值**。如果需要，增加对象的水平和垂直大小。填满容器。
  - `clip_vertical` Additional option that can be set to have the top and/or bottom edges of the child clipped to its container's bounds. The clip is based on the vertical gravity: a top gravity clips the bottom edge, a bottom gravity clips the top edge, and neither clips both edges.
  - `clip_horizontal` Additional option that can be set to have the left and/or right edges of the child clipped to its container's bounds. The clip is based on the horizontal gravity: a left gravity clips the right edge, a right gravity clips the left edge, and neither clips both edges.
- `android:mipMap`：布尔。Enables or disables the mipmap hint. See `setHasMipMap()` for more information. 默认false。
- `android:tileMode`：定义平铺模式。When the tile mode is enabled, the bitmap is repeated. 启用平铺模式后 Gravity 会被忽略。取值：
  - `disabled` **默认值**。不要铺满。
  - `clamp` Replicates the *edge color* if the shader draws outside of its original bounds
  - `repeat` Repeats the shader's image horizontally and vertically.
  - `mirror` Repeats the shader's image horizontally and vertically, alternating mirror images so that adjacent images always seam.

例子：

```xml
	<?xml version="1.0" encoding="utf-8"?>
	<bitmap xmlns:android="http://schemas.android.com/apk/res/android"
	    android:src="@drawable/icon"
	    android:tileMode="repeat" />
```

SEE ALSO:

http://developer.android.com/guide/topics/resources/providing-resources.html#AliasResources

#### Nine-Patch

[NinePatch](http://developer.android.com/reference/android/graphics/NinePatch.html)是一个 PNG 图像，你可以定义可拉伸区域，当视图的内容变大超过图像大小时，Android 可以缩放。这种类型图片一般作为视图背景，且至少一个维度是 `"wrap_content"`，当视图内容变大时，Nine-Patch 图像随着放大，匹配视图大小。Nine-Patch 的例子是，Android 标准按钮使用的背景。

For a complete discussion about how to create a Nine-Patch file with stretchable regions, see the [2D Graphics document](http://developer.android.com/guide/topics/graphics/2d-graphics.html#nine-patch).

例子，`res/drawable/filename.9.png`，文件名做资源名，编译后的资源类型是 `NinePatchDrawable`。引用资源 `R.drawable.filename`、`@[package:]drawable/filename`。

例如，`res/drawable/myninepatch.9.png`，使用在按钮上：

```xml
	<Button
	    android:layout_height="wrap_content"
	    android:layout_width="wrap_content"
	    android:background="@drawable/myninepatch" />
```

参见：

http://developer.android.com/guide/topics/graphics/2d-graphics.html#nine-patch
http://developer.android.com/reference/android/graphics/drawable/NinePatchDrawable.html

##### XML Nine-Patch

An XML Nine-Patch is a resource defined in XML that points to a Nine-Patch file. The XML can specify dithering for the image.

FILE LOCATION:
res/drawable/filename.xml

The filename is used as the resource ID.

Resource pointer to a NinePatchDrawable.

RESOURCE REFERENCE:
In Java: `R.drawable.filename`
In XML: `@[package:]drawable/filename`

语法：

```xml
	<?xml version="1.0" encoding="utf-8"?>
	<nine-patch
	    xmlns:android="http://schemas.android.com/apk/res/android"
	    android:src="@[package:]drawable/drawable_resource"
	    android:dither=["true" | "false"] />
```

特性：

- `android:src` Drawable resource. Required. Reference to a Nine-Patch file.
- `android:dither` Boolean. Enables or disables dithering of the bitmap if the bitmap does not have the same pixel configuration as the screen (for instance: a ARGB 8888 bitmap with an RGB 565 screen).

例子：

```xml
	<?xml version="1.0" encoding="utf-8"?>
	<nine-patch xmlns:android="http://schemas.android.com/apk/res/android"
	    android:src="@drawable/myninepatch"
	    android:dither="false" />
```

#### Layer List

LayerDrawable 管理一个 drawables 数组。数组中后面的元素绘制在最上面。

FILE LOCATION: `res/drawable/filename.xml`。文件名是资源名。编译后的资源类型`LayerDrawable`。引用`R.drawable.filename`、`@[package:]drawable/filename`。

语法：

```xml
	<?xml version="1.0" encoding="utf-8"?>
	<layer-list
	    xmlns:android="http://schemas.android.com/apk/res/android" >
	    <item
	        android:drawable="@[package:]drawable/drawable_resource"
	        android:id="@[+][package:]id/resource_name"
	        android:top="dimension"
	        android:right="dimension"
	        android:bottom="dimension"
	        android:left="dimension" />
	</layer-list>
```

`<item>` 中可以放 `<bitmap>`。`<item>` 的特性：

- `android:top` 整数。顶部偏移，单位像素。
- `android:right`
- `android:bottom`
- `android:left`

默认，所有items都会缩放适应容器视图的大小。如果将图片放入不同位置，可能增加视图大小，图片会缩放。要避免缩放，在 `<item>` 中嵌套 `<bitmap>`，定义 `gravity` 为不缩放的值，如 "center"。For example, the following `<item>` defines an item that scales to fit its container View:

```xml
	<item android:drawable="@drawable/image" />
```

为避免缩放，利用`<bitmap>`子元素：

```xml
	<item>
	  <bitmap android:src="@drawable/image" android:gravity="center" />
	</item>
```

例子：`res/drawable/layers.xml`

```xml
    <layer-list xmlns:android="http://schemas.android.com/apk/res/android">
        <item>
          <bitmap android:src="@drawable/android_red"
            android:gravity="center" />
        </item>
        <item android:top="10dp" android:left="10dp">
          <bitmap android:src="@drawable/android_green"
            android:gravity="center" />
        </item>
        <item android:top="20dp" android:left="20dp">
          <bitmap android:src="@drawable/android_blue"
            android:gravity="center" />
        </item>
    </layer-list>
```

This layout XML applies the drawable to a View:

```xml
	<ImageView
	    android:layout_height="wrap_content"
	    android:layout_width="wrap_content"
	    android:src="@drawable/layers" />
```

效果：

![](img/drawable_layers.png)

#### State List

`StateListDrawable` 定义在XML中，使用几个不同的图像表示相同图形对的不同状态。例如，按钮的状态有被按下、有焦点等。

每个状态用一个 `<item>` 元素表示。

注意，匹配状态时不是按最优匹配，而是从上到下匹配，匹配到适合当前状态后停止。

FILE LOCATION: `res/drawable/filename.xml`。文件名就是资源名。编译后的资源类型是 `StateListDrawable`。引用：`R.drawable.filename`、`@[package:]drawable/filename`。

SYNTAX:

```xml
	<selector xmlns:android="http://schemas.android.com/apk/res/android"
	    android:constantSize=["true" | "false"]
	    android:dither=["true" | "false"]
	    android:variablePadding=["true" | "false"] >
	    <item
	        android:drawable="@[package:]drawable/drawable_resource"
	        android:state_pressed=["true" | "false"]
	        android:state_focused=["true" | "false"]
	        android:state_hovered=["true" | "false"]
	        android:state_selected=["true" | "false"]
	        android:state_checkable=["true" | "false"]
	        android:state_checked=["true" | "false"]
	        android:state_enabled=["true" | "false"]
	        android:state_activated=["true" | "false"]
	        android:state_window_focused=["true" | "false"] />
	</selector>
```

`<selector>` 元素是根元素。它的特性有：

- `android:constantSize`：布尔。"true" if the drawable's reported internal size remains constant as the state changes (the size is the maximum of all of the states); "false" if the size varies based on the current state. 默认是false。
- `android:dither`：布尔。"true" to enable dithering of the bitmap if the bitmap does not have the same pixel configuration as the screen (for instance, an ARGB 8888 bitmap with an RGB 565 screen); "false" to disable dithering. Default is true.
- `android:variablePadding`：布尔。"true" if the drawable's padding should change based on the current state that is selected; "false" if the padding should stay the same (based on the maximum padding of all the states). Enabling this feature requires that you deal with performing layout when the state changes, 多数情况下是不支持的。默认false。

`<item>` 的特性：

- `android:drawable`：必需。
- `android:state_pressed`：布尔。"true"表示当对象被按下时应该使用此项；"false" if this item should be used in the default, non-pressed state.
- `android:state_focused`：布尔。"true"表示对象拥有焦点时应该使用此项；"false" if this item should be used in the default, non-focused state.
- `android:state_hovered`：布尔。"true"表示对象hovered by a cursor是该使用此项; "false" if this item should be used in the default, non-hovered state. 一般该状态与"focused"状态的效果相同。Introduced in API level 14.
- `android:state_selected`:布尔。"true" if this item should be used when the object is the current user selection when navigating with a directional control (such as when navigating through a list with a d-pad); "false" if this item should be used when the object is not selected. The selected state is used when focus (android:state_focused) is not sufficient （例如焦点在list view上，但其中的项可以d-pad选择）。
- `android:state_checkable`：布尔。"true" if this item should be used when the object is checkable; "false" if this item should be used when the object is not checkable. (Only useful if the object can transition between a checkable and non-checkable widget.)
- `android:state_checked`：布尔。"true" if this item should be used when the object is checked; "false" if it should be used when the object is un-checked.
- `android:state_enabled`：布尔。"true" if this item should be used when the object is enabled (capable of receiving touch/click events); "false" if it should be used when the object is disabled.
- `android:state_activated`：Boolean. "true" if this item should be used when the object is activated as the persistent selection (such as to "highlight" the previously selected list item in a persistent navigation view); "false" if it should be used when the object is not activated. Introduced in API level 11.
- `android:state_window_focused`：Boolean. "true" if this item should be used when the application window has focus (the application is in the foreground), "false" if this item should be used when the application window does not have focus (for example, if the notification shade is pulled down or a dialog appears).

例子：

XML file saved at `res/drawable/button.xml`:

```xml
	<selector xmlns:android="http://schemas.android.com/apk/res/android">
	    <item android:state_pressed="true"
	          android:drawable="@drawable/button_pressed" /> <!-- pressed -->
	    <item android:state_focused="true"
	          android:drawable="@drawable/button_focused" /> <!-- focused -->
	    <item android:state_hovered="true"
	          android:drawable="@drawable/button_focused" /> <!-- hovered -->
	    <item android:drawable="@drawable/button_normal" /> <!-- default -->
	</selector>
```

This layout XML applies the state list drawable to a `Button`:

```xml
	<Button
	    android:layout_height="wrap_content"
	    android:layout_width="wrap_content"
	    android:background="@drawable/button" />
```

参见：http://developer.android.com/reference/android/graphics/drawable/StateListDrawable.html

#### Level List

一个 Drawable，管理一组可替换的 Drawables, each assigned a maximum numerical value. Setting the level value of the drawable with `setLevel()` loads the drawable resource in the level list that has a `android:maxLevel` value greater than or equal to the value passed to the method.

FILE LOCATION: `res/drawable/filename.xml`. The filename is used as the resource ID.

Resource pointer to a `LevelListDrawable`.

RESOURCE REFERENCE:
In Java: R.drawable.filename
In XML: @[package:]drawable/filename

语法：

```xml
    <level-list xmlns:android="http://schemas.android.com/apk/res/android" >
        <item
            android:drawable="@drawable/drawable_resource"
            android:maxLevel="integer"
            android:minLevel="integer" />
    </level-list>
```

例子：

```xml
    <level-list xmlns:android="http://schemas.android.com/apk/res/android" >
        <item
            android:drawable="@drawable/status_off"
            android:maxLevel="0" />
        <item
            android:drawable="@drawable/status_on"
            android:maxLevel="1" />
    </level-list>
```

Once this is applied to a `View`, the level can be changed with `setLevel()` or `setImageLevel()`.

参见：http://developer.android.com/reference/android/graphics/drawable/LevelListDrawable.html

#### Transition Drawable

A `TransitionDrawable` is a drawable object that can cross-fade between the two drawable resources.

Each drawable is represented by an `<item>` element inside a single `<transition>` element. 最多支持两个元素。To transition forward, call `startTransition()`. To transition backward, call `reverseTransition()`.

FILE LOCATION: res/drawable/filename.xml
The filename is used as the resource ID.

Resource pointer to a TransitionDrawable.

RESOURCE REFERENCE:
In Java: R.drawable.filename
In XML: @[package:]drawable/filename

语法：

```xml
    <transition xmlns:android="http://schemas.android.com/apk/res/android" >
        <item
            android:drawable="@[package:]drawable/drawable_resource"
            android:id="@[+][package:]id/resource_name"
            android:top="dimension"
            android:right="dimension"
            android:bottom="dimension"
            android:left="dimension" />
    </transition>
```

`<item>` accepts child `<bitmap>` elements.

EXAMPLE:

XML file saved at res/drawable/transition.xml:

```xml
    <transition xmlns:android="http://schemas.android.com/apk/res/android">
        <item android:drawable="@drawable/on" />
        <item android:drawable="@drawable/off" />
    </transition>
```

This layout XML applies the drawable to a View:

```xml
    <ImageButton
        android:id="@+id/button"
        android:layout_height="wrap_content"
        android:layout_width="wrap_content"
        android:src="@drawable/transition" />
```

And the following code performs a 500ms transition from the first item to the second:

```java
ImageButton button = (ImageButton) findViewById(R.id.button);
TransitionDrawable drawable = (TransitionDrawable) button.getDrawable();
drawable.startTransition(500);
```

参见：http://developer.android.com/reference/android/graphics/drawable/TransitionDrawable.html

#### Inset Drawable

A drawable defined in XML that insets another drawable by a specified distance. This is useful when a View needs a background that is smaller than the View's actual bounds.

FILE LOCATION: res/drawable/filename.xml
The filename is used as the resource ID.

Resource pointer to a InsetDrawable.

RESOURCE REFERENCE:
In Java: R.drawable.filename
In XML: @[package:]drawable/filename

语法：

```xml
    <inset
        xmlns:android="http://schemas.android.com/apk/res/android"
        android:drawable="@drawable/drawable_resource"
        android:insetTop="dimension"
        android:insetRight="dimension"
        android:insetBottom="dimension"
        android:insetLeft="dimension" />
```

例子：

```xml
    <inset xmlns:android="http://schemas.android.com/apk/res/android"
        android:drawable="@drawable/background"
        android:insetTop="10dp"
        android:insetLeft="10dp" />
```

参见：http://developer.android.com/reference/android/graphics/drawable/InsetDrawable.html

#### Clip Drawable

A drawable defined in XML that clips another drawable based on this Drawable's current level. You can control how much the child drawable gets clipped in width and height based on the level, as well as a gravity to control where it is placed in its overall container. Most often used to implement things like progress bars.

FILE LOCATION: res/drawable/filename.xml
The filename is used as the resource ID.

Resource pointer to a ClipDrawable.

RESOURCE REFERENCE:
In Java: R.drawable.filename
In XML: @[package:]drawable/filename

语法：

```xml
    <clip
        xmlns:android="http://schemas.android.com/apk/res/android"
        android:drawable="@drawable/drawable_resource"
        android:clipOrientation=["horizontal" | "vertical"]
        android:gravity=["top" | "bottom" | "left" | "right" | "center_vertical" |
                         "fill_vertical" | "center_horizontal" | "fill_horizontal" |
                         "center" | "fill" | "clip_vertical" | "clip_horizontal"] />
```

`android:gravity` 可以用 `|` 组合多个常量。

- `top`：将对象放在容器的顶部，不改变它的大小。当 `clipOrientation` 是 `"vertical"`，裁掉图形底部。
- `bottom`
- `left`：将对象放在容器左边，不改变其大小。这是默认值。当 `clipOrientation` 是 `"horizontal"`，裁掉右边。
- `right`
- `center_vertical`：将对象放在容器的垂直中心，不改变其大小。Clipping behaves the same as when gravity is "center".
- `fill_vertical`：若需要，增加对象的垂直大小，填满容器。若 `clipOrientation` 是 `"vertical"`，不会发生裁切。(unless the drawable level is 0, in which case it's not visible).
- `center_horizontal`
- `fill_horizontal`
- `center`：将对象放在容器中心，不改变其大小。若 `clipOrientation` 为 `"horizontal"`，裁切左右。若 `clipOrientation` 为 `"vertical"`，裁切上下。
- `fill`：Grow the horizontal and vertical size of the object if needed so it completely fills its container. No clipping occurs because the drawable fills the horizontal and vertical space (unless the drawable level is 0, in which case it's not visible).
- `clip_vertical`：Additional option that can be set to have the top and/or bottom edges of the child clipped to its container's bounds. The clip is based on the vertical gravity: a top gravity clips the bottom edge, a bottom gravity clips the top edge, and neither clips both edges.
- `clip_horizontal`：Additional option that can be set to have the left and/or right edges of the child clipped to its container's bounds. The clip is based on the horizontal gravity: a left gravity clips the right edge, a right gravity clips the left edge, and neither clips both edges.

例子：

XML file saved at res/drawable/clip.xml:

```xml
    <clip xmlns:android="http://schemas.android.com/apk/res/android"
        android:drawable="@drawable/android"
        android:clipOrientation="horizontal"
        android:gravity="left" />
```

The following layout XML applies the clip drawable to a View:

```xml
    <ImageView
        android:id="@+id/image"
        android:background="@drawable/clip"
        android:layout_height="wrap_content"
        android:layout_width="wrap_content" />
```

The following code gets the drawable and increases the amount of clipping in order to progressively reveal the image:

```java
ImageView imageview = (ImageView) findViewById(R.id.image);
ClipDrawable drawable = (ClipDrawable) imageview.getDrawable();
drawable.setLevel(drawable.getLevel() + 1000);
```

Increasing the level reduces the amount of clipping and slowly reveals the image. Here it is at a level of 7000:

![](img/clip.png)

Note: The default level is 0, which is fully clipped so the image is not visible. When the level is 10,000, the image is not clipped and completely visible.

参见：http://developer.android.com/reference/android/graphics/drawable/ClipDrawable.html

#### Scale Drawable

A drawable defined in XML that changes the size of another drawable based on its current level.

FILE LOCATION: res/drawable/filename.xml
The filename is used as the resource ID.

Resource pointer to a ScaleDrawable.

RESOURCE REFERENCE:
In Java: R.drawable.filename
In XML: @[package:]drawable/filename

语法：

```xml
    <scale
        xmlns:android="http://schemas.android.com/apk/res/android"
        android:drawable="@drawable/drawable_resource"
        android:scaleGravity=["top" | "bottom" | "left" | "right" | "center_vertical" |
                              "fill_vertical" | "center_horizontal" | "fill_horizontal" |
                              "center" | "fill" | "clip_vertical" | "clip_horizontal"]
        android:scaleHeight="percentage"
        android:scaleWidth="percentage" />
```

`android:scaleGravity` 可以取多个关键字， `|` 分隔。

- `top`：Put the object at the top of its container, not changing its size.
- `bottom`：Put the object at the bottom of its container, not changing its size.
- `left`：Put the object at the left edge of its container, not changing its size. This is the default.
- `right`：Put the object at the right edge of its container, not changing its size.
- `center_vertical`：Place object in the vertical center of its container, not changing its size.
- `fill_vertical`：Grow the vertical size of the object if needed so it completely fills its container.
- `center_horizontal`：Place object in the horizontal center of its container, not changing its size.
- `fill_horizontal`：Grow the horizontal size of the object if needed so it completely fills its container.
- `center`：Place the object in the center of its container in both the vertical and horizontal axis, not changing its size.
- `fill`：Grow the horizontal and vertical size of the object if needed so it completely fills its container.
- `clip_vertical`：Additional option that can be set to have the top and/or bottom edges of the child clipped to its container's bounds. The clip is based on the vertical gravity: a top gravity clips the bottom edge, a bottom gravity clips the top edge, and neither clips both edges.
- `clip_horizontal`：Additional option that can be set to have the left and/or right edges of the child clipped to its container's bounds. The clip is based on the horizontal gravity: a left gravity clips the right edge, a right gravity clips the left edge, and neither clips both edges.

`android:scaleHeight` 和 `android:scaleWidth` 取百分比。相对于 drawable 的 bound。

例子：

```xml
    <scale xmlns:android="http://schemas.android.com/apk/res/android"
        android:drawable="@drawable/logo"
        android:scaleGravity="center_vertical|center_horizontal"
        android:scaleHeight="80%"
        android:scaleWidth="80%" />
```

参见：http://developer.android.com/reference/android/graphics/drawable/ScaleDrawable.html

#### Shape Drawable

This is a generic shape defined in XML.

FILE LOCATION: res/drawable/filename.xml
The filename is used as the resource ID.

Resource pointer to a `GradientDrawable`.

RESOURCE REFERENCE:
In Java: R.drawable.filename
In XML: @[package:]drawable/filename

```xml
    <shape
        xmlns:android="http://schemas.android.com/apk/res/android"
        android:shape=["rectangle" | "oval" | "line" | "ring"] >
        <corners
            android:radius="integer"
            android:topLeftRadius="integer"
            android:topRightRadius="integer"
            android:bottomLeftRadius="integer"
            android:bottomRightRadius="integer" />
        <gradient
            android:angle="integer"
            android:centerX="integer"
            android:centerY="integer"
            android:centerColor="integer"
            android:endColor="color"
            android:gradientRadius="integer"
            android:startColor="color"
            android:type=["linear" | "radial" | "sweep"]
            android:useLevel=["true" | "false"] />
        <padding
            android:left="integer"
            android:top="integer"
            android:right="integer"
            android:bottom="integer" />
        <size
            android:width="integer"
            android:height="integer" />
        <solid
            android:color="color" />
        <stroke
            android:width="integer"
            android:color="color"
            android:dashWidth="integer"
            android:dashGap="integer" />
    </shape>
```

**`<shape>` 元素**

`android:shape` 定义形状。若取 `"line"`，是一条水平线横亘容器宽度。此时需要一个 `<stroke>` 元素定义线的宽度。

下面的特性仅当 `android:shape="ring"` 时有效。

- `android:innerRadius`：中间洞的半径。
- `android:innerRadiusRatio`：浮点数。The radius for the inner part of the ring, expressed as a ratio of the ring's width. For instance, if `android:innerRadiusRatio="5"`, then the inner radius equals the ring's width divided by 5. This value is overridden by `android:innerRadius`. Default value is `9`.
- `android:thickness`：The thickness of the ring, as a dimension value or dimension resource.
- `android:thicknessRatio`：Float. The thickness of the ring, expressed as a ratio of the ring's width. For instance, if `android:thicknessRatio="2"`, then the thickness equals the ring's width divided by 2. This value is overridden by android:innerRadius. Default value is 3.
- `android:useLevel`：Boolean. "true" if this is used as a LevelListDrawable. This should normally be "false" or your shape may not appear.

**`<corners>` 元素**

仅当形状是矩形时可用。创建圆角。

- `android:radius`：Dimension. The radius for all corners, as a dimension value or dimension resource. This is overridden for each corner by the following attributes.
- `android:topLeftRadius`
- `android:topRightRadius`
- `android:bottomLeftRadius`
- `android:bottomRightRadius`

Note: Every corner must (initially) be provided a corner radius greater than 1, or else no corners are rounded.

**`<gradient>` 元素**

Specifies a gradient color for the shape.

- `android:angle`：整数。渐变的角度，单位度。0度水平向右，90度垂直向上。必须是45的倍数。默认0。
- `android:centerX`：浮点。The relative X-position for the center of the gradient (0 - 1.0).
- `android:centerY`：Float. The relative Y-position for the center of the gradient (0 - 1.0).
- `android:centerColor`：Color. Optional color that comes between the start and end colors, as a hexadecimal value or color resource.
- `android:endColor`：Color. The ending color, as a hexadecimal value or color resource.
- `android:gradientRadius`：Float. The radius for the gradient. Only applied when `android:type="radial"`.
- `android:startColor`：Color. The starting color, as a hexadecimal value or color resource.
- `android:type`：Keyword. The type of gradient pattern to apply. Valid values are: `"linear"`、 `"radial"`、`"sweep"`。
- `android:useLevel`：Boolean. "true" if this is used as a LevelListDrawable.

**`<padding>` 元素**

Padding to apply to the containing View element (this pads the position of the View content, not the shape).
attributes:

android:left
android:top
android:right
android:bottom

**`<size>` 元素**

形状的大小。

- `android:height`：形状的高度。
- `android:width`：形状的宽度。

Note: The shape scales to the size of the container View proportionate to the dimensions defined here, by default. When you use the shape in an `ImageView`, you can restrict scaling by setting the `android:scaleType` to `"center"`.

**`<solid>` 元素**

A solid color to fill the shape.

`android:color`：Color. The color to apply to the shape, as a hexadecimal value or color resource.

**`<stroke>` 元素**

A stroke line for the shape.

- `android:width`：Dimension. The thickness of the line, as a dimension value or dimension resource.
- `android:color`：Color. The color of the line, as a hexadecimal value or color resource.
- `android:dashGap`：Dimension. The distance between line dashes, as a dimension value or dimension resource. Only valid if `android:dashWidth` is set.
- `android:dashWidth`：Dimension. The size of each dash line, as a dimension value or dimension resource. Only valid if `android:dashGap` is set.


例子：

XML file saved at res/drawable/gradient_box.xml:

```xml
    <shape xmlns:android="http://schemas.android.com/apk/res/android"
        android:shape="rectangle">
        <gradient
            android:startColor="#FFFF0000"
            android:endColor="#80FF00FF"
            android:angle="45"/>
        <padding android:left="7dp"
            android:top="7dp"
            android:right="7dp"
            android:bottom="7dp" />
        <corners android:radius="8dp" />
    </shape>
```

This layout XML applies the shape drawable to a View:

```xml
    <TextView
        android:background="@drawable/gradient_box"
        android:layout_height="wrap_content"
        android:layout_width="wrap_content" />
```

This application code gets the shape drawable and applies it to a View:

```java
Resources res = getResources();
Drawable shape = res. getDrawable(R.drawable.gradient_box);

TextView tv = (TextView)findViewByID(R.id.textview);
tv.setBackground(shape);
```

参见：http://developer.android.com/reference/android/graphics/drawable/ShapeDrawable.html




