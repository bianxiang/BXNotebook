[toc]

2015年11月11日

## Intent 和 Intent Filter

http://developer.android.com/guide/components/intents-filters.html

An Intent is a messaging object you can use to request an action from another app component. Although intents facilitate communication between components in several ways, 一些基础的使用场景：

- 启动一个互动：You can start a new instance of an Activity by passing an Intent to `startActivity()`. If you want to receive a result from the activity when it finishes, call `startActivityForResult()`. Your activity receives the result as a separate Intent object in your activity's `onActivityResult()` callback.
- 启动一个服务：A Service is a component that performs operations in the background without a user interface. You can start a service to perform a one-time operation (such as download a file) by passing an Intent to `startService()`. The Intent describes the service to start and carries any necessary data. If the service is designed with a client-server interface, you can bind to the service from another component by passing an Intent to `bindService()`.
- To deliver a broadcast: A broadcast is a message that any app can receive. The system delivers various broadcasts for system events, such as when the system boots up or the device starts charging. You can deliver a broadcast to other apps by passing an Intent to `sendBroadcast()`, `sendOrderedBroadcast()`, or `sendStickyBroadcast()`.

### Intent 类型

有两种 Intent 类型：

- 显式 intents 指定要启动的租金的名字（全限类名）。You'll typically use an explicit intent to start a component in your own app, because you know the class name of the activity or service you want to start. For example, start a new activity in response to a user action or start a service to download a file in the background.
- Implicit intents do not name a specific component, but instead declare a general action to perform, which allows a component from another app to handle it. For example, if you want to show the user a location on a map, you can use an implicit intent to request that another capable app show a specified location on a map.

When you create an explicit intent to start an activity or service, the system immediately starts the app component specified in the Intent object.

When you create an implicit intent, the Android system finds the appropriate component to start by comparing the contents of the intent to the intent filters declared in the manifest file of other apps on the device. If the intent matches an intent filter, the system starts that component and delivers it the Intent object. If multiple intent filters are compatible, the system displays a dialog so the user can pick which app to use.

An intent filter is an expression in an app's manifest file that specifies the type of intents that the component would like to receive. For instance, by declaring an intent filter for an activity, you make it possible for other apps to directly start your activity with a certain kind of intent. Likewise, if you do not declare any intent filters for an activity, then it can be started only with an explicit intent.

Caution: To ensure your app is secure, always use an explicit intent when starting a Service and do not declare intent filters for your services. 利用隐式 intent 启动服务是一个安全陷阱，因为你不知道哪个服务会实际响应你的 intent，而且用户无法看到启动的是哪个服务。Beginning with Android 5.0 (API level 21), the system throws an exception if you call bindService() with an implicit intent.

### 构建一个 Intent

An Intent object carries information that the Android system uses to determine which component to start (such as the exact component name or component category that should receive the intent), plus information that the recipient component uses in order to properly perform the action (such as the action to take and the data to act upon).

The primary information contained in an Intent is the following:

**组件名**

The name of the component to start.
This is optional, but it's the critical piece of information that makes an intent explicit, meaning that the intent should be delivered only to the app component defined by the component name. Without a component name, the intent is implicit and the system decides which component should receive the intent based on the other intent information (such as the action, data, and category—described below). So if you need to start a specific component in your app, you should specify the component name.

This field of the Intent is a `ComponentName` object, which you can specify using a fully qualified class name of the target component, including the package name of the app. For example, `com.example.ExampleActivity`. You can set the component name with `setComponent()`, `setClass()`, `setClassName()`, or with the Intent constructor.

**Action**
A string that specifies the generic action to perform (such as view or pick).
In the case of a broadcast intent, this is the action that took place and is being reported. The action largely determines how the rest of the intent is structured—particularly what is contained in the data and extras.

You can specify your own actions for use by intents within your app (or for use by other apps to invoke components in your app), but you should usually use action constants defined by the Intent class or other framework classes. Here are some common actions for starting an activity:

- `ACTION_VIEW`：Use this action in an intent with `startActivity()` when you have some information that an activity can show to the user, such as a photo to view in a gallery app, or an address to view in a map app.
- `ACTION_SEND`：Also known as the "share" intent, you should use this in an intent with `startActivity()` when you have some data that the user can share through another app, such as an email app or social sharing app.

