# 样式和主题

一个样式（style）是一组属性的集合。样式中可以包含的属性有height, padding, font color, font size, background color等。样式定义在XML中。

例如，可以将以下布局：

	<TextView
	    android:layout_width="fill_parent"
	    android:layout_height="wrap_content"
	    android:textColor="#00FF00"
	    android:typeface="monospace"
	    android:text="@string/hello" />

转化成：

	<TextView
	    style="@style/CodeFont"
	    android:text="@string/hello" />

All of the attributes related to style have been removed from the layout XML and put into a style definition called CodeFont, which is then applied with the style attribute. You'll see the definition for this style in the following section.

主题是应用到整个Activity或应用的样式。When a style is applied as a theme, every View in the Activity or application will apply each style property that it supports. For example, you can apply the same CodeFont style as a theme for an Activity and 活动中素有的文本都将是green monospace。

## 定义样式

样式文件放在`res/values/`，文件名任意。根节点必须是`<resources>`。

一个样式是一个`<style>`元素，`name`特性指定样式名。每个属性是一个`<item>`。

	<?xml version="1.0" encoding="utf-8"?>
	<resources>
	    <style name="CodeFont" parent="@android:style/TextAppearance.Medium">
	        <item name="android:layout_width">fill_parent</item>
	        <item name="android:layout_height">wrap_content</item>
	        <item name="android:textColor">#00FF00</item>
	        <item name="android:typeface">monospace</item>
	    </style>
	</resources>

Each child of the <resources> element is converted into an application resource object at compile-time, which can be referenced by the value in the `<style>` element's `name` attribute. This example style can be referenced from an XML layout as `@style/CodeFont`。

`<style>`的`parent`特性是可选的。可以覆盖继承的样式。

Remember, a style that you want to use as an Activity or application theme is defined in XML exactly the same as a style for a View. 上面定义的样式可以应用于单个View（作为样式）或应用于整个Activity、应用（作为主题）。How to apply a style for a single View or as an application theme is discussed later.

### 继承

The `parent` attribute in the `<style>` element lets you specify a style from which your style should inherit properties. For example, you can inherit the Android platform's default text appearance and then modify it:

	<style name="GreenText" parent="@android:style/TextAppearance">
		<item name="android:textColor">#00FF00</item>
	</style>

如果想要继承你自己定义的样式，不一定要用`parent`特性。Instead, just prefix the name of the style you want to inherit to the name of your new style, separated by a period. For example, to create a new style that inherits the CodeFont style defined above, but make the color red, you can author the new style like this:

    <style name="CodeFont.Red">
        <item name="android:textColor">#FF0000</item>
    </style>

可以引用`@style/CodeFont.Red`。

可以继续继承，如：

    <style name="CodeFont.Red.Big">
        <item name="android:textSize">30sp</item>
    </style>

> Note: This technique for inheritance by chaining together names only works for styles defined by your own resources. You can't inherit Android built-in styles this way. To reference a built-in style, such as TextAppearance, you must use the `parent` attribute.

### 样式属性

要寻找某个View可以使用的属性，查询类文件，里面列出了所有支持的XML特性。For example, all of the attributes listed in the table of TextView XML attributes can be used in a style definition for a TextView element (or one of its subclasses). One of the attributes listed in the reference is android:inputType, so where you might normally place the android:inputType attribute in an <EditText> element, like this:

	<EditText
	    android:inputType="number"
	    ... />

You can instead create a style for the EditText element that includes this property:

	<style name="Numbers">
	  <item name="android:inputType">number</item>
	  ...
	</style>

So your XML for the layout can now implement this style:

	<EditText
	    style="@style/Numbers"
	    ... />

