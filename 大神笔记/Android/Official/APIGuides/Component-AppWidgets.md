[toc]

## App Widgets

http://developer.android.com/guide/topics/appwidgets/index.html

App Widgets are miniature application views that can be embedded in other applications (such as the Home screen) and receive periodic updates. These views are referred to as Widgets in the user interface, and you can publish one with an App Widget provider. An application component that is able to hold other App Widgets is called an **App Widget host**.

This document describes how to publish an App Widget using an App Widget provider.

> For information about how to design your app widget, read the [Widgets](http://developer.android.com/design/patterns/widgets.html) design guide.

### 基础

To create an App Widget, you need the following:

- `AppWidgetProviderInfo` object

Describes the metadata for an App Widget, such as the App Widget's layout, update frequency, and the AppWidgetProvider class. This should be defined in XML.

- `AppWidgetProvider` class implementation

Defines the basic methods that allow you to programmatically interface with the App Widget, based on broadcast events. Through it, you will receive broadcasts when the App Widget is updated, enabled, disabled and deleted.

- View layout

Defines the initial layout for the App Widget, defined in XML.
Additionally, you can implement an **App Widget configuration Activity**. This is an optional Activity that launches when the user adds your App Widget and allows him or her to modify App Widget settings at create-time.

### 在 Manifest 中声明一个 App Widget

First, declare the `AppWidgetProvider` class in your application's AndroidManifest.xml file. For example:

```
    <receiver android:name="ExampleAppWidgetProvider" >
        <intent-filter>
            <action android:name="android.appwidget.action.APPWIDGET_UPDATE" />
        </intent-filter>
        <meta-data android:name="android.appwidget.provider"
                   android:resource="@xml/example_appwidget_info" />
    </receiver>
```

The `<receiver>` element requires the `android:name` attribute, which specifies the `AppWidgetProvider` used by the App Widget.

The `<intent-filter>` element must include an `<action>` element with the `android:name` attribute. This attribute specifies that the `AppWidgetProvider` accepts the `ACTION_APPWIDGET_UPDATE` broadcast. This is the only broadcast that you must explicitly declare. The `AppWidgetManager` automatically sends all other App Widget broadcasts to the `AppWidgetProvider` as necessary.

The `<meta-data>` element specifies the `AppWidgetProviderInfo` resource and requires the following attributes:

- `android:name` - Specifies the metadata name. Use `android.appwidget.provider` to identify the data as the `AppWidgetProviderInfo` descriptor.
- `android:resource` - Specifies the `AppWidgetProviderInfo` resource location.

### 添加 AppWidgetProviderInfo 元数据

The `AppWidgetProviderInfo` defines the essential qualities of an App Widget, such as its minimum layout dimensions, its initial layout resource, how often to update the App Widget, and (optionally) a configuration Activity to launch at create-time. Define the `AppWidgetProviderInfo` object in an XML resource using a single `<appwidget-provider>` element and save it in the project's `res/xml/` folder.

For example:

```
    <appwidget-provider xmlns:android="http://schemas.android.com/apk/res/android"
        android:minWidth="40dp"
        android:minHeight="40dp"
        android:updatePeriodMillis="86400000"
        android:previewImage="@drawable/preview"
        android:initialLayout="@layout/example_appwidget"
        android:configure="com.example.android.ExampleAppWidgetConfigure" 
        android:resizeMode="horizontal|vertical"
        android:widgetCategory="home_screen">
    </appwidget-provider>
```

Here's a summary of the `<appwidget-provider>` attributes:

The values for the `minWidth` and `minHeight` attributes specify the minimum amount of space the App Widget consumes by default. The default Home screen positions App Widgets in its window based on **a grid of cells** that have a **defined** height and width. If the values for an App Widget's minimum width or height don't match the dimensions of the cells, then the App Widget dimensions round up to the nearest cell size.
See the [App Widget Design Guidelines](http://developer.android.com/guide/practices/ui_guidelines/widget_design.html#anatomy_determining_size) for more information on sizing your App Widgets.

> 若想让 app widget 适配更多设备，app widget 的最小大小不要超过 4 x 4 单元格。

The `minResizeWidth` and `minResizeHeight` attributes specify the App Widget's absolute minimum size. These values should specify the size below which the App Widget would be illegible or otherwise unusable. Using these attributes allows the user to resize the widget to a size that may be smaller than the default widget size defined by the `minWidth` and `minHeight` attributes. Introduced in Android 3.1.

The `updatePeriodMillis` attribute defines how often the App Widget framework should request an update from the `AppWidgetProvider` by calling the `onUpdate()` callback method. The actual update is not guaranteed to occur exactly on time with this value and we suggest updating as infrequently as possible—perhaps no more than once an hour to conserve the battery. You might also allow the user to adjust the frequency in a configuration—some people might want a stock ticker to update every 15 minutes, or maybe only four times a day.

Note: If the device is asleep when it is time for an update (as defined by updatePeriodMillis), then the device will wake up in order to perform the update. If you don't update more than once per hour, this probably won't cause significant problems for the battery life. If, however, you need to update more frequently and/or you do not need to update while the device is asleep, then you can instead perform updates based on an alarm that will not wake the device. To do so, set an alarm with an Intent that your `AppWidgetProvider` receives, using the `AlarmManager`. Set the alarm type to either `ELAPSED_REALTIME` or `RTC`, which will only deliver the alarm when the device is awake. Then set updatePeriodMillis to zero ("0").

The `initialLayout` attribute points to the layout resource that defines the App Widget layout.

The `configure` attribute defines the Activity to launch when the user adds the App Widget, in order for him or her to configure App Widget properties. This is optional.

The `previewImage` attribute specifies a preview of what the app widget will look like after it's configured, which the user sees when selecting the app widget. If not supplied, the user instead sees your application's launcher icon. This field corresponds to the `android:previewImage` attribute in the `<receiver>` element in the AndroidManifest.xml file.

The `autoAdvanceViewId` attribute specifies the view ID of the app widget subview that should be auto-advanced by the widget's host. Introduced in Android 3.0.

The `resizeMode` attribute specifies the rules by which a widget can be resized. You use this attribute to make homescreen widgets resizeable—horizontally, vertically, or on both axes. Users touch-hold a widget to show its resize handles, then drag the horizontal and/or vertical handles to change the size on the layout grid. Values for the resizeMode attribute include "horizontal", "vertical", and "none". To declare a widget as resizeable horizontally and vertically, supply the value "horizontal|vertical". Introduced in Android 3.1.

The `minResizeHeight` attribute specifies the minimum height (in dps) to which the widget can be resized. This field has no effect if it is greater than `minHeight` or if vertical resizing isn't enabled (see resizeMode). Introduced in Android 4.0.

The `minResizeWidth` attribute specifies the minimum width (in dps) to which the widget can be resized. This field has no effect if it is greater than minWidth or if horizontal resizing isn't enabled (see resizeMode). Introduced in Android 4.0.

The `widgetCategory` attribute declares whether your App Widget can be displayed on the home screen (`home_screen`), the lock screen (`keyguard`), or both. Only Android versions lower than 5.0 support lock-screen widgets. For Android 5.0 and higher, only `home_screen` is valid.

See the `AppWidgetProviderInfo` class for more information on the attributes accepted by the `<appwidget-provider>` element.

### 创建 App Widget 的布局

You must define an initial layout for your App Widget in XML and save it in the project's `res/layout/` directory. You can design your App Widget using the View objects listed below, but before you begin designing your App Widget, please read and understand the [App Widget Design Guidelines](http://developer.android.com/guide/practices/ui_guidelines/widget_design.html).

App Widget 的布局基于 `RemoteViews`，因此不是所有布局或视图都支持。

A `RemoteViews` object (and, consequently, an App Widget) can support the following layout classes:

FrameLayout
LinearLayout
RelativeLayout
GridLayout

And the following widget classes:

AnalogClock
Button
Chronometer
ImageButton
ImageView
ProgressBar
TextView
ViewFlipper
ListView
GridView
StackView
AdapterViewFlipper

这些类的子类不被支持！

`RemoteViews` also supports `ViewStub`, which is an invisible, zero-sized View you can use to lazily inflate layout resources at runtime.

#### 向 App Widgets 添加外边距

Widgets should not generally extend to screen edges and should not visually be flush with other widgets, so you should add margins on all sides around your widget frame.

As of Android 4.0, app widgets are automatically given padding between the widget frame and the app widget's bounding box to provide better alignment with other widgets and icons on the user's home screen. To take advantage of this strongly recommended behavior, set your application's `targetSdkVersion` to 14 or greater.

{{Android 4.0 以下如何实现略，见官方文档。}}

### 使用 AppWidgetProvider 类

You must declare your `AppWidgetProvider` class implementation as a broadcast receiver using the `<receiver>` element in the AndroidManifest (see Declaring an App Widget in the Manifest above).

The `AppWidgetProvider` class extends `BroadcastReceiver` as a convenience class to handle the App Widget broadcasts. The `AppWidgetProvider` receives only the event broadcasts that are relevant to the App Widget, such as when the App Widget is updated, deleted, enabled, and disabled. When these broadcast events occur, the `AppWidgetProvider` receives the following method calls:

`onUpdate()`
This is called to update the App Widget at intervals defined by the updatePeriodMillis attribute in the AppWidgetProviderInfo. This method is also called when the user adds the App Widget, so it should perform the essential setup, such as define event handlers for Views and start a temporary Service, if necessary. However, if you have declared a configuration Activity, this method is not called when the user adds the App Widget, but is called for the subsequent updates. It is the responsibility of the configuration Activity to perform the first update when configuration is done. (See Creating an App Widget Configuration Activity below.)

`onAppWidgetOptionsChanged()`
This is called when the widget is first placed and any time the widget is resized. You can use this callback to show or hide content based on the widget's size ranges. You get the size ranges by calling `getAppWidgetOptions()`, which returns a Bundle that includes the following:

`OPTION_APPWIDGET_MIN_WIDTH`—Contains the lower bound on the current width, in dp units, of a widget instance.
`OPTION_APPWIDGET_MIN_HEIGHT`—Contains the lower bound on the current height, in dp units, of a widget instance.
`OPTION_APPWIDGET_MAX_WIDTH`—Contains the upper bound on the current width, in dp units, of a widget instance.
`OPTION_APPWIDGET_MAX_HEIGHT`—Contains the upper bound on the current width, in dp units, of a widget instance.

This callback was introduced in API Level 16 (Android 4.1). If you implement this callback, make sure that your app doesn't depend on it since it won't be called on older devices.

`onDeleted(Context, int[])`
This is called every time an App Widget is deleted from the App Widget host.

`onEnabled(Context)`
This is called when an instance the App Widget is created for the first time. For example, if the user adds two instances of your App Widget, this is only called the first time. If you need to open a new database or perform other setup that only needs to occur once for all App Widget instances, then this is a good place to do it.

`onDisabled(Context)`
This is called when the last instance of your App Widget is deleted from the App Widget host. This is where you should clean up any work done in `onEnabled(Context)`, such as delete a temporary database.

`onReceive(Context, Intent)`
This is called for every broadcast and before each of the above callback methods. You normally don't need to implement this method because the default AppWidgetProvider implementation filters all App Widget broadcasts and calls the above methods as appropriate.

The most important `AppWidgetProvider` callback is `onUpdate()` because it is called when each App Widget is added to a host (unless you use a configuration Activity). If your App Widget accepts any user interaction events, then you need to register the event handlers in this callback. If your App Widget doesn't create temporary files or databases, or perform other work that requires clean-up, then onUpdate() may be the only callback method you need to define. For example, if you want an App Widget with a button that launches an Activity when clicked, you could use the following implementation of `AppWidgetProvider`:

```java
public class ExampleAppWidgetProvider extends AppWidgetProvider {

    public void onUpdate(Context context, AppWidgetManager appWidgetManager, int[] appWidgetIds) {
        final int N = appWidgetIds.length;

        // Perform this loop procedure for each App Widget that belongs to this provider
        for (int i=0; i<N; i++) {
            int appWidgetId = appWidgetIds[i];

            // Create an Intent to launch ExampleActivity
            Intent intent = new Intent(context, ExampleActivity.class);
            PendingIntent pendingIntent = PendingIntent.getActivity(context, 0, intent, 0);

            // Get the layout for the App Widget and attach an on-click listener
            // to the button
            RemoteViews views = new RemoteViews(context.getPackageName(), R.layout.appwidget_provider_layout);
            views.setOnClickPendingIntent(R.id.button, pendingIntent);

            // Tell the AppWidgetManager to perform an update on the current app widget
            appWidgetManager.updateAppWidget(appWidgetId, views);
        }
    }
}
```

Notice that it includes a loop that iterates through each entry in appWidgetIds, which is an array of IDs that identify each App Widget created by this provider. In this way, if the user creates more than one instance of the App Widget, then they are all updated simultaneously. However, only one updatePeriodMillis schedule will be managed for all instances of the App Widget. For example, if the update schedule is defined to be every two hours, and a second instance of the App Widget is added one hour after the first one, then they will both be updated on the period defined by the first one and the second update period will be ignored (they'll both be updated every two hours, not every hour).

Note: Because `AppWidgetProvider` is an extension of `BroadcastReceiver`, your process is not guaranteed to keep running after the callback methods return (see BroadcastReceiver for information about the broadcast lifecycle). If your App Widget setup process can take several seconds (perhaps while performing web requests) and you require that your process continues, consider starting a Service in the onUpdate() method. From within the Service, you can perform your own updates to the App Widget without worrying about the AppWidgetProvider closing down due to an Application Not Responding (ANR) error. See the [Wiktionary sample's AppWidgetProvider](http://code.google.com/p/wiktionary-android/source/browse/trunk/Wiktionary/src/com/example/android/wiktionary/WordWidget.java) for an example of an App Widget running a Service.

Also see the [ExampleAppWidgetProvider.java](http://developer.android.com/resources/samples/ApiDemos/src/com/example/android/apis/appwidget/ExampleAppWidgetProvider.html) sample class.

#### Receiving App Widget broadcast Intents

`AppWidgetProvider` is just a convenience class. If you would like to receive the App Widget broadcasts directly, you can implement your own `BroadcastReceiver` or override the `onReceive(Context, Intent)` callback. The Intents you need to care about are as follows:

`ACTION_APPWIDGET_UPDATE`
`ACTION_APPWIDGET_DELETED`
`ACTION_APPWIDGET_ENABLED`
`ACTION_APPWIDGET_DISABLED`
`ACTION_APPWIDGET_OPTIONS_CHANGED`

### Creating an App Widget Configuration Activity



