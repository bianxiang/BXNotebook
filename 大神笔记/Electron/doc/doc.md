[toc]

## Quick Start

http://electron.atom.io/docs/v0.35.0/tutorial/quick-start

Electron 可以看做服务器端的 Node.js 在桌面的变体。

运行 package.json 中 `main` 脚本的进程称为主进程。The script that runs in the main process can display a GUI by creating web pages.

Electron 中每个 Web 页面运行在自己的进程中，称为渲染进程。

Web 页面可以使用 Node.js APIs。

**主进程与渲染进程的区别**

主进程通过创建 `BrowserWindow` 实例来创建 Web 页面。每个 `BrowserWindow` 实例在它自己的渲染进程中绘制页面。

主进程管理所有的 WEB 页面和它们相应的渲染进程。

Web 页面不允许调用本地 GUI 接口。若想在页面中进程 GUI 操作，渲染进程需要与主进程通信，请求主进程执行这些操作。

Electron 提供 **ipc** 模块用于主进程与渲染进程通信。**remote** 模块用于 RPC 风格的通信。

### Write your First Electron App

Generally, an Electron app is structured like this:

    your-app/
    ├── package.json
    ├── main.js
    └── index.html

package.json 格式与 Node 模块格式相同。`main` 指定启动脚本。启动脚本运行主进程。
package.json 样例：

```
{
  "name"    : "your-app",
  "version" : "0.1.0",
  "main"    : "main.js"
}
```

若不指定 `main`，默认用 index.js。

main.js 负责创建窗口，处理系统事件，如：

```js
const electron = require('electron');
const app = electron.app;  // Module to control application life.
const BrowserWindow = electron.BrowserWindow; // Module to create native browser window.

// Report crashes to our server.
electron.crashReporter.start();

// Keep a global reference of the window object, if you don't, the window will
// be closed automatically when the JavaScript object is garbage collected.
var mainWindow = null;

// Quit when all windows are closed.
app.on('window-all-closed', function() {
  // On OS X it is common for applications and their menu bar
  // to stay active until the user quits explicitly with Cmd + Q
  if (process.platform != 'darwin') {
    app.quit();
  }
});

// This method will be called when Electron has finished
// initialization and is ready to create browser windows.
app.on('ready', function() {
  // Create the browser window.
  mainWindow = new BrowserWindow({width: 800, height: 600});
  // and load the index.html of the app.
  mainWindow.loadURL('file://' + __dirname + '/index.html');
  // Open the DevTools.
  mainWindow.webContents.openDevTools();
  // Emitted when the window is closed.
  mainWindow.on('closed', function() {
    // Dereference the window object, usually you would store windows
    // in an array if your app supports multi windows, this is the time
    // when you should delete the corresponding element.
    mainWindow = null;
  });
});
```

index.html 的内容：

```
    <!DOCTYPE html>
    <html>
      <head>
        <meta charset="UTF-8">
        <title>Hello World!</title>
      </head>
      <body>
        <h1>Hello World!</h1>
        We are using node <script>document.write(process.versions.node)</script>,
        Chrome <script>document.write(process.versions.chrome)</script>,
        and Electron <script>document.write(process.versions.electron)</script>.
      </body>
    </html>
```

### 运行应用

**electron-prebuilt**

如果你通过 npm **全局**安装了 **electron-prebuilt**，则只需要在你应用目录内执行：

```
electron .
```

如果不是全局安装，则：

```
./node_modules/.bin/electron .
```

**手工下载 Electron 可执行文件**

如果是手工下载的 Electron，需要使用可执行文件执行：

Windows

```
$ .\electron\electron.exe your-app\
```

Linux

```
$ ./electron/electron your-app/
```

OS X

```
$ ./Electron.app/Contents/MacOS/Electron your-app/
```

Electron.app here is part of the Electron's release package, you can download it from [here](https://github.com/atom/electron/releases).

**创建分发**

参见：http://electron.atom.io/docs/v0.35.0/tutorial/application-distribution

### 运行本实例