See the Intent class reference for more constants that define generic actions. Other actions are defined elsewhere in the Android framework, such as in Settings for actions that open specific screens in the system's Settings app.

You can specify the action for an intent with `setAction()` or with an Intent constructor.

你自定义的 action，要包含你的应用的包名做前缀：

```java
static final String ACTION_TIMETRAVEL = "com.example.action.TIMETRAVEL";
```

**Data**
The URI (a Uri object) that references the data to be acted on and/or the MIME type of that data. The type of data supplied is generally dictated by the intent's action. For example, if the action is `ACTION_EDIT`, the data should contain the URI of the document to edit.

When creating an intent, it's often important to specify the type of data (its MIME type) in addition to its URI. For example, an activity that's able to display images probably won't be able to play an audio file, even though the URI formats could be similar. So specifying the MIME type of your data helps the Android system find the best component to receive your intent. However, the MIME type can sometimes be inferred from the URI—particularly when the data is a `content:` URI, which indicates the data is located on the device and controlled by a **ContentProvider**, which makes the data MIME type visible to the system.

To set only the data URI, call `setData()`. To set only the MIME type, call `setType()`. If necessary, you can set both explicitly with `setDataAndType()`.

Caution: If you want to set both the URI and MIME type, do not call setData() and setType() because they each nullify the value of the other. Always use setDataAndType() to set both URI and MIME type.

**Category**
A string containing additional information about the kind of component that should handle the intent. Any number of category descriptions can be placed in an intent, but most intents do not require a category. Here are some common categories:

- `CATEGORY_BROWSABLE`：The target activity allows itself to be started by a web browser to display data referenced by a link—such as an image or an e-mail message.
- `CATEGORY_LAUNCHER`：The activity is the initial activity of a task and is listed in the system's application launcher.

See the Intent class description for the full list of categories.

You can specify a category with `addCategory()`.

These properties listed above (component name, action, data, and category) represent the defining characteristics of an intent. By reading these properties, the Android system is able to resolve which app component it should start.

However, an intent can carry additional information that does not affect how it is resolved to an app component. An intent can also supply:

**Extras**
Key-value pairs that carry additional information required to accomplish the requested action. Just as some actions use particular kinds of data URIs, some actions also use particular extras.

You can add extra data with various `putExtra()` methods, each accepting two parameters: the key name and the value. You can also create a `Bundle` object with all the extra data, then insert the Bundle in the Intent with `putExtras()`.

For example, when creating an intent to send an email with `ACTION_SEND`, you can specify the "to" recipient with the `EXTRA_EMAIL` key, and specify the "subject" with the `EXTRA_SUBJECT` key.

The Intent class specifies many `EXTRA_*` constants for standardized data types. If you need to declare your own extra keys (for intents that your app receives), be sure to include your app's package name as a prefix. For example:

```java
static final String EXTRA_GIGAWATTS = "com.example.EXTRA_GIGAWATTS";
```

**Flags**
Flags defined in the Intent class that function as metadata for the intent. The flags may instruct the Android system how to launch an activity (for example, which task the activity should belong to) and how to treat it after it's launched (for example, whether it belongs in the list of recent activities).
For more information, see the `setFlags()` method.

#### 显式 intent 的例子

An explicit intent is one that you use to launch a specific app component, such as a particular activity or service in your app. To create an explicit intent, define the component name for the `Intent` object—all other intent properties are optional.

For example, if you built a service in your app, named `DownloadService`, designed to download a file from the web, you can start it with the following code:

```java
// Executed in an Activity, so 'this' is the Context
// The fileUrl is a string URL, such as "http://www.example.com/image.png"
Intent downloadIntent = new Intent(this, DownloadService.class);
downloadIntent.setData(Uri.parse(fileUrl));
startService(downloadIntent);
```

The `Intent(Context, Class)` constructor supplies the app Context and the component a Class object. As such, this intent explicitly starts the `DownloadService` class in the app.

For more information about building and starting a service, see the Services guide.

#### 隐式 intent 的例子

An implicit intent specifies an action that can invoke any app on the device able to perform the action. Using an implicit intent is useful when your app cannot perform the action, but other apps probably can and you'd like the user to pick which app to use.

