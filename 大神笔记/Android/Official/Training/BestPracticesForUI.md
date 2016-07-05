[toc]

## Best Practices for User Interface

## 为多屏设计

### 支持不同的屏幕大小

#### 使用最小宽度限定符

Android 3.2 引入最小宽度限定符。最小宽度限定符筛选出宽度大于指定值的屏幕，单位`dp`。例如典型的 7" 平板的最小宽度为 600 dp，在布局文件名种通过`sw600dp`筛选出这类屏幕：

res/layout-sw600dp/main.xml, two-pane layout:

```xml
    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
        android:layout_width="fill_parent"
        android:layout_height="fill_parent"
        android:orientation="horizontal">
        <fragment android:id="@+id/headlines"
                  android:layout_height="fill_parent"
                  android:name="com.example.android.newsreader.HeadlinesFragment"
                  android:layout_width="400dp"
                  android:layout_marginRight="10dp"/>
        <fragment android:id="@+id/article"
                  android:layout_height="fill_parent"
                  android:name="com.example.android.newsreader.ArticleFragment"
                  android:layout_width="fill_parent" />
    </LinearLayout>
```

#### 布局别名

有时一个布局文件会在多种情况下使用，例如：

res/layout/main.xml: single-pane layout
res/layout-large: multi-pane layout
res/layout-sw600dp: multi-pane layout

为避免重复布局文件，可以使用别名。例如，先定义两个实际的布局文件：

res/layout/main.xml, single-pane layout
res/layout/main_twopanes.xml, two-pane layout

然后添加两个文件：

res/values-large/layout.xml:

```xml
    <resources>
        <item name="main" type="layout">@layout/main_twopanes</item>
    </resources>
```

res/values-sw600dp/layout.xml:

```xml
    <resources>
        <item name="main" type="layout">@layout/main_twopanes</item>
    </resources>
```

#### Use Orientation Qualifiers

Some layouts work well in both landscape and portrait orientations, but most of them can benefit from adjustments. In the News Reader sample app, here is how the layout behaves in each screen size and orientation:

small screen, portrait: single pane, with logo
small screen, landscape: single pane, with logo
7" tablet, portrait: single pane, with action bar
7" tablet, landscape: dual pane, wide, with action bar
10" tablet, portrait: dual pane, narrow, with action bar
10" tablet, landscape: dual pane, wide, with action bar
TV, landscape: dual pane, wide, with action bar

So each of these layouts is defined in an XML file in the res/layout/ directory. To then assign each layout to the various screen configurations, the app uses layout aliases to match them to each configuration:

res/layout/onepane.xml:

    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
        android:orientation="vertical"
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <fragment android:id="@+id/headlines"
                  android:layout_height="fill_parent"
                  android:name="com.example.android.newsreader.HeadlinesFragment"
                  android:layout_width="match_parent" />
    </LinearLayout>

res/layout/onepane_with_bar.xml:

    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
        android:orientation="vertical"
        android:layout_width="match_parent"
        android:layout_height="match_parent">
        <LinearLayout android:layout_width="match_parent"
                      android:id="@+id/linearLayout1"
                      android:gravity="center"
                      android:layout_height="50dp">
            <ImageView android:id="@+id/imageView1"
                       android:layout_height="wrap_content"
                       android:layout_width="wrap_content"
                       android:src="@drawable/logo"
                       android:paddingRight="30dp"
                       android:layout_gravity="left"
                       android:layout_weight="0" />
            <View android:layout_height="wrap_content"
                  android:id="@+id/view1"
                  android:layout_width="wrap_content"
                  android:layout_weight="1" />
            <Button android:id="@+id/categorybutton"
                    android:background="@drawable/button_bg"
                    android:layout_height="match_parent"
                    android:layout_weight="0"
                    android:layout_width="120dp"
                    style="@style/CategoryButtonStyle"/>
        </LinearLayout>

        <fragment android:id="@+id/headlines"
                  android:layout_height="fill_parent"
                  android:name="com.example.android.newsreader.HeadlinesFragment"
                  android:layout_width="match_parent" />
    </LinearLayout>

