[toc]

http://developer.android.com/sdk/installing/studio-build.html

## Build System Overview

The build system can run as an integrated tool from the Android Studio menu and independently from the command line. You can use the features of the build system to:

- Customize, configure, and extend the build process.
- Create multiple APKs for your app with different features using the same project and modules.
- Reuse code and resources across source sets.

To build an Android Studio project, see [Building and Running from Android Studio](http://developer.android.com/tools/building/building-studio.html). To configure custom build settings in an Android Studio project, see Configuring Gradle Builds.

**A Detailed Look at the Build Process**

The build process involves many tools and processes that generate intermediate files on the way to producing an .apk. If you are developing in Android Studio, the complete build process is done every time you run the Gradle build task for your project or modules. The build process is very flexible so it's useful, however, to understand what is happening under the hood since much of the build process is configurable and extensible. The following diagram depicts the different tools and processes that are involved in a build:

![](img/build.png)

If different folders contain resources with the same name or setting, the following override priority order is: dependencies override build types, which override product flavors, which override the main source directory.

Finally, if the application is being signed in release mode, you must align the .apk with the zipalign tool. Aligning the final .apk decreases memory usage when the application is running on a device.

The build generates an APK for each build variant in the `app/build` folder: the `app/build/outputs/apk/` directory contains packages named `app-<flavor>-<buildtype>.apk`; for example, `app-full-release.apk` and `app-demo-debug.apk`.

### （未）配置 Gradle 构建

http://developer.android.com/tools/building/configuring-gradle.html

#### 配置基础

Android Studio projects contain a top-level build file and a build file for each module. The build files are called `build.gradle`. In most cases, you only need to edit the build files at the module level. For example, the build file for the **app** module in the **BuildSystemExample** project looks like this:

```
apply plugin: 'com.android.application'

android {
    compileSdkVersion 19
    buildToolsVersion "19.0.0"

    defaultConfig {
        minSdkVersion 8
        targetSdkVersion 19
        versionCode 1
        versionName "1.0"
    }
    buildTypes {
        release {
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
}

dependencies {
    compile project(":lib")
    compile 'com.android.support:appcompat-v7:19.0.1'
    compile fileTree(dir: 'libs', include: ['*.jar'])
}
```

`apply plugin: 'com.android.application'` applies the Android plugin for Gradle to this build. This adds Android-specific build tasks to the top-level build tasks and makes the `android {...}` element available to specify Android-specific build options.

`android {...}` configures all the Android-specific build options:

- The `compileSdkVersion` property specifies the compilation target.
- The `buildToolsVersion` property specifies what version of the build tools to use.
- The `defaultConfig` element configures core settings and entries in the manifest file (AndroidManifest.xml) dynamically from the build system. The values in `defaultConfig` **override** those in the manifest file. The configuration specified in the `defaultConfig` element applies to all build variants, unless the configuration for a build variant overrides some of these values.
- The `buildTypes` element controls how to build and package your app. By default, the build system defines two build types: `debug` and `release`. The debug build type includes debugging symbols and is signed with the debug key. The release build type is not signed by default. In this example the build file configures the release version to use ProGuard.

##### 声明依赖

The app module in this example declares three dependencies:

```
...
dependencies {
    // Module dependency
    compile project(":lib")

    // Remote binary dependency
    compile 'com.android.support:appcompat-v7:19.0.1'

    // Local binary dependency
    compile fileTree(dir: 'libs', include: ['*.jar'])
}
```

**Module dependencies**

The **app** module depends on the **lib** module.

`compile project(":lib")` declares a dependency on the **lib** module of BuildSystemExample.

**Remote binary dependencies**

`compile 'com.android.support:appcompat-v7:19.0.1'` declares a dependency on version 19.0.1 of the Android Support Library by specifying its Maven coordinates. The Android Support Library is available in the Android Repository package of the Android SDK. If your SDK installation does not have this package, download and install it using the SDK Manager.

Android Studio configures projects to use the **Maven Central Repository** by default. (This configuration is included in the top-level build file for the project.)

**Local binary dependencies**

Some modules do not use any binary dependencies from the local file system. If you have modules that require local binary dependencies, copy the JAR files for these dependencies into `<moduleName>/libs` inside your project.

`compile fileTree(dir: 'libs', include: ['*.jar'])` tells the build system that any JAR file inside `app/libs` is a dependency and should be included in the compilation classpath and in the final package.

##### Run ProGuard

The build system can run ProGuard to obfuscate your classes during the build process. In BuildSystemExample, modify the build file for the app module to run ProGuard for the release build:

```
...
android {
    ...
    buildTypes {
        release {
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
}
...
```

`getDefaultProguardFile('proguard-android.txt')` obtains the default ProGuard settings from the Android SDK installation. Android Studio adds the module-specific rules file proguard-rules.pro at the root of the module, where you can add custom ProGuard rules.

##### Application ID for package identification

With the Android build system, the `applicationId` attribute is used to uniquely identify application packages for publishing. The application ID is set in the android section of the build.gradle file.

```
    apply plugin: 'com.android.application'

    android {
        compileSdkVersion 19
        buildToolsVersion "19.1"

    defaultConfig {
        applicationId "com.example.my.app"
        minSdkVersion 15
        targetSdkVersion 19
        versionCode 1
        versionName "1.0"
    }
    ...
```

When using build variants, the build system enables you to uniquely identify different packages for each product flavors and build types. The application ID in the build type is added as a suffix to those specified for the product flavors.

```
   productFlavors {
        pro {
            applicationId = "com.example.my.pkg.pro"
        }
        free {
            applicationId = "com.example.my.pkg.free"
        }
    }

    buildTypes {
        debug {
            applicationIdSuffix ".debug"
        }
    }
    ....
```

The package name must still be specified in the manifest file. It is used in your source code to refer to your R class and to resolve any relative activity/service registrations.

```
   package="com.example.app">
```

##### Configure signing settings

The debug and the release versions of the app differ on whether the application can be debugged on secure devices and on how the APK is signed. The build system signs the debug version with a default key and certificate using known credentials to avoid a password prompt at build time. The build system does not sign the release version unless you explicitly define a signing configuration for this build. If you do not have a release key, you can generate one as described in [Signing your Applications](http://developer.android.com/tools/publishing/app-signing.html).