For example, if you have content you want the user to share with other people, create an intent with the `ACTION_SEND` action and add extras that specify the content to share. When you call `startActivity()` with that intent, the user can pick an app through which to share the content.

Caution: It's possible that a user won't have any apps that handle the implicit intent you send to `startActivity()`. If that happens, the call will fail and your app will crash. To verify that an activity will receive the intent, call `resolveActivity()` on your Intent object. If the result is non-null, then there is at least one app that can handle the intent and it's safe to call `startActivity()`. If the result is null, you should not use the intent and, if possible, you should disable the feature that issues the intent.

```java
// Create the text message with a string
Intent sendIntent = new Intent();
sendIntent.setAction(Intent.ACTION_SEND);
sendIntent.putExtra(Intent.EXTRA_TEXT, textMessage);
sendIntent.setType("text/plain");

// Verify that the intent will resolve to an activity
if (sendIntent.resolveActivity(getPackageManager()) != null) {
    startActivity(sendIntent);
}
```

Note: In this case, a URI is not used, but the intent's data type is declared to specify the content carried by the extras.

When `startActivity()` is called, the system examines all of the installed apps to determine which ones can handle this kind of intent (an intent with the `ACTION_SEND` action and that carries "text/plain" data). If there's only one app that can handle it, that app opens immediately and is given the intent. If multiple activities accept the intent, the system displays a dialog so the user can pick which app to use.

#### Forcing an app chooser

When there is more than one app that responds to your implicit intent, the user can select which app to use and make that app the default choice for the action. This is nice when performing an action for which the user probably wants to use the same app from now on, such as when opening a web page (users often prefer just one web browser) .

However, if multiple apps can respond to the intent and the user might want to use a different app each time, you should explicitly show a chooser dialog. The chooser dialog asks the user to select which app to use for the action every time (the user cannot select a default app for the action). For example, when your app performs "share" with the `ACTION_SEND` action, users may want to share using a different app depending on their current situation, so you should always use the chooser dialog, as shown in figure 2.

To show the chooser, create an Intent using `createChooser()` and pass it to `startActivity()`. For example:

```java
Intent sendIntent = new Intent(Intent.ACTION_SEND);
...
// Always use string resources for UI text.
// This says something like "Share this photo with"
String title = getResources().getString(R.string.chooser_title);
// Create intent to show the chooser dialog
Intent chooser = Intent.createChooser(sendIntent, title);

// Verify the original intent will resolve to at least one activity
if (sendIntent.resolveActivity(getPackageManager()) != null) {
    startActivity(chooser);
}
```

This displays a dialog with a list of apps that respond to the intent passed to the createChooser() method and uses the supplied text as the dialog title.

### （未）Receiving an Implicit Intent

### 使用 Pending Intent

一个 PendingIntent 对象是一个 Intent 对象的包装。PendingIntent 的主要目的是授权外部应用使用它携带的 Intent，as if it were executed from your app's own process.

pending intent 的主要用例：

- Declare an intent to be executed when the user performs an action with your Notification (the Android system's NotificationManager executes the Intent).
- Declare an intent to be executed when the user performs an action with your App Widget (the Home screen app executes the Intent).
- Declare an intent to be executed at a specified time in the future (the Android system's AlarmManager executes the Intent).

Because each Intent object is designed to be handled by a specific type of app component (either an Activity, a Service, or a BroadcastReceiver), so too must a PendingIntent be created with the same consideration. When using a pending intent, your app will not execute the intent with a call such as startActivity(). You must instead declare the intended component type when you create the `PendingIntent` by calling the respective creator method:

- `PendingIntent.getActivity()` for an Intent that starts an Activity.
- `PendingIntent.getService()` for an Intent that starts a Service.
- `PendingIntent.getBroadcast()` for a Intent that starts an BroadcastReceiver.

Unless your app is receiving pending intents from other apps, the above methods to create a `PendingIntent` are the only PendingIntent methods you'll probably ever need.

Each method takes the current app Context, the Intent you want to wrap, and one or more flags that specify how the intent should be used (such as whether the intent can be used more than once).

More information about using pending intents is provided with the documentation for each of the respective use cases, such as in the **Notifications** and **App Widgets** API guides.

### （未）Intent Resolution

### （未）Common Intents