res/layout/twopanes.xml:

    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
        android:layout_width="fill_parent"
        android:layout_height="fill_parent"
        android:orientation="horizontal">
        <fragment android:id="@+id/headlines"
                  android:layout_height="fill_parent"
                  android:name="com.example.android.newsreader.HeadlinesFragment"
                  android:layout_width="400dp"
                  android:layout_marginRight="10dp"/>
        <fragment android:id="@+id/article"
                  android:layout_height="fill_parent"
                  android:name="com.example.android.newsreader.ArticleFragment"
                  android:layout_width="fill_parent" />
    </LinearLayout>

res/layout/twopanes_narrow.xml:

    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
        android:layout_width="fill_parent"
        android:layout_height="fill_parent"
        android:orientation="horizontal">
        <fragment android:id="@+id/headlines"
                  android:layout_height="fill_parent"
                  android:name="com.example.android.newsreader.HeadlinesFragment"
                  android:layout_width="200dp"
                  android:layout_marginRight="10dp"/>
        <fragment android:id="@+id/article"
                  android:layout_height="fill_parent"
                  android:name="com.example.android.newsreader.ArticleFragment"
                  android:layout_width="fill_parent" />
    </LinearLayout>

Now that all possible layouts are defined, it's just a matter of mapping the correct layout to each configuration using the configuration qualifiers. You can now do it using the layout alias technique:

res/values/layouts.xml:

    <resources>
        <item name="main_layout" type="layout">@layout/onepane_with_bar</item>
        <bool name="has_two_panes">false</bool>
    </resources>

{{后续在Activity中可以通过`getResources().getBoolean(R.bool.has_two_panes)`获取。}}

res/values-sw600dp-land/layouts.xml:

    <resources>
        <item name="main_layout" type="layout">@layout/twopanes</item>
        <bool name="has_two_panes">true</bool>
    </resources>

res/values-sw600dp-port/layouts.xml:

    <resources>
        <item name="main_layout" type="layout">@layout/onepane</item>
        <bool name="has_two_panes">false</bool>
    </resources>

res/values-large-land/layouts.xml:

    <resources>
        <item name="main_layout" type="layout">@layout/twopanes</item>
        <bool name="has_two_panes">true</bool>
    </resources>

res/values-large-port/layouts.xml:

    <resources>
        <item name="main_layout" type="layout">@layout/twopanes_narrow</item>
        <bool name="has_two_panes">true</bool>
    </resources>

### 为不同的屏幕密度设计

To generate these images, you should start with your raw resource in vector format and generate the images for each density using the following size scale:

xhdpi: 2.0
hdpi: 1.5
mdpi: 1.0 (baseline)
ldpi: 0.75

This means that if you generate a 200x200 image for xhdpi devices, you should generate the same resource in 150x150 for hdpi, 100x100 for mdpi and finally a 75x75 image for ldpi devices.

Then, place the generated image files in the appropriate subdirectory under res/ and the system will pick the correct one automatically based on the screen density of the device your application is running on:

    MyProject/
      res/
        drawable-xhdpi/
            awesomeimage.png
        drawable-hdpi/
            awesomeimage.png
        drawable-mdpi/
            awesomeimage.png
        drawable-ldpi/
            awesomeimage.png

Place your launcher icons in the `mipmap/` folders.

    res/...
        mipmap-ldpi/...
            finished_launcher_asset.png
        mipmap-mdpi/...
            finished_launcher_asset.png
        mipmap-hdpi/...
            finished_launcher_asset.png
        mipmap-xhdpi/...
            finished_launcher_asset.png
        mipmap-xxhdpi/...
            finished_launcher_asset.png
        mipmap-xxxhdpi/...
            finished_launcher_asset.png

