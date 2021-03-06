[toc]

## About Apache Cordova

Apache Cordova is a set of device APIs that allow a mobile app developer to access native device function such as the camera or accelerometer from JavaScript.

When using the Cordova APIs, an app can be built without any native code (Java, Objective-C, etc) from the app developer. Instead, web technologies are used, and they are hosted in the app itself locally (generally not on a remote http server).

Apps using Cordova are still packaged as apps using the platform SDKs, and can be made available for installation from each device's app store.

Cordova is available for the following platforms: iOS, Android, Blackberry, Windows Phone, Palm WebOS, Bada, and Symbian.

## 文档

http://cordova.apache.org/docs/en/4.0.0/

### 概述

Apache Cordova is an open-source mobile development framework. It allows you to use standard web technologies such as HTML5, CSS3, and JavaScript for cross-platform development. Applications execute within wrappers targeted to each platform, and rely on standards-compliant API bindings to access each device's sensors, data, and network status.

#### 基本组件

Apache Cordova应用依赖config.xml文件提供应用信息，设置参数（如是否响应屏幕方向变化）。This file adheres to the W3C's [Packaged Web App](http://www.w3.org/TR/widgets/), or widget, specification.

应用是一个本地的Web页面，一般是index.html。应用运行在一个WebView中。

The Cordova-enabled WebView may provide the application with its entire user interface. On some platforms, it can also be a component within a larger, hybrid application that mixes the WebView with native application components. (See [Embedding WebViews](http://cordova.apache.org/docs/en/4.0.0/guide_hybrid_webviews_index.md.html#Embedding%20WebViews) for details.)

一个插件接口用于Cordova和原生组件交互。接口让你可以用Javascript调用本地代码。As of version 3.0, plugins provide bindings to standard device APIs. 三方插件提供的绑定不一定在所有平台可用。

> NOTE: As of version 3.0, when you create a Cordova project it does not have any plugins present. This is the new default behavior. 任何插件，包括核心插件，都要显式添加。

Cordova自己不提供任何UI组件或类MVC框架。

#### 开发路径

从3.0开始，有两个基本的工作流可以创建一个手机App。虽然多数情况下二者都行，但各有优势。

- 跨平台（CLI）工作流：本工作流以*Cordova CLI*为中心。*Cordova CLI*在Cordova 3.0引入。CLI是一个高层工具，可以一次构建多个平台，屏蔽了底层脚本的细节。The CLI copies a common set of web assets into subdirectories for each mobile platform, makes any necessary configuration changes for each, runs build scripts to generate application binaries. The CLI also provides a common interface to apply plugins to your app. 推荐使用此工作流。
- 平台中心工作流。As a rule of thumb, use this workflow if you need to modify the project within the SDK. This workflow relies on a set of lower-level shell scripts that are tailored for each supported platform, and a separate *Plugman* utility that allows you to apply plugins. While you can use this workflow to build cross-platform apps, it is generally more difficult because the lack of a higher-level tool means separate build cycles and plugin modifications for each platform. Still, this workflow allows you greater access to development options provided by each SDK, and is essential for complex hybrid apps. See the various [Platform Guides](http://cordova.apache.org/docs/en/4.0.0/guide_platforms_index.md.html#Platform%20Guides) for details on each platform's available shell utilities.

使用跨平台流创建app最简单，参见The Command-line Interface。You then have the option to switch to a platform-centered workflow if you need the greater control the SDK provides. Lower-level shell utilities are available at cordova.apache.org in a separate distribution than the CLI. For projects initially generated by the CLI, these shell tools are also available in the project's various `platforms/*/cordova` directories.

> NOTE: Once you switch from the CLI-based workflow to one centered around the platform-specific SDKs and shell tools, you can't go back. The CLI maintains a common set of cross-platform source code, which on each build it uses to write over platform-specific source code. To preserve any modifications you make to the platform-specific assets, you need to switch to the platform-centered shell tools, which ignore the cross-platform source code, and instead relies on the platform-specific source code.

#### 安装Cordova

The installation of Cordova will differ depending on the workflow above you choose:

- Cross-platform workflow: see The Command-Line Interface.
- Platform-centered workflow: see the Platform Guides.

After installing Cordova, it is recommended that you review the *Platform Guides* for the mobile platforms that you will be developing for. It is also recommended that you also review the *Privacy Guide*, *Security Guide*, and *Next Steps*. For configuring Cordova, see *The config.xml File*. For accessing native function on a device from JavaScript, refer to the Plugin APIs. And refer to the other included guides as necessary.

### 平台支持

http://cordova.apache.org/docs/en/4.0.0/guide_support_index.md.html#Platform%20Support

### 命令行接口

本节介绍如何用CLI创建并部署应用到多个设备。The CLI is the main tool to use for the cross-platform workflow described in the Overview. Otherwise you can also use the CLI to initialize project code, then switch to various platforms' SDKs and shell tools for continued development.

#### 前提条件

运行前要安装平台的SDK。参见Platform Guides。

To add support or rebuild a project for any platform, you need to run the command-line interface from the same machine that supports the platform's SDK. The CLI supports the following combinations:

- iOS (Mac)
- Amazon Fire OS (Mac, Linux, Windows)
- Android (Mac, Linux, Windows)
- BlackBerry 10 (Mac, Linux, Windows)
- Windows Phone 8 (Windows)
- Windows (Windows)
- Firefox OS (Mac, Linux, Windows)


#### 安装Cordova CLI

Cordova命令行工具以npm包的形式分发。

安装`cordova`模块：

	$ sudo npm install -g cordova

The installation log may produce errors for any uninstalled platform SDKs.

Following installation, you should be able to run `cordova` on the command line with no arguments and it should print help text.

#### 创建App

在源码目录下执行：

    $ cordova create hello com.example.hello HelloWorld

Running the command with the `-d` option displays information about its progress.

第一个参数`hello`指定工程目录。这个目录不能存在。其内部，`www`子目录放着一个用的主页，资源在css, js, img目录下。These assets will be stored on the device's local filesystem, not served remotely. The `config.xml` file contains important metadata needed to generate and distribute the application.

第二个参数`com.example.hello`是可选的（此时第三个参数也要省略）。这个值后面可以通过编辑config.xml文件修改，but do be aware that there may be code generated outside of config.xml using this value, such as Java package names. The default value is `io.cordova.hellocordova`, but it is recommended that you select an appropriate value.

第三个参数`HelloWorld`表示应用显示名。此参数可选。You can edit this value later in the config.xml file, but do be aware that there may be code generated outside of config.xml using this value, such as Java class names. The default value is `HelloCordova`, but it is recommended that you select an appropriate value.

#### 添加平台

后续命令都要在工程目录下执行。

    $ cd hello

构建工程前，先要指定目标平台。能否执行这些命令取决于机器上是否有相应SDK。

Run any of these from a Mac:

    $ cordova platform add ios
    $ cordova platform add amazon-fireos
    $ cordova platform add android
    $ cordova platform add blackberry10
    $ cordova platform add firefoxos

Run any of these from a Windows machine, where wp refers to different versions of the Windows Phone operating system:

    $ cordova platform add wp8
    $ cordova platform add windows
    $ cordova platform add amazon-fireos
    $ cordova platform add android
    $ cordova platform add blackberry10
    $ cordova platform add firefoxos

Run this to check your current set of platforms:

    $ cordova platforms ls

(Note the `platform` and `platforms` commands are synonymous.)

Run either of the following synonymous commands to remove a platform:

    $ cordova platform remove blackberry10
    $ cordova platform rm amazon-fireos
    $ cordova platform rm android

Running commands to add or remove platforms affects the contents of the project's `platforms` directory, where each specified platform appears as a subdirectory. The `www` source directory is reproduced within each platform's subdirectory, appearing for example in `platforms/ios/www` or `platforms/android/assets/www`. Because the CLI constantly copies over files from the source `www` folder, you should only edit these files and not the ones located under the platforms subdirectories. If you use version control software, you should add this source `www` folder, along with the `merges` folder, to your version control system.

> 警告：When using the CLI to build your application, you should not edit any files in the `/platforms/` directory unless you know what you are doing, or if documentation specifies otherwise. The files in this directory are routinely overwritten when preparing applications for building, or when plugins are reinstalled.

If you wish at this point, you can use an SDK such as Eclipse or Xcode to open the project you created. You will need to open the derivative set of assets from the `/platforms/` directory to develop with an SDK. This is because the SDK specific metadata files are stored within the appropriate `/platform/` subdirectory. (See the Platform Guides for information on how to develop applications within each IDE.) Use this approach if you simply want to initialize a project using the CLI and then switch to an SDK for native work.

Read on if you wish to use the cross-platform workflow approach (the CLI) for the entire development cycle.

#### 构建App

By default, the cordova create script generates a skeletal web-based application whose home page is the project's `www/index.html` file. Edit this application however you want, but any initialization should be specified as part of the deviceready event handler, referenced by default from `www/js/index.js`.

运行下面的命令，迭代构建工程：

    $ cordova build

This generates platform-specific code within the project's `platforms` subdirectory. 还可以限制只构建某个平台：

    $ cordova build ios

The `cordova build` command is a shorthand for the following, which in this example is also targeted to a single platform:

    $ cordova prepare ios
    $ cordova compile ios

In this case, once you run `prepare`, you can use Apple's Xcode SDK as an alternative to modify and compile the platform-specific code that Cordova generates within `platforms/ios`. You can use the same approach with other platforms' SDKs.

#### 在模拟器或设备上测试

Run a command such as the following to rebuild the app and view it within a specific platform's emulator:

    $ cordova emulate android

Some mobile platforms emulate a particular device by default, such as the iPhone for iOS projects. For other platforms, you may need to first associate a device with an emulator.

NOTE: Emulator support is currently not available for Amazon Fire OS.

Alternately, you can plug the handset into your computer and test the app directly:

    $ cordova run android

Before running this command, you need to set up the device for testing, following procedures that vary for each platform. See Platform Guides for details on each platform's requirements.

#### Add Plugin Features

若应用需要访问设备的功能，需要添加能够访问插件Cordova核心API的插件。

A plugin is a bit of add-on code that provides an interface to native components. As of version 3.0, when you create a Cordova project it does not have any plugins present. 这是新的默认行为。任何插件，包括核心插件，都需要显式添加。

插件列表见 plugins.cordova.io。

The `cordova plugin add` command requires you to specify the repository for the plugin code. Here are examples of how you might use the CLI to add features to the app:

基本设备信息（Device API）：

	$ cordova plugin add org.apache.cordova.device

网络连接和电池事件：

    $ cordova plugin add org.apache.cordova.network-information
    $ cordova plugin add org.apache.cordova.battery-status

Accelerometer, Compass, and Geolocation:

    $ cordova plugin add org.apache.cordova.device-motion
    $ cordova plugin add org.apache.cordova.device-orientation
    $ cordova plugin add org.apache.cordova.geolocation

Camera, Media playback and Capture:

    $ cordova plugin add org.apache.cordova.camera
    $ cordova plugin add org.apache.cordova.media-capture
    $ cordova plugin add org.apache.cordova.media

Access files on device or network (File API):

    $ cordova plugin add org.apache.cordova.file
    $ cordova plugin add org.apache.cordova.file-transfer

Notification via dialog box or vibration:

    $ cordova plugin add org.apache.cordova.dialogs
    $ cordova plugin add org.apache.cordova.vibration

Contacts:

	$ cordova plugin add org.apache.cordova.contacts

Globalization:

	$ cordova plugin add org.apache.cordova.globalization

Splashscreen:

	$ cordova plugin add org.apache.cordova.splashscreen

Open new browser windows (InAppBrowser):

	$ cordova plugin add org.apache.cordova.inappbrowser

Debug console:

	$ cordova plugin add org.apache.cordova.console

NOTE: The CLI adds plugin code as appropriate for each platform. If you want to develop with lower-level shell tools or platform SDKs as discussed in the Overview, you need to run the **Plugman** utility to add plugins separately for each platform. (For more information, see Using Plugman to Manage Plugins.)

Use `plugin ls` (or `plugin list`, or `plugin` by itself) to view currently installed plugins. Each displays by its identifier:

    $ cordova plugin ls    # or 'plugin list'
    [ 'org.apache.cordova.console' ]

To remove a plugin, refer to it by the same identifier that appears in the listing. For example, here is how you would remove support for a debug console from a release version:

    $ cordova plugin rm org.apache.cordova.console
    $ cordova plugin remove org.apache.cordova.console    # same

You can batch-remove or add plugins by specifying more than one argument for each command:

    $ cordova plugin add org.apache.cordova.console  org.apache.cordova.device

#### 高级插件选项

When adding a plugin, several options allow you to specify from where to fetch the plugin. The examples above use a well-known registry.cordova.io registry, and the plugin is specified by the id:

    $ cordova plugin add org.apache.cordova.console

The id may also include the plugin's version number, appended after an @ character. The latest version is an alias for the most recent version. For example:

    $ cordova plugin add org.apache.cordova.console@latest
    $ cordova plugin add org.apache.cordova.console@0.2.1

If the plugin is not registered at registry.cordova.io but is located in another git repository, you can specify an alternate URL:

    $ cordova plugin add https://github.com/apache/cordova-plugin-console.git

The git example above fetches the plugin from the end of the master branch, but an alternate git-ref such as a tag or branch can be appended after a # character:

    $ cordova plugin add https://github.com/apache/cordova-plugin-console.git#r0.2.0

If the plugin (and its plugin.xml file) is in a subdirectory within the git repo, you can specify it with a : character. Note that the # character is still needed:

    $ cordova plugin add https://github.com/someone/aplugin.git#:/my/sub/dir

You can also combine both the git-ref and the subdirectory:

    $ cordova plugin add https://github.com/someone/aplugin.git#r0.0.1:/my/sub/dir

Alternately, specify a local path to the plugin directory that contains the plugin.xml file:

    $ cordova plugin add ../my_plugin_dir

#### 利用merges定制平台

While Cordova allows you to easily deploy an app for many different platforms, sometimes you need to add customizations. In that case, you don't want to modify the source files in various `www` directories within the top-level `platforms` directory, because they're regularly replaced with the top-level `www` directory's cross-platform source.

Instead, the top-level `merges` directory offers a place to specify assets to deploy on specific platforms. Each platform-specific subdirectory within `merges` mirrors the directory structure of the `www` source tree, allowing you to override or add files as needed. For example, here is how you might uses merges to boost the default font size for Android and Amazon Fire OS devices:

Edit the `www/index.html` file, adding a link to an additional CSS file, `overrides.css` in this case:

	<link rel="stylesheet" type="text/css" href="css/overrides.css" />

Optionally create an empty `www/css/overrides.css` file, which would apply for all non-Android builds, preventing a missing-file error.

Create a css subdirectory within `merges/android`, then add a corresponding `overrides.css` file.

	body { font-size:14px; }

When you rebuild the project, the Android version features the custom font size, while others remain unchanged.

You can also use `merges` to add files not present in the original `www` directory. For example, an app can incorporate a back button graphic into the iOS interface, stored in `merges/ios/img/back_button.png`, while the Android version can instead capture backbutton events from the corresponding hardware button.

#### Help Commands

Cordova features a couple of global commands, which may help you if you get stuck or experience a problem. The `help` command displays all available Cordova commands and their syntax:

    $ cordova help
    $ cordova        # same

Additionally, you can get more detailed help on a specific command. For example:

	$ cordova run --help

The `info` command produces a listing of potentially useful details, such as currently installed platforms and plugins, SDK versions for each platform, and versions of the CLI and node.js:

$ cordova info
It both presents the information to screen and captures the output in a local info.txt file.

NOTE: Currently, only details on iOS and Android platforms are available.

#### Updating Cordova and Your Project

After installing the cordova utility, you can always update it to the latest version by running the following command:

    $ sudo npm update -g cordova

Use this syntax to install a specific version:

    $ sudo npm install -g cordova@3.1.0-0.2.0

Run `cordova -v` to see which version is currently running. Run the `npm info` command for a longer listing that includes the current version along with other available version numbers:

    $ npm info cordova

Cordova 3.0 is the first version to support the command-line interface described in this section. If you are updating from a version prior to 3.0, you need to create a new project as described above, then copy the older application's assets into the top-level `www` directory. Where applicable, further details about upgrading to 3.0 are available in the Platform Guides. Once you upgrade to the cordova command-line interface and use `npm update` to stay current, the more time-consuming procedures described there are no longer relevant.

Cordova 3.0+ may still require various changes to project-level directory structures and other dependencies. After you run the npm command above to update Cordova itself, you may need to ensure your project's resources conform to the latest version's requirements. Run a command such as the following for each platform you're building:

    $ cordova platform update android
    $ cordova platform update ios
    ...etc.

### Platform Guides

各平台文档参见：

http://cordova.apache.org/docs/en/4.0.0/guide_platforms_index.md.html#Platform%20Guides

### Android平台指引

#### Android平台指引

This guide shows how to set up your SDK environment to deploy Cordova apps for Android devices, and how to optionally use Android-centered command-line tools in your development workflow.

##### 要求

Cordova支持2.3.x (Gingerbread, starting with Android API level 10)和 4.x。如果某个Android版本的使用率小于5%后，Cordova会停止支持。Android versions earlier than API level 10, and the 3.x versions (Honeycomb, API levels 11-13) fall significantly below that 5% threshold.

##### 安装Cordova命令行工具

If you want to use Cordova's Android-centered shell tools in conjunction with the SDK, download Cordova from [cordova.apache.org](http://cordova.apache.org/). 若你想使用跨平台CLI工具，忽略本节。

The Cordova download contains separate archives for each platform. Be sure to expand the appropriate archive, android in this case, within an empty directory. The relevant executible utilities are available in the top-level `bin` directory. (Consult the README file if necessary for more detailed directions.)

这些命令行工具允许你创建、构建和运行Android应用。For information on the additional command-line interface that enables plugin features across all platforms, see Using Plugman to Manage Plugins.

需要将SDK的`tools`和`platform-tools`目录假如`PATH`。

##### Open a New Project in the SDK

创建新工程，可以选择使用跨平台CLI工具，或Android中心的命令行工具。CLI方式：

    $ cordova create hello com.example.hello HelloWorld
    $ cd hello
    $ cordova platform add android
    $ cordova build

Here's the corresponding lower-level shell-tool approach:

    $ /path/to/cordova-android/bin/create /path/to/new/hello com.example.hello HelloWorld

##### 构建工程

若使用CLI开发，运行下面命令之一构建工程

    $ cordova build
    $ cordova build android   # do not rebuild other platforms

如果采用Android专有命令行工具开发，产生工程后，默认的应用源代码在`assets/www`子目录。Subsequent commands are available in its `cordova` subdirectory.

The `build` command cleans project files and rebuilds the app. The first pair of examples generate debugging information, and the second signs the apps for release:

    $ /path/to/project/cordova/build --debug

    $ /path/to/project/cordova/build --release

##### 部署到模拟器

At this point you can use the cordova CLI utility to deploy the application to the emulator from the command line:

    $ cordova emulate android

Otherwise use the alternate shell interface:

    $ /path/to/project/cordova/run --emulator

Instead of relying on whichever emulator is currently enabled within the SDK, you can refer to each by the names you supply:

    $ /path/to/project/cordova/run --target=NAME

When you run the app, you also build it. You can append additional `--debug`, `--release`, and `--nobuild` flags to control how it is built, or even whether a rebuild is necessary:

    $ /path/to/project/cordova/run --emulator --nobuild

If instead you are working within Eclipse, right-click the project and choose *Run As → Android Application*. You may be asked to specify an AVD if none are already open.

##### 部署到设备

To push an app directly to the device, make sure USB debugging is enabled on your device as described on the Android Developer Site, and use a mini USB cable to plug it into your system.

You can use this CLI command to push the app to the device:

    $ cordova run android

...or use this Android-centered shell interface:

    $ /path/to/project/cordova/run --device

With no flags specified, the run command detects a connected device, or a currently running emulator if no device is found, otherwise it prompts to specify an emulator.

To run the app from within Eclipse, right-click the project and choose *Run As → Android Application*.

##### 其他命令

The following generates a detailed log of the app as it runs:

    $ /path/to/project/cordova/log

The following cleans the project files:

    $ /path/to/project/cordova/clean

#### Android命令行工具

This guide shows how to use Cordova's set of platform-centered shell tools to develop Android apps.

To enable shell tools for Android, download *Cordova* from cordova.apache.org. The download contains separate archives for each platform. Expand each you wish to target, android in this case. The relevant tools are typically available in the top-level `bin` directory, otherwise consult the README file for more detailed directions.

These tools allow you to create, build, and run Android apps. For information on the additional command-line interface that enables plugin features across all platforms, see Using Plugman to Manage Plugins.

**创建一个工程**

运行`create`命令，指定工程路径，包名，应用显示名。

    $ /path/to/cordova-android/bin/create /path/to/project com.example.project_name ProjectName

**构建**

清理然后构建工程。

调试版：

    $ /path/to/project/cordova/build --debug

发布版：

	$ /path/to/project/cordova/build --release

**运行**

The `run` command accepts the following optional parameters:

- Target specification. This includes --emulator, --device, or `--target=<targetID>`.
- Build specification. This includes --debug, --release, or --nobuild.

运行：

	$ /path/to/project/cordova/run [Target] [Build]

Make sure you create at least one Android Virtual Device, otherwise you're prompted to do so with the android command. If more than one AVD is available as a target, you're prompted to select one. By default the run command detects a connected device, or a currently running emulator if no device is found.

**日志**

    $ /path/to/project/cordova/log

**清理**

    $ /path/to/project/cordova/clean

**使用Ant**

If you wish to call Ant directly from the command line such as `ant debug install`, you need to specify additional parameters to the ant command:

    ant debug install -Dout.dir=ant-build -Dgen.absolute.dir=ant-gen

This is because the directories used by Cordova's Ant scripts are different than the default. This is done to avoid conflicts when Ant is run from the command line versus inside Eclipse/ADT.

These additional parameters are automatically added for you when using the `cordova/build` and `cordova/run` scripts described above. For this reason it is recommended to use the cordova/build and cordova/run scripts instead of calling Ant directly from the command line.

**（未）Building with Gradle (Experimental!)**

#### Android配置

The config.xml file controls an app's basic settings that apply across each application and CordovaWebView instance. 本节只介绍Android特有的配置。全局配置参见**The config.xml File**。

`KeepRunning` (boolean, defaults to true): 决定在应用收到[pause](http://cordova.apache.org/docs/en/4.0.0/cordova_events_events.md.html#pause)事件后是否继续运行。设为`false`，在收到暂停事件后并不是杀死应用，而是终止cordova webview中的执行。

	<preference name="KeepRunning" value="false"/>

`LoadUrlTimeoutValue` (number in milliseconds, default to 20000, 20 seconds): When loading a page, the amount of time to wait before throwing a timeout error. This example specifies 10 seconds rather than 20:

	<preference name="LoadUrlTimeoutValue" value="10000"/>

`SplashScreen` (string, defaults to `splash`): The name of the file minus its extension in the `res/drawable` directory. Various assets must share this common name in various subdirectories.

	<preference name="SplashScreen" value="mySplash"/>

`SplashScreenDelay` (number in milliseconds, defaults to 3000): The amount of time the splash screen image displays.

	<preference name="SplashScreenDelay" value="10000"/>

`InAppBrowserStorageEnabled` (boolean, defaults to `true`): Controls whether pages opened within an InAppBrowser can access the **same** localStorage and WebSQL storage as pages opened with the default browser.

	<preference name="InAppBrowserStorageEnabled" value="true"/>

`LoadingDialog` (string, defaults to null): If set, displays a dialog with the specified title and message, and a spinner, when loading the first page of an application. The title and message are separated by a comma in this value string, and that comma is removed before the dialog is displayed.

	<preference name="LoadingDialog" value="My Title,My Message"/>

`LoadingPageDialog` (string, defaults to null): The same as LoadingDialog, but for loading every page after the first page in the application.

	<preference name="LoadingPageDialog" value="My Title,My Message"/>

`ErrorUrl` (URL, defaults to null): If set, will display the referenced page upon an error in the application instead of a dialog with the title "Application Error".

	<preference name="ErrorUrl" value="myErrorPage.html"/>

`ShowTitle` (boolean, defaults to `false`): Show the title at the top of the screen.

	<preference name="ShowTitle" value="true"/>

`LogLevel` (string, defaults to `ERROR`): Sets the minimum log level through which log messages from your application will be filtered. Valid values are ERROR, WARN, INFO, DEBUG, and VERBOSE.

	<preference name="LogLevel" value="VERBOSE"/>

`SetFullscreen` (boolean, defaults to `false`): Same as the Fullscreen parameter in the global configuration of this xml file. This Android-specific element is deprecated in favor of the global Fullscreen element, and will be removed in a future version.

`AndroidLaunchMode` (string, defaults to `singleTop`): Sets the Activity `android:launchMode` attribute. This changes what happens when the app is launched from app icon or intent and is already running. Valid values are `standard`, `singleTop`, `singleTask`, `singleInstance`.

	<preference name="AndroidLaunchMode" value="singleTop"/>

#### （未）Android插件

http://cordova.apache.org/docs/en/4.0.0/guide_platforms_android_plugin.md.html#Android%20Plugins

本节介绍如何实现Android原生插件。Before reading this, see Application Plugins for an overview of the plugin's structure and its common JavaScript interface. This section **continues** to demonstrate the sample echo plugin that communicates from the Cordova webview to the native platform and back. For another sample, see also the comments in [CordovaPlugin.java](http://https://github.com/apache/cordova-android/blob/master/framework/src/org/apache/cordova/CordovaPlugin.java).

#### （未）Android WebViews

http://cordova.apache.org/docs/en/4.0.0/guide_platforms_android_webview.md.html#Android%20WebViews

#### （未）升级Android

http://cordova.apache.org/docs/en/4.0.0/guide_platforms_android_upgrade.md.html#Upgrading%20Android
