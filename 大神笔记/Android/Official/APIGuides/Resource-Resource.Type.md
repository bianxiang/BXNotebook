[toc]

## 资源类型

http://developer.android.com/guide/topics/resources/available-resources.html

### 动画资源

http://developer.android.com/guide/topics/resources/animation-resource.html

#### xxx 属性动画

#### （未）视图动画

The view animation framework supports both tween and frame by frame animations, which can both be declared in XML. The following sections describe how to use both methods.

##### 补间动画

An animation defined in XML that performs transitions such as rotating, fading, moving, and stretching on a graphic.

定义在文件：res/anim/filename.xml。The filename will be used as the resource ID.

Resource pointer to an Animation.

在 Java 中通过 `R.anim.filename` 引用。在 XML 中通过 `@[package:]anim/filename` 引用。

语法：

```xml
    <set xmlns:android="http://schemas.android.com/apk/res/android"
        android:interpolator="@[package:]anim/interpolator_resource"
        android:shareInterpolator=["true" | "false"] >
        <alpha
            android:fromAlpha="float"
            android:toAlpha="float" />
        <scale
            android:fromXScale="float"
            android:toXScale="float"
            android:fromYScale="float"
            android:toYScale="float"
            android:pivotX="float"
            android:pivotY="float" />
        <translate
            android:fromXDelta="float"
            android:toXDelta="float"
            android:fromYDelta="float"
            android:toYDelta="float" />
        <rotate
            android:fromDegrees="float"
            android:toDegrees="float"
            android:pivotX="float"
            android:pivotY="float" />
        <set>
            ...
        </set>
    </set>
```

The file must have a single root element: either an `<alpha>`, `<scale>`, `<translate>`, `<rotate>`, or `<set>` element that holds a group (or groups) of other animation elements (even nested `<set>` elements).

### xxx 颜色状态列表

http://developer.android.com/guide/topics/resources/color-list-resource.html

### xxx Drawable 资源

http://developer.android.com/guide/topics/resources/drawable-resource.html

### xxx 布局资源

http://developer.android.com/guide/topics/resources/layout-resource.html

### （未）菜单资源

http://developer.android.com/guide/topics/resources/menu-resource.html

### xxx 字符串资源

http://developer.android.com/guide/topics/resources/string-resource.html

### xxx Style Resource

http://developer.android.com/guide/topics/resources/style-resource.html

### （未）More Resource Types

http://developer.android.com/guide/topics/resources/more-resources.html

This page defines more types of resources you can externalize, including:

- Bool：XML resource that carries a boolean value.
- Color：XML resource that carries a color value (a hexadecimal color).
- Dimension：XML resource that carries a dimension value (with a unit of measure).
- ID：XML resource that provides a unique identifier for application resources and components.
- Integer：XML resource that carries an integer value.
- Integer Array：XML resource that provides an array of integers.
- Typed Array：XML resource that provides a TypedArray (which you can use for an array of drawables).

#### 布尔

A boolean value defined in XML.

Note: A bool is a simple resource that is referenced using the value provided in the name attribute (not the name of the XML file). As such, you can combine bool resources with other simple resources in the one XML file, under one `<resources>` element.

FILE LOCATION: res/values/filename.xml
The filename is arbitrary. The `<bool>` element's name will be used as the resource ID.

RESOURCE REFERENCE:
In Java: R.bool.bool_name
In XML: @[package:]bool/bool_name

```xml
    <resources>
        <bool name="bool_name">[true | false]</bool>
    </resources>
```

例子：

XML file saved at res/values-small/bools.xml:

```xml
    <resources>
        <bool name="screen_small">true</bool>
        <bool name="adjust_view_bounds">true</bool>
    </resources>
```

This application code retrieves the boolean:

```java
Resources res = getResources();
boolean screenIsSmall = res.getBoolean(R.bool.screen_small);
```

This layout XML uses the boolean for an attribute:

```java
    <ImageView
        android:layout_height="fill_parent"
        android:layout_width="fill_parent"
        android:src="@drawable/logo"
        android:adjustViewBounds="@bool/adjust_view_bounds" />
```

#### 颜色

A color value defined in XML. The color is specified with an RGB value and alpha channel. You can use a color resource any place that accepts a hexadecimal color value. You can also use a color resource when a drawable resource is expected in XML (for example, `android:drawable="@color/green"`).

The value always begins with a pound (#) character and then followed by the Alpha-Red-Green-Blue information in one of the following formats:

    #RGB
    #ARGB
    #RRGGBB
    #AARRGGBB

Note: A color is a simple resource that is referenced using the value provided in the name attribute (not the name of the XML file). As such, you can combine color resources with other simple resources in the one XML file, under one `<resources>` element.

FILE LOCATION: res/values/colors.xml
The filename is arbitrary. The `<color>` element's name will be used as the resource ID.

RESOURCE REFERENCE:
In Java: R.color.color_name
In XML: @[package:]color/color_name

语法：

```xml
    <resources>
        <color name="color_name">hex_color</color>
    </resources>
```