Note: You should place all launcher icons in the `res/mipmap-[density]/` folders, rather than `drawable/` folders to ensure launcher apps use the best resolution icon. For more information about using the `mipmap` folders, see Managing Projects Overview.

## 管理系统UI

## （未）Creating Custom Views

http://developer.android.com/training/custom-views/index.html

### 创建一个 View 类

http://developer.android.com/training/custom-views/create-view.html

#### 创建 `View` 的子类

继承 `View` 类，或其子类，如 `Button`。

为了 Android Studio，你必须提供一个构造器，取 Context 和 AttributeSet 两个参数。

```java
class PieChart extends View {
    public PieChart(Context context, AttributeSet attrs) {
        super(context, attrs);
    }
}
```

#### 定义定制特性

若要在XML中使用你的定制类，需要在 `<declare-styleable>` 中定义定制的特性。

创建 res/values/attrs.xml 文件：

```xml
<resources>
   <declare-styleable name="PieChart">
       <attr name="showText" format="boolean" />
       <attr name="labelPosition" format="enum">
           <enum name="left" value="0"/>
           <enum name="right" value="1"/>
       </attr>
   </declare-styleable>
</resources>
```

上面代码声明了两个特性，`showText` 和 `labelPosition`, that belong to a styleable entity named `PieChart`. `declare-styleable` 的 `name` 默认取定制视图的类的名字。

在布局XML中使用这些定制特性，需要用另外的命名空间。不能再使用 `http://schemas.android.com/apk/res/android`，要使用 `http://schemas.android.com/apk/res/[your package name]`。例如：

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
	xmlns:custom="http://schemas.android.com/apk/res/com.example.customviews">
	<com.example.customviews.charting.PieChart
		custom:showText="true"
		custom:labelPosition="left" />
</LinearLayout>
```

为避免使用很长的命名空间 URI，利用 `xmlns` 指令，给 `http://schemas.android.com/apk/res/com.example.customviews` 指定别名 `custom`。别名你可以随意定。

定制视图的 XML 标签是定制类的全限类名。如果视图类是内部类，还需要加外面的类： `com.example.customviews.charting.PieChart$PieView`。

#### 使用定制的特性

When a view is created from an XML layout, all of the attributes in the XML tag are read from the resource bundle and passed into the view's constructor as an `AttributeSet`. Although it's possible to read values from the `AttributeSet` directly, doing so has some disadvantages:

- Resource references within attribute values are not resolved
- Styles are not applied

Instead, pass the `AttributeSet` to `obtainStyledAttributes()`. This method passes back a `TypedArray` array of values that have already been dereferenced and styled.

The Android resource compiler does a lot of work for you to make calling `obtainStyledAttributes()` easier. For each `<declare-styleable>` resource in the `res` directory, the generated `R.java` defines both an array of attribute ids and a set of constants that define the index for each attribute in the array. You use the predefined constants to read the attributes from the `TypedArray`. Here's how the `PieChart` class reads its attributes:

```java
public PieChart(Context context, AttributeSet attrs) {
   super(context, attrs);
   TypedArray a = context.getTheme().obtainStyledAttributes(
        attrs,
        R.styleable.PieChart,
        0, 0);

   try {
       mShowText = a.getBoolean(R.styleable.PieChart_showText, false);
       mTextPos = a.getInteger(R.styleable.PieChart_labelPosition, 0);
   } finally {
       a.recycle();
   }
}
```

Note that `TypedArray` objects are a shared resource and must be recycled after use.

#### 添加属性和事件

Attributes are a powerful way of controlling the behavior and appearance of views, but they can only be read when the view is initialized. To provide dynamic behavior, expose a property getter and setter pair for each custom attribute. The following snippet shows how `PieChart` exposes a property called `showText`:

```java
public boolean isShowText() {
   return mShowText;
}

public void setShowText(boolean showText) {
   mShowText = showText;
   invalidate();
   requestLayout();
}
```

