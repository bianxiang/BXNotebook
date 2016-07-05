[toc]

## Building Apps for TV

### 构建 TV 应用

**Dependencies and Prerequisites: Android 5.0 (API level 21) or higher**

Android offers a rich user experience that's optimized for apps running on large screen devices.

TV apps use the same structure as those for phones and tablets. 但TV的交互模型与手机或平板不同。新布局必须能在十码外轻松理解。导航只用directional pad和选择按钮就能控制。

> Note: You are encouraged to use Android Studio for building TV apps, because it provides project setup, library inclusion, and packaging conveniences. This training assumes you are using Android Studio.

#### TV 应用入门

> Important: There are specific requirements your app must meet to qualify as an **Android TV app on Google Play**. For more information, see the requirements listed in [TV App Quality](http://developer.android.com/distribute/essentials/quality/tv.html).

##### 支持的媒体格式

See the following documentation for information about the codecs, protocols, and formats supported by Android TV.

- [Supported Media Formats](http://developer.android.com/guide/appendix/media-formats.html)
- [DRM](https://source.android.com/devices/drm.html)
- [android.drm](http://developer.android.com/reference/android/drm/package-summary.html)
- [ExoPlayer](http://developer.android.com/guide/topics/media/exoplayer.html)
- [android.media.MediaPlayer](http://developer.android.com/reference/android/media/MediaPlayer.html)

##### Set up a TV Project

This section discusses how to modify an existing app to run on TV devices, or create a new one. These are the main components you must use to create an app that runs on TV devices:

- Activity for TV (Required) - In your application manifest, declare an activity that is intended to run on TV devices.
- TV Support Libraries (Optional) - There are several Support Libraries available for TV devices that provide widgets for building user interfaces.

**Prerequisites**

Before you begin building apps for TV, you must:

- Update your SDK tools to version 24.0.0 or higher
- Update your SDK with Android 5.0 (API 21) or higher

**Declare a TV Activity**

An application intended to run on TV devices must declare a launcher activity for TV in its manifest using a `CATEGORY_LEANBACK_LAUNCHER` intent filter. This filter identifies your app as being enabled for TV, and is required for your app to be considered a TV app in Google Play. Declaring this intent also identifies **which activity** in your app to launch when a user selects its icon on the TV home screen.

The following code snippet shows how to include this intent filter in your manifest:

```xml
    <application
      android:banner="@drawable/banner" >
      ...
      <activity
        android:name="com.example.android.MainActivity"
        android:label="@string/app_name" >
        <intent-filter>
          <action android:name="android.intent.action.MAIN" />
          <category android:name="android.intent.category.LAUNCHER" />
        </intent-filter>
      </activity>

      <activity
        android:name="com.example.android.TvActivity"
        android:label="@string/app_name"
        android:theme="@style/Theme.Leanback">
        <intent-filter>
          <action android:name="android.intent.action.MAIN" />
          <category android:name="android.intent.category.LEANBACK_LAUNCHER" />
        </intent-filter>

      </activity>
    </application>
```

The second activity manifest entry in this example specifies that activity as the one to launch on a TV device.

> Caution: If you do not include the `CATEGORY_LEANBACK_LAUNCHER` intent filter in your app, it is not visible to users running the Google Play store on TV devices. Also, if your app does not have this filter when you load it onto a TV device using developer tools, 引用不会出现在 TV UI 中。

For guidelines on designing an app for TV, see the [TV Design](http://developer.android.com/design/tv/index.html) guide.

**Declare Leanback support**

Declare that your app uses the Leanback user interface required by Android TV. If you are developing an app that runs on mobile (phones, wearables, tablets, etc.) as well as Android TV, set the required attribute value to false. If you set the required attribute value to true, your app will run only on devices that use the Leanback UI.

```xml
<manifest>
    <uses-feature android:name="android.software.leanback"
        android:required="false" />
    ...
</manifest>
```

**Declare touchscreen not required**

Applications that are intended to run on TV devices do not rely on touch screens for input. In order to make this clear, the manifest of your TV app must declare that a the `android.hardware.touchscreen` feature is not required. This setting identifies your app as being able to work on a TV device, and is required for your app to be considered a TV app in Google Play. The following code example shows how to include this manifest declaration:

```xml
<manifest>
    <uses-feature android:name="android.hardware.touchscreen"
              android:required="false" />
    ...
</manifest>
```

> Caution: You must declare that a touch screen is not required in your app manifest, as shown this example code, or your app cannot appear in the Google Play store on TV devices.

**Provide a home screen banner**

An application must provide a **home screen banner** for each localization if it includes a Leanback launcher intent filter. The banner is the app launch point that appears on the home screen in the apps and games rows. Desribe the banner in the manifest as follows:

```xml
    <application
        ...
        android:banner="@drawable/banner" >
        ...
    </application>
```

Use the `android:banner` attribute with the `<application>` tag to supply a default banner for all application activities, or with the `<activity>` tag to supply a banner for a specific activity.

See [Banners](http://developer.android.com/design/tv/patterns.html#banner) in the UI Patterns for TV design guide.

##### 添加 TV 支持库

The Android SDK includes support libraries that are intended for use with TV apps. These libraries provide APIs and user interface widgets for use on TV devices. The libraries are located in the `<sdk>/extras/android/support/` directory. Here is a list of the libraries and their general purpose:

- v17 leanback library - Provides user interface widgets for TV apps, particularly for apps that do media playback.
- v7 recyclerview library - Provides classes for managing display of long lists in a memory efficient manner. Several classes in the v17 leanback library depend on the classes in this library.
- v7 cardview library - Provides user interface widgets for displaying information cards, such as media item pictures and descriptions.

Note: You are not required to use these support libraries for your TV app. However, we strongly recommend using them, particularly for apps that provide a media catalog browsing interface.

If you decide to use the v17 leanback library for your app, you should note that it is dependent on the v4 support library. This means that apps that use the leanback support library should include all of these support libraries:

- v4 support library
- v7 recyclerview support library
- v17 leanback support library

The v17 leanback library contains resources, which require you to take specific steps to include it in app projects. For instructions on importing a support library with resources, see Support Library Setup.

##### Build TV Apps

After you have completed the steps described above, it's time to start building apps for the big screen! Check out these additional topics to help you build your app for TV:

Building TV Playback Apps - TVs are built to entertain, so Android provides a set of user interface tools and widgets for building TV apps that play videos and music, and let users browse for the content they want.
Helping Users Find Your Content on TV - With all the content choices at users' fingertips, helping them find content they enjoy is almost as important as providing that content. This training discusses how to surface your content on TV devices.
Building TV Games - TV devices are a great platform for games. See this topic for information on building great game experiences for TV.
Building Live TV Apps - Present your video content in a linear, "broadcast TV" style with channels and programs that your users can access through a program guide as well as the channel up/down buttons.
Run TV Apps
Running your app is an important part of the development process. The AVD Manager in the Android SDK provides the device definitions that allow you to create virtual TV devices for running and testing your applications.

To create an virtual TV device:

Start the AVD Manager. For more information, see the AVD Manager help.
In the AVD Manager dialog, click the Device Definitions tab.
Select one of the Android TV device definitions and click Create AVD.
Select the emulator options and click OK to create the AVD.
Note: For best performance of the TV emulator device, enable the Use Host GPU option and, where supported, use virtual device acceleration. For more information on hardware acceleration of the emulator, see Using the Emulator.

To test your application on the virtual TV device:

Compile your TV application in your development environment.
Run the application from your development environment and choose the TV virtual device as the target.
For more information about using emulators see, Using the Emulator. For more information on deploying apps from Android Studio to virtual devices, see Debugging with Android Studio. For more information about deploying apps to emulators from Eclipse with ADT, see Building and Running from Eclipse with ADT.