Clone and run the code in this tutorial by using the [atom/electron-quick-start](https://github.com/atom/electron-quick-start) repository.

Note: Running this requires Git and Node.js (which includes npm) on your system.

```
# Clone the repository
$ git clone https://github.com/atom/electron-quick-start
# Go into the repository
$ cd electron-quick-start
# Install dependencies and run the app
$ npm install && npm start
```

## 支持的平台

http://electron.atom.io/docs/v0.35.0/tutorial/supported-platforms/

Following platforms are supported by Electron:

**OS X**

Only 64-bit binaries are provided for OS X, and the minimum OS X version supported is OS X 10.8.

**Windows**

Windows 7 and later are supported, older operating systems are not supported (and do not work).

Both x86 and amd64 (x64) binaries are provided for Windows. Please note, the ARM version of Windows is not supported for now.

**Linux**

The prebuilt ia32(i686) and x64(amd64) binaries of Electron are built on Ubuntu 12.04, the arm binary is built against ARM v7 with hard-float ABI and NEON for Debian Wheezy.

Whether the prebuilt binary can run on a distribution depends on whether the distribution includes the libraries that Electron is linked to on the building platform, so only Ubuntu 12.04 is guaranteed to work, but following platforms are also verified to be able to run the prebuilt binaries of Electron:

- Ubuntu 12.04 and later
- Fedora 21
- Debian 8

## 应用的分发

http://electron.atom.io/docs/v0.35.0/tutorial/application-distribution/

包含你的应用的文件夹要命名为 `app`，放入 Electron 的 `resources` 目录。在 OS X 上是 `Electron.app/Contents/Resources/`，在 Linux 和 Windows 上是 `resources/`：

On OS X:

    electron/Electron.app/Contents/Resources/app/
    ├── package.json
    ├── main.js
    └── index.html

On Windows and Linux:

    electron/resources/app
    ├── package.json
    ├── main.js
    └── index.html

然后执行 `Electron.app` 或 `electron` 或 `electron.exe`。

### 将应用打包成一个文件

为防止源码泄露，可以将应用打包进一个 [asar](https://github.com/atom/asar) 归档。

To use an `asar` archive to replace the app folder, you need to rename the archive to `app.asar`, and put it under Electron's resources directory like below, and Electron will then try to read the archive and start from it.

On OS X:

    electron/Electron.app/Contents/Resources/
    └── app.asar

On Windows and Linux:

    electron/resources/
    └── app.asar

More details can be found in **Application packaging**.

### Rebranding with Downloaded Binaries

After bundling your app into Electron, you will want to rebrand Electron before distributing it to users.

**Windows**

可以将 `electron.exe` 重命名为任何名字。利用 [rcedit](https://github.com/atom/rcedit) 或 [ResEdit](http://www.resedit.net/) 修改图标等信息。

**OS X**

可以将 **Electron.app** 重命名为任何名字。还要重命名以下文件的 `CFBundleDisplayName`、 `CFBundleIdentifier`、 `CFBundleName` 字段：

    Electron.app/Contents/Info.plist
    Electron.app/Contents/Frameworks/Electron Helper.app/Contents/Info.plist

You can also rename the helper app to avoid showing Electron Helper in the Activity Monitor, but make sure you have renamed the helper app's executable file's name.

The structure of a renamed app would be like:

    MyApp.app/Contents
    ├── Info.plist
    ├── MacOS/
    │   └── MyApp
    └── Frameworks/
        ├── MyApp Helper EH.app
        |   ├── Info.plist
        |   └── MacOS/
        |       └── MyApp Helper EH
        ├── MyApp Helper NP.app
        |   ├── Info.plist
        |   └── MacOS/
        |       └── MyApp Helper NP
        └── MyApp Helper.app
            ├── Info.plist
            └── MacOS/
                └── MyApp Helper

**Linux**

You can rename the `electron` executable to any name you like.

### Rebranding by Rebuilding Electron from Source

It is also possible to rebrand Electron by changing the product name and building it from source. To do this you need to modify the `atom.gyp` file and have a clean rebuild.

**grunt-build-atom-shell**

手工检出 Electron 代码，重新构建代码可能较为复杂，因此我们创建了一个 Grunt 任务自动处理：`grunt-build-atom-shell`。

This task will automatically handle editing the `.gyp` file, building from source, then rebuilding your app's native Node modules to match the new executable name.

## （未）应用的打包

http://electron.atom.io/docs/v0.35.0/tutorial/application-packaging/


## （未）Online/Offline Event Detection

http://electron.atom.io/docs/v0.35.0/tutorial/online-offline-events/

## （未）Desktop Environment Integration

http://electron.atom.io/docs/v0.35.0/tutorial/desktop-environment-integration/