Notice that `setShowText` calls `invalidate()` and `requestLayout()`. 若你的更改可能影响界面，需要调用 `invalidate()`，让系统知道可能需要重绘。若更改可能影响视图的大小和形状，需要调用 `requestLayout()`。

Custom views should also support event listeners to communicate important events. For instance, `PieChart` exposes a custom event called `OnCurrentItemChanged` to notify listeners that the user has rotated the pie chart to focus on a new pie slice.

#### Design For Accessibility

Your custom view should support the widest range of users. This includes users with disabilities that prevent them from seeing or using a touchscreen. To support users with disabilities, you should:

- Label your input fields using the `android:contentDescription` attribute
- Send accessibility events by calling `sendAccessibilityEvent()` when appropriate.
- Support alternate controllers, such as D-pad and trackball

For more information on creating accessible views, see [Making Applications Accessible](http://developer.android.com/guide/topics/ui/accessibility/apps.html#custom-viewshttp://developer.android.com/guide/topics/ui/accessibility/apps.html#custom-views) in the Android Developers Guide.

### 定制绘制

http://developer.android.com/intl/zh-cn/training/custom-views/custom-drawing.html

#### 重写 onDraw()

The most important step in drawing a custom view is to override the `onDraw()` method. The parameter to `onDraw()` is a `Canvas` object that the view can use to draw itself. The `Canvas` class defines methods for drawing text, lines, bitmaps, and many other graphics primitives. You can use these methods in `onDraw()` to create your custom user interface (UI).

Before you can call any drawing methods, though, it's necessary to create a `Paint` object. The next section discusses `Paint` in more detail.

#### Create Drawing Objects

The `android.graphics` framework divides drawing into two areas:

- What to draw, handled by `Canvas`
- How to draw, handled by `Paint`.

例如 `Canvas` 提供丰富绘制直线，`Paint` 提供丰富定义线的颜色。

So, before you draw anything, you need to create one or more `Paint` objects. The `PieChart` example does this in a method called `init`, which is called from the constructor:

```java
private void init() {
   mTextPaint = new Paint(Paint.ANTI_ALIAS_FLAG);
   mTextPaint.setColor(mTextColor);
   if (mTextHeight == 0) {
       mTextHeight = mTextPaint.getTextSize();
   } else {
       mTextPaint.setTextSize(mTextHeight);
   }

   mPiePaint = new Paint(Paint.ANTI_ALIAS_FLAG);
   mPiePaint.setStyle(Paint.Style.FILL);
   mPiePaint.setTextSize(mTextHeight);

   mShadowPaint = new Paint(0);
   mShadowPaint.setColor(0xff101010);
   mShadowPaint.setMaskFilter(new BlurMaskFilter(8, BlurMaskFilter.Blur.NORMAL));

   ...
```

预先创建对象是一项重要的优化。Views are redrawn very frequently, and many drawing objects require expensive initialization. Creating drawing objects within your `onDraw()` method significantly reduces performance and can make your UI appear sluggish.

#### 处理布局事件

要正确绘制你的视图，先要知道它的大小。Complex custom views often need to perform multiple layout calculations depending on the size and shape of their area on screen. You should never make assumptions about the size of your view on the screen.

`View` 有很多方法负责测量，其中多数必须被重写。若你的视图并不需要对尺寸的特殊处理，你只需要覆盖一个方法：`onSizeChanged()`。

`onSizeChanged()` is called when your view is first assigned a size, and again if the size of your view changes for any reason. Calculate positions, dimensions, and any other values related to your view's size in `onSizeChanged()`, instead of recalculating them every time you draw. In the PieChart example, `onSizeChanged()` is where the `PieChart` view calculates the bounding rectangle of the pie chart and the relative position of the text label and other visual elements.

When your view is assigned a size, the layout manager assumes that the size includes all of the view's padding. You must handle the padding values when you calculate your view's size. Here's a snippet from `PieChart.onSizeChanged()` that shows how to do this:

```java
   // Account for padding
   float xpad = (float)(getPaddingLeft() + getPaddingRight());
   float ypad = (float)(getPaddingTop() + getPaddingBottom());

   // Account for the label
   if (mShowText) xpad += mTextWidth;

   float ww = (float)w - xpad;
   float hh = (float)h - ypad;

   // Figure out how big we can make the pie.
   float diameter = Math.min(ww, hh);
```
   
If you need finer control over your view's layout parameters, implement `onMeasure()`. This method's parameters are `View.MeasureSpec` values that tell you how big your view's parent wants your view to be, and whether that size is a hard maximum or just a suggestion. As an optimization, these values are stored as packed integers, and you use the static methods of `View.MeasureSpec` to unpack the information stored in each integer.

Here's an example implementation of `onMeasure()`. In this implementation, PieChart attempts to make its area big enough to make the pie as big as its label:

```java
@Override
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
   // Try for a width based on our minimum
   int minw = getPaddingLeft() + getPaddingRight() + getSuggestedMinimumWidth();
   int w = resolveSizeAndState(minw, widthMeasureSpec, 1);

   // Whatever the width ends up being, ask for a height that would let the pie
   // get as big as it can
   int minh = MeasureSpec.getSize(w) - (int)mTextWidth + getPaddingBottom() + getPaddingTop();
   int h = resolveSizeAndState(MeasureSpec.getSize(w) - (int)mTextWidth, heightMeasureSpec, 0);

   setMeasuredDimension(w, h);
}
```

There are three important things to note in this code:

- The calculations take into account the view's padding. As mentioned earlier, this is the view's responsibility.
- The helper method `resolveSizeAndState()` is used to create the final width and height values. This helper returns an appropriate `View.MeasureSpec` value by comparing the view's desired size to the spec passed into `onMeasure()`.
- `onMeasure()` has no return value. Instead, the method communicates its results by calling `setMeasuredDimension()`. Calling this method is mandatory. If you omit this call, the View class throws a runtime exception.

#### Draw!

Once you have your object creation and measuring code defined, you can implement `onDraw()`. Every view implements `onDraw()` differently, but there are some common operations that most views share:

- Draw text using `drawText()`. Specify the typeface by calling `setTypeface()`, and the text color by calling `setColor()`.
- Draw primitive shapes using `drawRect()`, `drawOval()`, and `drawArc()`. Change whether the shapes are filled, outlined, or both by calling `setStyle()`.
- Draw more complex shapes using the `Path` class. Define a shape by adding lines and curves to a `Path` object, then draw the shape using `drawPath()`. Just as with primitive shapes, paths can be outlined, filled, or both, depending on the `setStyle()`.
- Define gradient fills by creating `LinearGradient` objects. Call setShader() to use your LinearGradient on filled shapes.
- Draw bitmaps using `drawBitmap()`.

For example, here's the code that draws `PieChart`. It uses a mix of text, lines, and shapes.

```java
protected void onDraw(Canvas canvas) {
   super.onDraw(canvas);

   // Draw the shadow
   canvas.drawOval(mShadowBounds, mShadowPaint);

   // Draw the label text
   canvas.drawText(mData.get(mCurrentItem).mLabel, mTextX, mTextY, mTextPaint);

   // Draw the pie slices
   for (int i = 0; i < mData.size(); ++i) {
       Item it = mData.get(i);
       mPiePaint.setShader(it.mShader);
       canvas.drawArc(mBounds,
               360 - it.mEndAngle,
               it.mEndAngle - it.mStartAngle,
               true, mPiePaint);
   }

   // Draw the pointer
   canvas.drawLine(mTextX, mPointerY, mPointerX, mPointerY, mTextPaint);
   canvas.drawCircle(mPointerX, mPointerY, mPointerSize, mTextPaint);
}
```