For a reference of all available style properties, see the [R.attr](http://developer.android.com/reference/android/R.attr.htmlhttp://developer.android.com/reference/android/R.attr.html) reference. Keep in mind that all View objects don't accept all the same style attributes, so you should normally refer to the specific View class for supported style properties. However, if you apply a style to a View that does not support all of the style properties, the View will apply only those properties that are supported and simply ignore the others.

一些样式属性，不会任何View支持，只能作用域主题。这些样式属性作用域整个窗口，但不是任何View。例如作用域应用的样式属性可以隐藏标题、状态条、改变窗口背景。hese theme-only style properties, look at the R.attr reference for attributes that begin with `window`. For instance, `windowNoTitle` and `windowBackground` are style properties that are effective only when the style is applied as a theme to an Activity or application. See the next section for information about applying a style as a theme.

> Note: Don't forget to prefix the property names in each `<item>` element with the android: namespace. For example: `<item name="android:inputType">`.

## 应用样式和主题到UI

设置一个样式有两种方式：

- 到单个View，项View的XML添加`style`特性
- 到真个活动或应用，添加`android:theme`特性到`<activity>`或`<application>`。

如果一个样式作用域一个ViewGroup，子View不会继承样式属性，样式只影响直接作用的元素。However, you can apply a style so that it applies to all View elements—by applying the style as a theme.

To apply a style definition as a theme, you must apply the style to an Activity or application in the Android manifest. 此时，活动或应用中的每个View将应用它们支持的属性。不支持的属性将被忽略。

### Apply a theme to an Activity or application

向整个应用应用主题：

	<application android:theme="@style/CustomTheme">

如果只想作用域单个活动，只需将`android:theme`设置到`<activity>`。

利用内建的Dialog主题可以使得Activity显得像一个对话框：

	<activity android:theme="@android:style/Theme.Dialog">

如果想让背景透明，就

	<activity android:theme="@android:style/Theme.Translucent">

可以扩展系统主题。例如，改变light主题的配色：

	<color name="custom_theme_color">#b0b0ff</color>

	<style name="CustomTheme" parent="android:Theme.Light">
	    <item name="android:windowBackground">@color/custom_theme_color</item>
	    <item name="android:colorBackground">@color/custom_theme_color</item>
	</style>

(Note that the color needs to supplied as a separate resource here because the android:windowBackground attribute only supports a reference to another resource; unlike android:colorBackground, it can not be given a color literal.)

使用定制的`CustomTheme`主题：

	<activity android:theme="@style/CustomTheme">

根据平台版本选择主题

新版本的Android会带来新的主题，如果想使用它们同时又兼容旧的平台，You can accomplish this through a custom theme that uses resource selection to switch between different parent themes, based on the platform version.

For example, here is the declaration for a custom theme which is simply the standard platforms default light theme. It would go in an XML file under `res/values` (typically `res/values/styles.xml`):

	<style name="LightThemeSelector" parent="android:Theme.Light">
	    ...
	</style>

To have this theme use the newer holographic theme when the application is running on Android 3.0 (API Level 11) or higher, you can place an alternative declaration for the theme in an XML file in `res/values-v11`, but make the parent theme the holographic theme:

	<style name="LightThemeSelector" parent="android:Theme.Holo.Light">
	    ...
	</style>

Now use this theme like you would any other, and your application will automatically switch to the holographic theme if running on Android 3.0 or higher.

A list of the standard attributes that you can use in themes can be found at [R.styleable.Theme](http://developer.android.com/reference/android/R.styleable.html#Theme).

## 使用平台样式和主题

The Android platform provides a large collection of styles and themes that you can use in your applications. You can find a reference of all available styles in the [R.style](http://developer.android.com/reference/android/R.style.html) class. To use the styles listed here, replace all underscores in the style name with a period. For example, you can apply the `Theme_NoTitleBar` theme with "`@android:style/Theme.NoTitleBar`".

The `R.style` reference, however, is not well documented and does not thoroughly describe the styles, so viewing the actual source code for these styles and themes will give you a better understanding of what style properties each one provides. For a better reference to the Android styles and themes, see the following source code:

- [Android Styles (styles.xml)](https://android.googlesource.com/platform/frameworks/base/+/refs/heads/master/core/res/res/values/styles.xml)
- [Android Themes (themes.xml)](https://android.googlesource.com/platform/frameworks/base/+/refs/heads/master/core/res/res/values/themes.xml)

These files will help you learn through example. For instance, in the Android themes source code, you'll find a declaration for `<style name="Theme.Dialog">`. In this definition, you'll see all of the properties that are used to style dialogs that are used by the Android framework.

For more information about the syntax for styles and themes in XML, see the Style Resource document.