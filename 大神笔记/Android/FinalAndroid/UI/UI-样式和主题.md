[toc]

## 样式

若在布局XML中有一些公共的样式，例如，表单标签都具有下面的样子。可以将它们抽象到 `style.xml` 中。

```
    <resources>
        <style name="form_label">
            <item name="android:layout_width">wrap_content</item>
            <item name="android:layout_height">wrap_content</item>
            <item name="android:gravity">right</item>
            <item name="android:paddingRight">10dp</item>
        </style>
    </resources>
```

于是在布局XML中，可以应用此样式。可以覆盖样式中的特性。注意 `style` 特性没有 `android:` 前缀。

```
    <TextView style="@style/form_label" android:text="@string/proname" >
    </TextView>
```

样式可以继承。如继承系统默认的文本样式：

```
    <style name="GreenText" parent="@android:style/TextAppearance">
        <item name="android:textColor">#00FF00</item>
    </style>
```

如果样式中包含视图不识别的属性，会被忽略。例如 `LinearLayout` 没有 `android:text` 特性。如果指定了会被忽略。

继承有一种简单写法。把父样式名在放在签名，加点分隔，然后是新名字。例如 `RedText.Small` 继承自 `RedText`：

```
    <style name="RedText.Small" >
        <item name="android:textSize">8dip</item>
    </style>
```

### 继承与引用

有时想要引用继承的主题中的**特性**。此时需要用到语法 `?android:attr/`。例如：

```
    <resources>
    <style name="activated" parent="android:Theme.Holo">
        <item name="android:background">?android:attr/activatedBackgroundIndicator</item>
    </style>
    </resources>
```

`activatedBackgroundIndicator`是继承的主题中的一个**特性**。

有时 `style` 本身也是引用。如：

```xml
    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
        android:orientation="vertical"
        android:layout_width="fill_parent"
        android:layout_height="fill_parent" >
        <ProgressBar android:id="@+id/progress"
            style="?android:attr/progressBarStyleHorizontal"
            android:layout_width="fill_parent"
            android:layout_height="wrap_content" />
    </LinearLayout>
```

这里 `style` 特性指向**主题**提供的一个**特性**（`progressBarStyleHorizontal`）。在 Android 源代码中，它是一个 style 资源：`@android:style/Widget.ProgressBar.Horizontal`。Hence, we are saying to Android that we want our `ProgressBar` styled as `@android:style/Widget.ProgressBar.Horizontal`, via the indirection of `?android:attr/progressBarStyleHorizontal`.

This is one place where inheriting a style becomes important. In the first example shown in this section, we inherited from `Theme.Holo`, because we specifically wanted the `activatedBackgroundIndicator` value from `Theme.Holo`. That value might not exist in other styles, or it might not have the value we want.

## 主题

样式针对一个视图生效，不会对视图的孩子生效。要为孩子应用样式，需要为每个孩子设置 `style` 属性。

若想对整个活动或App指定某个样式，设置 `<activity>` 或 `<application>` 的 `android:theme` 特性。

使用主题的例子：

```
    <activity android:name=”.ExampleActivity”
        android:theme=”@android:style/Theme.Holo” >
    </activity>
```

主题可以设定 Activity 的**样式**。例如下面是 `@android:style/Theme.NoTitleBar.Fullscreen` 主题：

```xml
    <!-- Variant of the default (dark) theme that has no title bar and fills the entire screen -->
    <style name="Theme.NoTitleBar.Fullscreen">
      <item name="android:windowFullscreen">true</item>
      <item name="android:windowContentOverlay">@null</item>
    </style>
```

It specifies that the activity should take over the entire screen, removing the status bar on Android 1.x and 2.x devices (`android:windowFullscreen` set to true), and the action bar on Android 3.x and 4.x devices. `android:windowContentOverlay` 设为 `@null` 表示移除*content overlay*。*content overlay*是围绕 Activity 的*Content view*的布局。移出它的效果是移除了*title bar*。

