# Chrome拓展

[toc]

## 入门
https://developer.chrome.com/extensions/getstarted

我们将实现一个UI元素：[browser action](https://developer.chrome.com/extensions/browserAction)。它是Chrome地址栏旁的一个可点击的图标。点击后弹出一个窗口。

效果：
![](gettingstarted-1.jpg)


### 需要声明的内容

`manifest.json`文件。JSON格式。

```json
{
  "manifest_version": 2,
  "name": "One-click Kittens",
  "description": "This extension demonstrates a browser action with kittens.",
  "version": "1.0",

  "permissions": [
    "https://secure.flickr.com/"
  ],
  "browser_action": {
    "default_icon": "icon.png",
    "default_popup": "popup.html"
  }
}
```

`manifest_version`一定要是2。

The final block first requests permission to work with data on https://secure.flickr.com/.

### 资源

`icon.png`将显示在地址栏右侧。下载[icon.png](https://developer.chrome.com/extensions/examples/tutorials/getstarted/icon.png)。它是一个19像素的PNG。

`popup.html`将称为弹出菜单。下载[popup.html](https://developer.chrome.com/extensions/examples/tutorials/getstarted/popup.html)。

`popup.html`需要一些Javascript收集图片，下载[popup.js](https://developer.chrome.com/extensions/examples/tutorials/getstarted/popup.js)。

目录中应该有四个文件：icon.png, manifest.json, popup.html, popup.js。


### 加载扩展

1. 打开`chrome://extensions`。
1. Ensure that the **Developer mode** checkbox in the top right-hand corner is checked.
1. Click **Load unpacked extension…** to pop up a file-selection dialog.
1. Navigate to the directory in which your extension files live, and select it.

If the extension is valid, it'll be loaded up and active right away! If it's invalid, an error message will be displayed at the top of the page.

### 代码实验

修改一下`popup.js`。将`var QUERY = 'kittens';`改为`var QUERY = 'puppies';`。

If you click on your extension's browser action again, you'll note that your change hasn't yet had an effect. You'll need to let Chrome know that something has happened, either explicitly by going back to the extension page (chrome://extensions, or Tools > Extensions under the Chrome menu), and clicking Reload under your extension, or by reloading the extensions page itself (either via the reload button to the left of the Omnibox, or by hitting F5 or Ctrl-R).

### 下一步

## 概述

https://developer.chrome.com/extensions/overview

### 基础

扩展是一组HTML、CSS、JavaScript和图片和集合。

扩展可以利用c[ontent scripts](https://developer.chrome.com/extensions/content_scripts)与Web页面交互，利用[跨域XMLHttpRequests](https://developer.chrome.com/extensions/xhr)与服务器交互。扩展还可以与浏览器功能交互，如[书签](https://developer.chrome.com/extensions/bookmarks)和[标签](https://developer.chrome.com/extensions/tabs)。

### 扩展的UI

两种形式的UI：browser actions和page actions。扩展至多有一个browser action或page action。Choose a browser action when the extension is relevant to most pages. Choose a page action when the extension's icon should appear or disappear, depending on the page.


























## 开发手册

https://developer.chrome.com/extensions/devguide




